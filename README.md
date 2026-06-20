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

**Dev Deployment**:
```bash
helm upgrade --install organistation ./ \
  -n dev-ns --create-namespace \
  -f dev-values.yaml \
  --set global.imageTag=latest
```

**Prod Deployment**:
```bash
helm upgrade --install organistation ./ \
  -n prod-ns --create-namespace \
  -f prod-values.yaml \
  --set global.imageTag=v1.0.0 --atomic
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
