# Privacera Uber Helm Chart

[![Release](https://img.shields.io/github/v/release/evilgenius-jp/privacera-uber-chart)](https://github.com/evilgenius-jp/privacera-uber-chart/releases)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

A comprehensive Helm chart for deploying Privacera components including Config Portal, Connector, and Operator.

This Helm chart deploys both the Privacera Connector and Operator using the [privacera-base-chart](https://evilgenius-jp.github.io/my-base-chart) as a dependency. The base chart provides all Kubernetes resources, while this chart contains only the service-specific configuration overrides.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- Access to ECR repository: `944725613590.dkr.ecr.us-east-1.amazonaws.com/jp/connector`

## Adding the Helm Repository

To use this chart, first add the Privacera Helm repository:

```bash
helm repo add privacera-uber https://evilgenius-jp.github.io/privacera-uber-chart
helm repo update
```

## Installation

### 1. Deploy Operator Only (d2p mode)

```bash
helm install my-operator privacera-uber/privacera-uber \
  --set tags.d2p=true \
  --namespace my-data-plane \
  --create-namespace
```

### 2. Deploy with Custom Values File

```bash
# Download values file from config-portal UI, then:
helm install my-operator privacera-uber/privacera-uber \
  -f my-runtime-plane-values.yaml \
  --namespace my-data-plane \
  --create-namespace
```

### 3. Custom Installation

```bash
# Override specific values
helm install privacera-uber . \
  --set connector.image.tag=v1.2.0 \
  --set operator.env.LOG_LEVEL=DEBUG \
  --namespace jp-test \
  --create-namespace

# Use custom values file
helm install privacera-uber . \
  -f custom-values.yaml \
  --namespace jp-test \
  --create-namespace
```

## Configuration

The chart configuration is under the `connector` and `operator` keys, which map to the base chart values. Only essential overrides are included by default.

### Key Configuration

#### Connector
```yaml
connector:
  # Application settings
  app:
    name: "privacera-connector"
    namespace: "jp-test"

  # Container image
  image:
    hub: "944725613590.dkr.ecr.us-east-1.amazonaws.com"
    repository: "jp/connector"
    tag: "latest"

  # Environment variables
  env:
    CONFIG_SERVER_URL: "http://config-server:4549"
    TENANT_ID: "17568760930036"
    # ... other variables

  # Secrets (base64 encoded)
  secrets:
    data:
      DB_PASSWORD: "d2VsY29tZTE="
```

#### Operator
```yaml
operator:
  # Application settings
  app:
    name: "privacera-operator"
    namespace: "jp-test"

  # Container image
  image:
    hub: "944725613590.dkr.ecr.us-east-1.amazonaws.com"
    repository: "jp/operator"
    tag: "latest"

  # Environment variables
  env:
    TRUST3_SERVER_URL: "http://config-server:4549"
    TENANT_ID: "17568760930036"
    # ... other variables
```

### Common Customizations

#### Change Image Tag
```yaml
connector:
  image:
    tag: "v1.0.1"
```

#### Update Environment Variables
```yaml
connector:
  env:
    variables:
      LOG_LEVEL: "DEBUG"
      QUERY_INTERVAL: "10"
```

#### Add Resource Limits
```yaml
connector:
  resources:
    enabled: true
    requests:
      cpu: "200m"
      memory: "1Gi"
    limits:
      cpu: "1000m"
      memory: "2Gi"
```

#### Enable Persistent Storage
```yaml
connector:
  persistentVolumeClaims:
    logs-storage:
      enabled: true
      size: "5Gi"
      mountPath: "/app/logs"
```

## Management Commands

```bash
# Upgrade
helm upgrade privacera-connector . --namespace privacera

# Check status
helm status privacera-connector --namespace privacera

# View values
helm get values privacera-connector --namespace privacera

# Uninstall
helm uninstall privacera-connector --namespace privacera
```

## Troubleshooting

```bash
# Check pod status
kubectl get pods -n privacera -l app=privacera-connector

# View logs
kubectl logs -n privacera -l app=privacera-connector -f

# Describe pod
kubectl describe pod -n privacera -l app=privacera-connector
```

## Architecture

This chart uses the privacera-base-chart dependency to provide:
- Deployment
- ConfigMaps
- Secrets
- Service Account
- EmptyDir volumes
- All other standard Kubernetes resources

The connector runs as a single-replica deployment that connects to:
- Config Server (for configuration management)
- Database (for data processing)
- No inbound network access required