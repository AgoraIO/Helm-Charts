# Developers Guide (Chart Maintainers)

This guide is for developers maintaining the rtc‑egress Helm chart and related release workflows.

## Overview

- Chart path: `charts/rtc-egress`
- Chart versioning: `charts/rtc-egress/Chart.yaml` (`version`, `appVersion`)
- Published Helm repo: `https://helm.agora.build`
- Images are built in the RTC‑Egress source repository and pushed to GHCR: `ghcr.io/agoraio/rtc-egress/<service>:<tag>`

## Local Development

Prerequisites:
- Helm v3+
- chart-testing (optional for CI‑like checks)

Common commands:
```bash
# Lint chart
helm lint charts/rtc-egress

# Render templates with current defaults
helm template test charts/rtc-egress > out.yaml

# Dry-run an install
helm install my-rtc-egress charts/rtc-egress --dry-run --debug \
  --set agora.appId=TEST --set redis.external.host=redis.example --set redis.external.port=6379
```

Key design points to keep in sync with the code:
- Each service reads a YAML config file. The chart mounts a ConfigMap at `/opt/rtc_egress/config` and sets `CONFIG_FILE` to the service‑specific file:
  - `api_server_config.yaml`
  - `egress_config.yaml`
  - `flexible_recorder_config.yaml`
  - `uploader_config.yaml`
  - `webhook_notifier_config.yaml`
- Config resolution (in binaries): `CONFIG_FILE` > `--config` > `CONFIG_DIR/<file>` > search `./config`, `/opt/rtc_egress/config`, `/etc/rtc_egress`. The process logs the path used and fails fast if none found.
- Health ports are value‑driven:
  - API 8191, Egress 8192, Flexible 8193, Uploader 8194, Notifier 8195
- Queue subscription patterns are rendered from values into config:
  - Defaults include both global and regional patterns.

## Mandatory Values (for users)

Reference: see the "Mandatory Parameters" section in `charts/rtc-egress/README.md`.

## Versioning & Release

1) Bump chart version and appVersion in `charts/rtc-egress/Chart.yaml`.
2) Push to `main` — the repo’s Pages workflow packages charts and publishes the Helm repo index.
3) Verify:
```bash
helm repo update
helm search repo agora/rtc-egress --versions | head
helm show chart agora/rtc-egress --version <new-version>
```

## Images & Tags

- Images are built in the RTC‑Egress code repo (not here). The chart supports:
  - Per‑service tags (`{service}.image.tag`)
  - Global fallback `image.tag` applied to all services not explicitly tagged
- For releases, prefer immutable tags (semver or date+sha) and avoid `latest` in production

## Testing

- Use `helm template` and `kubectl apply -f -` in a dev cluster to validate manifests
- Check ConfigMap content and mounts:
```bash
kubectl get cm my-rtc-egress-config -n <ns> -o yaml | sed -n '1,200p'
kubectl get deploy -n <ns> -o yaml | rg -n 'CONFIG_FILE|mountPath: /opt/rtc_egress/config'
```

## Security & Secrets

- Do not commit secrets. Prefer Kubernetes Secrets or CI‑injected values
- S3 credentials must be provided by the user environment (values or Secrets)

## Troubleshooting

- Pods crashloop at start: check `CONFIG_FILE` and the mounted file content
- Health probe failures: confirm containerPort and health ports match values
- Uploader fails: ensure S3 values are set or disable uploader in dev

## Repository Admins

For setting up GitHub Pages and Helm repo publishing, see `REPOSITORY-SETUP.md`.
