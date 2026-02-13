# Environment Strategy

## Cluster Model

This platform uses a single K3s cluster hosted on AWS EC2.

Environment separation is implemented using Kubernetes namespaces.

Namespaces:
- dev
- prod

---

## Key Differences Between Environments

### Replicas
- dev: 1 replica
- prod: 2+ replicas

### Resource Limits
- dev: minimal CPU/memory
- prod: production-grade resource requests and limits

### Domains
- dev: dev.example.com
- prod: app.example.com

### Alerts
- dev: warning-level alerts
- prod: critical alerts enabled

---

## Deployment Model

Each environment will be managed by Argo CD using separate Application definitions.

The Git repository will contain environment-specific overlays under:

- envs/dev/
- envs/prod/

This ensures clean separation of configuration while sharing the same base manifests.
