# Contributing to k8s-multitenant

Thank you for taking the time to contribute! This document explains how to get started.

## Development Setup

```bash
git clone https://github.com/clouddrove/k8s-multitenant.git
cd k8s-multitenant
```

### Prerequisites

- Helm v3.10+
- kubectl
- [chart-testing (ct)](https://github.com/helm/chart-testing) — for lint checks
- A local Kubernetes cluster ([kind](https://kind.sigs.k8s.io/) or [k3d](https://k3d.io/) recommended)

## Making Changes

### Helm Chart Changes

1. Edit files under `charts/k8s-multitenant/`
2. Bump `version` in `charts/k8s-multitenant/Chart.yaml` following [SemVer](https://semver.org/)
3. Update `CHANGELOG.md` with a description of the change
4. Add or update example values in `examples/` if relevant

### Linting

```bash
helm lint charts/k8s-multitenant --strict
```

### Template Testing

```bash
helm template test charts/k8s-multitenant -f examples/basic-tenant/values.yaml
helm template test charts/k8s-multitenant -f examples/advanced-tenant/values.yaml
```

### Integration Testing (kind)

```bash
kind create cluster --name k8s-multitenant-test

helm install test-release charts/k8s-multitenant \
  -f examples/basic-tenant/values.yaml \
  --namespace platform-system \
  --create-namespace

kubectl get namespaces
kubectl get resourcequota -A
kubectl get networkpolicy -A

kind delete cluster --name k8s-multitenant-test
```

## Pull Request Guidelines

- Keep PRs focused — one feature or fix per PR
- Add an entry to `CHANGELOG.md` under `## [Unreleased]`
- Ensure `helm lint` passes
- Ensure template rendering tests pass for all example files
- Update documentation in `docs/` if you add or change any values

## Reporting Bugs

Use the [bug report template](.github/ISSUE_TEMPLATE/bug_report.md).

## Requesting Features

Use the [feature request template](.github/ISSUE_TEMPLATE/feature_request.md).

## Code of Conduct

Be respectful and constructive. We follow the [Contributor Covenant](https://www.contributor-covenant.org/).
