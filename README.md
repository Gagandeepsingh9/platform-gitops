## 📐 Architecture Overview

![Architecture Diagram](./architecture.png)

This repository is the **single source of truth** for the Kubernetes platform and application delivery using a **GitOps-first DevSecOps model** powered by **Argo CD**, **Helm**, and **Kubernetes** on **AWS EKS**.

It manages:
- Platform services (Argo CD, Ingress, Cert-Manager, Prometheus, Grafana)
- Application delivery for **Gemini Clone App**
- Secure, automated, self-healing deployments

---

**Core Design**
- Private EKS Cluster inside AWS VPC  
- Bastion Host with AWS SSM (No SSH)  
- Two Node Groups:
  - `system-node-group`: platform services
  - `gemini-app-node-group`: application workloads
- GitOps via Argo CD (App of Apps Pattern)
- Dedicated LoadBalancers for:
  - Argo CD
  - Ingress Controller
  - Prometheus
  - Grafana

---

## 🔗 Related Repositories

| Purpose | Repository |
|-------|-----------|
| Infrastructure (Terraform) | https://github.com/Gagandeepsingh9/gemini-clone-infra |
| Application Source & CI | https://github.com/Gagandeepsingh9/Project-gemini-clone |
| Platform GitOps (This Repo) | https://github.com/Gagandeepsingh9/platform-gitops |

---

## 🧱 Repository Structure

```bash
platform-gitops/
├── apps/                     # ArgoCD Applications (App of Apps)
│   ├── application-cert-manager.yml
│   ├── application-ingress.yml
│   ├── application-pg.yml
│   ├── application-gemini.yml
├── argocd/
│   └── values-argocd.yml
├── platform/
│   ├── cert-manager/values-cert-manager.yml
│   ├── ingress-nginx/values-ingress.yml
│   └── prometheus-grafana/values-pg.yml
├── gemini-clone-app/k8s/      # Application manifests
├── root-application.yml      
```

---

## 🚀 GitOps Deployment Flow

### 1. Infrastructure Provisioning  
Handled in: `gemini-clone-infra`

Terraform provisions:
- AWS VPC (Private Subnets)
- Bastion Host
- EKS Cluster (Private Endpoint)
- Node Groups (System + App)

---

### 2. Bootstrap Argo CD

```bash
kubectl apply -f argocd/
```

Argo CD is exposed using a LoadBalancer as defined in `values-argocd.yml`.

---

### 3. Root GitOps Application

```bash
kubectl apply -f root-application.yml
```

This automatically deploys all platform and application components.

---

## 📦 Platform Components

### Cert-Manager
- Installed via Helm
- Sync Wave: `0`
- Handles TLS with Let’s Encrypt
- Runs on `system-node-group`

### Ingress NGINX
- Helm-based installation
- Dedicated LoadBalancer
- Routes traffic to Gemini App

### Prometheus + Grafana
- Installed via Helm using Chart `kube-prometheus-stack`
- Persistent volumes for metrics
- Dedicated LoadBalancers
- Runs on system nodes

### Argo CD
- GitOps engine
- App of Apps pattern
- Self-healing & auto-sync enabled

---

## 📱 Application Delivery – Gemini Clone

### Source & CI: `Project-gemini-clone`

Pipeline:
1. Clone Repo  
2. Build Docker Image  
3. Scan with Trivy  
4. Push to DockerHub  
5. Update GitOps Repo (this repo)

Jenkins updates:
```bash
gemini-clone-app/k8s/gemini-deployment.yml
```

### CD via Argo CD

```yaml
apps/application-gemini.yml
```

- Namespace: `gemini`
- Automated Sync
- Self-heal enabled

---

## 🔐 Security Design

- Private EKS Endpoint
- No SSH – only AWS SSM
- Node Isolation by Workload
- Image Scanning via Trivy
- TLS via cert-manager

---

## 📊 Monitoring Flow

- Prometheus scrapes:
  - Nodes
  - Pods
  - Ingress
- Grafana visualizes dashboards
- Access via dedicated LoadBalancer

---

## 🌐 Traffic Flow

### Admin Access
```text
Admin → SSM → Bastion → Private EKS Endpoint → Cluster
```

### User Access
```text
User → DNS → Ingress LB → Ingress → Gemini Service → Pods
```

### Observability Access
```text
Admin → Grafana/Prometheus LB → Services → Pods
```

---

## 🛠️ How to Use

```bash
# After infra & ArgoCD installed
kubectl apply -f root-application.yml
```

Everything else is automated.

---

