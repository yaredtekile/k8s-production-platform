# K3s Production Platform – Ingress Controller Conflict & Recovery
## Incident Report + Architectural Evolution

---

# 1. Context

This document describes a real production-style debugging incident that occurred during the GitOps migration phase of the K3s-based production platform running on AWS.

The platform architecture at the time included:

- AWS EC2 (t3.small)
- K3s (single-node control plane)
- ingress-nginx
- cert-manager (Let's Encrypt)
- Argo CD (GitOps)
- Webhook-based auto-sync

All components were being progressively converted into GitOps-managed resources using an App-of-Apps model.

---

# 2. Incident Summary

After converting ingress-nginx into a GitOps-managed Helm Application via Argo CD:

- `argocd.yared.site` stopped working
- Argo CD CLI returned TLS verification errors
- Eventually, the endpoint became completely unreachable

This was not a simple misconfiguration. It was a layered interaction between:

- K3s defaults
- Multiple ingress controllers
- Service exposure types
- AWS networking
- GitOps reconciliation

---

# 3. Initial Symptoms

## 3.1 TLS Certificate Mismatch

Running:

```bash
curl -vkI https://argocd.yared.site
```

Revealed:

```text
subject: CN=TRAEFIK DEFAULT CERT
issuer: CN=TRAEFIK DEFAULT CERT
```

Instead of the expected Let's Encrypt certificate.

Argo CD CLI returned:

```text
tls: failed to verify certificate
x509: certificate valid for TRAEFIK DEFAULT CERT
```

## 3.2 After Disabling Traefik

Once Traefik was disabled:

```yaml
disable:
  - traefik
```

New error:

```text
connection refused
```

Port 443 was no longer reachable.

# 4. Root Cause Analysis

This issue had three independent layers.

## 4.1 Layer 1 – K3s Default Traefik

K3s installs Traefik automatically unless explicitly disabled.

Even though ingress-nginx was installed and functioning, Traefik was still:

- Running in kube-system
- Binding host ports 80 and 443
- Serving its default self-signed certificate

Traffic to:

`https://argocd.yared.site`

Was terminating at Traefik, not nginx.

That explains the `TRAEFIK DEFAULT CERT`.

## 4.2 Layer 2 – Removing Traefik Broke Host Port Binding

Once Traefik was disabled and K3s restarted:

Nothing was binding host ports 80/443.

Why?

Because ingress-nginx was configured as:

```yaml
type: NodePort
```

NodePort exposes ports in the 30000–32767 range. It does NOT bind host port 80/443 automatically.

AWS Security Group only allowed:

- 80
- 443

NodePorts were not exposed externally.

Result: `connection refused`

## 4.3 Layer 3 – GitOps Reconciliation Reverted Manual Fix

Manually patching the Service:

```bash
kubectl patch svc ingress-nginx-controller ...
```

Changed it temporarily to:

```yaml
type: LoadBalancer
```

But it reverted back to:

```yaml
type: NodePort
```

Why?

Because ingress-nginx was now managed by:

- Argo CD
- Helm
- Git values file

And Git still declared:

```yaml
controller:
  service:
    type: NodePort
```

GitOps reconciliation correctly enforced declared state.

This confirmed: **Source of truth = Git**

# 5. Final Resolution

The correct fix was to resolve the issue at the Git level.

## Step 1 – Permanently Disable Traefik

Created `/etc/rancher/k3s/config.yaml` with:

```yaml
disable:
  - traefik
```

Restarted K3s.

- Traefik pods removed.
- Traefik service removed.

## Step 2 – Update ingress-nginx to LoadBalancer (Git)

Updated `platform/ingress/ingress-nginx-values.yaml`.

From:

```yaml
controller:
  service:
    type: NodePort
```

To:

```yaml
controller:
  service:
    type: LoadBalancer
```

Committed and pushed.

## Step 3 – K3s svclb Pods Bound Host Ports

K3s automatically created `svclb-ingress-nginx-*`.

These pods:

- Bind host port 80
- Bind host port 443
- Forward traffic to ingress-nginx controller

Traffic flow restored:

`Internet → EC2:443 → svclb → ingress-nginx → Argo CD`

Argo CD endpoint restored. Let's Encrypt certificate served correctly.

# 6. Technical Deep Dive

## 6.1 NodePort vs LoadBalancer (In K3s)

**NodePort:**
- Exposes service on high port (30000+)
- Requires SG changes to expose publicly
- Does NOT bind host 80/443

**LoadBalancer (K3s):**
- Creates `svclb-*` pods
- Binds host ports 80/443 directly
- No cloud ELB created
- Acts as bare-metal LB

## 6.2 Why This Did Not Fail Earlier

Originally:

- Traefik was binding 80/443
- Even though nginx existed, Traefik terminated traffic first

Once Traefik was removed, ingress-nginx had no host binding.
The architecture changed subtly, but significantly.

## 6.3 GitOps Enforcement

Argo CD enforced:

- Helm release configuration
- Values file state
- Service type definition

Manual changes were reverted.
This confirms reconciliation was working as intended.

# 7. Lessons Learned

**✔ K3s Defaults Matter**
Always explicitly disable components you do not use. Never rely on implicit defaults.

**✔ Only One Ingress Controller Should Own 80/443**
Running multiple ingress controllers creates:
- TLS conflicts
- Certificate mismatches
- Traffic ambiguity

**✔ NodePort Is Not Internet Exposure**
NodePort exposes ports internally. LoadBalancer (in K3s) binds host ports.

**✔ Git Is The Only Truth In GitOps**
Manual fixes are temporary. Permanent fixes must modify declared state.

**✔ Debugging Order Matters**
Correct approach:
1. Inspect certificate being served
2. Identify ingress controller handling traffic
3. Inspect Service type
4. Verify host port binding
5. Check Git desired state
6. Fix in Git
7. Let reconciliation converge

# 8. Architectural Outcome

After resolution:

- Traefik permanently disabled
- ingress-nginx is LoadBalancer
- Host ports 80/443 bound correctly
- Argo CD fully Git-managed
- ingress-nginx fully Git-managed
- cert-manager fully Git-managed
- Echo app Git-managed
- Webhook auto-sync enabled
- Platform deterministic and reproducible

# 9. Platform Maturity Level

This incident demonstrates:

- Multi-layer debugging
- TLS inspection at certificate level
- Kubernetes Service exposure mechanics
- Helm + Argo CD reconciliation behavior
- Cloud networking interaction
- Production-style recovery

This is no longer a simple Kubernetes demo cluster. It is now a real GitOps production platform with hardened architecture.

# 10. Final Statement

This debugging process reinforced a critical rule:

**In GitOps-driven systems, runtime behavior must always be evaluated against declared state.**

The final architecture is cleaner, deterministic, and production-ready.
