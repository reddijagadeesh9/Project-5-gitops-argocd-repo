
# Project 5 - GitOps Workflow with ArgoCD

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/c49faf33-2dc8-440e-a67d-0c800890512a" />

# Overview

This project demonstrates a complete GitOps workflow implementation using ArgoCD and Kubernetes. The entire Kubernetes deployment configuration is stored in a GitHub repository, and ArgoCD continuously monitors and synchronizes the Kubernetes cluster state with the desired state defined in Git.

The project implements automated synchronization, self-healing, drift detection, multi-environment deployments, Kustomize overlays, ApplicationSet automation, GitHub Actions integration, Sealed Secrets setup, and sync window policies following production-grade GitOps practices.

---

# Project Description

Implemented an end-to-end GitOps deployment workflow using ArgoCD, Kubernetes, GitHub Actions, Kustomize, and KIND Kubernetes cluster. Configured automated synchronization, self-healing, and drift correction by storing Kubernetes manifests in GitHub as the single source of truth.

Implemented multi-environment deployments using Kustomize overlays and ApplicationSet automation for development, staging, and production environments. Configured GitHub Actions for automated image updates and implemented Sealed Secrets setup for secure Kubernetes secret encryption.

---

# Architecture

```text
Developer
    ↓
GitHub Repository
    ↓
GitHub Actions
    ↓
ArgoCD
    ↓
Kubernetes Cluster
    ↓
Application Deployment
````

---

# Tools and Technologies Used

* Kubernetes
* KIND Cluster
* ArgoCD
* GitHub
* GitHub Actions
* Docker
* Helm
* Kustomize
* Sealed Secrets
* Ubuntu EC2
* AWS EC2

---

# Features Implemented

* GitOps Workflow
* ArgoCD Installation and Configuration
* GitHub Repository Integration
* Automatic Synchronization
* Self-Healing
* Drift Detection
* Multi-Environment Deployment
* Kustomize Overlays
* ApplicationSet Automation
* GitHub Actions CI/CD
* Sync Window Policies
* Sealed Secrets Setup and Secret Encryption
* Production-Style GitOps Architecture

---

# Infrastructure Setup

## EC2 Instance Configuration

| Configuration | Value        |
| ------------- | ------------ |
| OS            | Ubuntu 22.04 |
| Instance Type | t3.medium    |
| Storage       | 25 GB        |

---

# STEP 1 - Launch EC2 Instance

Launch Ubuntu EC2 instance and connect using SSH.

```bash
ssh -i key.pem ubuntu@<PUBLIC-IP>
```

---

# STEP 2 - Install Required Tools

## Update Server

```bash
sudo apt update -y
sudo apt upgrade -y
```

## Install Docker

```bash
curl -fsSL https://get.docker.com | bash

sudo usermod -aG docker ubuntu

newgrp docker
```

## Install Kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl

sudo mv kubectl /usr/local/bin/
```

## Install KIND

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64

chmod +x kind

sudo mv kind /usr/local/bin/
```

## Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## Install Kustomize

```bash
curl -s https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh | bash

sudo mv kustomize /usr/local/bin/
```

## Install Git

```bash
sudo apt install git -y
```

---

# STEP 3 - Create Kubernetes Cluster

## Create Cluster Configuration

Create file:

```bash
vim kind-cluster.yaml
```

Paste:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

## Create Cluster

```bash
kind create cluster --name gitops-cluster --config kind-cluster.yaml
```

## Verify Cluster

```bash
kubectl get nodes
```
<img width="873" height="206" alt="Screenshot 2026-05-06 225509" src="https://github.com/user-attachments/assets/3c16d350-2b10-4c85-a883-452fa292b47b" />

---

# STEP 4 - Create GitHub Repository Structure

## Repository Structure

```text
Project-5-gitops-argocd-repo
│
├── apps
│   └── nginx
│       ├── deployment.yaml
│       ├── service.yaml
│       └── kustomization.yaml
│
├── overlays
│   ├── dev
│   ├── staging
│   └── prod
│
├── .github
│   └── workflows
│
├── applicationset.yaml
└── project-sync-window.yaml
```

---

# STEP 5 - Install ArgoCD

## Add Helm Repository

```bash
helm repo add argo https://argoproj.github.io/argo-helm

helm repo update
```

## Create Namespace

```bash
kubectl create namespace argocd
```

## Install ArgoCD

```bash
helm install argocd argo/argo-cd -n argocd
```

## Verify Pods

```bash
kubectl get pods -n argocd
```
<img width="1075" height="280" alt="Screenshot 2026-05-06 230145" src="https://github.com/user-attachments/assets/2e3a8f7c-bf16-4e75-93c1-d3a46022bde1" />

---

# STEP 6 - Expose ArgoCD UI

## Change Service Type

```bash
kubectl patch svc argocd-server -n argocd \
-p '{"spec": {"type": "NodePort"}}'
```

## Port Forward ArgoCD

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address 0.0.0.0
```

## Access UI

```text
https://<EC2-PUBLIC-IP>:8080
```

---

# STEP 7 - Get ArgoCD Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d
```

Username:

```text
admin
```
<img width="1016" height="755" alt="Screenshot 2026-05-06 232223" src="https://github.com/user-attachments/assets/a6674f7f-8818-40a6-b94d-de39b79ef4d5" />

---

# STEP 8 - Create Kubernetes Application Manifests

## Create Application Directory

```bash
mkdir -p apps/nginx
```

---

## Deployment Manifest

Create:

```bash
vim apps/nginx/deployment.yaml
```

Paste:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx-app

  template:
    metadata:
      labels:
        app: nginx-app

    spec:
      containers:
      - name: nginx
        image: nginx:latest

        ports:
        - containerPort: 80
```

---

## Service Manifest

Create:

```bash
vim apps/nginx/service.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx-app

  ports:
  - port: 80
    targetPort: 80

  type: NodePort
```
<img width="957" height="278" alt="Screenshot 2026-05-06 232242" src="https://github.com/user-attachments/assets/8037d328-22ed-4fbc-9ed8-433860e43ce9" />

---

## Kustomization File

Create:

```bash
vim apps/nginx/kustomization.yaml
```

Paste:

```yaml
resources:
  - deployment.yaml
  - service.yaml
```

---

# STEP 9 - Push Code to GitHub

```bash
git add .

git commit -m "Initial GitOps setup"

git push origin main
```

---

# STEP 10 - Create Application in ArgoCD

Configured application from ArgoCD UI using:

* Repository URL
* Path: apps/nginx
* Auto Sync Enabled
* Self Heal Enabled

<img width="1597" height="853" alt="Screenshot 2026-05-06 232511" src="https://github.com/user-attachments/assets/f04759e1-31a6-42ef-a09b-11f452db1cc3" />

---

# STEP 11 - Verify GitOps Workflow

Updated replicas in Git repository and verified ArgoCD automatically synchronized changes to the cluster.

Example:

```yaml
replicas: 2
```

Changed to:

```yaml
replicas: 4
```
<img width="796" height="172" alt="Screenshot 2026-05-06 232540" src="https://github.com/user-attachments/assets/6764edc5-bc3b-4390-b269-41cee5b0d8cb" />

---

# STEP 12 - Test Self-Healing

Deleted deployment manually using:

```bash
kubectl delete deployment nginx-app
```

ArgoCD automatically recreated the deployment.

---
<img width="1024" height="296" alt="Screenshot 2026-05-06 232806" src="https://github.com/user-attachments/assets/969005df-aa95-40e9-96b0-84775542205c" />

# STEP 13 - Configure GitHub Actions

## Create Workflow Directory

```bash
mkdir -p .github/workflows
```

---

## GitHub Actions Workflow

Create:

```bash
vim .github/workflows/update-image.yaml
```

Paste:

```yaml
name: Update Image Tag

on:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  update-image:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Update Image Tag
        run: |
          sed -i 's/nginx:latest/nginx:1.25/' apps/nginx/deployment.yaml

      - name: Commit Changes
        run: |
          git config --global user.email "actions@github.com"
          git config --global user.name "github-actions"

          git add .

          git commit -m "Updated nginx image tag to 1.25" || echo "No changes to commit"

      - name: Push Changes
        run: |
          git push origin main
```
<img width="1916" height="869" alt="Screenshot 2026-05-06 233840" src="https://github.com/user-attachments/assets/5725fe6b-c710-4563-b8ff-b97aaa99611f" />

<img width="1895" height="729" alt="Screenshot 2026-05-06 233850" src="https://github.com/user-attachments/assets/6afd38b5-93d9-4923-9ea7-0ec1555f00ed" />

<img width="1641" height="868" alt="Screenshot 2026-05-06 233816" src="https://github.com/user-attachments/assets/edeb2d79-2fab-4afe-8574-90bb987f0f03" />

---

# STEP 14 - Configure Sealed Secrets

## Add Helm Repository

```bash
helm repo add sealed-secrets \
https://bitnami-labs.github.io/sealed-secrets

helm repo update
```

---

## Install Sealed Secrets Controller

```bash
helm install sealed-secrets-controller \
sealed-secrets/sealed-secrets \
-n kube-system
```

---

## Install Kubeseal CLI

```bash
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.25.0/kubeseal-0.25.0-linux-amd64.tar.gz

tar -xvzf kubeseal-0.25.0-linux-amd64.tar.gz

sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

---

## Create Kubernetes Secret

```bash
kubectl create secret generic db-secret \
--from-literal=username=admin \
--from-literal=password=admin123 \
--dry-run=client -o yaml > secret.yaml
```

---

## Encrypt Secret

```bash
kubeseal \
--controller-name=sealed-secrets-controller \
--controller-namespace=kube-system \
-o yaml < secret.yaml > sealed-secret.yaml
```

---

## Apply Sealed Secret

```bash
kubectl apply -f sealed-secret.yaml
```
<img width="664" height="214" alt="Screenshot 2026-05-06 233029" src="https://github.com/user-attachments/assets/2ac86bde-dce8-41d7-bc5c-31c276e8ddb2" />
<img width="1107" height="176" alt="Screenshot 2026-05-06 233100" src="https://github.com/user-attachments/assets/64bdb539-12b1-41ec-913d-f50f184c3d51" />

---

# STEP 15 - Configure Environment Overlays

## Create Overlay Directories

```bash
mkdir -p overlays/dev
mkdir -p overlays/staging
mkdir -p overlays/prod
```

---

## DEV Overlay

Create:

```bash
vim overlays/dev/kustomization.yaml
```

Paste:

```yaml
resources:
  - ../../apps/nginx

patchesStrategicMerge:
  - replica-patch.yaml
```

---

## DEV Replica Patch

Create:

```bash
vim overlays/dev/replica-patch.yaml
```

Paste:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-app

spec:
  replicas: 2
```

---

## STAGING Replica Patch

```yaml
spec:
  replicas: 3
```

---

## PROD Replica Patch

```yaml
spec:
  replicas: 5
```

---

# STEP 16 - Configure ApplicationSet

## Create Namespaces

```bash
kubectl create namespace dev

kubectl create namespace staging

kubectl create namespace prod
```

---

## Create ApplicationSet File

```bash
vim applicationset.yaml
```

Paste:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet

metadata:
  name: nginx-appset
  namespace: argocd

spec:
  generators:
    - list:
        elements:
          - env: dev
            namespace: dev

          - env: staging
            namespace: staging

          - env: prod
            namespace: prod

  template:
    metadata:
      name: '{{env}}-nginx'

    spec:
      project: default

      source:
        repoURL: https://github.com/reddijagadeesh9/Project-5-gitops-argocd-repo.git
        targetRevision: HEAD
        path: overlays/{{env}}

      destination:
        server: https://kubernetes.default.svc
        namespace: '{{namespace}}'

      syncPolicy:
        automated:
          prune: true
          selfHeal: true

        syncOptions:
          - CreateNamespace=true
```

---

## Apply ApplicationSet

```bash
kubectl apply -f applicationset.yaml
```
<img width="1051" height="191" alt="Screenshot 2026-05-07 000553" src="https://github.com/user-attachments/assets/d1f37494-f82c-4902-820e-49a562e93240" />

<img width="1705" height="713" alt="Screenshot 2026-05-07 001005" src="https://github.com/user-attachments/assets/5f893c3b-2762-441f-94a4-6e64bd90dbce" />


---

# STEP 17 - Configure Sync Window Policy

## Create Sync Window File

```bash
vim project-sync-window.yaml
```

Paste:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject

metadata:
  name: production-project
  namespace: argocd

spec:
  destinations:
    - namespace: '*'
      server: '*'

  sourceRepos:
    - '*'

  syncWindows:
    - kind: deny
      schedule: '0 9 * * *'
      duration: 8h
      applications:
        - prod-nginx
      manualSync: true
```

---

## Apply Project

```bash
kubectl apply -f project-sync-window.yaml
```
<img width="1170" height="129" alt="Screenshot 2026-05-07 001132" src="https://github.com/user-attachments/assets/c86f2522-43a1-41a1-8a97-1c2a55f035e6" />

<img width="1536" height="645" alt="Screenshot 2026-05-07 001255" src="https://github.com/user-attachments/assets/abb73199-0ab4-4791-9cc6-dea4d118b97e" />

<img width="1629" height="528" alt="Screenshot 2026-05-07 001307" src="https://github.com/user-attachments/assets/51eb953b-6ba3-4b7d-8417-23efa4799b78" />

---

# Verification Commands

## Check Applications

```bash
kubectl get applications -n argocd
```

## Check Pods

```bash
kubectl get pods -A
```

## Check Deployments

```bash
kubectl get deployments -A
```

## Check Services

```bash
kubectl get svc -A
```
<img width="1073" height="382" alt="Screenshot 2026-05-07 001349" src="https://github.com/user-attachments/assets/00a13ee8-2ecc-4e19-9052-ad845ebec771" />

---

# GitOps Workflow

```text
Developer Pushes Code
        ↓
GitHub Repository Updated
        ↓
GitHub Actions Triggered
        ↓
ArgoCD Detects Changes
        ↓
Cluster Automatically Synced
        ↓
Application Updated
```

---

# Multi-Environment Deployment

| Environment | Namespace | Replicas |
| ----------- | --------- | -------- |
| DEV         | dev       | 2        |
| STAGING     | staging   | 3        |
| PROD        | prod      | 5        |

---
DEV
<img width="1658" height="865" alt="Screenshot 2026-05-07 001739" src="https://github.com/user-attachments/assets/948d2212-10d5-422f-bf8f-0be21c8dec20" />

STAGIGN
<img width="1642" height="819" alt="Screenshot 2026-05-07 001829" src="https://github.com/user-attachments/assets/1bf13e7f-f72b-4118-bb14-ee0c9d728919" />

PROD
<img width="1634" height="849" alt="Screenshot 2026-05-07 001810" src="https://github.com/user-attachments/assets/fd02006a-5797-4818-9608-987e5d7267fa" />


# Project Outcome

Successfully implemented a production-style GitOps workflow using ArgoCD with automated synchronization, drift correction, self-healing, multi-environment deployment strategy, GitHub Actions integration, Sealed Secrets setup, Kustomize overlays, and ApplicationSet automation.

---

# Author

REDDI JAGADEESWARA RAO

```
```
