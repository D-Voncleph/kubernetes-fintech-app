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

## Deployment Runbooks

This architecture is platform-agnostic and can be deployed to both local and cloud-native Kubernetes environments.

### Option A: Local Development (Minikube)

1. Start the local cluster: `minikube start`
2. Apply the manifests: `kubectl apply -f .`
3. Access the application: `minikube service frontend-service`

### Option B: Ephemeral Cloud / Production EKS

For zero-cost testing, utilize an ephemeral sandbox (e.g., Killercoda), or authenticate to your AWS EKS cluster via `aws eks update-kubeconfig`.

1. Clone this repository into the deployment terminal.

2. Apply the configuration and security layers:

   ```bash
   kubectl apply -f frontend-configmap.yaml
   kubectl apply -f backend-secret.yaml
   ```

3. Apply the application tiers:

   ```bash
   kubectl apply -f backend-deployment.yaml && kubectl apply -f backend-service.yaml
   kubectl apply -f frontend-deployment.yaml && kubectl apply -f frontend-service.yaml
   ```

4. Verify deployment health: `kubectl get pods -o wide`



# Layer 7 Networking & Routing (Ingress)

To ensure secure, cost-effective scaling, this architecture strictly avoids exposing individual microservices via Layer 4 Network Load Balancers. Instead, all external traffic is routed through a single Layer 7 Ingress Controller.

## The Routing Manifest (`frontend-ingress.yaml`)

This declarative rulebook acts as the cluster's traffic cop, dynamically routing incoming HTTP requests to the isolated internal services.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fintech-ingress
spec:
  ingressClassName: nginx # Note: Change to 'alb' for AWS EKS deployments
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

## Ingress Deployment Strategies

This application is engineered for platform agnosticism and supports multiple Ingress controllers depending on the target environment.

### Option A: Ephemeral Sandbox (NGINX Ingress Controller)

For zero-cost validation in environments like Killercoda or bare-metal:

1. Provision the platform-agnostic controller:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.2/deploy/static/provider/baremetal/deploy.yaml
```

2. Apply the routing rulebook:

```bash
kubectl apply -f frontend-ingress.yaml
```

3. Retrieve the exposed NodePort to access the application:

```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

### Option B: Enterprise AWS EKS (Application Load Balancer)

For production deployments requiring AWS native integration:

1. Ensure the AWS Load Balancer Controller is provisioned via the cluster's Terraform state.
2. Update the `ingressClassName` in the manifest to `alb`.
3. Apply the routing rulebook:

```bash
kubectl apply -f frontend-ingress.yaml
```

4. Retrieve the AWS-generated public DNS address:

```bash
kubectl get ingress
```
---
