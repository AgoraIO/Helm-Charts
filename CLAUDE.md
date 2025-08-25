# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains Helm charts for self-hosted Agora services, specifically the RTC Egress Server which provides real-time video recording and snapshot capabilities. The main chart is located at `charts/rtc-egress/` and is designed for Kubernetes deployment.

## Repository Structure

```
charts/
└── rtc-egress/           # Agora RTC Egress Server Helm chart
    ├── Chart.yaml        # Chart metadata and dependencies
    ├── values.yaml       # Default configuration values
    ├── values-production.yaml  # Production-specific overrides
    ├── templates/        # Kubernetes resource templates
    │   ├── _helpers.tpl  # Template helper functions
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   ├── configmap.yaml
    │   ├── pvc.yaml      # Persistent volume claims
    │   ├── serviceaccount.yaml
    │   ├── ingress.yaml
    │   └── hpa.yaml      # Horizontal Pod Autoscaler
    └── README.md         # Chart-specific documentation
```

## Common Development Tasks

### Chart Testing and Validation

```bash
# Lint all charts
ct lint --all

# Install charts in kind cluster for testing
ct install --all

# Helm template rendering (dry run)
helm template my-release ./charts/rtc-egress --values ./charts/rtc-egress/values.yaml

# Validate chart against Kubernetes cluster
helm lint ./charts/rtc-egress
```

### Chart Installation Examples

```bash
# Add Agora helm repository
helm repo add agora https://helm.agora.build
helm repo update

# Basic installation
helm install egress agora/rtc-egress --set agora.appId=YOUR_APP_ID

# Production installation with custom values
helm install egress agora/rtc-egress -f values-production.yaml
```

## Chart Architecture

### Core Components

- **RTC Egress Server**: Main application container (`ag_egress:latest`)
- **Persistent Storage**: Three volumes for recordings, snapshots, and logs
- **Redis Integration**: External Redis required for session management
- **Multi-port Service**: API (8080), Health (8182), Canvas Template (3000)

### Key Configuration Areas

1. **Agora Settings**: App ID, access tokens, channel configuration
2. **Redis Configuration**: External Redis connection parameters
3. **Persistence**: Configurable storage for recordings, snapshots, logs
4. **Resources**: CPU/memory limits and requests
5. **Autoscaling**: HPA configuration for production workloads
6. **Security**: Pod security contexts, service accounts

### Template Helpers

The `_helpers.tpl` file contains standard Helm helper templates for:
- Resource naming (`rtc-egress.name`, `rtc-egress.fullname`)
- Label generation (`rtc-egress.labels`, `rtc-egress.selectorLabels`)
- Service account naming (`rtc-egress.serviceAccountName`)

## Prerequisites

- Helm 3.2.0+
- Kubernetes 1.19+
- chart-testing tool for validation
- External Redis server for production deployments
- Valid Agora App ID and credentials

## Configuration Patterns

### Production Values
Use `values-production.yaml` for production-specific overrides including:
- Resource limits and requests
- External Redis configuration
- Persistence settings
- Security contexts

### Development Setup
For local development, the chart can be configured to use internal Redis and reduced resource requirements.