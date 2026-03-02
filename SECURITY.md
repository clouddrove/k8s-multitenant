# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| 1.x | Yes |

## Reporting a Vulnerability

**Please do not open a public GitHub issue for security vulnerabilities.**

Email **security@clouddrove.com** with:

- A description of the vulnerability
- Steps to reproduce
- Potential impact
- Any suggested mitigations

We aim to acknowledge reports within 48 hours and provide a fix or mitigation plan within 7 days for critical issues.

## Security Considerations

- This chart creates RBAC bindings using the subjects you provide. Always audit `values.yaml` before deploying to production.
- NetworkPolicy enforcement requires a CNI plugin that supports it (e.g., Calico, Cilium, Weave). Without a supported CNI, policies are silently unenforced.
- ResourceQuota and LimitRange protect against resource exhaustion but are not a substitute for proper cluster-level autoscaling policies.
