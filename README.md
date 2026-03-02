# k8s-multitenant — Kubernetes Multi-Tenancy Helm Chart

[![Helm Lint](https://github.com/clouddrove/k8s-multitenant/actions/workflows/helm-lint.yml/badge.svg)](https://github.com/clouddrove/k8s-multitenant/actions/workflows/helm-lint.yml)
[![Helm Release](https://github.com/clouddrove/k8s-multitenant/actions/workflows/helm-release.yml/badge.svg)](https://github.com/clouddrove/k8s-multitenant/actions/workflows/helm-release.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Helm: v3](https://img.shields.io/badge/Helm-v3-blue?logo=helm)](https://helm.sh)
[![Kubernetes: 1.24+](https://img.shields.io/badge/Kubernetes-1.24%2B-326CE5?logo=kubernetes)](https://kubernetes.io)
[![GitHub Stars](https://img.shields.io/github/stars/clouddrove/k8s-multitenant?style=social)](https://github.com/clouddrove/k8s-multitenant)

> **Production-ready Helm chart for Kubernetes multi-tenancy** — automate namespace provisioning, RBAC, resource quotas, network policies, and LimitRanges for every tenant in a single declarative `values.yaml`.

---

## Why k8s-multitenant?

Manually managing dozens (or hundreds) of tenant namespaces is error-prone and doesn't scale. `k8s-multitenant` gives platform teams a **single Helm chart** to declaratively enforce:

- **Namespace isolation** per tenant
- **Fine-grained RBAC** — roles bound to team identities
- **ResourceQuota + LimitRange** — CPU/memory guardrails per tier
- **Network isolation** — default-deny ingress/egress policies
- **Consistent governance** — every tenant gets the same baseline, every time

---

## Features

| Feature | Description |
|---|---|
| Namespace automation | Create and annotate one or many namespaces per tenant |
| RBAC | ClusterRole / Role + RoleBindings scoped to tenant namespaces |
| Resource quotas | Per-tenant CPU, memory, and object-count limits |
| LimitRanges | Default container requests/limits injected at admission |
| Network policies | Tenant-to-tenant isolation with allow-lists |
| Labels & annotations | Standardized metadata for cost allocation and tooling |
| Multi-tier support | `small`, `medium`, `large`, `custom` resource profiles |
| GitOps-ready | Pure declarative — works with Argo CD, Flux, or plain `helm upgrade` |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Platform Team                         │
│              helm install / helm upgrade                │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│              k8s-multitenant Helm Chart                 │
│                                                         │
│  ┌─────────────┐  ┌──────────┐  ┌────────────────────┐ │
│  │  Namespace  │  │   RBAC   │  │  Resource Quotas   │ │
│  │  Template   │  │ Template │  │  + LimitRanges     │ │
│  └──────┬──────┘  └────┬─────┘  └─────────┬──────────┘ │
│         │              │                  │            │
│  ┌──────▼──────────────▼──────────────────▼──────────┐ │
│  │            Network Policy Templates               │ │
│  └────────────────────────────────────────────────────┘ │
└──────────────────────────┬──────────────────────────────┘
                           │ creates
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │  ns/team-a  │  │  ns/team-b  │  │  ns/team-c  │
   │  quota      │  │  quota      │  │  quota      │
   │  rbac       │  │  rbac       │  │  rbac       │
   │  netpol     │  │  netpol     │  │  netpol     │
   └─────────────┘  └─────────────┘  └─────────────┘
```

---

## Requirements

| Tool | Minimum Version |
|---|---|
| Kubernetes | 1.24+ |
| Helm | 3.10+ |
| kubectl | 1.24+ |

---

## Installation

The chart is distributed as an OCI artifact on the GitHub Container Registry. Helm 3.8+ supports OCI natively — no `helm repo add` required.

### Install with defaults

```bash
helm install my-tenants oci://ghcr.io/clouddrove/k8s-multitenant \
  --version 1.0.0 \
  --namespace platform-system \
  --create-namespace
```

### Install with a custom values file

```bash
helm install my-tenants oci://ghcr.io/clouddrove/k8s-multitenant \
  --version 1.0.0 \
  --namespace platform-system \
  --create-namespace \
  -f my-tenants-values.yaml
```

### Upgrade

```bash
helm upgrade my-tenants oci://ghcr.io/clouddrove/k8s-multitenant \
  --version 1.0.0 \
  -f my-tenants-values.yaml
```

### Pull default values

```bash
helm show values oci://ghcr.io/clouddrove/k8s-multitenant --version 1.0.0
```

---

## Quick Start

Create a `values.yaml` to onboard your first tenant:

```yaml
tools:
  create: true
  namespace: k8s-tools

tenants:
  - name: team-alpha
    labels:
      cost-center: "eng-platform"
      environment: "production"
    rbac:
      subjects:
        - kind: Group
          name: team-alpha-admins
          apiGroup: rbac.authorization.k8s.io

rbac:
  create: true
  serviceAccountName: default

resourceQuota:
  enabled: true
  default:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    pods: "20"
    services: "5"

networkPolicy:
  enabled: true
  vpcCidr: "10.0.0.0/8"       # set to your VPC CIDR
  allowInternetEgress: false   # set true if pods need external HTTP/HTTPS
```

Apply it:

```bash
helm install tenants k8s-multitenant/k8s-multitenant -f values.yaml -n platform-system --create-namespace
```

---

## Configuration Reference

### Global Settings

| Parameter | Description | Default |
|---|---|---|
| `global.labels` | Labels applied to all managed resources | `{}` |
| `global.annotations` | Annotations applied to all managed resources | `{}` |

### Tools Namespace

| Parameter | Description | Default |
|---|---|---|
| `tools.create` | Create a shared tools namespace | `true` |
| `tools.namespace` | Name of the tools namespace | `"k8s-tools"` |
| `tools.labels` | Additional labels | `{}` |
| `tools.annotations` | Additional annotations | `{}` |

### Tenant Settings

| Parameter | Description | Default |
|---|---|---|
| `tenants[].name` | Tenant identifier — becomes the namespace name | **required** |
| `tenants[].labels` | Additional labels on the namespace | `{}` |
| `tenants[].annotations` | Additional annotations on the namespace | `{}` |
| `tenants[].resourceQuota` | Per-tenant quota spec (overrides `resourceQuota.default`) | `{}` |
| `tenants[].limitRange` | Per-tenant LimitRange spec (overrides `limitRange.default`) | `{}` |

### RBAC

| Parameter | Description | Default |
|---|---|---|
| `rbac.create` | Create Role + RoleBinding for each tenant namespace | `true` |
| `rbac.serviceAccountName` | ServiceAccount to bind to the tenant role | `"default"` |
| `rbac.defaultRules` | RBAC rules granted in each tenant namespace | see `values.yaml` |

### Resource Quota

| Parameter | Description | Default |
|---|---|---|
| `resourceQuota.enabled` | Create a ResourceQuota in each tenant namespace | `true` |
| `resourceQuota.default` | Default hard quota applied to all tenants | see `values.yaml` |

### LimitRange

| Parameter | Description | Default |
|---|---|---|
| `limitRange.enabled` | Create a LimitRange in each tenant namespace | `true` |
| `limitRange.default` | Default container limits applied to all tenants | see `values.yaml` |

### Network Policy

| Parameter | Description | Default |
|---|---|---|
| `networkPolicy.enabled` | Global default; create NetworkPolicy in every tenant namespace | `true` |
| `networkPolicy.vpcCidr` | VPC CIDR used for ALB ingress + internal service egress rules | `"10.0.0.0/8"` |
| `networkPolicy.allowInternetEgress` | Allow outbound HTTPS/HTTP to the public internet | `false` |
| `tenants[].networkPolicy.enabled` | Per-tenant override for `networkPolicy.enabled` | inherits global |

---

## Examples

See the [`examples/`](examples/) directory for ready-to-use configurations:

| Example | Description |
|---|---|
| [`basic-tenant/`](examples/basic-tenant/) | Single tenant with default settings |
| [`advanced-tenant/`](examples/advanced-tenant/) | Multi-team setup with custom quotas and network policies |
| [`production-tenant/`](examples/production-tenant/) | Production-grade config with strict isolation and cost labels |

---

## GitOps Integration

### Argo CD

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform-tenants
  namespace: argocd
spec:
  project: default
  source:
    repoURL: ghcr.io/clouddrove
    chart: k8s-multitenant
    targetRevision: 1.0.0
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: platform-system
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Flux

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: k8s-multitenant
  namespace: flux-system
spec:
  interval: 1h
  type: oci
  url: oci://ghcr.io/clouddrove
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: platform-tenants
  namespace: platform-system
spec:
  chart:
    spec:
      chart: k8s-multitenant
      version: ">=1.0.0"
      sourceRef:
        kind: HelmRepository
        name: k8s-multitenant
        namespace: flux-system
  values: {}
```

---

## Security Considerations

- All namespaces are created with a **default-deny network policy** unless explicitly overridden.
- RBAC bindings are **namespace-scoped** — no cluster-wide privilege escalation.
- LimitRanges enforce **default CPU/memory limits** so unbounded containers can't consume all cluster resources.
- Resource quotas prevent **noisy-neighbour** scenarios.
- See [SECURITY.md](SECURITY.md) for vulnerability reporting.

---

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

```bash
# Clone the repo
git clone https://github.com/clouddrove/k8s-multitenant.git
cd k8s-multitenant

# Lint the chart
helm lint charts/k8s-multitenant

# Run template tests
helm template test-release charts/k8s-multitenant -f examples/basic-tenant/values.yaml
```

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Related Projects

- [kiosk](https://github.com/loft-sh/kiosk) — multi-tenancy extension for Kubernetes
- [HNC (Hierarchical Namespace Controller)](https://github.com/kubernetes-sigs/hierarchical-namespaces) — namespace hierarchy for Kubernetes
- [Capsule](https://github.com/projectcapsule/capsule) — Kubernetes multi-tenancy operator
- [vcluster](https://github.com/loft-sh/vcluster) — virtual clusters for stronger isolation

---

<p align="center">
  Made with ❤️ by <a href="https://github.com/clouddrove">CloudDrove</a>
</p>
