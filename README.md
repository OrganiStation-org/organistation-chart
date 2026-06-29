# ⛵ OrganiStation Helm Umbrella Chart

This repository contains the **Kubernetes Manifest Orchestration** for the OrganiStation platform. It implements the **Helm Umbrella Chart** pattern to package, configure, and manage all 8 platform microservices as a single deployable application.

---

## 🏗️ Chart Architecture

The deployment is managed by a parent (umbrella) chart that coordinates the deployment, networking, and configuration of 8 sub-charts located in the `charts/` directory:

- `gateway`: Express reverse-proxy routing all external API traffic.
- `frontend`: React SPA web dashboard.
- `auth`: Identity provider issuing JWTs, seeding permissions, and managing RBAC.
- `ai`: RAG backend using ChromaDB vector database and Groq LLM inference.
- `hr`: Employee records, rosters, and leave applications.
- `projects`: Kanban task boards, milestones, and ticketing.
- `finance`: Revenue charts, invoices, and expense claims.
- `notification`: Real-time websocket notifications (via Azure WebPubSub) and transactional emails (via Azure Communication Services).

### Global Resources
* **Unified Ingress**: Configured at the top-level chart (`templates/ingress.yaml`) to route public DNS traffic through a single Application Gateway/Ingress Controller directly to the gateway and frontend.
* **Workload Identity**: Shared `ServiceAccount` and CSI Secret Provider Class configurations mapping Azure Key Vault secrets directly into sub-chart pods.

---

## 🌓 Environment Decoupling

The platform uses custom value files to override the base configurations in `values.yaml` for environment isolation:

### 🧪 Development (`dev-values.yaml`)
- **Target Namespace**: `dev-ns`
- **Resource Limits**: Requests/limits tuned down for cost-savings in test environments.
- **Replicas**: 1 pod per microservice.

### 🚀 Production (`prod-values.yaml`)
- **Target Namespace**: `prod-ns` (**Strict Enforced Isolation**)
- **Resource Limits**: Configured with high performance limits.
- **High Availability**: 2+ replicas per service with Pod Anti-Affinity rules.

---

## 🚚 CD Deployment via ArgoCD

The chart is continuously deployed to Azure Kubernetes Service (AKS) using **ArgoCD**. The ArgoCD application manifests are stored in the `shared-workflows` repository under the `argocd/` directory.

ArgoCD automatically tracks changes to the `develop` branch (for the Dev environment) and the `main` branch (for the Prod environment) in this repository and synchronizes the cluster state.

### Dev Environment Sync Manifest (`dev-argocd.yaml`)
- Deploys the chart from the `develop` branch.
- Target Namespace: `dev-ns`

### Prod Environment Sync Manifest (`prod-argocd.yaml`)
- Deploys the chart from the `main` branch.
- Target Namespace: `prod-ns`

---

## 🚀 Manual Deployment via CLI

If you need to install or upgrade the release manually, run the following commands from the root of this directory:

### 1. Dev Environment Deploy
```bash
helm upgrade --install organistation ./ \
  -n dev-ns --create-namespace \
  -f dev-values.yaml \
  --set global.namespace=dev-ns \
  --set auth.image.name=<your-acr>.azurecr.io/auth \
  --set auth.image.tag=<image-sha> \
  --set gateway.image.name=<your-acr>.azurecr.io/gateway \
  --set gateway.image.tag=<image-sha>
```

### 2. Prod Environment Deploy
```bash
helm upgrade --install organistation ./ \
  -n prod-ns --create-namespace \
  -f prod-values.yaml \
  --set global.namespace=prod-ns \
  --set auth.image.tag=<image-sha> \
  --atomic
```

---

## 🕵️ Troubleshooting & Verification

### 1. Check Pod Rollout Status
```bash
kubectl get pods -n dev-ns
```

### 2. Verify Key Vault Secrets Sync
If pods are stuck in `ContainerCreating` or `CreateContainerConfigError`, check the CSI driver integration status:
```bash
kubectl describe secretproviderclass organistation-kv -n dev-ns
```

### 3. Verify Ingress Rules
To see the hostnames and backend paths routed by Ingress:
```bash
kubectl get ingress -n dev-ns
```
