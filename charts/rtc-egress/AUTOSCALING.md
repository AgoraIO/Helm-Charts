# Dynamic Horizontal Scaling Guide

The Agora RTC Egress Helm chart supports **dynamic horizontal scaling (autoscaling)** for all 5 services using Kubernetes HPA (Horizontal Pod Autoscaler).

## ✅ Current Autoscaling Support

### Production Configuration (values-production-separated.yaml)

| Service | Min | Max | Scaling Triggers | Status |
|---------|-----|-----|------------------|--------|
| **API Server** | 1 | 3 | CPU 70%, Memory 80% | ✅ Enabled |
| **Egress** | 3 | 15 | CPU 70%, Memory 80%, Redis Queue | ✅ Enabled |
| **Flexible Recorder** | 2 | 6 | CPU 70%, Memory 80%, Redis Queue | ✅ Enabled |
| **Uploader** | 1 | 4 | CPU 70%, Memory 80% | ✅ Enabled |
| **Webhook Notifier** | 2 | 8 | CPU 70%, Memory 80% | ✅ Enabled |

### Default Configuration (values.yaml)

**All services have autoscaling disabled by default** for development/testing environments. You can enable it per service.

## 🚀 How to Enable/Configure Autoscaling

### 1. Enable Autoscaling for Specific Services

```yaml
apiServer:
  autoscaling:
    enabled: true          # Enable autoscaling
    minReplicas: 1          # Minimum pods
    maxReplicas: 3          # Maximum pods
    targetCPUUtilizationPercentage: 70     # Scale when CPU > 70%
    targetMemoryUtilizationPercentage: 80  # Scale when Memory > 80%

egress:
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 15
    targetCPUUtilizationPercentage: 70
    targetMemoryUtilizationPercentage: 80
    redis:
      enabled: true                    # Enable Redis-based scaling
      queueName: "egress:record:*"      # Queue pattern to monitor
      targetQueueLength: "5"            # Scale when queue > 5 tasks per pod

flexibleRecorder:
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 6
    targetCPUUtilizationPercentage: 70
    targetMemoryUtilizationPercentage: 80
    redis:
      enabled: true                    # Monitor web recording queues
      queueName: "egress:web:*"
      targetQueueLength: "3"

uploader:
  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 4
    targetCPUUtilizationPercentage: 70
    targetMemoryUtilizationPercentage: 80

webhookNotifier:
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 8
    targetCPUUtilizationPercentage: 70
    targetMemoryUtilizationPercentage: 80
```

### 2. Install with Autoscaling Enabled

```bash
# Production with all autoscaling enabled
helm install rtc-egress agora/rtc-egress \
  --values values-production-separated.yaml \
  --set agora.appId=YOUR_APP_ID

# Selective autoscaling for development
helm install rtc-egress agora/rtc-egress \
  --set egress.autoscaling.enabled=true \
  --set flexibleRecorder.autoscaling.enabled=true \
  --set agora.appId=YOUR_APP_ID
```

### 3. Runtime Scaling Control

```bash
# Check current HPA status
kubectl get hpa -n rtc-egress-prod

# View scaling events
kubectl describe hpa rtc-egress-egress -n rtc-egress-prod

# Manual scaling (overrides HPA temporarily)
kubectl scale deployment rtc-egress-egress --replicas=5 -n rtc-egress-prod

# Enable/disable autoscaling at runtime
helm upgrade rtc-egress agora/rtc-egress \
  --set apiServer.autoscaling.enabled=true \
  --set apiServer.autoscaling.maxReplicas=5
```

## 📊 Scaling Triggers

### CPU & Memory Based Scaling
- **CPU Usage > 70%**: Add more pods
- **Memory Usage > 80%**: Add more pods  
- **CPU/Memory < threshold**: Remove excess pods

### Redis Queue Based Scaling (Advanced)
For `egress` and `flexibleRecorder` services:
- **Queue Length > Target**: Scale up when too many pending tasks
- **Queue Length < Target**: Scale down when queues are empty

**Requirements for Redis scaling:**
- Prometheus with Redis Exporter
- ServiceMonitor for metrics collection
- Custom metrics API server

Note:
- Ensure your service subscribes to the same Redis queue namespace that the HPA watches.
- The chart renders `redis.worker_patterns` into each service config (defaults include both global and regional patterns). HPA examples above use patterns like `egress:record:*` and `egress:web:*` — keep them consistent with your `worker_patterns`.

### 4. Monitoring Scaling Activities

```bash
# Real-time scaling status
watch kubectl get hpa,pods -n rtc-egress-prod

# Scaling history
kubectl get events --field-selector involvedObject.kind=HorizontalPodAutoscaler -n rtc-egress-prod

# Pod resource usage
kubectl top pods -n rtc-egress-prod
```

## ⚠️ Best Practices

1. **Start Conservative**: Begin with smaller max replicas and adjust based on load
2. **Monitor Resources**: Ensure cluster has sufficient CPU/memory for scaling
3. **Test Scaling**: Use load testing to validate autoscaling behavior
4. **Set Resource Requests**: Required for CPU/memory-based scaling to work
5. **Consider Costs**: More pods = higher costs

## 🔧 Troubleshooting

### HPA Not Scaling
```bash
# Check metrics availability
kubectl top pods -n rtc-egress-prod

# Verify resource requests are set
kubectl describe deployment rtc-egress-egress -n rtc-egress-prod

# Check HPA conditions
kubectl describe hpa rtc-egress-egress -n rtc-egress-prod
```

### Common Issues
- **No metrics**: Ensure metrics-server is running
- **Resource requests missing**: Set CPU/memory requests in deployment
- **Redis scaling not working**: Verify Prometheus and Redis Exporter setup

## 📈 Scaling Scenarios

### High Load Scenario
```
Initial: 1 api + 3 egress + 2 recorder + 1 uploader + 2 notifier = 9 pods
High Load: 3 api + 15 egress + 6 recorder + 4 uploader + 8 notifier = 36 pods
```

### Cost-Optimized Scenario  
```
Low Traffic: 1 api + 3 egress + 2 recorder + 1 uploader + 2 notifier = 9 pods
Burst Capacity: Scale only when needed, return to baseline
```

This provides **elastic scaling** that adapts to your workload demands automatically! 🎉
