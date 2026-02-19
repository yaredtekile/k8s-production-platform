# AWS K3s Terraform

Terraform configuration for provisioning the AWS host used by this platform.

## Scope

This module creates a single-node EC2 foundation for running K3s.

## What It Creates

- Default VPC and subnet lookup
- Security group with:
  - `22/tcp` from `my_ip_cidr`
  - `6443/tcp` from `my_ip_cidr`
  - `80/tcp` from `0.0.0.0/0`
  - `443/tcp` from `0.0.0.0/0`
- EC2 key pair from local key `~/.ssh/k3s_aws.pub`
- Ubuntu EC2 instance
- GP3 root disk (default: `30GB`)

## Inputs

- `aws_region` (default: `eu-west-1`)
- `my_ip_cidr` (required, ex: `203.0.113.10/32`)
- `instance_type` (default: `t3.micro`)
- `root_volume_gb` (default: `30`)

## Outputs

- `public_ip`
- `ssh_command`

## Usage

```bash
cd infra/terraform/aws-k3s
terraform init
terraform plan -var="my_ip_cidr=<YOUR_PUBLIC_IP>/32"
terraform apply -var="my_ip_cidr=<YOUR_PUBLIC_IP>/32"
```

## Post-Apply

```bash
terraform output public_ip
terraform output ssh_command
```

## Notes

- This is intentionally cost-oriented and single-node (not HA).
- It assumes the target region has a default VPC.
- The SSH public key path is currently hardcoded in `main.tf`.
