# 🚀 End-to-End Flask App Deployment on Kubernetes (Kind) using Helm & Argo CD + Image Updater

A fully automated **GitOps** pipeline where a new Docker image (e.g. `1.0.0 → 1.0.1`) is
automatically detected, committed to GitHub, and deployed to Kubernetes — **without any
manual step**.

> Docker Hub → GitHub → Kubernetes, all connected automatically via Argo CD + Argo CD Image Updater.

---

## 📌 Table of Contents

1. [Objective](#-objective)
2. [Problem Statement](#-problem-statement)
3. [Industry Use Case](#-industry-use-case-as-a-devops-engineer)
4. [Architecture](#-architecture)
5. [How Argo CD Auto-Sync Works](#-how-argo-cd-auto-sync-works)
6. [Argo CD UI vs Argo CD YAML (GitOps)](#-argo-cd-ui-vs-argo-cd-yaml-gitops)
7. [Argo CD Image Updater — Kaise Kaam Karta Hai](#-argo-cd-image-updater--kaise-kaam-karta-hai-auto-workflow)
8. [Project Structure](#-project-structure)
9. [Components Overview](#-components-overview)
10. [Step-by-Step Implementation](#-step-by-step-implementation)
11. [Manifests](#-manifests)
12. [Configuring Auto Git Commits](#-configuring-argo-cd-image-updater-for-auto-git-commits)
13. [Automated Workflow (Real Example)](#-automated-workflow-real-example)
14. [Commands Summary (Cheat Sheet)](#-commands-summary-cheat-sheet)
15. [Extra Tip: Why UI Refresh Lagta Hai](#-extra-tip-why-you-need-to-refresh-the-argocd-ui)
16. [Conclusion](#-conclusion)

---

## 🎯 Objective

Ek aisa **GitOps workflow** implement karna jahan:

- Docker image ka naya version Docker Hub par push hote hi automatically detect ho jaye
- Uska naya tag GitHub repo mein automatically commit ho jaye
- Argo CD us change ko dekh kar Kubernetes cluster ko automatically update kar de
- Koi bhi insaan manually kuch na kare — pura process **self-healing** aur **audit-trail** ke sath ho

---

## ❗ Problem Statement

Zyada tar companies mein deployment aur version update **manually** hota hai:

- Developer naya image Docker Hub par push karta hai
- Ops team ko manually YAML file mein image tag update karna parta hai
- Isse human error, delay, aur configuration drift hoti hai
- Multiple environments (dev/staging/prod) manage karna aur bhi mushkil ho jata hai

**Solution:** Argo CD + Argo CD Image Updater ka combination — jo pura process automate kar deta hai.

---

## 🏢 Industry Use Case (as a DevOps Engineer)

| # | Kya Hota Hai |
|---|---|
| 1 | **Automated Deployment** — Git mein koi bhi change (config ya version) automatically cluster par deploy ho jata hai |
| 2 | **Continuous Image Updates** — naya image Docker Hub par aate hi manifest Git mein khud update ho jata hai |
| 3 | **Security & Compliance** — latest patches automatically apply hote hain, vulnerabilities kam hoti hain |
| 4 | **Version Control & Audit Trail** — har deployment Git commit history mein record hota hai |
| 5 | **Scalability** — same workflow multiple services/environments par repeat ho sakta hai |
| 6 | **Operational Efficiency** — manual kaam khatam, rollback fast, release cycle tez |

---

## 🏗 Architecture

```
Developer  →  Docker Hub  →  Argo CD Image Updater  →  GitHub Repo  →  Argo CD  →  Kubernetes Cluster
 (pushes        (stores          (detects new             (commits          (auto-syncs      (running
  new image)     image)           image tag)               new tag)          desired state)    app)
```

---

## 🔄 How Argo CD Auto-Sync Works

```
   Git (source of truth)
        │  1. Git is the source of truth
        ▼
   ┌─────────────┐
   │   Argo CD   │──── 3. Compare desired vs actual state ────►  Kubernetes Cluster
   └─────────────┘                                                    │
        │                                                             │
        │ 4. Auto-sync                                 5. Verification &
        ▼                                                  health checks
   ✅ Synced  ◄──────────────────────────────────────────────────────┘
```

| Concept | Explanation |
|---|---|
| 🧩 GitOps Automation | Argo CD continuously watches your Git repo and auto-updates the cluster whenever a change (new image tag or Helm value) is pushed. |

---

## 🖥 Argo CD UI vs Argo CD YAML (GitOps)

### 1️⃣ Argo CD UI (Web Interface)

**Description:** Web dashboard to manage apps, view sync status & logs.

✅ Pros:
- Easy & visually intuitive
- Good for monitoring/quick manual sync or rollback
- Great for demos/small projects

❌ Cons:
- Manual changes break GitOps consistency
- Hard to track history
- Not suitable for production automation

### 2️⃣ Argo CD YAML (GitOps) — **Recommended**

**Description:** Applications defined as YAML in Git; Argo CD syncs cluster to match Git.

✅ Pros:
- Fully version controlled
- Reproducible clusters
- CI/CD-friendly & automatable
- Meets audit/compliance needs

❌ Cons:
- Slightly complex initial setup
- Requires Git workflow knowledge

### 3️⃣ Industry Best Practice

> **Rule of Thumb:** `Git → Argo CD → Kubernetes`
> UI is only for **observation** or **emergency manual override** — never for permanent production changes.

---

## 🤖 Argo CD Image Updater — Kaise Kaam Karta Hai (Auto Workflow)

Ye tool hai jo **container registry (Docker Hub)** ko continuously monitor karta hai aur
naye image version milte hi Git repo ko khud update kar deta hai — bina kisi manual YAML edit ke.

### Step-by-step (bilkul simple zubaan mein):

1. **Watch** – Image Updater har kuch der baad Docker Hub check karta hai ke koi naya
   image tag (jaise `1.0.0 → 1.0.1`) aaya hai ya nahi.
2. **Detect** – Naya tag milte hi, ye configured **update strategy** (jaise `semver`) ke
   through decide karta hai ke ye version deploy karne layak hai ya nahi.
3. **Update in Git** – Ye khud GitHub repo mein jaake manifest (values.yaml/deployment.yaml)
   ka image tag update karta hai aur ek naya **commit** push karta hai.
4. **Argo CD Detects Change** – Argo CD Git repo ko continuously watch kar raha hota hai,
   naya commit dikhte hi wo "OutOfSync" state show karta hai.
5. **Auto-Sync** – `syncPolicy.automated` on hone ki wajah se Argo CD khud hi cluster ko
   naye desired state (naya image) ke sath sync/deploy kar deta hai.
6. **Self-Heal + Verification** – Agar koi manually cluster mein change karde, Argo CD use
   wapas Git ke desired state par le aata hai (self-heal), aur health check bhi karta hai.

### Is process ke 3 core steps (short version):

```
1. Monitor registry  →  2. Update Git manifest  →  3. Argo CD auto-deploys
```

### Key Benefits:
- ❌ Koi manual YAML edit nahi
- ✅ Sab kuch Git ke through hi flow karta hai (GitOps-friendly)
- ✅ Versioning policies support karta hai (`latest`, semantic versioning `semver`)

---

## 🧱 Project Structure

```
flask-app/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml          # optional
└── argocd-application.yaml
```

---

## ⚙️ Components Overview

| Component | Description |
|---|---|
| Flask Todo App | Sample Python app, Dockerized |
| Kubernetes (KIND) | Target cluster jahan app deploy hota hai |
| Argo CD | GitOps CD tool jo Kubernetes ko Git ke sath sync karta hai |
| Argo CD Image Updater | Naye image tags detect karke Git manifests update karta hai |
| GitHub | GitOps repo jahan manifests store hain |
| Docker Hub | Container registry jahan naye image versions push hote hain |

---

## ⚙️ Step-by-Step Implementation

### 1. Install Docker & KIND

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
```

### 2. Install kubectl & KIND

```bash
# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
kubectl version --client

# KIND
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

### 3. Kind Cluster Config

1 control-plane + 2 worker nodes, with ports 80 & 443 mapped:

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.28.0
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
  - role: worker
    image: kindest/node:v1.28.0
  - role: worker
    image: kindest/node:v1.28.0
```

```bash
kind create cluster --name helm-argocd --config=kind-config.yaml
kubectl cluster-info
kubectl get nodes
```

### 4. Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### 5. Clone Repo & Install via Helm

```bash
git clone 
helm create flask-app
helm install flask-app . --namespace dev --create-namespace

# if needed:
helm uninstall flask-stack --namespace default
```

### 6. Install & Configure Argo CD

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Argo CD CLI
curl -sSL -o argocd-linux-amd64 \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
argocd version
```

### 7. Run Argo CD Server & Login

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address 0.0.0.0

# Get initial admin password
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
# Username: admin

argocd login localhost:8080 --username admin --password <ARGOCD_ADMIN_PASSWORD> --insecure
```

### 8. Install Argo CD Image Updater

```bash
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml

# OR via Helm
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd-image-updater argo/argocd-image-updater -n argocd
```

Verify it's running:

```bash
kubectl get pods -n argocd | grep image
# OR
kubectl -n argocd get pods -l app.kubernetes.io/name=argocd-image-updater
```

Expected: `argocd-image-updater-xxxxx   1/1   Running` ✅

---

## 🧩 Manifests

### 1️⃣ Deployment Annotations (Image Updater Config)

Add these annotations in your app's `deployment.yaml`:

```yaml
annotations:
  # Which image to watch
  argocd-image-updater.argoproj.io/image-list: "{{ .Chart.Name }}={{ .Values.image.repository }}"

  # Use semantic versioning strategy (1.0.0 → 1.1.0)
  argocd-image-updater.argoproj.io/{{ .Chart.Name }}.update-strategy: "semver"

  # Allow 'latest' or semver tags
  argocd-image-updater.argoproj.io/{{ .Chart.Name }}.allow-tags: "regexp:^(latest|\\d+\\.\\d+\\.\\d+)$"

  # Commit updated tag back to Git (not directly to cluster)
  argocd-image-updater.argoproj.io/write-back-method: git

  # Branch to commit to
  argocd-image-updater.argoproj.io/git-branch: main
```

### 2️⃣ Argo CD Application (`argocd-application.yaml`)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: flask-todo
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: "flask-todo=misacademy/flask-todo"
    argocd-image-updater.argoproj.io/flask-todo.update-strategy: "semver"
    argocd-image-updater.argoproj.io/flask-todo.allow-tags: "regexp:^v?\\d+\\.\\d+\\.\\d+$"
    argocd-image-updater.argoproj.io/write-back-method: git
    argocd-image-updater.argoproj.io/git-branch: main
spec:
  project: default
  source:
    repoURL: https://github.com/umair1012/flask-todo-argo.git
    path: .
    targetRevision: main
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply it:

```bash
kubectl apply -f argocd-application.yaml
```

---

## 🔐 Configuring Argo CD Image Updater for Auto Git Commits

### 1. Create GitHub Personal Access Token

- Go to: `https://github.com/settings/tokens`
- Scopes required: ✅ `repo`, ✅ `read:packages`, ✅ `write:packages`
- Copy the token (starts with `ghp_...`)

### 2. Create Secret in Kubernetes

```bash
kubectl -n argocd create secret generic git-creds \
  --from-literal=username=<your-github-username> \
  --from-literal=password=<your-github-personal-access-token>
```

### 3. Add Repo to Argo CD

```bash
argocd repo add https://github.com/umair1012/flask-todo-argo.git \
  --username <your-username> --password <your-personal-access-token>
```

### 4. Update Image Updater ConfigMap

```yaml
# argocd-image-updater-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-image-updater-config
  namespace: argocd
data:
  registries.conf: |
    registries:
      - name: Docker Hub
        api_url: https://registry-1.docker.io
        prefix: docker.io
        default: true
        credentials: secret:dockerhub-creds   # optional, for private repos

  git.commit.message.template: |
    [ArgoCD Image Updater] Updated {{.AppName}} image to {{.ImageUpdated}}

  git.write-back.method: git
  git.creds: secret:github-creds
```

```bash
kubectl apply -f argocd-image-updater-configmap.yaml
```

### 5. Add Image Updater Custom Resource

```yaml
# image-updater-cr.yaml
apiVersion: argocd-image-updater.argoproj.io/v1alpha1
kind: ImageUpdater
metadata:
  name: default
  namespace: argocd
spec:
  namespace: argocd
  applicationRefs:
    - namePattern: "*"       # watch all apps
  images:
    - alias: flask-todo
      imageName: misacademy/flask-todo:v*   # track any tag starting with v
```

```bash
kubectl apply -f image-updater-cr.yaml
```

### 6. Restart & Verify

```bash
kubectl -n argocd rollout restart deployment argocd-image-updater
# OR
kubectl -n argocd rollout restart deployment argocd-image-updater-controller

# Check logs
kubectl -n argocd get pods | grep image-updater
kubectl -n argocd logs deployment/argocd-image-updater -f

# Check application health
kubectl get applications -n argocd
```

---

## 🔁 Automated Workflow (Real Example)

```
1️⃣ Developer builds & pushes new image:
     umair1012/flask-todo-app:1.0.1  →  Docker Hub

2️⃣ Argo CD Image Updater detects new tag
     → commits "1.0.1" update to GitHub repo

3️⃣ Argo CD detects Git change
     → auto-syncs & deploys updated container to Kubernetes

4️⃣ Flask app is updated automatically
     → zero manual deployment steps
```

Ye exactly wohi pipeline hai jo real-world Kubernetes-based companies use karti hain jahan
reliability, automation aur traceability critical hoti hai.

---

## 🧩 Commands Summary (Cheat Sheet)

| Step | Command |
|---|---|
| Install Argo CD | `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` |
| Install Image Updater | `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml` |
| Apply Application | `kubectl apply -f argocd-application.yaml` |
| Access Argo CD UI | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |

---

## 💡 Extra Tip: Why You Need to Refresh the Argo CD UI

**1. Argo CD UI live-stream nahi karta**
Argo CD ka web UI React-based frontend hai jo API ko **periodically poll** karta hai —
real-time WebSocket connection nahi hai. Isliye Git commit ya sync status turant nahi dikhta,
jab tak aap manually refresh na karein ya background polling (usually 3–5 min) na ho.

**2. Backend (`argocd-server`) turant update ho jata hai**
Aapka actual sync **real-time** hota hai — pods, deployments sab turant update ho jate hain.
Bas UI ko naya state dikhane ke liye agla poll ya browser refresh chahiye hota hai.

> ✅ Matlab: Sync already ho chuka hota hai, sirf UI par dikhne mein thoda time lagta hai.

---

## 🧠 Conclusion

Argo CD + Argo CD Image Updater ko integrate karke humne ek **hands-free CI/CD pipeline**
Kubernetes ke liye achieve kiya hai. Ye workflow:

- Manual updates khatam karta hai
- Human error kam karta hai
- Deployment ko consistent, traceable aur automated banata hai
- Modern GitOps best practices ke sath align hota hai

Ye setup **production-grade DevOps automation** ke liye ek reusable template ki tarah
kaam karta hai — kisi bhi microservice-based architecture ke liye applicable.

---
