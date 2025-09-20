# 🏗️ AURYNTO Infrastructure (PoC) — End-to-End Guide

This doc captures **exactly what we set up**, **how to run it locally**, **what the team needs to know**, and **how to troubleshoot** common issues on Windows using **VS Code + GitHub Desktop + Docker Desktop**.

---

## 0) What this repo does

- Holds a **Helm chart** that deploys:
  - **Frontend** (nginx serving our `public/index.html` static page)
  - **Backend** (image placeholder wired, you can enable later)
- Includes a **GitHub Actions CI** workflow (`infrastructure-ci`) that **lints the chart on PRs**.
- Designed to run locally on a **kind** Kubernetes cluster (Kubernetes-in-Docker) with **port-forwarding** for access during the PoC.

---

## 1) Repository layout

aurynto-infrastructure/
├─ helm/
│ └─ aurynto/
│ ├─ Chart.yaml
│ ├─ values.yaml
│ ├─ .helmignore
│ └─ templates/
│ ├─ backend-deployment.yaml
│ ├─ backend-service.yaml
│ ├─ frontend-deployment.yaml
│ ├─ frontend-service.yaml
│ ├─ ingress.yaml # optional, disabled by default
│ └─ NOTES.txt
└─ .github/
└─ workflows/
└─ helm-ci.yaml # lints the chart on pull requests  

**Key configurable bits** are in `values.yaml`:
- `serviceType: NodePort` (for local kind)
- `frontend.image`, `backend.image`
- `imagePullSecrets: []` (set to `["ghcr-cred"]` for private GHCR)

---

## 2) Prereqs (Windows 10/11)

Install once:

1. **Docker Desktop** — start it and leave it running
2. **VS Code**
3. **GitHub Desktop** (optional but recommended for Git workflow)
4. **kubectl**  
   - Download `kubectl.exe` and place it in a folder on PATH (e.g., `C:\Program Files\kubectl\`)
5. **kind**  
   - Download `kind.exe` → put in `C:\Program Files\kind\` → add that folder to **PATH**
6. **Helm**  
   - Download `helm.exe` → put in `C:\Program Files\helm\` → add to **PATH**

> After editing PATH, **restart VS Code** (its terminal won’t see new PATH until you reopen).  
> Verify each tool:
```powershell
> docker version
> kubectl version --client
> kind --version
> helm version  
```
---

## 3) Local Kubernetes (kind) — create & prepare cluster

```powershell

# Create cluster
kind create cluster --name aurynto

# Create namespace for the app
kubectl create namespace aurynto

```

---

## Private GHCR images? Create a pull secret  

Use a **GitHub Personal Access Token (PAT)** with read:packages only.

1. Create PAT: GitHub → Profile → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens** (classic)
→ **Generate new token (classic)** → name it (e.g., `docker-ghcr`), scope read:packages → **Generate** → copy it.

2. Create secret in the cluster: 

```powershell

kubectl create secret docker-registry ghcr-cred `
  --namespace aurynto `
  --docker-server=ghcr.io `
  --docker-username=YOUR_GITHUB_USERNAME `
  --docker-password=YOUR_GHCR_PAT  

```

3. Tell the chart to use it (`values.yaml`):  
`imagePullSecrets: ["ghcr-cred"]`  

If your GHCR packages are **public**, you can leave `imagePullSecrets`: [].  

---

## 4) Deploy with Helm (PoC uses port-forwarding)  

Install/upgrade the release into the aurynto namespace:  

```powershell

helm upgrade --install aurynto .\helm\aurynto -n aurynto

```

**Note:** If you ever created the aurynto namespace before installing and you included a namespace.yaml in the chart, Helm may error with “invalid ownership metadata”. Two fixes:  

- **Label the namespace once:**  

```powershell

kubectl label namespace aurynto app.kubernetes.io/managed-by=Helm --overwrite
kubectl annotate namespace aurynto meta.helm.sh/release-name=aurynto --overwrite
kubectl annotate namespace aurynto meta.helm.sh/release-namespace=aurynto --overwrite

```

- **Recommended pattern for future: Don’t template the Namespace. `Remove templates/namespace.yaml` and use `--create-namespace` on first install.**  

Check pods & services:  

```powershell  

kubectl get pods -n aurynto
kubectl get svc  -n aurynto  

```
You should see `frontend` **Running** and a `Service` named `frontend`.  

**Access via port-forwarding (our PoC choice)**  

```powershell 

kubectl -n aurynto port-forward svc/frontend 8080:80
# open http://localhost:8080

```

Why port-forward? On Windows with kind/WSL2, NodePort doesn’t always bind to localhost without extra config. Port-forward is simple and reliable for a PoC.

***(Optional) If you prefer NodePort on localhost, recreate kind with extra port mappings:***

```# kind-aurynto.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 30080
  - containerPort: 30081
    hostPort: 30081
```

