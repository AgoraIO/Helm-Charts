# Deployment Modes Guide

The Agora RTC Egress Helm chart supports **two deployment architectures**:

## 🏗️ Architecture Comparison

| Feature | Separated Architecture | Monolithic Architecture |
|---------|----------------------|------------------------|
| **Containers** | 5 separate services | 1 unified container |
| **Scaling** | Independent per service | All services together |
| **Resource Usage** | Optimized per service | Fixed allocation |
| **Complexity** | Higher (production) | Lower (simple) |
| **Fault Tolerance** | High (isolated failures) | Lower (single point) |
| **Recommended For** | Production | Development/Testing |

## 🚀 Separated Architecture (Recommended)

### Overview
Runs 5 independent services, each with its own container, scaling, and resources:

- **API Server** (1-3 pods) - HTTP API that publishes tasks to Redis
- **Egress** (3-15 pods) - Native C++ recording/snapshot workers
- **Flexible Recorder** (2-6 pods) - Web recording engine
- **Uploader** (1-4 pods) - File upload to S3
- **Webhook Notifier** (2-8 pods) - Event notifications

### Deployment Commands

```bash
# Production deployment
helm install rtc-egress agora/rtc-egress \
  --namespace rtc-egress-prod \
  --values values-production-separated.yaml \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST \
  --set s3.bucket=YOUR_S3_BUCKET

# Development deployment  
helm install rtc-egress agora/rtc-egress \
  --namespace rtc-egress-dev \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST

# With custom instance counts
helm install rtc-egress agora/rtc-egress \
  --set apiServer.replicaCount=2 \
  --set egress.replicaCount=5 \
  --set flexibleRecorder.replicaCount=3 \
  --set uploader.replicaCount=2 \
  --set webhookNotifier.replicaCount=2
```

### Benefits
- ✅ **Independent scaling** - Scale each service based on load
- ✅ **Resource optimization** - Right-size CPU/memory per service
- ✅ **Fault isolation** - Service failures don't affect others
- ✅ **Flexible deployment** - Enable/disable services as needed
- ✅ **Production ready** - Battle-tested architecture

### Monitoring
```bash
# Check all service pods
kubectl get pods -n rtc-egress-prod -l app.kubernetes.io/name=rtc-egress

# Monitor autoscaling
kubectl get hpa -n rtc-egress-prod

# View service-specific logs
kubectl logs -n rtc-egress-prod -l app.kubernetes.io/component=api-server
kubectl logs -n rtc-egress-prod -l app.kubernetes.io/component=egress
```

## 🏢 Monolithic Architecture (Legacy)

### Overview
Runs all services in a single container - simpler but less flexible:

- **Single Container** with all 5 services embedded
- **Unified scaling** - All services scale together
- **Simpler operations** - One deployment to manage

### Deployment Commands

```bash
# Monolithic deployment
helm install rtc-egress-mono agora/rtc-egress \
  --namespace rtc-egress-mono \
  --values values-monolithic.yaml \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=YOUR_REDIS_HOST

# With custom scaling
helm install rtc-egress-mono agora/rtc-egress \
  --values values-monolithic.yaml \
  --set replicaCount=5 \
  --set autoscaling.maxReplicas=20

# Simple development setup
helm install rtc-egress agora/rtc-egress \
  --set architecture=monolithic \
  --set agora.appId=YOUR_AGORA_APP_ID \
  --set redis.external.host=localhost:6379
```

### Benefits
- ✅ **Simple deployment** - One container to manage
- ✅ **Lower complexity** - Fewer moving parts
- ✅ **Resource efficiency** - No inter-service overhead
- ✅ **Development friendly** - Quick setup and testing
- ✅ **Backward compatible** - Works with existing configs

### Monitoring
```bash
# Check monolithic pods
kubectl get pods -n rtc-egress-mono -l app.kubernetes.io/name=rtc-egress

# View combined logs
kubectl logs -n rtc-egress-mono deployment/rtc-egress-mono

# Monitor unified scaling
kubectl get hpa rtc-egress-mono -n rtc-egress-mono
```

## ⚡ Switching Between Architectures

### From Monolithic to Separated
```bash
# 1. Deploy separated architecture
helm install rtc-egress-new agora/rtc-egress \
  --values values-production-separated.yaml

# 2. Test the new deployment
# 3. Switch traffic gradually
# 4. Remove monolithic deployment
helm uninstall rtc-egress-mono
```

### From Separated to Monolithic  
```bash
# 1. Deploy monolithic version
helm install rtc-egress-simple agora/rtc-egress \
  --values values-monolithic.yaml

# 2. Switch traffic
# 3. Remove separated services
helm uninstall rtc-egress
```

## 🎯 When to Use Each Architecture

### Use **Separated Architecture** when:
- **Production environment** with high availability needs
- **Different scaling patterns** for each service
- **Resource optimization** is important
- **Team has Kubernetes expertise**
- **Monitoring and observability** are priorities

### Use **Monolithic Architecture** when:
- **Development/testing** environments
- **Simpler operations** are preferred
- **Getting started** with the platform
- **Resource constraints** (fewer pods)
- **Backward compatibility** is needed

## 📊 Resource Usage Comparison

### Separated Architecture
```
Production: ~36 pods max (1+15+6+4+8+2) = High resource usage, optimal performance
Development: ~9 pods min (1+3+2+1+2) = Moderate resource usage
```

### Monolithic Architecture  
```
Production: ~10 pods max (10+0+0+0+0) = Lower resource usage, unified scaling
Development: ~3 pods min (3+0+0+0+0) = Minimal resource usage
```

Choose the architecture that best fits your operational requirements! 🎉