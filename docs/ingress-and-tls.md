# Ingress + TLS (cert-manager) — Notes & Fixes

## Current state
- Ingress controller: ingress-nginx (IngressClass: `nginx`)
- DNS: `echo.yared.site` -> EC2 public IP
- TLS: cert-manager + Let's Encrypt **production**
- Certificate secret: `dev/echo-tls`
- Auto-renewal: cert-manager handles renewal before expiry

## What went wrong (real-world debugging)
During HTTP-01 validation, cert-manager kept reporting:

- `wrong status code '404', expected '200'`

Meaning: the ACME challenge URL was reachable, but the solver path was not being served.

### Root cause
K3s ships with Traefik by default, and Traefik was effectively handling port 80.
So requests to:
`http://echo.yared.site/.well-known/acme-challenge/<token>`
returned a plain text 404 (Traefik-style), not the solver response.

### Fix
- Ensure ingress-nginx is the controller handling external HTTP traffic (port 80/443)
- After nginx owned port 80, the challenge path returned `200 OK`
- Challenge became `valid`, then the certificate issued successfully

## Proof commands
```bash
kubectl get ingressclass
kubectl get ingress -n dev
kubectl get certificate -n dev
curl -i http://echo.yared.site/.well-known/acme-challenge/<token>
curl -vI https://echo.yared.site
