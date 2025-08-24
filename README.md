# AgoraIO Helm Charts

[![Release Charts](https://github.com/AgoraIO/helm-charts/actions/workflows/release.yml/badge.svg)](https://github.com/AgoraIO/helm-charts/actions/workflows/release.yml)
[![Lint and Test Charts](https://github.com/AgoraIO/helm-charts/actions/workflows/lint-test.yml/badge.svg)](https://github.com/AgoraIO/helm-charts/actions/workflows/lint-test.yml)

This repository contains Helm charts for AgoraIO services.

## Usage

Add the AgoraIO helm repository:

```bash
helm repo add agora https://helm.agora.build
helm repo update
```

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| [rtc-egress](./charts/rtc-egress) | Real-time video recording and snapshot service | 1.0.0 |

## Installing Charts

### RTC Egress

```bash
helm install egress agora/rtc-egress
```

With custom values:

```bash
helm install egress agora/rtc-egress -f my-values.yaml
```

## Development

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

## Contributing

Please read our contributing guidelines before submitting pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.