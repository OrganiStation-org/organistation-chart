# ⛵ OrganiStation Helm Umbrella Chart

This repository contains the **Kubernetes Manifest Orchestration** for the OrganiStation platform. It uses the "Umbrella Chart" pattern to manage 8 microservices as a single logical unit.

---

## 🏗️ Chart Architecture

The platform is deployed using a top-level chart that coordinates these internal sub-charts:
- `auth`, `ai`, `hr`, `finance`, `projects`, `gateway`, `frontend`, `notification`.

### Key Global Features:
*   **Unified Ingress**: All services are routed through a single Application Gateway / Ingress controller.
*   **Workload Identity**: Shared `ServiceAccount` and `SecretProviderClass` logic for Azure Key Vault synchronization.
*   **Consolidated HPA**: Horizontal Pod Autoscaling is standardized across the stack.

---

## 🌓 Environment Decoupling

We use strict value separation to prevent development changes from affecting production stability.

### 🧪 Development (`dev-values.yaml`)
- **Namespace**: `dev-ns`
- **Resource Limits**: Requests/Limits tuned for cost savings.
- **Replicas**: 1 per service.
- **Public Access**: Direct via LoadBalancer (if needed).

### 🚀 Production (`prod-values.yaml`)
- **Namespace**: `prod-ns` (**STRICT ENFORCEMENT**)
- **Resource Limits**: Guaranteed QOS (Quality of Service) with higher limits.
- **High Availability**: 2+ replicas with Anti-Affinity rules.
- **Security**: Only accessible via private link/Internal App Gateway.

---

## 🚀 Deployment via CLI

While the GitHub Action is preferred, you can deploy manually using:

Images are tagged in ACR with the **first 7 characters of the git commit SHA** (e.g. `a1b2c3d`). Each microservice has its own tag in ACR.

The deploy workflow resolves the **latest tag per repository** from ACR automatically. You can also pass one SHA for all services via workflow dispatch.

**Manual deploy (per-service tags from your ACR list)**:
```bash
helm upgrade --install organistation ./ \
  -n dev-ns --create-namespace \
  -f dev-values.yaml \
  --set global.namespace=dev-ns \
  --set auth.image.name=mahesh6l0facrprod.azurecr.io/auth \
  --set auth.image.tag=37af23c \
  --set notification.image.name=mahesh6l0facrprod.azurecr.io/notification \
  --set notification.image.tag=d528b9c \
  --set gateway.image.name=mahesh6l0facrprod.azurecr.io/gateway \
  --set gateway.image.tag=80dd4f3
  # ...repeat for ai, hr, finance, projects
```

**Prod deploy**:
```bash
helm upgrade --install organistation ./ \
  -n prod-ns --create-namespace \
  -f prod-values.yaml \
  --set auth.image.tag=<latest-auth-sha> \
  --atomic
```

---

## 🕵️ Troubleshooting & Monitoring

### 1. Check Pod Status
```bash
kubectl get pods -n dev-ns
```

### 2. Verify Key Vault Sync
If pods fail to start, check the CSI driver:
```bash
kubectl describe secretproviderclass organistation-kv -n dev-ns
```

### 3. Service Rollout Status
```bash
kubectl rollout status deployment/gateway -n dev-ns
```
