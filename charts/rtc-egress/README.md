# Agora RTC Egress Server Helm Chart

This Helm chart deploys the Agora RTC Egress Server on a Kubernetes cluster using the Helm package manager.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- Redis server (external or as subchart)
- Valid Agora App ID and credentials

## Installing the Chart

To install the chart from the Agora Helm repository:

```bash
# Add the Agora Helm repository
helm repo add agora https://helm.agora.build
helm repo update

# Create namespace
kubectl create namespace com.myrtc.egress

# Install the chart
helm install my-rtc-egress agora/rtc-egress \
  --namespace com.myrtc.egress \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST \
  --values values.yaml
```

To install from source:

```bash
# Clone the repository
git clone https://github.com/AgoraIO/RTC-Egress.git
cd RTC-Egress

# Install the chart
helm install my-rtc-egress ./charts/rtc-egress \
  --namespace com.myrtc.egress \
  --create-namespace \
  --set agora.appId=YOUR_AGORA_APP_ID
```

## Uninstalling the Chart

To uninstall/delete the `my-rtc-egress` deployment:

```bash
helm delete my-rtc-egress --namespace com.myrtc.egress
```

## Configuration

The following table lists the configurable parameters of the rtc-egress chart and their default values.

### Image Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.registry` | Container image registry | `ghcr.io` |
| `image.repository` | Container image repository | `AgoraIO/RTC-Egress` |
| `image.tag` | Container image tag | `latest` |
| `image.pullPolicy` | Container image pull policy | `IfNotPresent` |

### Agora Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `agora.appId` | Agora Application ID (required) | `YOUR_AGORA_APP_ID` |
| `agora.accessToken` | Agora channel access token | `""` |
| `agora.channelName` | Default channel name | `""` |
| `agora.egressUid` | Egress UID | `"42"` |

### Redis Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `redis.external.enabled` | Use external Redis | `true` |
| `redis.external.host` | External Redis host | `redis-server` |
| `redis.external.port` | External Redis port | `6379` |
| `redis.external.password` | External Redis password | `""` |
| `redis.external.database` | Redis database number | `0` |

### Server Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `server.ginMode` | Gin framework mode | `release` |
| `server.apiPort` | API server port | `8080` |
| `server.healthPort` | Health check port | `8182` |
| `server.canvasTemplatePort` | Canvas template port | `3000` |
| `server.logLevel` | Log level | `info` |

### Persistence Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.recordings.size` | Recordings volume size | `100Gi` |
| `persistence.snapshots.size` | Snapshots volume size | `50Gi` |
| `persistence.logs.size` | Logs volume size | `10Gi` |

### Recording Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `recording.enabled` | Enable recording functionality | `false` |
| `recording.format` | Recording format | `mp4` |
| `recording.video.width` | Video width | `1280` |
| `recording.video.height` | Video height | `720` |
| `recording.video.fps` | Video frame rate | `30` |
| `recording.video.bitrate` | Video bitrate | `2000000` |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU resource limits | `2000m` |
| `resources.limits.memory` | Memory resource limits | `4Gi` |
| `resources.requests.cpu` | CPU resource requests | `500m` |
| `resources.requests.memory` | Memory resource requests | `1Gi` |

### Autoscaling

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable horizontal pod autoscaler | `false` |
| `autoscaling.minReplicas` | Minimum number of replicas | `1` |
| `autoscaling.maxReplicas` | Maximum number of replicas | `10` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization | `80` |

## Examples

### Basic Installation

```bash
helm install my-egress agora/rtc-egress \
  --set agora.appId=your-app-id \
  --set redis.external.host=redis.example.com
```

### Production Installation

```bash
helm install prod-egress agora/rtc-egress \
  --namespace production \
  --values values-production.yaml \
  --set agora.appId=your-app-id \
  --set redis.external.host=redis.prod.example.com \
  --set redis.external.password=your-redis-password
```

### Development with Local Redis

```bash
helm install dev-egress agora/rtc-egress \
  --namespace development \
  --set agora.appId=your-app-id \
  --set redis.subchart.enabled=true \
  --set redis.external.enabled=false
```

## Monitoring

The chart includes health checks and readiness probes:

- Health endpoint: `http://pod-ip:8182/health`
- API endpoint: `http://pod-ip:8080/egress/v1/task/status`

## Troubleshooting

### Common Issues

1. **Pod not starting**: Check if Agora App ID is configured correctly
   ```bash
   kubectl logs -l app.kubernetes.io/name=rtc-egress
   ```

2. **Redis connection issues**: Verify Redis configuration
   ```bash
   kubectl exec -it deployment/rtc-egress-server -- env | grep REDIS
   ```

3. **Storage issues**: Check PVC status
   ```bash
   kubectl get pvc -l app.kubernetes.io/name=rtc-egress
   ```

### Debugging

Enable debug mode:
```bash
helm upgrade my-egress agora/rtc-egress \
  --set server.ginMode=debug \
  --set server.logLevel=debug
```

## Contributing

Please read the [contributing guidelines](../../../CONTRIBUTING.md) before submitting pull requests.

## License

This chart is licensed under the MIT License. See [LICENSE](../../../LICENSE) for more information.