# AWS K3s Terraform

Terraform code to provision AWS infrastructure for a single-node K3s Kubernetes cluster.

## What it creates
- EC2 instance (Ubuntu)
- Security Group allowing:
  - SSH (22) from `my_ip_cidr`
  - Kubernetes API (6443) from `my_ip_cidr`
  - HTTP (80) from anywhere
  - HTTPS (443) from anywhere
- EC2 key pair (public key uploaded to AWS)
- 30GB root disk

## Usage

Initialize:
```bash
terraform init
