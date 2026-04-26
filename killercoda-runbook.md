# Playbook: Ephemeral Cloud Testing (Killercoda)

This runbook documents the standard operating procedure for testing the FinTech application architecture in a zero-cost, ephemeral cloud environment prior to production deployment.

## The Value of Ephemeral Sandboxes

Before provisioning expensive managed services (like Amazon EKS), infrastructure manifests must be tested in a remote, multi-node cloud environment. Killercoda provides an instant, browser-based Kubernetes cluster that self-destructs after 60 minutes, ensuring zero billing risk while validating cloud compatibility.

## Deployment Procedure

To deploy the architecture to a fresh Killercoda environment, execute the following within the provided browser terminal:

### 1. Pull the Infrastructure as Code (IaC)

```bash
git clone https://github.com/<your-username>/kubernetes-fintech-app.git
cd kubernetes-fintech-app
```

### 2. Initialize Configuration & Security First

```bash
kubectl apply -f frontend-configmap.yaml
kubectl apply -f backend-secret.yaml
```

### 3. Deploy Data and Presentation Tiers

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

### 4. Verify Portability

```bash
kubectl get all
```

If all Pods reach a `Running` state, the architecture is verified as cloud-agnostic and is cleared for deployment to enterprise environments like EKS or GKE.
