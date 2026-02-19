# DevExpertHelm Copilot Instructions

## Project Overview
This is a Kubernetes Helm chart project that packages a simple multi-resource application (`myapp`) using the hashicorp/http-echo container image. The chart demonstrates best practices for deploying various Kubernetes resource types.

## Architecture & Key Components

### Chart Structure
- **Chart root**: `charts/myapp/` - Single production Helm chart
- **Chart metadata**: [Chart.yaml](charts/myapp/Chart.yaml) defines `apiVersion: v2`, `version: 0.1.0`, `appVersion: "0.2.3"`
- **Values configuration**: [values.yaml](charts/myapp/values.yaml) centralizes all parameterizable settings
- **Templates directory**: `templates/` contains 7 Kubernetes resource templates

### Deployed Resource Types
The chart creates these Kubernetes resources with consistent naming:
1. **Deployment** ([deployment.yaml](charts/myapp/templates/deployment.yaml)) - Main application pod (1 replica)
2. **DaemonSet** ([daemonset.yaml](charts/myapp/templates/daemonset.yaml)) - Runs on every node
3. **Service** ([service.yaml](charts/myapp/templates/service.yaml)) - ClusterIP exposing port 5678
4. **Job** ([job.yaml](charts/myapp/templates/job.yaml)) - One-time batch task
5. **CronJob** ([cronjob.yaml](charts/myapp/templates/cronjob.yaml)) - Runs every 5 minutes (schedule: `*/5 * * * *`)
6. **ConfigMap** ([configmap.yaml](charts/myapp/templates/configmap.yaml)) - Stores MESSAGE and PORT config
7. **Secret** ([secret.yaml](charts/myapp/templates/secret.yaml)) - Stores base64-encoded secrets

### Naming Convention
All resources use a derived fullname pattern:
```
{{- $fullname := printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" -}}
```
Example: Release `myapp` + Chart `myapp` = `myapp-myapp` (resource names like `myapp-myapp-config`, `myapp-myapp-cron`).

## Critical Values & Configuration

### Core Image Settings
Located in [values.yaml](charts/myapp/values.yaml):
- **Image**: `hashicorp/http-echo:0.3.0` with `IfNotPresent` pull policy
- **Container args**: Deployment uses `-text=Hello from Helm` (parameterized via `.Values.image.command`)
- **Port**: Service and DaemonSet use port `5678` (from `.Values.service.port`)

### ConfigMap/Secret Integration
- **ConfigMap entries**: MESSAGE and PORT passed as data keys
- **Secret dummy field**: Contains base64-encoded placeholder (`temp` → `dGVtcA==`)
- These are mounted/referenced in deployment specs for environment configuration

## Common Workflows

### Helm Install
```bash
helm install myapp charts/myapp/
```
Creates all 7 resources in the default namespace with Chart version 0.1.0, app version 0.2.3.

### Helm Upgrade
```bash
helm upgrade myapp charts/myapp/
```
Updates existing release; watch for immutability issues (see Known Issues below).

### Helm Rollback
```bash
helm rollback myapp <revision>
```
Reverts to previous deployment state; `outputs/helm-rollback.txt` confirms success.

### Verify Deployment
```bash
kubectl get all -n default
kubectl describe job myapp-myapp-job
kubectl logs -l app=myapp-myapp
```

## Known Issues & Patterns

### Job Immutability Bug
Historical failure (revision 2): Job spec template fields become immutable after creation. When upgrading with label/annotation changes, Kubernetes rejects the update. **Solution**: Jobs cannot be modified in-place; consider `Job.spec.backoffLimit` or deletion before re-creation strategies.

### Truncation Logic
Different templates use different truncation lengths:
- ConfigMap/Service/Deployment: `trunc 63` (DNS-1123 subdomain compliance)
- Job/DaemonSet: `trunc 55` or `trunc 50` (accounting for suffixes like `-job`, `-ds`)

Maintain consistency when adding templates or extending resource names.

### CronJob History Limits
[cronjob.yaml](charts/myapp/templates/cronjob.yaml) sets:
- `successfulJobsHistoryLimit: 1`
- `failedJobsHistoryLimit: 1`

This prevents accumulation of completed job objects; adjust if audit trail is needed.

## Development Conventions

### Template Helpers
- Use `printf "%s-%s" .Release.Name .Chart.Name` for consistent resource naming
- Use `toYaml` filter for complex value structures (e.g., `{{ toYaml .Values.image.command | nindent 12 }}`)
- Quote string values: `{{ .Values.config.message | quote }}`
- Base64 encode sensitive data: `{{ "temp" | b64enc | quote }}`

### Testing Changes
After modifying templates or values:
1. Run `helm template myapp charts/myapp/` to preview rendered YAML before applying
2. Use `helm upgrade myapp charts/myapp/ --dry-run --debug` for full validation
3. Check `outputs/` directory for helm command logs (install, upgrade, rollback records)

### File Locations
- **Always edit values first** for parameterization changes (values.yaml)
- **Template changes** only when adding new resources or modifying structure
- **Chart.yaml changes** only for version bumps (use semantic versioning)

## Integration Points
- **Kubernetes API**: All resources target Kubernetes batch/v1, apps/v1, and v1 APIs
- **Container image**: External `hashicorp/http-echo:0.3.0` - update tag in values.yaml only
- **Namespace**: Defaults to `default`; can be set with `helm install -n <namespace>`

## Resources
- Helm documentation: Official Helm chart best practices
- Kubernetes API: Reference Job, CronJob, DaemonSet immutability rules
- Related files: Review `outputs/helm-history.txt` for deployment history and errors
