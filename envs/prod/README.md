# Production Environment

## Purpose

The `prod` environment represents the live, user-facing system.

It runs inside the same K3s cluster but is isolated using a dedicated Kubernetes namespace.

---

## Characteristics

- Multiple replicas for availability
- Production-grade resource requests and limits
- Strict alerting configuration
- Public domain (e.g., app.example.com)
- HTTPS enforced via cert-manager and Let’s Encrypt

---

## Deployment Model

This environment is managed by Argo CD using a dedicated Application definition.

Configuration overrides (replicas, resources, domains, scaling) are applied through environment-specific overlays.

---

## Goal

The production environment prioritizes stability, reliability, and observability.
