# Kubernetes Production Platform on AWS

## Overview

This project implements a production-oriented Kubernetes platform running on AWS EC2 using K3s.

The platform follows GitOps principles and includes:

- Automated deployments with Argo CD
- HTTPS with cert-manager and Let’s Encrypt
- Ingress-based traffic routing
- Metrics monitoring (Prometheus + Grafana)
- Centralized logging (Loki)

The system is structured to resemble a real-world DevOps / SRE production environment.

---

## Architecture Diagram

![Architecture](./architecture.png)

---

## Architecture Layers

### 1️⃣ Developer & Git Layer

**Developer Laptop**  
Developers push application and platform manifests to GitHub.

**GitHub Repository**  
Acts as the single source of truth for the entire cluster state following the GitOps model.

---

### 2️⃣ GitOps Control Layer

**Argo CD (inside Kubernetes)**  
Argo CD continuously monitors the GitHub repository and synchronizes the desired state into the Kubernetes cluster.

It applies manifests via the Kubernetes API and automatically reconciles drift.

This ensures:
- Fully declarative deployments
- No manual cluster changes
- Continuous synchronization with Git

---

### 3️⃣ Cloud Infrastructure Layer

**AWS EC2 Instance**  
Hosts the Kubernetes cluster.

**K3s Kubernetes Cluster**  
A lightweight Kubernetes distribution managing workloads and platform services.

All applications and add-ons run inside this cluster.

---

### 4️⃣ Traffic & Ingress Layer

**Internet Users**  
External users access services via HTTPS.

**Ingress Controller**  
Receives incoming traffic and routes it to internal services (e.g., API / Worker).

**cert-manager**  
Automates TLS certificate provisioning and renewal.

**Let’s Encrypt**  
Certificate Authority used to issue trusted TLS certificates via ACME challenges.

---

### 5️⃣ Application Layer

**API / Worker (Placeholder Application)**  
Represents business workloads running inside the cluster.

Key characteristics:
- Exposes health endpoints
- Uses environment-based configuration
- Designed for autoscaling
- Emits structured logs

---

### 6️⃣ Observability Layer

**Prometheus**  
Scrapes metrics from application pods and cluster components.

**Grafana**  
Visualizes metrics using dashboards.

**Loki**  
Aggregates logs from application pods for centralized log analysis.

This enables:
- Performance monitoring
- Error rate tracking
- Resource visibility
- Troubleshooting

---

## Flow Explanation

### 🔵 Deployment Flow (GitOps)

Developer → GitHub → Argo CD → Kubernetes Cluster

All changes are pushed to GitHub.  
Argo CD detects changes and applies them to the cluster automatically.

---

### 🟢 User Traffic Flow

Internet Users → Ingress → Application

Incoming HTTPS traffic is routed through the Ingress controller to the appropriate service.

---

### 🟣 Observability Flow

Application → Prometheus (metrics)  
Prometheus → Grafana (dashboards)  
Application → Loki (logs)

The platform continuously monitors application health and cluster state.

---

## Key Platform Capabilities

- GitOps continuous delivery
- Automated TLS via Let’s Encrypt
- Centralized logging
- Metrics-based monitoring
- Clear infrastructure boundaries
- Production-ready architecture separation
