# Development Environment (`dev`)

## Purpose

The `dev` environment is used for testing and validation before production changes.
It runs in the same K3s cluster as prod but is isolated by namespace.

## Characteristics

- Namespace: `dev`
- Faster iteration with lower blast radius
- Lower resource profile than production
- HTTPS ingress and certificate flow can be validated safely

## Deployment Model

`dev` is GitOps-managed by Argo CD.
Environment separation strategy is documented in `docs/environment-strategy.md`.

## Operational Checks

```bash
kubectl get pods -n dev
kubectl get ingress -n dev
kubectl get certificate -n dev
```
