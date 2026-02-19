# Production Environment (`prod`)

## Purpose

The `prod` environment is for live, user-facing workloads.
It is isolated in the `prod` namespace and reconciled through GitOps.

## Characteristics

- Namespace: `prod`
- Higher stability and reliability expectations than `dev`
- Production-grade resource planning and scaling posture
- HTTPS enforced with cert-manager + Let's Encrypt
- Stricter monitoring and incident response expectations

## Deployment Model

Production changes must be committed to Git and applied by Argo CD.
Environment strategy and planned overlay model are documented in
`docs/environment-strategy.md`.

## Operational Checks

```bash
kubectl get pods -n prod
kubectl get ingress -n prod
kubectl get events -n prod --sort-by=.lastTimestamp
```
