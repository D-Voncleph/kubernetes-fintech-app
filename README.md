# FinTech App: Kubernetes Infrastructure

This repository contains the Kubernetes manifest files used to orchestrate, scale, and manage the FinTech application container.

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