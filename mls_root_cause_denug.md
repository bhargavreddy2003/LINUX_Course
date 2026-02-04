# MKS Production Debugging Playbook (Reverse-Engineering Approach)

_Environment: 3-node MKS cluster (1 control-plane + 2 workers) managed via Rafay. This playbook assumes **production** mindset and uses both `kubectl` and on-node Linux debugging. Focus is on how to think like a senior engineer: triage, reverse-engineer, identify root cause, and apply safe fixes._

---

## 0. Philosophy: How Seniors Debug Production Clusters

### 0.1 Goals

- **Protect production first**: avoid making things worse (no random reboots or restarts without reason).
- **Narrow the blast radius**: identify if the issue is global (whole cluster), scoped (a node, namespace), or local (one pod).
- **Use layers**: app → Kubernetes Object → Node → OS/Network/Hardware.
- **Reverse-engineer from symptoms**: start from the visible failure and walk backward towards the root cause.

### 0.2 Layers to reason about

1. **Application layer**: containers, images, environment variables, configMaps, DB connectivity.
2. **Kubernetes logical layer**: Pods, Deployments, Services, Ingress, Jobs, CronJobs.
3. **Kubernetes control-plane**: API server, etcd, scheduler, controller-manager.
4. **Node / kubelet layer**: node status, kubelet, containerd/Docker, CNI.
5. **Infrastructure layer**: OS, CPU/RAM, disk, network, DNS, firewalls.
6. **Rafay/MKS orchestration layer**: Salt minion, chisel, Rafay Controller, rctl tasks, upgrades, preflights.

Every incident belongs to one or more of these layers. Debugging is about **identifying the layer first**, then drilling down.

---

## 1. Global Triage Framework (First 5–10 Minutes)

> Use this for any unknown failure. Do **not** start by `kubectl logs` randomly.

### 1.1 Step 1 – Ask: "Who is complaining and how?"

- User-facing symptoms:
  - 5xx from API gateway.
  - Timeouts from services.
  - Latency spikes.
- Internal symptoms:
  - Alerts: NodeNotReady, PodCrashLoopBackOff, etcd high latency, Rafay cluster offline.
  - Rafay UI: cluster red/orange, tasks failing.

This tells you **scope and urgency**.

### 1.2 Step 2 – Cluster-wide quick scan (from workstation)

```bash
# Are the control-plane and nodes healthy?
kubectl get nodes -o wide

# Are core system pods healthy?
kubectl get pods -n kube-system -o wide

# Any Rafay/MKS pods in trouble?
kubectl get pods -A -o wide | egrep 'rafay|chisel|salt|agent'
```

Interpretation:

- If **control-plane node is NotReady** or system pods are failing → jump to control-plane / node sections.
- If **only workloads** are failing → app/storage/config/CNI layers.
- If **Rafay pods** are failing but cluster is otherwise ok → cluster works but management plane degraded.

### 1.3 Step 3 – Check namespace or service-level impact

```bash
# List pods by namespace
kubectl get pods -n NAMESPACE

# Look for patterns: Pending, CrashLoopBackOff, ImagePullBackOff, Error, Completed
kubectl get pods -A | egrep 'CrashLoopBackOff|Error|ImagePullBackOff|Pending'
```

This tells you which app/team is impacted. Now you know **WHERE** to go deep.

### 1.4 Step 4 – Decide the primary layer to inspect

- **Pods failing to start or crashing** → Kubernetes logical + app + CNI.
- **Nodes NotReady / Unknown** → Node + infra + Rafay agent.
- **API errors (`kubectl` failing, etcd alerts)** → Control-plane + etcd + networking.
- **Rafay UI shows cluster offline but kubectl OK** → Rafay/MKS layer (Salt minion, chisel, controller connectivity).

Once you decide, use the relevant section below.

---

## 2. Node & Infra Failures (Production-Focused)

### 2.1 Symptom set A: Node NotReady / Unknown

#### 2.1.1 What it usually means

- Node’s kubelet is not heartbeating to API server.
- Underlying OS or container runtime is down or overloaded.
- CNI or iptables broken, causing API server connectivity loss.
- For MKS: node agent operations may have partially modified iptables or network.

#### 2.1.2 Debug sequence (control-plane + node)

1. **From workstation**:

   ```bash
   kubectl get nodes
   kubectl describe node NODE_NAME
   ```

   Look at `Conditions`:

   - `Ready: False` with `KubeletNotReady`: check kubelet.
   - `NetworkUnavailable: True`: check CNI and iptables.
   - `MemoryPressure` or `DiskPressure`: check resources.

2. **SSH into the node**:

   ```bash
   # Resource checks
   top
   free -h
   df -h

   # Services
   systemctl status kubelet
   systemctl status containerd   # or docker

   # Logs
   journalctl -u kubelet -e --no-pager | tail -n 200
   journalctl -u containerd -e --no-pager | tail -n 200
   ```

3. **Network and API connectivity from node**:

   ```bash
   ip addr
   ip route

   ping -c 4 CONTROL_PLANE_IP
   curl -k https://CONTROL_PLANE_IP:6443/healthz

   sudo iptables -L -n -v
   ```

4. **Rafay-specific: Salt minion & chisel** (particularly if node became NotReady after a management operation):

   ```bash
   systemctl status salt-minion-rafay
   tail -n 200 /opt/rafay/salt/var/log/salt/minion

   ps aux | grep -i chisel | grep -v grep
   sudo crictl ps | grep -i chisel || sudo docker ps | grep -i chisel
   ```

#### 2.1.3 Common root causes & fixes

- **Disk pressure**: `DiskPressure` true, `df -h` shows >90% on `/var` or etcd directories.
  - Fix: clean logs (`/var/log`), prune old images (`crictl rmi` / `docker image prune`), rotate logs.
- **Kubelet crashed or misconfigured**:
  - Fix: inspect `journalctl -u kubelet`, correct config files under `/var/lib/kubelet/` or `/etc/sysconfig/kubelet`, then `sudo systemctl restart kubelet`.
- **Container runtime down**:
  - Fix: inspect runtime logs, ensure no disk/permission issues, restart containerd/Docker.
- **Network / iptables** problems:
  - Fix: if recent change to firewall/iptables, temporarily flush (`sudo iptables -F`), confirm connectivity, then implement proper rules.

---

### 2.2 Symptom set B: Cluster network failures (Services intermittently unreachable)

#### 2.2.1 What it usually means

- CNI plugin issues (Calico/Flannel pods crashing, misconfigured IP pools).
- iptables/nftables partially applied or broken.
- Node-to-node routing broken (wrong routes after network changes).

#### 2.2.2 Debug sequence

1. **Check CNI/system pods**:

   ```bash
   kubectl get pods -n kube-system -o wide | egrep 'calico|cni|flannel|weave'
   ```

   - If CNI pods are `CrashLoopBackOff`, inspect logs:
     ```bash
     kubectl logs POD_NAME -n kube-system
     ```

2. **From one affected node**:

   ```bash
   ip addr
   ip route

   # Check if podCIDR matches cluster design (see node annotations, cluster config)
   
   # Iptables rules used by kube-proxy
   sudo iptables -L -n -v
   ```

3. **Service-to-pod path**:

   - Check `Endpoints` of the Service:
     ```bash
     kubectl get svc SVC_NAME -n NAMESPACE -o wide
     kubectl get endpoints SVC_NAME -n NAMESPACE -o wide
     ```
   - Then confirm the backend pod IP is reachable from an affected node.

#### 2.2.3 Common root causes & fixes

- **CNI misconfiguration after upgrade**:
  - Fix: revert CNI config, roll back to last-known-good DaemonSet version.
- **Node network changed (new interface, VPN, default route changed)**:
  - Fix: restore correct routing; ensure podCIDR routing rules are present.
- **iptables corrupted by external tooling**:
  - Fix: flush rules (`iptables -F`) as a diagnostic; if it fixes the issue, reapply kube-proxy/CNI-managed rules and reconfigure external firewall management.

---

## 3. Control-Plane & etcd Failures

### 3.1 Symptom set C: `kubectl` fails / API server down

#### 3.1.1 Symptoms

- `kubectl get nodes` hangs or returns connection refused.
- Rafay UI shows cluster as offline or unreachable.
- Alerts about API server or etcd.

#### 3.1.2 Debug from control-plane node

1. **Check kubelet and runtime**:

   ```bash
   systemctl status kubelet
   systemctl status containerd    # or docker

   journalctl -u kubelet -e --no-pager | tail -n 200
   ```

2. **Check control-plane containers**:

   ```bash
   sudo crictl ps | egrep 'etcd|kube-apiserver|kube-controller-manager|kube-scheduler' \
     || sudo docker ps | egrep 'etcd|kube-apiserver|kube-controller-manager|kube-scheduler'
   ```

   - If `kube-apiserver` is missing or restarting, inspect its logs.

3. **Inspect static pod manifests**:

   ```bash
   ls -l /etc/kubernetes/manifests
   sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
   sudo cat /etc/kubernetes/manifests/etcd.yaml
   ```

4. **Check etcd health**:

   ```bash
   export ETCDCTL_API=3

   ETCDCTL_API=3 etcdctl \
     --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     endpoint health
   ```

5. **API server endpoints**:

   ```bash
   sudo ss -tulpn | grep 6443
   curl -k https://127.0.0.1:6443/healthz
   curl -k https://127.0.0.1:6443/livez
   curl -k https://127.0.0.1:6443/readyz
   ```

#### 3.1.3 Root cause patterns & fixes

- **etcd unhealthy or out of disk**:
  - `etcdctl endpoint health` fails, etcd container logs show I/O errors.
  - Fix: free disk, move etcd data to larger partition, or perform etcd restore if corrupted.
- **API server misconfigured** (e.g., after manual changes):
  - Wrong cert paths, misconfigured flags, improper etcd endpoint.
  - Fix: correct manifest, apply minimal changes, let kubelet restart pod.
- **Certificates expired**:
  - Logs show x509 expiration errors.
  - Fix: rotate certificates using kubeadm workflows or management tooling (for MKS, through Rafay-supported process).

---

### 3.2 Symptom set D: Scheduler or controller-manager issues

Symptoms:

- Pods stuck in `Pending` with no events indicating scheduling.
- Deployments not updating, Jobs not progressing.

Debug:

```bash
# Are scheduler/controller-manager containers running?
sudo crictl ps | egrep 'kube-scheduler|kube-controller-manager' \
  || sudo docker ps | egrep 'kube-scheduler|kube-controller-manager'

# Logs via container runtime
sudo crictl logs SCHEDULER_CONTAINER_ID
sudo crictl logs CONTROLLER_MANAGER_CONTAINER_ID
```

Common issues:

- Misconfigured flags in static pod manifests.
- Permissions/RBAC issues when communicating with API server.

Fix: correct manifests, ensure containers can talk to API server, and watch logs for new errors.

---

## 4. Rafay/MKS-Specific Failures

### 4.1 Symptom set E: Rafay shows cluster offline, but `kubectl` still works

This means:

- Kubernetes cluster itself is alive.
- Rafay’s management connection to the cluster is broken.

Debug path:

1. **Check Rafay operator/agent pods in cluster**:

   ```bash
   kubectl get pods -A -o wide | egrep 'rafay|agent|chisel'
   kubectl describe pod POD_NAME -n NAMESPACE
   kubectl logs POD_NAME -n NAMESPACE
   ```

2. **Check outbound connectivity from those pods/nodes**:

   - Node-level:
     ```bash
     ping -c 4 YOUR_RAFAY_ENDPOINT_DOMAIN
     curl -v https://YOUR_RAFAY_ENDPOINT_DOMAIN/
     ```

3. **Check Salt minion logs on nodes (MKS)**:

   ```bash
   systemctl status salt-minion-rafay
   tail -n 200 /opt/rafay/salt/var/log/salt/minion
   ```

4. **Check chisel tunnel**:

   ```bash
   ps aux | grep -i chisel | grep -v grep
   sudo crictl ps | grep -i chisel || sudo docker ps | grep -i chisel
   ```

Fixes:

- Restore network and DNS to Rafay Controller.
- Restart Salt minion and chisel if they crashed.
- Adjust any proxies (`http_proxy`, `https_proxy`, `no_proxy`).

---

### 4.2 Symptom set F: MKS upgrade or provisioning breaks nodes

Common patterns:

- Nodes go NotReady during or after upgrade.
- Rafay tasks stuck in `InProgress` or `Failed` during MKS upgrade.

Debug:

1. **Check affected node(s)**:

   ```bash
   systemctl status salt-minion-rafay
   tail -n 200 /opt/rafay/salt/var/log/salt/minion

   systemctl status kubelet
   systemctl status containerd   # or docker
   ```

2. **Look for iptables issues** (commonly called out in MKS troubleshooting):

   ```bash
   sudo iptables -L -n -v
   sudo iptables -F   # diagnostic step
   ```

3. **Check for partial upgrades**:

   - Node kernel changed but CNI modules not loaded or mismatched.
   - kubelet or runtime versions changed but config left incompatible.

4. **Use rctl status** (from workstation) to see which nodes/tasks failed:

   ```bash
   rctl status apply TASKSET_ID
   ```

Fix:

- For iptables-related downtime: flush and reconstruct rules, ensure CNI and kube-proxy reapply correct chains.
- For partial upgrades: either complete upgrade or roll back to previous image/config.

---

## 5. Application & Developer-Facing Failures

### 5.1 Symptom set G: Pods in CrashLoopBackOff

Debug path for a specific pod:

1. **From workstation**:

   ```bash
   kubectl describe pod POD_NAME -n NAMESPACE
   kubectl logs POD_NAME -n NAMESPACE
   kubectl logs POD_NAME -n NAMESPACE -p   # previous container instance
   ```

2. **Identify error class**:

   - App code issue: stack traces, misconfig, missing env.
   - Infrastructure: `CrashLoopBackOff` with `CreateContainerConfigError`, `CreateContainerError`, or CNI errors.

3. **If infra-related (e.g., `CreateContainerConfigError`)**:

   - Check volume mounts, secrets, configMaps in `describe` output.
   - On node: inspect container logs via runtime if they started at all.

Fix patterns:

- App bugs: rollback image to previous working version.
- Misconfig: fix configMap/secret and restart deployment.
- Volume errors: correct PVC/StorageClass, fix permissions.

---

### 5.2 Symptom set H: Performance degradation / latency spikes

Production logic:

1. **Check pod distribution & resource usage**:

   ```bash
   kubectl top nodes
   kubectl top pods -A
   ```

2. **Check node-level CPU/IO pressure**:

   ```bash
   top
   iostat -x 1 10   # if installed, for disk I/O
   df -h
   ```

3. **Check network saturation**:

   ```bash
   ip -s link
   ss -s
   ```

Root causes & fixes:

- Node underprovisioned: scale out nodes or pods, tune resource requests/limits.
- Noisy neighbour: misconfigured pod hogging CPU or I/O; apply resource limits and QoS.
- Storage bottleneck: slow disks; move critical workloads to faster volumes or separate nodes.

---

## 6. Reverse-Engineering Patterns & Mindset

### 6.1 Reverse from symptom to layer

Example: "Service A is timing out from client."

1. **Confirm if Service A pods are healthy**:

   ```bash
   kubectl get pods -n A_NAMESPACE -o wide
   ```

2. **Check Service & Endpoints**:

   ```bash
   kubectl get svc A_SERVICE -n A_NAMESPACE -o wide
   kubectl get endpoints A_SERVICE -n A_NAMESPACE -o wide
   ```

3. **From an arbitrary node or debug pod**:

   ```bash
   curl -v http://POD_IP:PORT
   ```

4. **If pod works but Service DNS fails**:

   - Check CoreDNS pods in `kube-system`.
   - Check cluster DNS config in node `/etc/resolv.conf` and CoreDNS configMap.

This is reverse-engineering: start where it breaks, walk down layers.

### 6.2 When in doubt, divide and conquer

- **Is it one namespace or all?**
- **Is it one node or multiple?**
- **Is it internal-only or also external-facing?**

Each answer halves the search space.

---

## 7. Senior-Level Production Checklists

### 7.1 Production incident checklist (generic)

1. **Stabilize**
   - Confirm incident scope with `kubectl get nodes` and `kubectl get pods -A`.
   - If only one app is affected, avoid touching cluster-wide components.

2. **Collect facts**
   - On affected nodes: `top`, `free -h`, `df -h`, `systemctl status` for kubelet/runtime/minion.
   - In cluster: `kubectl describe` and `kubectl logs` for failing pods.

3. **Identify primary failing layer**
   - Control-plane vs node vs app vs MKS management.

4. **Apply minimal, reversible changes only**
   - Prefer scaling or cordoning over rebooting entire node.
   - Prefer rolling back a deployment over editing live containers.

5. **Verify recovery**
   - Ensure metrics and logs confirm improvement.
   - Remove any temporary hacks (e.g. iptables flush) after implementing permanent fixes.

### 7.2 Node incident checklist (short version)

On any problematic node:

```bash
# Resources
 top
 free -h
 df -h
 lsblk

# Services
 systemctl status kubelet
 systemctl status containerd   # or docker
 systemctl status salt-minion-rafay

# Logs
 journalctl -u kubelet -e --no-pager | tail -n 200
 journalctl -u containerd -e --no-pager | tail -n 200
 journalctl -u salt-minion-rafay -e --no-pager | tail -n 200
 tail -n 200 /opt/rafay/salt/var/log/salt/minion

# Runtime
 sudo crictl ps || sudo docker ps
 sudo crictl ps -a || sudo docker ps -a

# Networking
 ip addr
 ip route
 ping -c 4 CONTROL_PLANE_IP
 curl -k https://CONTROL_PLANE_IP:6443/healthz
 ping -c 4 YOUR_RAFAY_ENDPOINT_DOMAIN
 curl -v https://YOUR_RAFAY_ENDPOINT_DOMAIN/

# Firewall
 sudo iptables -L -n -v
 # optional diagnostic step: sudo iptables -F
```

This, plus the layered reasoning from earlier sections, is the core of a **senior-level, reverse-engineering approach** to MKS debugging in production.
