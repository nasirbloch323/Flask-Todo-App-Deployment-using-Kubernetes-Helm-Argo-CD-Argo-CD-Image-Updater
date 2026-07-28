# 🚀 End-to-End Flask Todo App Deployment using Kubernetes, Helm, Argo CD & Argo CD Image Updater

> Beginner-friendly GitOps guide.

## Overview

This project deploys a Flask Todo application on a local Kubernetes
(KIND) cluster using Helm. Argo CD watches the Git repository and keeps
the cluster synchronized. Argo CD Image Updater monitors Docker Hub for
new image tags, updates the Git repository automatically, and Argo CD
deploys the new version.

## Workflow

Developer → Docker Hub → Argo CD Image Updater → GitHub → Argo CD →
Kubernetes → Running Flask App

## Prerequisites

-   Docker
-   kubectl
-   KIND
-   Helm
-   Git
-   GitHub account
-   Docker Hub account

## Project Structure

``` text
flask-app/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── argocd-application.yaml
```

## Step 1 -- Install Docker

``` bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
```

This installs Docker and enables the service.

## Step 2 -- Install kubectl

``` bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

## Step 3 -- Install KIND

``` bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
kind version
```

## Step 4 -- Create Cluster

Create `kind-config.yaml` then:

``` bash
kind create cluster --name helm-argocd --config kind-config.yaml
kubectl get nodes
```

## Step 5 -- Install Helm

``` bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

## Step 6 -- Install Argo CD

``` bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Port forward:

``` bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address 0.0.0.0
```

Get admin password:

``` bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

## Step 7 -- Install Argo CD Image Updater

``` bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

## Step 8 -- Configure Application

Update `argocd-application.yaml` with your GitHub repository, branch,
namespace and Helm chart.

## Step 9 -- Configure GitHub Token

Create a GitHub Personal Access Token with `repo` permission and create
a Kubernetes secret.

## Step 10 -- Deploy

``` bash
kubectl apply -f argocd-application.yaml
```

## Testing Auto Updates

1.  Build a new Docker image.
2.  Push it to Docker Hub.
3.  Image Updater detects the new tag.
4.  Git repository is updated.
5.  Argo CD syncs automatically.
6.  Kubernetes runs the latest version.

## Troubleshooting

-   `kubectl get pods -A`
-   `kubectl get applications -n argocd`
-   `kubectl logs deployment/argocd-image-updater -n argocd`
-   `argocd app sync <app-name>`

## Conclusion

This project demonstrates a complete beginner-friendly GitOps pipeline
using Kubernetes, Helm, Argo CD and Argo CD Image Updater. All
deployments are version-controlled, repeatable and automated.
