# Platform Standards

This document defines baseline requirements for workloads deployed on this Kubernetes platform.

These standards ensure reliability, security, and operational consistency.

---

## 1. Container Standards

All containers must:

- Expose `/healthz` endpoint (liveness)
- Expose `/readyz` endpoint (readiness)
- Emit structured JSON logs
- Run as non-root user
- Handle graceful shutdown (SIGTERM)

Containers must not:

- Hardcode secrets
- Depend on localhost-only assumptions

---

## 2. Kubernetes Workload Requirements

All Deployments must define:

- Resource requests and limits (CPU and memory)
- Liveness probe
- Readiness probe
- Explicit ServiceAccount
- Namespace (no default namespace usage)

Rolling updates must be configured to avoid downtime.

---

## 3. Secrets Management Policy

- No plaintext secrets stored in Git
- Non-sensitive configuration stored in ConfigMaps
- Sensitive values sourced from external secret provider (planned: AWS SSM / Secrets Manager)
- Secrets injected via environment variables or mounted volumes

---

## 4. Namespace Strategy

Namespaces must be used for environment isolation:

- dev
- prod

No workloads should run in the default namespace.

---

## 5. Observability Requirements

All applications must:

- Expose Prometheus-compatible metrics
- Emit structured logs
- Be observable via dashboards in Grafana

Alerts must be defined for:

- Pod crash loops
- High error rates
- Resource saturation
