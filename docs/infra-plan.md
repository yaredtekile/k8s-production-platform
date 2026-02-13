# Infrastructure Plan (AWS)

## Goal
Provision a low-cost Kubernetes cluster using K3s on a single AWS EC2 instance, suitable for a production-style portfolio project.

## Resources (Minimum)
- EC2 instance (Ubuntu 22.04/24.04) running K3s
- Security Group:
  - SSH: 22 (restricted to my IP)
  - HTTP: 80 (public)
  - HTTPS: 443 (public)
- Public IP (instance public IP; Elastic IP optional)
- DNS:
  - A record for app domain pointing to instance public IP
  - Optional wildcard record for subdomains

## Instance Sizing
Planned instance type: t2.micro
Disk: 30 GB

## Access Model
- SSH key-based access
- kubeconfig retrieved securely and used from laptop via kubectl

## Notes / Tradeoffs
- Single-node cluster is cost-effective but not HA
- Namespace separation used for dev/prod
