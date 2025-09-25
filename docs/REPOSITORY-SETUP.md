# Helm Repository Setup Guide

This guide explains how to set up `https://helm.agora.build` as a working Helm repository using GitHub Pages and GitHub Actions.

## Prerequisites

- GitHub repository with admin access
- Domain `agora.build` configured for GitHub Pages
- Charts in the `charts/` directory

## Setup Steps

### 1. Enable GitHub Pages

1. Go to your repository **Settings** → **Pages**
2. Set **Source** to "Deploy from a branch"
3. Select **Branch**: `gh-pages` (will be created automatically)
4. Set **Custom domain** to: `helm.agora.build`
5. Enable **Enforce HTTPS**

### 2. Configure DNS

Add a CNAME record for your domain:
```
helm.agora.build → your-github-username.github.io
```

### 3. GitHub Actions Setup

The repository includes two workflows:

- `.github/workflows/lint-test.yml`: Validates charts on pull requests and pushes
- `.github/workflows/release.yml`: Packages charts from `charts/*`, builds an index, and publishes to GitHub Pages (`gh-pages`) so `https://helm.agora.build` serves the repository index and packages.

### 4. Chart Versioning

For the release workflow to work properly:

1. **Bump chart version** in `charts/*/Chart.yaml` when making changes
2. **Push to main branch** to trigger the release workflow (it will package all charts under `charts/`)
3. Optionally create Git tags for app releases; the chart is versioned via `Chart.yaml`.

Example version bump:
```yaml
# charts/rtc-egress/Chart.yaml
apiVersion: v2
name: rtc-egress
version: 1.0.1  # <- Increment this
appVersion: "1.0.1"
```

### 5. First Release

1. Ensure chart version is set (e.g., `1.0.0`)
2. Commit and push to main branch:
   ```bash
   git add .
   git commit -m "Initial chart release"
   git push origin main
   ```
3. The GitHub Action will:
   - Package the chart
   - Create a GitHub release
   - Update the `gh-pages` branch with the Helm repository index

### 6. Verify Setup

After the first successful run:

```bash
# Add the repository
helm repo add agora https://helm.agora.build
helm repo update

# Search for charts and versions
helm search repo agora/rtc-egress --versions | head

# Install a chart
helm install my-egress agora/rtc-egress --set agora.appId=YOUR_APP_ID
```

## Troubleshooting

### Common Issues

1. **404 on helm.agora.build**: Check DNS configuration and GitHub Pages settings
2. **Workflow fails**: Ensure chart versions are incremented and valid
3. **Charts not appearing**: Check that `gh-pages` branch exists and contains `index.yaml`

### Manual Release

If needed, you can manually trigger a release:

```bash
# Install chart-releaser
go install github.com/helm/chart-releaser/cmd/cr@latest

# Package and upload
cr package charts/rtc-egress
cr upload --owner AgoraIO --git-repo Helm-Charts
cr index --owner AgoraIO --git-repo Helm-Charts --charts-repo https://helm.agora.build
```

### Checking Repository Status

Visit these URLs to verify:
- Repository index: `https://helm.agora.build/index.yaml`
- Chart package: `https://helm.agora.build/rtc-egress-1.0.0.tgz`

## Repository Structure

After setup, your `gh-pages` branch will contain:
```
├── index.yaml           # Helm repository index
├── rtc-egress-1.0.0.tgz # Chart packages
└── ...
```

## Maintenance

- **Update charts**: Increment version in Chart.yaml and push to main
- **Monitor workflows**: Check GitHub Actions tab for build status  
- **Review releases**: GitHub Releases page shows published chart versions
