# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Helm chart repository for MailDev, an SMTP server and web interface for development and testing. The repository contains a single chart (`charts/maildev/`) that packages the MailDev application for Kubernetes deployment.

### Key Files and Structure

- **`charts/maildev/Chart.yaml`** — Chart metadata including version and appVersion
- **`charts/maildev/values.yaml`** — Default configuration values for the chart
- **`charts/maildev/templates/`** — Kubernetes manifests:
  - `deployment.yaml` — Main MailDev deployment with extensive configuration
  - `service-smtp.yaml`, `service-web.yaml` — Services for SMTP (port 1025) and web UI (port 1080)
  - `cm-auto-relay-rules.yaml` — ConfigMap for SMTP auto-relay rules
  - `serviceaccount.yaml` — ServiceAccount for the deployment
  - `_helpers.tpl` — Shared template definitions
- **`ct.yaml`** — chart-testing configuration (target branch: main, chart directory: charts)
- **`.github/workflows/`** — CI/CD pipelines:
  - `lint-test.yaml` — Runs on PRs; lints and tests changed charts
  - `release.yaml` — Publishes charts to GitHub Pages on merges to main

## Development Commands

### Linting and Testing Charts

**Lint all charts:**
```bash
ct lint --config ct.yaml
```

**List changed charts (since the target branch):**
```bash
ct list-changed --config ct.yaml
```

**Install and test charts on a local Kubernetes cluster:**
```bash
# Requires a running Kubernetes cluster (e.g., kind, minikube, Docker Desktop)
ct install --config ct.yaml
```

### Prerequisites for Local Testing

- **Helm** (v3.16.2+ as used in CI)
- **chart-testing** (Python-based tool)
- **Kubernetes cluster** (for `ct install` tests)

Install chart-testing: https://github.com/helm/chart-testing

## Chart Configuration Notes

The MailDev chart models the following major configurations:

1. **Image** — Repository, tag, and pull policy (defaults to maildev/maildev at Chart.appVersion)
2. **Ports** — SMTP (1025) and web UI (1080) ports are configurable
3. **Outgoing Relay** — Can relay emails to an external SMTP server with auto-relay rules
4. **Web Interface** — Can be disabled; supports basic auth via user/pass
5. **HTTPS** — Can be enabled with custom key and cert
6. **Escape Hatches** — `extraArgs` (for unmapped MailDev CLI flags) and `extraEnv` (for custom environment variables)

The deployment uses liveness and readiness probes against `/healthz` on the web port.

## Release Process

Chart releases are automated:
1. Changes are merged to `main`
2. The release workflow runs chart-releaser-action, which:
   - Detects version bumps in Chart.yaml
   - Packages and indexes the chart
   - Publishes to GitHub Pages

To release a new version, bump `version:` in `charts/maildev/Chart.yaml`.
