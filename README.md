# DevExpertHelm
---
## Purpose of each resource

This chart deploys several Kubernetes resources to practice writing Helm templates from scratch and managing releases with Helm.

* **Deployment**
  Runs the main application workload with a configurable number of replicas. It uses values from `values.yaml` (image repository/tag/args) and consumes the ConfigMap and Secret.

* **Service**
  Exposes the Deployment internally in the cluster (default `ClusterIP`) on a configurable port.

* **DaemonSet**
  Runs one pod per node. Used to practice deploying a workload across all nodes (and also consumes the ConfigMap and Secret).

* **CronJob**
  Runs a scheduled task based on a cron expression (e.g., periodic echo/log task). Used to practice scheduled workloads and Helm upgrades/rollbacks.

* **Job**
  Runs a one-time task until completion (e.g., print a message). Used to practice batch workloads and Helm lifecycle behavior.

* **ConfigMap**
  Stores non-sensitive configuration such as application message and port. Mounted and/or injected into pods.

* **Secret**
  Stores sensitive data (e.g., API token). Rendered using base64 encoding in the template and mounted and/or injected into pods.

---

## Commands used:

### Install / Upgrade (idempotent)

```bash
helm upgrade --install myapp ./charts/myapp
```

### Verify release status

```bash
helm status myapp
```

### Show release history

```bash
helm history myapp
```

### Rollback to a previous revision

```bash
helm rollback myapp 1
```

### (Optional) Render manifests locally for review

```bash
helm template myapp ./charts/myapp
```

---

## Verification steps

### Verify Kubernetes resources

```bash
kubectl get all
kubectl get configmap
kubectl get secret
```

### Verify pods and image version (after upgrades/rollbacks)

```bash
kubectl describe deploy -l app=myapp-myapp | grep -i image
kubectl describe ds -l app=myapp-myapp | grep -i image
```

### Verify ConfigMap/Secret were injected (if mounted/env is used)

```bash
kubectl describe pod -l app=myapp-myapp | less
```

Look for:

* Environment variables sourced from ConfigMap/Secret
* Volume mounts (e.g., `/etc/myapp/config`, `/etc/myapp/secret`)

---
