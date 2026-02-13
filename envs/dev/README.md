# Development Environment

## Purpose

The `dev` environment is used for testing, experimentation, and validation of new changes before promoting them to production.

It runs inside the same K3s cluster but is isolated using a dedicated Kubernetes namespace.

---

## Characteristics

- Single replica workloads
- Lower CPU and memory requests
- Reduced alert severity
- Development domain (e.g., dev.example.com)

---

## Deployment Model

This environment will be managed by Argo CD using a separate Application definition.

Environment-specific configuration (replicas, resources, domain) will override the base manifests.

---

## Goal

The development environment allows safe iteration while maintaining a structure similar to production.
