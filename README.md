# Agora Helm Charts

[![Release Charts](https://github.com/AgoraIO/helm-charts/actions/workflows/release.yml/badge.svg)](https://github.com/AgoraIO/helm-charts/actions/workflows/release.yml)
[![Lint and Test Charts](https://github.com/AgoraIO/helm-charts/actions/workflows/lint-test.yml/badge.svg)](https://github.com/AgoraIO/helm-charts/actions/workflows/lint-test.yml)

This repository contains Helm charts for self-hosted Agora services.

## Usage

Add the Agora helm repository:

```bash
helm repo add agora https://helm.agora.build
helm repo update
```

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| [rtc-egress](./charts/rtc-egress) | Real-time video recording, snapshot service and more | 1.0.0 |

## Installing Charts

### RTC Egress

```bash
# Create the secret for private registry
kubectl create secret docker-registry ghcr-secret \
    --docker-server=ghcr.io \
    --docker-username=YOUR_GITHUB_USERNAME \
    --docker-password=YOUR_GITHUB_TOKEN \
    --namespace myrtc-egress
```

With custom values:

```bash
helm install my-rtc-egress agora/rtc-egress -f my-values.yaml
```

Full values.yaml:

```bash
helm install my-rtc-egress agora/rtc-egress \
    --namespace myrtc-egress \
    --set agora.appId=$AGORA_APP_ID \
    --set redis.external.host=$REDIS_HOST \
    --set redis.external.port=$REDIS_PORT \
    --set image.tag=$IMAGE_TAG \
    --set 'global.imagePullSecrets[0].name=ghcr-secret'
```

```bash
helm list -n myrtc-egress
```

```bash
kubectl get pods -n myrtc-egress
```

```bash
kubectl logs -n myrtc-egress my-rtc-egress-69b7b86d5b-sfzcq
```

If you want to upgrade the chart:
```bash
helm upgrade my-rtc-egress ./charts/rtc-egress \
    --namespace myrtc-egress \
    --set agora.appId=$AGORA_APP_ID \
    --set redis.external.host=$REDIS_HOST \
    --set redis.external.port=$REDIS_PORT \
    --set image.tag=$IMAGE_TAG \
    --set 'global.imagePullSecrets[0].name=ghcr-secret'
```

If you want to uninstall the chart:
```bash
helm uninstall my-rtc-egress -n myrtc-egress
```

## Development(Chart Maintainer)

### Prerequisites

- [Helm](https://helm.sh/docs/intro/install/) v3.0+
- [chart-testing](https://github.com/helm/chart-testing) (for testing)

### Testing Charts Locally

```bash
# Lint charts
ct lint --all

# Install charts in kind cluster
ct install --all
```

### Adding New Charts

1. Create a new directory under `charts/`
2. Follow the [Helm chart best practices](https://helm.sh/docs/chart_best_practices/)
3. Ensure your chart passes linting and testing
4. Submit a pull request

## Repository Setup(Repository Admin)

To enable `https://helm.agora.build`, see the [REPOSITORY-SETUP.md](REPOSITORY-SETUP.md) guide for configuring GitHub Pages and Actions.

## Contributing

Please read our contributing guidelines before submitting pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.