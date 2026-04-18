# 🚀 FinTech Application Kubernetes Stack

This repository contains the declarative Infrastructure as Code (IaC) to deploy a highly available, multi-tier FinTech application on Kubernetes.

## ⚠️ SECURITY WARNING: Secret Management

**DO NOT USE THIS PATTERN IN PRODUCTION.**

The `backend-secret.yaml` manifest in this repository contains Base64 encoded credentials. It has been committed to version control strictly for educational purposes. In a true production environment, Secrets must be excluded via `.gitignore`, encrypted via SOPS, or managed by an external KMS (like HashiCorp Vault).

---

## 🌐 Network Architecture

This application utilizes a decoupled, multi-tier network. The backend is securely isolated, while the frontend is exposed via a NodePort.

```text
[Public Internet / Web Browser]
           │
           ▼  (External Traffic via Port 30000+)
  ┌─────────────────────────────┐
  │ Service: fintech-frontend   │  <-- Type: NodePort
  └─────────────┬───────────────┘
                │ (Internal Port 80)
                ▼
  ┌─────────────────────────────┐
  │ Pods: Nginx Web Servers     │  <-- Presentation Tier
  └─────────────┬───────────────┘
                │ (Internal HTTP Requests)
                ▼
  ┌─────────────────────────────┐
  │ Service: fintech-backend    │  <-- Type: ClusterIP (Internal DNS)
  └─────────────┬───────────────┘
                │ (Internal Port 5000)
                ▼
  ┌─────────────────────────────┐
  │ Pods: Node.js API           │  <-- Data / Logic Tier
  └─────────────────────────────┘
```

---

## 🛠️ Deployment Guide (Minikube)

Follow these steps to launch the entire stack on a local Minikube cluster.

### Prerequisites

- Minikube installed and running (`minikube start`)
- `kubectl` installed and configured

### Step 1: Initialize Configuration & Security

Deploy the static data that the Pods rely on to boot successfully:

```bash
kubectl apply -f frontend-configmap.yaml
kubectl apply -f backend-secret.yaml
```

### Step 2: Deploy the Data Tier (Backend)

Deploy the Node.js API and abstract it behind an internal ClusterIP:

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
```

### Step 3: Deploy the Presentation Tier (Frontend)

Deploy the Nginx web servers and expose them externally:

```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

### Step 4: Verify and Access

Confirm all resources are running:

```bash
kubectl get all
```

Establish a tunnel to view the application in your local browser:

```bash
minikube service fintech-frontend
```
