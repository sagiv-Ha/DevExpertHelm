## Part 5 – ConfigMap and Secret Usage
### ConfigMap Usage

The ConfigMap is used to store non-sensitive configuration data for the application.
In this chart, the ConfigMap stores:
message – the application response text
port – the container port
The values are defined in values.yaml:

```bash
config:
  message: "Hello from Helm"
  port: "5678"
```


The ConfigMap template reads these values:

```bash
data:
  message: {{ .Values.config.message }}
  port: {{ .Values.config.port }}
```


The Deployment and DaemonSet consume the ConfigMap in two ways:
1. As environment variables:
MESSAGE
2. As mounted files inside the container:
/etc/myapp/config
This allows dynamic configuration without rebuilding the container image.

---

### Secret Usage
The Secret is used to store sensitive information.
In this chart, the Secret stores:
* apiToken
The value is defined in values.yaml:

```bash 
secret:
  apiToken: "change-me"
```

The Secret template encodes the value using base64:

```bash
apiToken: {{ .Values.secret.apiToken | b64enc }}
```

The Deployment and DaemonSet consume the Secret:
1. As an environment variable:
API_TOKEN
2. As a mounted volume:
/etc/myapp/secret

This ensures sensitive data is separated from the container image and can be managed securely.

Why This Is Important

Configuration is externalized from the image.
Sensitive data is separated from non-sensitive configuration.
Changes to ConfigMap or Secret can be applied using Helm upgrades.
This follows Kubernetes best practices for configuration management.

----

## 1. Helm Lifecycle Explanation

Helm manages Kubernetes applications through a release lifecycle.

The main lifecycle phases are:

### Install

When running:

```bash
helm install myapp ./charts/myapp
```

Helm:

* Renders templates using `values.yaml`
* Sends the generated Kubernetes manifests to the cluster
* Creates a new release with revision 1

### Upgrade

When running:

```bash
helm upgrade myapp ./charts/myapp
```

Helm:

* Re-renders templates with updated values
* Applies changes to existing resources
* Creates a new revision (e.g., revision 2, 3, etc.)
* Maintains release history

### Rollback

When running:

```bash
helm rollback myapp <revision>
```

Helm:

* Re-applies manifests from a previous revision
* Creates a new revision entry for the rollback
* Restores the previous state of resources

### Uninstall

When running:

```bash
helm uninstall myapp
```

Helm:

* Deletes all Kubernetes resources created by the release
* Removes release tracking information

Helm keeps track of revisions, making upgrades and rollbacks predictable and auditable.

---

## 2. Why `helm upgrade --install` Is Preferred

The command:

```bash
helm upgrade --install myapp ./charts/myapp
```

Is preferred because:

* It installs the release if it does not exist.
* It upgrades the release if it already exists.
* It is idempotent.
* It simplifies CI/CD pipelines.
* It prevents errors caused by trying to install an already existing release.

Using this command ensures consistent deployment behavior in both first-time and repeated executions.

---

## 3. Explanation of Each Template and Resource

### Chart.yaml

Defines metadata about the chart:

* Name
* Version
* Type
* Description

This file is required for Helm to recognize the chart.

---

### values.yaml

Contains configurable parameters such as:

* Image repository and tag
* Command arguments
* Service port
* ConfigMap values
* Secret values

This allows customization without modifying templates.

---

### deployment.yaml

Defines a Kubernetes Deployment resource.

Purpose:

* Runs the main application workload.
* Supports scaling via replica count.
* Uses image and tag from `values.yaml`.
* Consumes ConfigMap and Secret as environment variables and mounted volumes.

---

### service.yaml

Defines a Kubernetes Service.

Purpose:

* Exposes the Deployment internally in the cluster.
* Provides stable networking and load balancing.
* Uses configurable port and service type.

---

### daemonset.yaml

Defines a Kubernetes DaemonSet.

Purpose:

* Ensures one pod runs on each node.
* Demonstrates node-level workload deployment.
* Uses the same configuration model as the Deployment.

---

### cronjob.yaml

Defines a Kubernetes CronJob.

Purpose:

* Runs scheduled tasks using a cron expression.
* Demonstrates batch workload management.

---

### job.yaml

Defines a Kubernetes Job.

Purpose:

* Executes a one-time task until completion.
* Used to demonstrate Helm lifecycle management on batch workloads.

---

### configmap.yaml

Defines a Kubernetes ConfigMap.

Purpose:

* Stores non-sensitive configuration data.
* Injected into pods via environment variables and/or mounted volumes.
* Allows configuration updates without rebuilding container images.

---

### secret.yaml

Defines a Kubernetes Secret.

Purpose:

* Stores sensitive information such as API tokens.
* Encoded using base64 in the template.
* Injected securely into pods.
* Keeps secrets separate from container images and source code.

---
