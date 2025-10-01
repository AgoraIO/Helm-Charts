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

## Who is this for?

- If you want to deploy and use the chart → see Chart User Docs below.
- If you want to modify, package, and publish the chart → see Chart Developer Docs below.

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| [rtc-egress](./charts/rtc-egress) | Real-time egress (recording, snapshots, web recording, upload, webhooks) | see Chart.yaml |

## Chart User Docs (Install & Operate)

### RTC Egress

```bash
# Create the secret for private registry (if needed)
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
    --set-string agora.appId="$AGORA_APP_ID" \
    --set-string redis.external.host="$REDIS_HOST" \
    --set-string redis.external.port="$REDIS_PORT" \
    --set s3.enabled=true \
    --set-string s3.bucket="$S3_BUCKET" \
    --set-string s3.region="$S3_REGION" \
    --set-string s3.accessKey="$S3_ACCESS_KEY" \
    --set-string s3.secretKey="$S3_SECRET_KEY" \
    --set-string s3.endpoint="$S3_ENDPOINT" \
    --set-string webhookNotifier.webhook.url="$WEBHOOK_URL" \
    --set-string image.tag="$IMAGE_TAG"
```

please add secret when install from private registry
--set 'global.imagePullSecrets[0].name=ghcr-secret'


```bash
helm list -n myrtc-egress
```

```bash
kubectl get pods -n myrtc-egress
```

```bash
kubectl -n myrtc-egress get svc
kubectl describe pod -n myrtc-egress my-rtc-egress-69b7b86d5b-sfzcq
kubectl logs -n myrtc-egress my-rtc-egress-69b7b86d5b-sfzcq
```

If you want to upgrade the chart:
```bash
helm upgrade --install my-rtc-egress agora/rtc-egress \
    --reuse-values \
    --namespace myrtc-egress \
    --set-string agora.appId="$AGORA_APP_ID" \
    --set-string redis.external.host="$REDIS_HOST" \
    --set-string redis.external.port="$REDIS_PORT" \
    --set s3.enabled=true \
    --set-string s3.bucket="$S3_BUCKET" \
    --set-string s3.region="$S3_REGION" \
    --set-string s3.accessKey="$S3_ACCESS_KEY" \
    --set-string s3.secretKey="$S3_SECRET_KEY" \
    --set-string s3.endpoint="$S3_ENDPOINT" \
    --set-string webhookNotifier.webhook.url="$WEBHOOK_URL" \
    --set-string image.tag="$IMAGE_TAG"

See `charts/rtc-egress/README.md` for mandatory parameters, configuration files, and architecture details.
```

If you want to uninstall the chart:
```bash
helm uninstall my-rtc-egress -n myrtc-egress
```

## Chart Developer Docs (Build & Publish)

See the [DEVELOPERS.md](docs/DEVELOPERS.md) for:
- Local development and linting
- Versioning and release to GitHub Pages (https://helm.agora.build)
- How images are tagged and consumed by the chart
- Security/secrets guidance

## Repository Setup(Repository Admin)

To enable `https://helm.agora.build`, see the [REPOSITORY-SETUP.md](docs/REPOSITORY-SETUP.md) guide for configuring GitHub Pages and Actions.

## Contributing

Please read our contributing guidelines before submitting pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
