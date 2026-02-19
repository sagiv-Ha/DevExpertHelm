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

*Why This Is Important*

Configuration is externalized from the image.

Sensitive data is separated from non-sensitive configuration.

Changes to ConfigMap or Secret can be applied using Helm upgrades.

This follows Kubernetes best practices for configuration management.