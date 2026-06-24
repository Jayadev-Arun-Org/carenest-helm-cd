# CareNest GitOps & Helm CD (carenest-helm-cd)

Welcome to the **CareNest Continuous Deployment (CD)** repository! This repository acts as the Single Source of Truth for the CareNest infrastructure and application state running on Azure Kubernetes Service (AKS).

CareNest utilizes a **GitOps** methodology, powered by ArgoCD, to continuously reconcile the state of the AKS cluster against the configuration defined in this repository.

## 🧭 Repository Overview

This repository contains the official Helm chart and Kubernetes manifests required to deploy the entire CareNest microservices ecosystem, including:

- **Ingress & Routing:** AKS Ingress Controller mapping domain routes to microservices.
- **Core Microservices:** Frontend, Auth, Appointment, Pharmacy, Notify, and AI.
- **Background Jobs:** AIOps CronJob for anomaly detection.
- **Secrets Management:** Azure Key Vault Secret Store CSI Driver configurations.

## 📂 Helm Chart Structure

The main Helm chart is located in the `/carenest` directory.

```text
carenest-helm-cd/
├── carenest/
│   ├── Chart.yaml             # Helm chart metadata
│   ├── values.yaml            # Single Source of Truth for configs & image tags
│   └── templates/
│       ├── deployments/       # Microservice Deployment manifests
│       ├── services/          # ClusterIP Service manifests
│       ├── ingress/           # Ingress routing rules
│       ├── cronjobs/          # AIOps Agent CronJob definition
│       └── secrets/           # Azure Key Vault SecretProviderClass manifests
└── README.md
```

## 🔄 GitOps Workflow (ArgoCD)

We use **ArgoCD** to automatically deploy changes from this repository to the `carenest-dev` and `prod` namespaces in our AKS cluster.

1. **Continuous Integration (CI):** When code is merged into `carenest-app-ci`, a GitHub Action automatically builds the Docker images, pushes them to ACR, and commits the new image tags into this repository's `values.yaml` file.
2. **Reconciliation:** ArgoCD (running inside the AKS cluster) detects the commit in this repository within 3 minutes.
3. **Deployment:** ArgoCD automatically applies the changes to the cluster, ensuring zero drift between the repository and the live environment.

## 🔐 Azure Key Vault Integration

Instead of storing sensitive secrets (like MongoDB URIs, JWT Secrets, Slack Webhooks, or OpenAI Keys) in plain text or standard Kubernetes Secrets, we utilize the **Azure Key Vault Secret Store CSI Driver**.

- The `secret-provider-class.yaml` explicitly maps secrets from the Azure Key Vault (`jd-carenest-new-kv`).
- When a pod spins up, the CSI driver dynamically retrieves the secret from Key Vault using the cluster's User-Assigned Managed Identity and mounts it directly into the pod as an environment variable or volume mount.

## 🤖 AIOps Agent

This repository manages the `aiops-cronjob.yaml`. The agent runs every 5 minutes to:
1. Query Azure Log Analytics for CPU/Memory metrics of the AKS nodes.
2. Pass the data to Azure AI Foundry for anomaly detection (with tracing sent to Langfuse Cloud).
3. Dispatch high-severity alerts to a Slack Webhook if the cluster approaches a scaling threshold.
