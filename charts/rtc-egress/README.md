# Agora RTC Egress Suite Helm Chart

This Helm chart deploys the complete Agora RTC Egress Suite on a Kubernetes cluster using the Helm package manager.

## Architecture Overview

The suite supports **two deployment architectures**:

### Production Architecture (Separated - Recommended)

- **API Server**: 1 pod (default) - HTTP API that publishes tasks to Redis queues (port 8080, health 8181)
- **Egress**: 3 pods (default) with auto-scaling - Native C++ recording/snapshot workers (health port 8182)
- **Flexible Recorder**: 2 pods (default) - Web recording via web recorder engine (health port 8183)
- **Uploader**: 1 pod (default) - File upload service with S3 integration (health port 8184)
- **Webhook Notifier**: 2 pods (default) with auto-scaling - Redis keyspace notifications → HTTP webhooks (port 8080, health 8185)

### Monolithic Architecture (Legacy)

All services run in a single container - simpler but less flexible for scaling and fault isolation.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- Redis server (external)
- Valid Agora App ID and credentials
- S3 bucket (for production file uploads)

## Installing the Chart

### Production Installation (Separated Architecture)

For production with the recommended separated 5-service architecture:

```bash
# Add the Agora Helm repository
helm repo add agora https://helm.agora.build
helm repo update

# Create namespace
kubectl create namespace rtc-egress-prod

# Install with production values (1 api-server, 3 egress, 2 flexible-recorder, 1 uploader, 2 webhook-notifier)
helm install rtc-egress agora/rtc-egress \
  --namespace rtc-egress-prod \
  --values values-production-separated.yaml \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST \
  --set s3.bucket=YOUR_S3_BUCKET
```

### Development Installation

For development or testing:

```bash
# Install with default values (separated architecture - uses values.yaml automatically)
helm install my-rtc-egress agora/rtc-egress \
  --namespace rtc-egress-dev \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST
```

### Monolithic Installation (Legacy Mode)

For backward compatibility or simpler deployments:

```bash
# Install in monolithic mode (single container with all services)
helm install rtc-egress-mono agora/rtc-egress \
  --namespace rtc-egress-mono \
  --values values-monolithic.yaml \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST
```

### Private Registry Installation

For private GitHub Container Registry images:

```bash
# Create image pull secret
kubectl create secret docker-registry ghcr-secret \
  --namespace rtc-egress \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GITHUB_TOKEN

# Install with private registry using command-line overrides
helm install my-rtc-egress agora/rtc-egress \
  --namespace rtc-egress \
  --create-namespace \
  --set agora.appId=$AGORA_APP_ID \
  --set redis.external.host=$REDIS_HOST \
  --set redis.external.port=$REDIS_PORT \
  --set 'global.imagePullSecrets[0].name=ghcr-secret'
```

### From Source Installation

```bash
# Clone the repository
git clone https://github.com/AgoraIO/Helm-Charts.git
cd Helm-Charts

# Install the chart
helm install my-rtc-egress ./charts/rtc-egress \
  --namespace rtc-egress \
  --create-namespace \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST
```

## Configuration

The chart supports both **separated** and **monolithic** architectures.

### Values Files

| File | Purpose | Usage |
|------|---------|-------|
| `values.yaml` | Development/testing defaults | Default (auto-loaded) |
| `values-production-separated.yaml` | Production separated architecture | `--values values-production-separated.yaml` |
| `values-monolithic.yaml` | Monolithic single-container mode | `--values values-monolithic.yaml` |

### Architecture Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `architecture` | Deployment architecture (`separated` or `monolithic`) | `separated` |

### Separated Architecture - Service Configuration

#### API Server
| Parameter | Description | Default |
|-----------|-------------|---------|
| `apiServer.enabled` | Enable API Server service | `true` |
| `apiServer.image.repository` | API Server image repository | `agoraio/api-server` |
| `apiServer.replicaCount` | Number of API Server replicas | `1` |
| `apiServer.autoscaling.enabled` | Enable autoscaling | `false` |
| `apiServer.autoscaling.maxReplicas` | Maximum replicas | `5` |
| `apiServer.resources.limits.cpu` | CPU limit | `500m` |
| `apiServer.resources.limits.memory` | Memory limit | `512Mi` |

#### Egress Service
| Parameter | Description | Default |
|-----------|-------------|---------|
| `egress.enabled` | Enable Egress service | `true` |
| `egress.image.repository` | Egress image repository | `agoraio/egress` |
| `egress.replicaCount` | Number of Egress replicas | `3` |
| `egress.workers` | Number of worker processes | `4` |
| `egress.autoscaling.enabled` | Enable autoscaling | `false` |
| `egress.autoscaling.maxReplicas` | Maximum replicas | `10` |
| `egress.resources.limits.cpu` | CPU limit | `2000m` |
| `egress.resources.limits.memory` | Memory limit | `4Gi` |

#### Flexible Recorder
| Parameter | Description | Default |
|-----------|-------------|---------|
| `flexibleRecorder.enabled` | Enable Flexible Recorder service | `true` |
| `flexibleRecorder.image.repository` | Flexible Recorder image repository | `agoraio/flexible-recorder` |
| `flexibleRecorder.replicaCount` | Number of replicas | `2` |
| `flexibleRecorder.autoscaling.enabled` | Enable autoscaling | `false` |
| `flexibleRecorder.autoscaling.maxReplicas` | Maximum replicas | `8` |
| `flexibleRecorder.webRecorder.baseUrl` | Web Recorder Engine URL | `http://localhost:8001` |

#### Uploader Service
| Parameter | Description | Default |
|-----------|-------------|---------|
| `uploader.enabled` | Enable Uploader service | `true` |
| `uploader.image.repository` | Uploader image repository | `agoraio/uploader` |
| `uploader.replicaCount` | Number of replicas | `1` |
| `uploader.autoscaling.enabled` | Enable autoscaling | `false` |
| `uploader.autoscaling.maxReplicas` | Maximum replicas | `3` |

#### Webhook Notifier
| Parameter | Description | Default |
|-----------|-------------|---------|
| `webhookNotifier.enabled` | Enable Webhook Notifier service | `true` |
| `webhookNotifier.image.repository` | Webhook Notifier image repository | `agoraio/webhook-notifier` |
| `webhookNotifier.replicaCount` | Number of replicas | `2` |
| `webhookNotifier.autoscaling.enabled` | Enable autoscaling | `false` |
| `webhookNotifier.autoscaling.maxReplicas` | Maximum replicas | `4` |

### Monolithic Architecture - Legacy Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.registry` | Container image registry | `ghcr.io` |
| `image.repository` | Container image repository | `agoraio/rtc-egress` |
| `image.tag` | Container image tag | `latest` |
| `image.pullPolicy` | Container image pull policy | `Always` |
| `replicaCount` | Number of monolithic replicas | `3` |

### Agora Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `agora.appId` | Agora Application ID (required) | `YOUR_AGORA_APP_ID` |
| `agora.accessToken` | Agora channel access token | `""` |
| `agora.channelName` | Default channel name | `dev-test` (separated), `egress_test` (monolithic) |
| `agora.egressUid` | Egress UID | `"42"` |

### Redis Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `redis.external.enabled` | Use external Redis | `true` |
| `redis.external.host` | External Redis host | `localhost` |
| `redis.external.port` | External Redis port | `6379` |
| `redis.external.password` | External Redis password | `""` |
| `redis.external.database` | Redis database number | `0` |

### Server Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `server.ginMode` | Gin framework mode | `debug` (dev), `release` (prod) |
| `server.apiPort` | API server port | `8080` |
| `server.healthPort` | Health check port | `8182` |
| `server.canvasTemplatePort` | Canvas template port | `3000` |
| `server.logLevel` | Log level | `debug` (dev), `info` (prod) |
| `pod.region` | Pod region identifier | `""` |
| `pod.workers` | Number of worker processes | `4` |

### Persistence Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.recordings.size` | Recordings volume size | `100Gi` (dev), `1Ti` (prod) |
| `persistence.recordings.accessMode` | Access mode | `ReadWriteOnce` (dev), `ReadWriteMany` (prod) |
| `persistence.snapshots.size` | Snapshots volume size | `50Gi` (dev), `500Gi` (prod) |
| `persistence.logs.size` | Logs volume size | `10Gi` (dev), `100Gi` (prod) |

### S3 Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `s3.enabled` | Enable S3 upload | `false` (dev), `true` (prod) |
| `s3.bucket` | S3 bucket name | `""` |
| `s3.region` | S3 region | `us-west-2` |
| `s3.accessKey` | S3 access key | `""` |
| `s3.secretKey` | S3 secret key | `""` |

### Autoscaling Configuration

Each service supports independent autoscaling:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `{service}.autoscaling.enabled` | Enable HPA for service | `false` (dev), varies (prod) |
| `{service}.autoscaling.minReplicas` | Minimum replicas | Service-specific |
| `{service}.autoscaling.maxReplicas` | Maximum replicas | Service-specific |
| `{service}.autoscaling.targetCPUUtilizationPercentage` | CPU scaling threshold | `70` |
| `{service}.autoscaling.targetMemoryUtilizationPercentage` | Memory scaling threshold | `80` |

### Monitoring Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `monitoring.enabled` | Enable monitoring | `false` (dev), `true` (prod) |
| `monitoring.serviceMonitor.enabled` | Enable Prometheus ServiceMonitor | `false` (dev), `true` (prod) |
| `monitoring.redis.enabled` | Enable Redis metrics | `false` (dev), `true` (prod) |

## Advanced Configuration Examples

### Enable Autoscaling for All Services

```bash
helm install rtc-egress agora/rtc-egress \
  --set apiServer.autoscaling.enabled=true \
  --set egress.autoscaling.enabled=true \
  --set flexibleRecorder.autoscaling.enabled=true \
  --set uploader.autoscaling.enabled=true \
  --set webhookNotifier.autoscaling.enabled=true \
  --set agora.appId=YOUR_AGORA_APP_ID
```

### Custom Instance Counts

```bash
helm install my-rtc-egress agora/rtc-egress \
  --namespace rtc-egress \
  --set apiServer.replicaCount=2 \
  --set egress.replicaCount=5 \
  --set flexibleRecorder.replicaCount=3 \
  --set uploader.replicaCount=2 \
  --set webhookNotifier.replicaCount=3 \
  --set agora.appId=$AGORA_APP_ID \
  --set redis.external.host=$REDIS_HOST
```

### Selective Service Deployment

```bash
# Deploy only API Server and Egress
helm install rtc-egress agora/rtc-egress \
  --set apiServer.enabled=true \
  --set egress.enabled=true \
  --set flexibleRecorder.enabled=false \
  --set uploader.enabled=false \
  --set webhookNotifier.enabled=false \
  --set agora.appId=YOUR_AGORA_APP_ID
```

### Production with Enhanced Security

```bash
helm install rtc-egress agora/rtc-egress \
  --values values-production-separated.yaml \
  --set podSecurityContext.runAsUser=1001 \
  --set podSecurityContext.runAsGroup=1001 \
  --set securityContext.readOnlyRootFilesystem=false \
  --set agora.appId=YOUR_AGORA_APP_ID
```

## Monitoring and Observability

### Health Checks

Each service provides health endpoints:
- API Server: `http://pod-ip:8181/health`
- Egress: `http://pod-ip:8182/health`
- Flexible Recorder: `http://pod-ip:8183/health`
- Uploader: `http://pod-ip:8184/health`
- Webhook Notifier: `http://pod-ip:8185/health`

### Monitoring Commands

```bash
# Check all service pods
kubectl get pods -n rtc-egress -l app.kubernetes.io/name=rtc-egress

# Monitor autoscaling
kubectl get hpa -n rtc-egress

# View service-specific logs
kubectl logs -n rtc-egress -l app.kubernetes.io/component=api-server
kubectl logs -n rtc-egress -l app.kubernetes.io/component=egress

# Real-time monitoring
watch kubectl get hpa,pods -n rtc-egress
```

## Troubleshooting

### Common Issues

1. **Pods not starting**: Check Agora App ID configuration
   ```bash
   kubectl logs -l app.kubernetes.io/name=rtc-egress -n rtc-egress
   ```

2. **Redis connection issues**: Verify Redis configuration
   ```bash
   kubectl get pods -n rtc-egress -l app.kubernetes.io/component=api-server
   kubectl logs <pod-name> -n rtc-egress
   ```

3. **Storage issues**: Check PVC status
   ```bash
   kubectl get pvc -n rtc-egress -l app.kubernetes.io/name=rtc-egress
   ```

4. **Service discovery issues**: Check service endpoints
   ```bash
   kubectl get svc -n rtc-egress
   kubectl get endpoints -n rtc-egress
   ```

### Enable Debug Mode

```bash
# Enable debug logging
helm upgrade rtc-egress agora/rtc-egress \
  --set server.ginMode=debug \
  --set server.logLevel=debug \
  --reuse-values
```

### Scale Services Manually

```bash
# Scale specific services
kubectl scale deployment rtc-egress-egress --replicas=5 -n rtc-egress
kubectl scale deployment rtc-egress-api-server --replicas=2 -n rtc-egress
```

## Uninstalling the Chart

```bash
# Uninstall the release
helm uninstall rtc-egress -n rtc-egress

# Clean up persistent volumes (if needed)
kubectl delete pvc -l app.kubernetes.io/name=rtc-egress -n rtc-egress
```

## Additional Documentation

- [Autoscaling Guide](AUTOSCALING.md) - Complete guide to dynamic scaling
- [Deployment Modes](DEPLOYMENT-MODES.md) - Comparison of architectures and deployment patterns

## Contributing

Please read the [contributing guidelines](../../../CONTRIBUTING.md) before submitting pull requests.

## License

This chart is licensed under the MIT License. See [LICENSE](../../../LICENSE) for more information.