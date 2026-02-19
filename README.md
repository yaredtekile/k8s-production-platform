# Kubernetes Production Platform (AWS + K3s + GitOps)

Production-style Kubernetes platform running on AWS EC2 with K3s, fully managed through GitOps with Argo CD.

## Quick Navigation

- Architecture: `docs/architecture.md`
- Platform standards: `docs/platform-standards.md`
- Environment strategy: `docs/environment-strategy.md`
- Ingress/TLS notes: `docs/ingress-and-tls.md`
- Incident report: `docs/ingress-controller-conflict-and-recovery.md`
- Infra provisioning guide: `infra/terraform/aws-k3s/README.md`
- Dev environment guide: `envs/dev/README.md`
- Prod environment guide: `envs/prod/README.md`

## Why This Project

This repository demonstrates how to build and operate a realistic DevOps/SRE platform with:

- declarative infrastructure (Terraform)
- declarative platform services (Argo CD App-of-Apps)
- ingress and TLS automation (ingress-nginx + cert-manager + Let's Encrypt)
- observability (Prometheus, Alertmanager, Grafana)
- centralized logging (Loki + Promtail)

## Architecture

![Platform architecture](docs/architecture.png)

Architecture details: `docs/architecture.md`

## Portfolio Highlights

- Real GitOps reconciliation model with automated sync, prune, and self-heal.
- Multi-app platform composition using Argo CD and Helm chart sources.
- TLS debugging and incident recovery documented in detail:
  `docs/ingress-controller-conflict-and-recovery.md`
- Monitoring and logging stack integrated end-to-end.

## Tech Stack

- Terraform (`infra/terraform/aws-k3s`)
- AWS EC2 (single-node host)
- K3s Kubernetes
- Argo CD (App-of-Apps pattern)
- ingress-nginx Helm chart `4.14.3`
- kube-prometheus-stack Helm chart `66.3.0`
- Loki Helm chart `6.25.0`
- Promtail Helm chart `6.16.6`
- cert-manager ClusterIssuers (staging + prod)

## Argo CD Managed Apps

- `argocd-config`
- `argocd-ingress`
- `cert-manager-config`
- `echo`
- `ingress-nginx`
- `kube-prometheus-stack`
- `loki`
- `monitoring-ingress`
- `promtail`

## Repository Structure

```text
.
├── apps/                  # Workload manifests (example app)
├── docs/                  # Architecture, standards, incident write-ups, screenshots
├── envs/                  # Environment-specific documentation/overlays
├── infra/terraform/       # AWS infrastructure provisioning code
└── platform/              # GitOps-managed platform services and config
```

## Prerequisites

- AWS account and credentials configured locally
- Terraform `>= 1.5.0`
- `kubectl`, `helm`, and `argocd` CLI
- A domain with DNS records pointed to your EC2 public IP

## Bootstrap Flow

1. Provision EC2 infrastructure:
   `infra/terraform/aws-k3s`
2. Install K3s on the instance (Traefik disabled).
3. Install Argo CD in namespace `argocd`.
4. Bootstrap root application:
   `kubectl apply -n argocd -f platform/argocd/apps/root-app.yaml`
5. Argo CD reconciles platform and app manifests from this repo.

## Proof Checklist

- Argo CD: applications are `Synced` and `Healthy`
- Ingress: external routes resolve and serve expected services
- cert-manager: certificates are issued and renewed
- Grafana: cluster metrics and logs are visible

## Platform Proof (Screenshots)

### Argo CD

![Argo CD applications](docs/screenshots/ArgoCD-Apps.png)
![Argo CD kube-prometheus-stack detail](docs/screenshots/ArgoCD-Kube-Prometheus-Stack-Detail.png)
![Argo CD ingress-nginx detail](docs/screenshots/ArgoCD-Ingress-Nginx-Detail.png)

### Grafana + Loki

![Grafana dashboard](docs/screenshots/Grafana-Dashboard.png)
![Grafana Loki logs](docs/screenshots/Grafana-Loki.png)
![Grafana node exporter](docs/screenshots/Grafana-Node-Exporter.png)
![Grafana alertmanager](docs/screenshots/Grafana-Alert-Manager.png)

### Runtime + TLS Proof

![Cluster state proof](docs/screenshots/Cluster-State-Of-Proof.png)
![Cert-manager issuance proof](docs/screenshots/Cert-Manager.png)
![TLS proof](docs/screenshots/TLS-Proof.png)

## Useful Verification Commands

```bash
# Show timestamp in proof capture
date -u

# Cluster state
kubectl get nodes -o wide
kubectl get pods -A
kubectl get ingress -A

# Certificate issuance
kubectl get certificate -A
kubectl describe certificate echo-tls -n dev

# TLS certificate proof
echo | openssl s_client -connect echo.yared.site:443 -servername echo.yared.site 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
curl -I https://echo.yared.site
```

## Current Trade-offs

- Single-node cluster (cost-efficient, not HA)
- Some bootstrap steps are manual (K3s and Argo CD installation)
- Environment overlays (`envs/dev`, `envs/prod`) are documented and evolving

## Roadmap

- Add CI checks (`yamllint`, `terraform validate`, schema checks)
- Fully GitOps-manage cert-manager installation (not only issuer config)
- Implement environment-specific overlays for dev/prod workloads
- Add policy and security hardening resources under `platform/security`
