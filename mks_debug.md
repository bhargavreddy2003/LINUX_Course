# MKS Node-Level Operations & Debug Runbook (No kubectl)

_Environment: 3-node MKS cluster (1 control-plane + 2 workers) managed via Rafay. Focus is **only** on node-side Linux, systemd, networking, container runtime, and Rafay components. All investigations start from SSH into a node. No `kubectl` is used in this runbook._

---

## 1. Fundamentals: Kubernetes, MKS, and Rafay on a Node

### 1.1 Kubernetes architecture recap (from a node perspective)

At the node level, a Kubernetes cluster is mostly just:

- **Processes managed by systemd** (e.g., `kubelet`, `containerd`, `docker`, `salt-minion-rafay`).
- **Containers** running inside the container runtime (e.g., `etcd`, `kube-apiserver`, application pods).
- **Network configuration** (interfaces, routes, iptables, CNI binaries and config).
- **Local data directories** (for container images, volumes, etcd data, logs).

Key entities:

- **Pods**: groups of one or more containers with shared network namespace and volumes.
- **Containers**: processes started by the container runtime, isolated by namespaces/cgroups.
- **Container runtime**: `containerd` or `docker` on the node, responsible for pulling images, starting/stopping containers.
- **Kubelet**: node agent that instructs the container runtime which pods to run.

On most modern clusters (including most MKS configurations):

- `kubelet` is a systemd service.
- `containerd` is the default runtime; sometimes Docker is still used.
- Control-plane components (API server, etcd, controller-manager, scheduler) run as **pods** that kubelet manages using **static pod manifests** in `/etc/kubernetes/manifests/`.

### 1.2 Control-plane vs Worker from node view

**Control-plane node** typically runs:

- Systemd services:
  - `kubelet`
  - `containerd` or `docker`
- Containers (via runtime):
  - `etcd`
  - `kube-apiserver`
  - `kube-controller-manager`
  - `kube-scheduler`
- Rafay components (node agent stack):
  - `salt-minion-rafay` (Salt minion customized for Rafay)
  - `chisel` tunnel (process/container)

**Worker nodes** typically run:

- Systemd services:
  - `kubelet`
  - `containerd` or `docker`
  - `salt-minion-rafay`
- Containers:
  - CNI daemon (e.g., `calico-node`, `flannel`, etc.)
  - `kube-proxy`
  - User workload pods
  - Possibly Rafay agent/chisel as containers

### 1.3 Rafay MKS integration on a node (conceptual)

On an MKS node provisioned via Rafay:

- **Conjurer / node bootstrap** installs Rafay node agent components.
- A **Salt minion** (`salt-minion-rafay`) is configured to talk to Rafay infrastructure.
- A **chisel tunnel** process is typically used to establish secure outbound connectivity.
- The node maintains a **long-lived outbound WebSocket/HTTPS** connection to the Rafay Controller.

From the node’s perspective, MKS adds:

- Additional systemd services (e.g., `salt-minion-rafay`).
- Extra processes/containers (e.g., chisel).
- Extra logs and configuration directories under `/opt/rafay/`.

Understanding these is critical for debugging cluster-state issues from the node, especially when the Rafay console shows nodes/clusters offline.

---

## 2. Linux & Systemd Fundamentals for Node Debugging

This section explains the basic tools you will always use: processes, memory, disk, systemd, and logs. Treat these as your **first layer** of checks before going into Kubernetes/Rafay specifics.

### 2.1 Process and resource monitoring

#### 2.1.1 CPU, load, and processes

```bash
# Live process and CPU view (press 'q' to exit)
top

# If installed, more visual and navigable
top  # or 'htop'
```

Key points:

- **Load average** significantly higher than number of CPU cores usually indicates CPU saturation or I/O wait.
- Identify any processes consuming 100% CPU (especially `etcd`, `kube-apiserver`, application processes, or runaway scripts).

#### 2.1.2 Memory usage

```bash
# Display total, used, free memory and swap
free -h
```

Interpretation:

- Very low `available` memory and significant `swap` usage can cause performance problems or OOM kills.
- If memory pressure is high, kubelet may start evicting pods or the OOM killer may terminate critical components.

### 2.2 Disk and filesystem health

```bash
# Disk usage for all mounted filesystems
df -h

# Block devices and partitions
lsblk
```

Important mount points to watch:

- `/` and `/var`: often where container runtime data and logs are written.
- `/var/lib/etcd`, `/var/lib/containerd`, `/var/lib/docker`: high usage can cause etcd and runtime failures.

Guidelines:

- Keep these volumes under ~80–90% usage.
- If disk is full, clean logs and unused images **before** restarting services.

### 2.3 Systemd basics

#### 2.3.1 Listing and inspecting services

```bash
# Show failed services
systemctl --failed

# List all active services (can be large)
systemctl list-units --type=service

# Inspect a specific service (e.g., kubelet)
systemctl status kubelet
systemctl status containerd
systemctl status docker
systemctl status salt-minion-rafay
```

`systemctl status` shows:

- Current state: `active (running)`, `inactive`, `failed`.
- Recent logs from the journal.
- Main PID and process tree.

#### 2.3.2 Starting/stopping/restarting services

```bash
# Start a service
sudo systemctl start SERVICE_NAME

# Stop a service
sudo systemctl stop SERVICE_NAME

# Restart a service
sudo systemctl restart SERVICE_NAME

# Enable on boot
sudo systemctl enable SERVICE_NAME

# Disable on boot
sudo systemctl disable SERVICE_NAME
```

Use restart carefully in production; prefer **inspect logs first**.

### 2.4 Journald and log inspection

```bash
# Show detailed recent system events (all units)
journalctl -xe --no-pager | tail -n 200

# Logs for specific unit (e.g., kubelet)
journalctl -u kubelet -e --no-pager | tail -n 200
journalctl -u containerd -e --no-pager | tail -n 200
journalctl -u docker -e --no-pager | tail -n 200
journalctl -u salt-minion-rafay -e --no-pager | tail -n 200
```

Key patterns to watch for:

- Authentication/SSL errors.
- Network timeouts.
- File path / permission issues.
- Out-of-resource messages.

---

## 3. Container Runtime Deep Dive (containerd & Docker)

### 3.1 Containerd basics

`containerd` is a modern container runtime. `crictl` is the CLI used to inspect containers managed by containerd.

#### 3.1.1 Core commands

```bash
# Check if containerd is running
systemctl status containerd

# Restart containerd (use only if crashed or stuck)
sudo systemctl restart containerd

# List running containers
sudo crictl ps

# List all containers (running + exited)
sudo crictl ps -a

# Detailed inspect for a specific container
sudo crictl inspect CONTAINER_ID | less

# Logs from a container (if runtime/logging configured)
sudo crictl logs CONTAINER_ID
```

Important fields in `crictl inspect`:

- `metadata.name` – container/pod name.
- `status.state` – running, exited, etc.
- `status.exitCode` – useful for crash diagnosis.
- `status.reason` and `status.message` – human-readable reason.

#### 3.1.2 Filtering by name

```bash
# Find core Kubernetes and Rafay-related containers
sudo crictl ps | egrep 'etcd|kube-apiserver|kube-controller-manager|kube-scheduler|kube-proxy|calico|flannel|salt|minion|chisel|rafay'
```

### 3.2 Docker basics

If Docker is used as the container runtime:

```bash
# Docker daemon status
systemctl status docker

# Restart if necessary
sudo systemctl restart docker

# Running containers
sudo docker ps

# All containers
sudo docker ps -a

# Logs for a container
sudo docker logs --tail=200 CONTAINER_ID_OR_NAME
sudo docker logs -f CONTAINER_ID_OR_NAME

# Detailed inspect
sudo docker inspect CONTAINER_ID_OR_NAME | less
```

### 3.3 Common failure diagnostics at runtime level

1. **CrashLoop containers**:
   - Repeatedly exiting containers visible via `crictl ps -a` or `docker ps -a`.
   - Use `crictl logs` / `docker logs` and `inspect` exit codes.

2. **Image pull issues**:
   - Errors about `ImagePullBackOff` or `ErrImagePull` in kubelet logs.
   - Check registry credentials, reachability, and DNS.

3. **Volume/permission issues**:
   - Check mount paths referenced in container spec.
   - Confirm host paths exist and have correct permissions.

---

## 4. Linux Networking & iptables Deep Dive

### 4.1 IP interfaces and routing

```bash
# Interfaces and addresses
ip addr

# Routing table
ip route
```

Interpretation:

- Verify expected IPs assigned to the correct interfaces.
- Ensure default route points to correct gateway.

### 4.2 DNS and external reachability

```bash
# DNS resolution
ping -c 4 google.com

# or (if ping blocked, use DNS tools)
dig google.com || nslookup google.com || host google.com

# External connectivity by IP
ping -c 4 8.8.8.8
```

### 4.3 Connectivity to control-plane node and API server

```bash
# From worker -> control-plane
ping -c 4 CONTROL_PLANE_IP

# If you know API server IP/hostname and certs allow
curl -k https://CONTROL_PLANE_IP:6443/healthz
```

### 4.4 Checking listening ports

Common Kubernetes-related ports:

- API server: `6443` (control-plane node)
- etcd: `2379` (client), `2380` (peer)
- kubelet: `10250`
- Salt minion/master: `4505` and `4506` (for Salt stack in general)

```bash
sudo ss -tulpn | egrep '6443|2379|2380|10250|4505|4506'
```

### 4.5 iptables troubleshooting

iptables rules can break traffic between nodes, pods, and Rafay Controller. Rafay’s MKS troubleshooting guidance explicitly suggests flushing iptables as a diagnostic step when worker nodes lose connectivity.

```bash
# View current rules
sudo iptables -L -n -v

# Flush all filter rules (use with caution, especially on public-facing hosts)
sudo iptables -F
```

After flushing, test connectivity again. If the issue is resolved, persist proper firewall rules using a configuration management approach rather than ad-hoc iptables rules.

---

## 5. Control-Plane Node Runbook (No kubectl)

This section is **only** for the master/control-plane node.

### 5.1 Validate kubelet

```bash
systemctl status kubelet

journalctl -u kubelet -e --no-pager | tail -n 200
```

Key log messages to look for:

- Failed to connect to API server.
- Certificate or x509 errors.
- CNI plugin errors when creating pod sandboxes.

### 5.2 Static pod manifests for core components

Control-plane pods are defined as static pod manifests in `/etc/kubernetes/manifests/`.

```bash
ls -l /etc/kubernetes/manifests

# Confirm expected manifests exist
ls /etc/kubernetes/manifests | egrep 'etcd|kube-apiserver|controller-manager|scheduler'

# Inspect specific manifest
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
sudo cat /etc/kubernetes/manifests/etcd.yaml
```

If a manifest is syntactically wrong or references wrong file paths/certs, kubelet will fail to launch the pod. The errors will appear in `journalctl -u kubelet`.

### 5.3 Confirm core control-plane containers are running

```bash
# containerd
sudo crictl ps | egrep 'etcd|kube-apiserver|kube-controller-manager|kube-scheduler'

# docker (if used)
sudo docker ps | egrep 'etcd|kube-apiserver|kube-controller-manager|kube-scheduler'
```

If one is missing:

1. Inspect its static pod manifest in `/etc/kubernetes/manifests/`.
2. Check kubelet logs for errors referencing that pod.
3. Use runtime logs (`crictl logs` / `docker logs`) if the container is repeatedly crashing.

### 5.4 etcd health diagnostics

If `etcdctl` is installed and certificates are in the default locations:

```bash
export ETCDCTL_API=3

ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

If unhealthy:

- Check disk space on etcd data volume (`df -h`).
- Check etcd container logs using the runtime.
- Avoid deleting `/var/lib/etcd` unless performing a planned restore or one-node cluster recovery.

### 5.5 API server health from the same node

```bash
# Confirm port 6443 is listening
sudo ss -tulpn | grep 6443

# Health endpoints
curl -k https://127.0.0.1:6443/healthz
curl -k https://127.0.0.1:6443/livez
curl -k https://127.0.0.1:6443/readyz
```

If these fail:

- Inspect the `kube-apiserver.yaml` manifest for:
  - etcd endpoints
  - certificate and key file paths
- Check the apiserver container logs via `crictl logs` / `docker logs`.

### 5.6 Network checks specific to control-plane

From the control-plane node:

```bash
# General network
ip addr
ip route

# DNS and connectivity to workers
ping -c 4 WORKER_NODE_IP
```

If control-plane can’t reach workers, or vice versa, the cluster will be unstable.

---

## 6. Worker Node Runbook (No kubectl)

### 6.1 Validate kubelet on worker

```bash
systemctl status kubelet

journalctl -u kubelet -e --no-pager | tail -n 200
```

Look for:

- Errors reaching the API server (e.g., `dial tcp CONTROL_PLANE_IP:6443` failures).
- CNI/plugin errors when setting up pod networking.

### 6.2 Validate container runtime on worker

```bash
# containerd
systemctl status containerd
sudo crictl ps
sudo crictl ps -a

# or docker
systemctl status docker
sudo docker ps
sudo docker ps -a
```

If container runtime is down, kubelet cannot run any pods.

### 6.3 CNI plugin presence and health (Calico/Flannel/etc.)

Even without `kubectl`, you can inspect the CNI plugin on the node:

```bash
# CNI configuration files
ls -l /etc/cni/net.d

# Look for Calico/Flannel processes
ps aux | grep -i calico | grep -v grep
ps aux | grep -i flannel | grep -v grep
ps aux | grep -i cni | grep -v grep
```

If there are no CNI configs or processes, pods will fail to obtain IPs and networking will break.

### 6.4 Network checks from worker to control-plane and internet

```bash
ip addr
ip route

# Reach control-plane
ping -c 4 CONTROL_PLANE_IP

# Reach API server health endpoint (if reachable by IP)
curl -k https://CONTROL_PLANE_IP:6443/healthz

# Check outbound internet if required (for image pulls, Rafay controller)
ping -c 4 8.8.8.8
```

### 6.5 iptables and worker-specific firewall considerations

If a worker is intermittently disconnecting or failing to route traffic correctly, iptables could be misconfigured.

```bash
sudo iptables -L -n -v

# As a troubleshooting step suggested for MKS environments:
sudo iptables -F
```

After flushing iptables, verify:

- Worker can reach control-plane and Rafay Controller.
- If this fixes the problem, implement a better long-term firewall policy.

---

## 7. Rafay Node Agent, Salt Minion, and Chisel

### 7.1 Salt minion service (`salt-minion-rafay`)

On MKS nodes, Rafay uses a Salt minion variant to orchestrate actions. It generally appears as a systemd unit.

```bash
# Check if the Salt minion service is running
systemctl status salt-minion-rafay

# Start or restart if necessary
sudo systemctl start salt-minion-rafay
sudo systemctl restart salt-minion-rafay

# Enable/disable on boot (should normally be enabled)
sudo systemctl enable salt-minion-rafay
```

### 7.2 Minion logs and interpretation

Rafay’s upstream troubleshooting references Salt minion logs located under `/opt/rafay/salt/var/log/salt/`.

```bash
# List log directory
ls -l /opt/rafay/salt/var/log/salt

# Follow minion log in real time
tail -f /opt/rafay/salt/var/log/salt/minion

# Or show last 200 lines
tail -n 200 /opt/rafay/salt/var/log/salt/minion
```

Typical issues visible here:

- DNS resolution failures for the Rafay Controller hostname.
- Connection timeouts or SSL handshake errors.
- Authentication/activation problems when node is bootstrapping.

### 7.3 Chisel tunnel process

Chisel provides a secure TCP tunnel from node to Rafay infrastructure. It may run as a process, systemd service, or container.

**Check as a process:**

```bash
ps aux | grep -i chisel | grep -v grep
```

**Check as a systemd service:**

```bash
systemctl list-units | grep -i chisel

# If present
systemctl status chisel.service
journalctl -u chisel.service -e --no-pager | tail -n 200
```

**Check as a container:**

```bash
# containerd
sudo crictl ps | grep -i chisel

# docker
sudo docker ps | grep -i chisel
```

If chisel is not running or is repeatedly restarting, Rafay may show the cluster/node as offline even if Kubernetes is locally healthy.

### 7.4 Connectivity from node-agent to Rafay Controller

On any node where minion/chisel runs:

```bash
# Resolve Rafay Controller domain
ping -c 4 YOUR_RAFAY_ENDPOINT_DOMAIN
# or
nslookup YOUR_RAFAY_ENDPOINT_DOMAIN

# Check HTTPS reachability
curl -v https://YOUR_RAFAY_ENDPOINT_DOMAIN/
```

If your environment uses HTTP/HTTPS proxies:

```bash
echo $http_proxy
echo $https_proxy
echo $no_proxy
```

Ensure proxies allow outbound 443 to the Rafay Controller, and that `no_proxy` is configured correctly for internal cluster IP ranges.

---

## 8. Scenario-Based Node-Only Playbooks

This section gives **from-scratch, on-node** sequences for common problems.

### 8.1 Scenario A: Rafay shows node/cluster down, but node seems healthy

You are on the node that appears offline in Rafay.

1. **Check systemd for Salt minion and chisel**

   ```bash
   systemctl status salt-minion-rafay
   systemctl status chisel.service   # if such a unit exists
   ```

   - If either is `inactive` or `failed`, restart:
     ```bash
     sudo systemctl restart salt-minion-rafay
     sudo systemctl restart chisel.service
     ```

2. **Check Salt minion logs**

   ```bash
   tail -n 200 /opt/rafay/salt/var/log/salt/minion
   ```

   - Look for errors like DNS failures or TLS handshake issues.

3. **Check chisel process or container**

   ```bash
   ps aux | grep -i chisel | grep -v grep
   sudo crictl ps | grep -i chisel   # or sudo docker ps | grep -i chisel
   ```

   - If chisel is restarting, inspect its logs via `journalctl` or container logs.

4. **Check outbound connectivity to Rafay Controller**

   ```bash
   ping -c 4 YOUR_RAFAY_ENDPOINT_DOMAIN
   curl -v https://YOUR_RAFAY_ENDPOINT_DOMAIN/
   ```

5. **Check iptables**

   ```bash
   sudo iptables -L -n -v

   # As a test step (with caution):
   sudo iptables -F
   ```

6. **Re-check Salt minion logs and service status** after any changes.

---

### 8.2 Scenario B: Worker node fails to join MKS / stuck during provisioning

On the worker node being provisioned:

1. **Validate basic OS requirements**

   ```bash
   # OS details
   cat /etc/os-release

   # CPU cores
   nproc

   # Memory
   free -h

   # Disk usage
   df -h
   ```

   - Ensure OS version, CPU, RAM, and disk meet Rafay/MKS requirements.

2. **Check Salt minion service**

   ```bash
   systemctl status salt-minion-rafay
   sudo systemctl restart salt-minion-rafay
   ```

3. **Check minion logs for preflight failures**

   ```bash
   tail -n 200 /opt/rafay/salt/var/log/salt/minion
   ```

   - Look for preflight check failures: OS, CPU, memory, internet connectivity, registry reachability, etc.

4. **Check connectivity to Rafay Controller and registries**

   ```bash
   ping -c 4 YOUR_RAFAY_ENDPOINT_DOMAIN
   curl -v https://YOUR_RAFAY_ENDPOINT_DOMAIN/
   ```

5. **Check container runtime & kubelet** (if they are already being installed)

   ```bash
   systemctl status containerd    # or docker
   systemctl status kubelet

   journalctl -u kubelet -e --no-pager | tail -n 200
   ```

6. **Adjust OS or network, then retry provisioning** from Rafay side or via `rctl`.

---

### 8.3 Scenario C: Control-plane unstable / API server flapping

On the control-plane node:

1. **Resource checks**

   ```bash
   top
   free -h
   df -h
   ```

2. **Check core services**

   ```bash
   systemctl status kubelet
   systemctl status containerd    # or docker
   ```

3. **Check control-plane containers**

   ```bash
   sudo crictl ps | egrep 'etcd|kube-apiserver|kube-controller-manager|kube-scheduler' \
     || sudo docker ps | egrep 'etcd|kube-apiserver|kube-controller-manager|kube-scheduler'
   ```

4. **Check kubelet and etcd logs**

   ```bash
   journalctl -u kubelet -e --no-pager | tail -n 200
   
   # etcd health and logs
   export ETCDCTL_API=3
   ETCDCTL_API=3 etcdctl ... endpoint health
   # (fill in cert flags as per section 5.4)
   ```

5. **Check API server listening and endpoints**

   ```bash
   sudo ss -tulpn | grep 6443
   curl -k https://127.0.0.1:6443/healthz
   curl -k https://127.0.0.1:6443/livez
   curl -k https://127.0.0.1:6443/readyz
   ```

6. **Check Salt minion logs** for any cluster-wide orchestration issues

   ```bash
   tail -n 200 /opt/rafay/salt/var/log/salt/minion
   ```

---

### 8.4 Scenario D: Pods repeatedly crashing on a node (no kubectl access)

On the affected node:

1. **List containers via runtime**

   ```bash
   # containerd
   sudo crictl ps -a

   # docker
   sudo docker ps -a
   ```

   - Look for containers with repeated exit/restart patterns.

2. **Inspect and get logs of a crashing container**

   ```bash
   # containerd
   sudo crictl inspect CONTAINER_ID | less
   sudo crictl logs CONTAINER_ID

   # docker
   sudo docker inspect CONTAINER_ID_OR_NAME | less
   sudo docker logs CONTAINER_ID_OR_NAME
   ```

   - Identify whether the issue is app-level (stack trace, env var missing) or infra-level (missing volumes, permissions, networking).

3. **Cross-check kubelet logs**

   ```bash
   journalctl -u kubelet -e --no-pager | tail -n 200
   ```

4. **Check CNI** if symptoms are network-related (no IPs, failing to connect services):

   ```bash
   ls -l /etc/cni/net.d
   ps aux | grep -i calico | grep -v grep
   ps aux | grep -i flannel | grep -v grep
   ```

5. **Check node’s network** (to external services or internal endpoints that pods need).

   ```bash
   ip addr
   ip route
   ping -c 4 SERVICE_OR_DB_IP
   ```

---

## 9. Node Debug Checklist (Copy-Paste Sequence)

When you SSH into **any** MKS node (control-plane or worker) to debug, you can run this exact sequence and then look at outputs:

```bash
# 1) OS health

# CPU, load, and top processes
top

# Memory
free -h

# Disk
df -h
lsblk


# 2) Core services

systemctl status kubelet
systemctl status containerd   # or docker
systemctl status salt-minion-rafay


# 3) Logs (recent)

journalctl -u kubelet -e --no-pager | tail -n 200
journalctl -u containerd -e --no-pager | tail -n 200
journalctl -u docker -e --no-pager | tail -n 200
journalctl -u salt-minion-rafay -e --no-pager | tail -n 200

tail -n 200 /opt/rafay/salt/var/log/salt/minion


# 4) Runtime containers

sudo crictl ps || sudo docker ps
sudo crictl ps -a || sudo docker ps -a


# 5) Networking

ip addr
ip route

ping -c 4 CONTROL_PLANE_IP
ping -c 4 YOUR_RAFAY_ENDPOINT_DOMAIN

curl -k https://CONTROL_PLANE_IP:6443/healthz      # if you know APIServer IP
curl -v https://YOUR_RAFAY_ENDPOINT_DOMAIN/


# 6) iptables sanity

sudo iptables -L -n -v
# As last-resort diagnostic step
# sudo iptables -F
```

After collecting this data, you can:

- Decide whether the issue is **resource-related** (CPU, RAM, disk).
- Narrow down to **service problems** (kubelet, containerd/docker, salt-minion-rafay, chisel).
- Confirm or rule out **network/firewall issues** (routing, DNS, iptables).
- Escalate with specific logs and commands already run.

This runbook is intentionally written so that a **junior engineer** can follow it step-by-step, while also giving sufficient depth and syntax details for a **senior engineer** to perform advanced diagnosis entirely from the node side without needing `kubectl`.
