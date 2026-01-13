# Complete Rafay RCTL Command Reference

The Rafay **`rctl`** (Rafay Control) CLI is your primary tool for automating infrastructure and workload management across the Rafay platform. It supports both **declarative** (GitOps-style) and **imperative** (direct command) patterns for resource management.

---

## Global Flags

All `rctl` commands support these universal flags:

- **`-p, --project`** – set project context for the command
- **`-o, --output`** – format output as `table` (default), `json`, or `yaml`
- **`-d, --debug`** – enable debug logging for troubleshooting
- **`-v, --verbose`** – verbose mode with detailed output
- **`--wait`** – block until long-running operations (cluster create, workload publish) complete
- **`--dry-run`** – preview changes without applying them
- **`--force`** – force delete (skip validation checks)
- **`-c, --config`** – specify a custom config file path

---

## Configuration Setup

### Initialize Configuration

**Config file method:**
```bash
./rctl config init <path-to-config-file>
```
Download the config file from the Rafay Console → **My Tools** → **Download CLI Config**.

**Environment variable method (for CI/CD):**
```bash
export RCTL_REST_ENDPOINT="console.rafay.dev"
export RCTL_OPS_ENDPOINT="console.rafay.dev"
export RCTL_API_KEY="your-api-key"
export RCTL_PROJECT="defaultproject"
```
ENV variables take precedence over config files.

**View current configuration:**
```bash
./rctl config show
```

**Set project context:**
```bash
./rctl config set project <project-name>
```
Note: project names are case-sensitive.

---

## Cluster Management

### Create/Update Cluster
```bash
./rctl apply -f cluster.yaml
./rctl apply -f cluster.yaml --dry-run          # preview changes
./rctl apply -f cluster.yaml --wait             # wait for completion
```

Declarative approach is strongly recommended for reproducible infrastructure. EKS, AKS, GKE, and bare metal (MKS) clusters are all supported via YAML specification.

### List Clusters
```bash
./rctl get cluster                              # basic list
./rctl get cluster --v3                         # detailed list with timestamps/blueprints
./rctl get cluster -p <project-name>            # specific project
```

### Get Cluster Details
```bash
./rctl get cluster <cluster-name>               # table format
./rctl get cluster <cluster-name> -o json       # JSON (useful for scripting)
./rctl get cluster <cluster-name> -o yaml       # YAML (good for templates)
```

### Download Kubeconfig
```bash
./rctl download kubeconfig                      # unified kubeconfig for all clusters
./rctl download kubeconfig --cluster <name>     # specific cluster kubeconfig
```
Access is proxied through Rafay's Zero Trust Kubectl proxy for secure, audit-logged kubectl access.

### Delete Cluster
```bash
./rctl delete cluster <cluster-name>            # standard delete (checks for associated resources)
./rctl delete cluster <cluster-name> --force    # force delete (only removes controller entry)
```
Use `--force` only when cloud resources are already gone but controller entry persists.

---

## Namespace Management

### Create/Update Namespace
```bash
./rctl apply -f namespace.yaml
```

You can define labels, annotations, resource quotas, network policies, and placement across specific clusters.

### List Namespaces
```bash
./rctl get namespace --v3                       # all namespaces in project
./rctl get namespace <namespace-name> --v3      # specific namespace
./rctl get namespace <namespace-name> -o yaml   # YAML output for templating
```

### Check Namespace Status
```bash
./rctl status namespace <namespace-name> --v3
```
Shows deployment status, assigned clusters, failed clusters.

### Namespace Label Associations
```bash
./rctl apply -f namespacelabelassoc.yaml        # apply labels across namespaces
./rctl get projtagassoc                         # view all associations
./rctl get projtagassoc <association-name>      # specific association
./rctl delete projtagassoc <association-name>   # remove association
```

### Delete Namespace
```bash
./rctl delete namespace -f config.yaml --v3
```
Warning: deletes from all clusters where the namespace is published.

---

## Workload Management

### Create & Publish Workload
```bash
./rctl apply -f workload.yaml                   # creates and publishes in one command
```

Supports **Helm** charts (Helm3), **Kubernetes YAML**, and **Kustomize**. Workloads can pull charts/values from Git or Helm repositories.

### List Workloads
```bash
./rctl get workload --v3                        # all workloads
./rctl get workload -p <project-name> --v3      # specific project
```

### Check Workload Status
```bash
./rctl status workload <workload-name>          # status across all clusters
./rctl status workload <workload-name> --detailed --clusters=cluster1,cluster2
```
Detailed view includes K8s object names, latest conditions, and cluster events.

### Publish/Unpublish Workload
```bash
./rctl publish workload <workload-name>         # publish to clusters
./rctl unpublish workload <workload-name> --v3  # remove from clusters (keep definition)
```

### Delete Workload
```bash
./rctl delete workload <workload-name> --v3     # unpublish and delete definition
```

### Update Workload Configuration
```bash
./rctl update workload <path-to-definition> --v3
```
You can update placement, versions, or configuration even if the workload is already running.

---

## Pipeline Management

### Create/Update Pipeline
```bash
./rctl apply -f pipeline.yaml
```

Define multi-stage pipelines with approval gates, deployment stages, and error handling.

### List & Retrieve Pipelines
```bash
./rctl get pipeline -p <project-name>           # all pipelines
./rctl get pipeline <pipeline-name> -p <project-name>     # specific pipeline
./rctl get pipeline <pipeline-name> -o json     # JSON output
```

### Activate/Deactivate Pipeline
```bash
./rctl activate pipeline <pipeline-name>
./rctl deactivate pipeline <pipeline-name>
./rctl start pipeline <pipeline-name>           # begin execution
./rctl stop pipeline <pipeline-name>            # stop execution
```

### Delete Pipeline
```bash
./rctl delete pipeline <pipeline-name>
```

---

## Repository Management

### Create/Update Repository
```bash
./rctl apply -f repository.yaml
```

Repositories can be Git (for pipelines/values) or Helm (for charts). Share repositories across projects for reuse.

### List Repositories
```bash
./rctl get repository -p <project-name> --v3    # all repositories
./rctl get repository <repo-name> --v3          # specific repository
```

### Validate Repository
```bash
./rctl validate repository <repo-name>          # checks connectivity and config
./rctl validate repository <repo-name> -o yaml  # detailed YAML output
```
Validation runs against all linked agents, showing success/failure per agent.

### Share Repository
Update spec with `sharing.enabled: true` and list projects, then:
```bash
./rctl apply -f repository.yaml --v3
```

### Delete Repository
```bash
./rctl delete repository <repo-name> --v3
```

---

## Add-ons, Blueprints & Cloud Credentials

### Add-ons
```bash
./rctl apply -f addon.yaml                      # create/update
./rctl get addon                                # list all
./rctl get addon <addon-name> -o yaml           # specific addon
./rctl delete addon <addon-name>                # delete
```

### Blueprints (Cluster Templates)
```bash
./rctl apply -f blueprint.yaml                  # create/update
./rctl get blueprint                            # list all
./rctl delete blueprint <blueprint-name>        # delete
```

### Cloud Credentials
```bash
./rctl apply -f credentials.yaml                # create/update (AWS/Azure/GCP)
./rctl get cloudcredential                      # list all
./rctl delete cloudcredential <name>            # delete
```

---

## Registries & Secrets

### Container Registries (ECR, ACR, GCR, Docker Hub)
```bash
./rctl apply -f registry.yaml                   # create/update
./rctl get registry                             # list all
./rctl delete registry <registry-name>          # delete
```

### Secret Stores (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault)
```bash
./rctl apply -f secret-store.yaml               # create/update
./rctl get secretstore                          # list all
./rctl delete secretstore <store-name>          # delete
```

### Secret Groups
```bash
./rctl apply -f secret-group.yaml               # create/update
./rctl get secretgroup                          # list all
./rctl get secretgroup <group-name> -o yaml     # specific group
./rctl delete secretgroup <group-name>          # delete
```

---

## Projects & Multi-Tenancy

### Manage Projects
```bash
./rctl apply -f project.yaml                    # create/update
./rctl get project                              # list all
./rctl get project <project-name>               # specific project
```

Note: Project creation is typically an Org Admin operation via the web console, as RBAC assignments must accompany creation.

---

## Advanced Features

### Templating (Multi-environment, Bulk Creation)
```bash
./rctl apply -t template.tmpl --values values.yaml
```
Use Go templating to generate multiple resources (namespaces, workloads) with variable substitution. Useful for creating dev/stage/prod environments from a single template.

### Validation (Pre-merge CI Checks)
```bash
./rctl validate workload <workload-name>        # validate workload definition
./rctl validate repository <repo-name>          # validate repository connectivity
```

### Dry Run (Preview Changes Safely)
```bash
./rctl apply -f cluster.yaml --dry-run          # shows operations without applying
```
Supported for clusters (EKS, AKS, GKE, MKS); previews cluster creation, node group, and blueprint sync operations.

### Exit Codes (CI/CD Integration)
```bash
#!/bin/bash
./rctl apply -f cluster.yaml
if [ $? -eq 0 ]; then
  echo "Success"
else
  echo "Failure"
fi
```
Exit code 0 = success; any other number = failure. Use in automation pipelines to chain commands.

---

## Resource Support Matrix

The table below shows which resources support **declarative** (`rctl apply`), **imperative** (`rctl create/get/delete`), or both approaches:

| Resource | Declarative | Imperative | Use Case |
|----------|:-----------:|:----------:|----------|
| Clusters | ✓ | ✓ | Provision EKS/AKS/GKE/bare metal clusters |
| Namespaces | ✓ | ✓ | Multi-cluster namespace lifecycle & quotas |
| Workloads | ✓ | ✓ | Deploy Helm charts & K8s YAML across clusters |
| Pipelines | ✓ | ✓ | GitOps deployment workflows with approval gates |
| Repositories | ✓ | ✓ | Git & Helm repository management |
| Add-ons | ✓ | ✓ | Cluster extensions (monitoring, logging, etc.) |
| Blueprints | ✓ | ✓ | Reusable cluster configurations |
| Registries | ✓ | ✓ | Container image registry integration |
| Cloud Credentials | ✓ | ✓ | AWS/Azure/GCP authentication |
| Secret Stores | ✓ | ✓ | Vault/Secrets Manager integration |
| Secret Groups | ✓ | ✓ | Grouped secret management |
| Projects | ✓ | ✓ | Multi-tenancy boundaries |
| Backup/Restore | ✗ | ✓ | Backup policies, jobs, restore workflows |

---

## Common Use Cases & When to Use

| Scenario | Commands |
|----------|----------|
| **First-time setup** | `rctl config init <config-file>` |
| **Create production cluster** | `rctl apply -f cluster.yaml --wait` |
| **Deploy multi-cluster app** | `rctl apply -f workload.yaml` |
| **Check workload health** | `rctl status workload <name> --detailed` |
| **GitOps automation (CI/CD)** | `rctl apply -f resources/ --wait` + exit code checks |
| **Safe preview of changes** | `rctl apply -f resource.yaml --dry-run` |
| **Pre-merge validation** | `rctl validate workload <name>` in CI pipeline |
| **Troubleshooting failures** | `rctl --debug get <resource>` or `rctl --verbose get <resource>` |
| **Download kubeconfig for kubectl** | `rctl download kubeconfig --cluster <name>` |
| **Bulk resource creation** | `rctl apply -t template.tmpl --values vars.yaml` |
| **Unpublish without deleting** | `rctl unpublish workload <name>` |
| **Remove cluster safely** | `rctl delete cluster <name>` (or `--force` if already deleted in cloud) |

---

## Best Practices

1. **Always use declarative approach** – store YAML in Git for version control and audit trails
2. **Validate before deploying** – use `--dry-run` and `rctl validate` in CI pipelines
3. **Use `--wait` in automation** – ensures operations complete before continuing
4. **Set project context** – always specify `-p <project>` or use `rctl config set project`
5. **Use JSON/YAML output for automation** – `rctl get <resource> -o json` for scripting
6. **Check exit codes in scripts** – validate success/failure for CI/CD pipelines
7. **Environment variables for CI/CD** – use `RCTL_*` env vars instead of config files in pipelines
8. **Template for multi-environment consistency** – use templating for dev/stage/prod
9. **Dry run before major changes** – test cluster updates with `--dry-run` first
10. **Enable verbose/debug when troubleshooting** – `--verbose` and `--debug` flags help diagnose issues

---

## Installation & PATH Setup

After downloading `rctl`, add it to your system PATH:

**macOS/Linux:**
```bash
export PATH=$PATH:/path/to/rctl
```

**Windows:**
- Press Windows key → search "environment variables"
- Edit system environment variables → **PATH** → Add path to `rctl.exe`
- Restart command prompt

---

## Summary

`rctl` is a powerful CLI that unifies cluster provisioning, workload deployment, pipeline management, and multi-tenancy across the Rafay platform. **Prefer declarative YAML** (`rctl apply`) for production, use **imperative commands** for ad-hoc tasks, and leverage **flags like `--wait`, `--dry-run`, and `-o json`** for robust automation. Save this reference for quick lookups on command syntax and use cases.
