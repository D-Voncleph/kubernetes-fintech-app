# FinTech App: Kubernetes Infrastructure

This repository contains the Kubernetes manifest files used to orchestrate, scale, and manage the FinTech application container.

## ⚠️ SECURITY WARNING: Secret Management

**DO NOT USE THIS PATTERN IN PRODUCTION.**

The `backend-secret.yaml` manifest in this repository contains Base64 encoded database credentials. It has been committed to version control **strictly for educational and demonstration purposes** to illustrate the Kubernetes Secret injection lifecycle.

In a true production environment (e.g., Upscale DV client architecture), raw Kubernetes Secrets should **never** be committed to Git. Production best practices dictate:
1. Adding all `*-secret.yaml` files to `.gitignore`.
2. Utilizing an encryption tool like Mozilla SOPS to encrypt the YAML file before committing.
3. Completely bypassing native Secrets in favor of an external Secrets Manager (like HashiCorp Vault or AWS KMS) integrated with the cluster.

## 🏗️ Architecture Overview

The local development and testing environment is built on a Linux-native architecture utilizing **Windows Subsystem for Linux (WSL2)** and **Docker Desktop**. This ensures parity with production Linux servers while maintaining local development speed.

## ⚙️ Prerequisites & Toolchain Setup

To run this cluster locally, ensure the following tools are installed and configured:

### 1. The Container Engine (Docker Desktop)

Minikube requires a container hypervisor to run the local cluster nodes.

- Install [Docker Desktop](https://www.docker.com/products/docker-desktop/).
- **Crucial:** Navigate to Settings > Resources > WSL Integration and enable integration with your default WSL (Ubuntu) distribution.

### 2. The CLI Client (kubectl)

This is the command-line interface used to communicate with the Kubernetes API Server. Install it directly inside your WSL environment:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### 3. The Local Cluster (Minikube)

Minikube provisions a single-node Kubernetes cluster (combining the Control Plane and Worker Node) inside a Docker container. Install it in WSL:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## 🚀 Bootstrapping the Cluster

Once the toolchain is installed, boot the cluster using the Docker driver:

```bash
minikube start --driver=docker
```

Verify the API Server is responding and the node is ready:

```bash
kubectl get nodes
```
## 🚀 Kubernetes Deployment

This application utilizes a decoupled, multi-tier architecture orchestrated by Kubernetes. The frontend and backend are managed as separate Deployments, allowing them to be scaled and updated independently.

### Prerequisites
* A running Kubernetes cluster (e.g., Minikube or Docker Desktop).
* `kubectl` CLI installed and configured.

### Deploying the Application

To spin up the entire application stack, apply both declarative manifests to your cluster:

1. **Deploy the Backend API:**
   This provisions the Node.js backend using the custom Docker image.
```bash
   kubectl apply -f backend-deployment.yaml
```

2. **Deploy the Frontend Web Server:**
   This provisions the Nginx web server.
```bash
   kubectl apply -f frontend-deployment.yaml
```

3. **Verify the Deployment:**
   Ensure both Deployments and their respective Pods are running successfully:
```bash
   kubectl get deployments
   kubectl get pods
```

---

### 🚀 Step 2: Push Your Code and Docs

Now, let's get that new `frontend-deployment.yaml` and your updated `README.md` safely stored in version control.


## 🌐 Network Architecture

This application utilizes a decoupled, multi-tier Kubernetes network. External traffic is strictly controlled, and backend services are securely isolated from the public internet.

### Traffic Flow Diagram

\```text
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
\```

### Network Components

1. **Frontend Service (NodePort):** Acts as the public entry point. It opens a physical port on the cluster's worker nodes, routing external internet traffic down to the Nginx pods.
2. **Backend Service (ClusterIP):** Acts as an internal load balancer and static DNS record (`http://fintech-backend`). It completely walls off the Node.js API from the outside world, ensuring the backend can only be accessed by other authorized pods within the cluster.
