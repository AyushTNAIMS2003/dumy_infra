# 🏗️ AURYNTO Infrastructure (PoC) — Guide 1

## 📅 Development Plans
- [Week 1 Plan](docs/WEEK1_PLAN.md)

This doc captures **exactly what to set up**, **how to run it locally**, **what the team needs to know**, and **how to troubleshoot** common issues on Windows using **VS Code + GitHub Desktop + Docker Desktop**.

---

## 0) What this repo does

- Holds a **Helm chart** that deploys:
  - **Frontend** (nginx serving our `public/index.html` static page)
  - **Backend** (image placeholder wired, you can enable later)
- Includes a **GitHub Actions CI** workflow (`infrastructure-ci`) that **lints the chart on PRs**.
- Designed to run locally on a **kind** Kubernetes cluster (Kubernetes-in-Docker) with **port-forwarding** for access during the PoC.

---

## 1) Repository layout

```text
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

```  

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
Then:  

```powershell 

kind delete cluster --name aurynto
kind create cluster --name aurynto --config kind-aurynto.yaml
kubectl create namespace aurynto
# recreate ghcr-cred if needed
helm upgrade --install aurynto .\helm\aurynto -n aurynto
# visit http://localhost:30080

```

---

## 5) CI for Infrastructure (helm lint on PRs)  

```.github/workflows/helm-ci.yaml:```  

```  

name: infrastructure-ci
on:
  pull_request:
    branches: [ "main" ]
    paths: [ "helm/**" ]
permissions:
  contents: read
jobs:
  helm-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/setup-helm@v4
      - run: helm lint ./helm/aurynto

```  

**Branch protection / required checks (Team plan)**  

- Use **repo-level rulesets** (Team plan doesn’t include org-wide rulesets for private repos).  

- In the repo: **Settings** → **Rules** → **Rulesets** → Require:  

    - Pull requests

    - 1 approval

    - **Required status checks** → **Add from recent runs**
    **Select exactly:** ```infrastructure-ci / helm-lint (pull_request)```

⚠️ Avoid **mismatch** (“Expected — waiting” forever) by selecting checks from recent runs (exact string). Remove old/generic entries like just ```infrastructure-ci```.  

---

## 6) Team workflow (GitHub Desktop + VS Code)  

**For any change (values, templates, CI):**  

1. GitHub Desktop → **Current Branch** → **New Branch** (e.g., `update-frontend-service`)

2. Edit files in VS Code

3. GitHub Desktop → **Commit to <branch>** → **Push origin**

4. **Create Pull Request**

6. Wait for infrastructure-ci (lint) to pass → review → Merge pull request

Branch protection error `GH013: Changes must be made through a pull request`. means you tried to push to `main`. Create a branch & PR instead.  

---

## 7) Verify the deployment  

From inside the cluster:  

```powershell 

# HTML should print if service is reachable in cluster DNS
kubectl -n aurynto run curl --rm -it --image=curlimages/curl --restart=Never -- curl -sS http://frontend:80

```  

From your **host** (with port-forward running):  

- Open http://localhost:8080
 — you should see the AURYNTO Frontend placeholder page.

---

## 8) Common issues & quick fixes  

A) `ImagePullBackOff`  
    - Images are **private** and you didn’t set a pull secret:  
        1. Create `ghcr-cred` secret (see §3).

        2. In `values.yaml`:
        `imagePullSecrets: ["ghcr-cred"]`

        3. `helm upgrade --install aurynto ...`  

B) **NodePort unreachable on Windows/kind** 

    - Use port-forwarding (PoC choice):
    `kubectl -n aurynto port-forward svc/frontend 8080:80`  

    - `Or recreate kind with extraPortMappings` (see §4).

C) **Helm namespace ownership error**  

“Namespace exists and cannot be imported… missing `app.kubernetes.io/managed-by: Helm` …”  

- **One-time fix**: label & annotate the `aurynto` namespace (see §4).  

**Better pattern** for later: don’t template the namespace; use `--create-namespace`.  

D) **VS Code terminal can’t find `helm` but PowerShell can**  

- VS Code was opened **before** PATH changed. Close all VS Code windows and re-open.  

- Ensure `terminal.integrated.inheritEnv` is true.  

- As a session workaround:
    `$env:Path = "$env:Path;C:\Program Files\helm"`  

D) **Required check stuck at “Expected — waiting”**  

- In repo ruleset, remove mismatched checks.

- **Add from recent runs** the exact context name:
    `infrastructure-ci / helm-lint (pull_request)`

- If locked, temporarily unrequire checks → merge → re-enable with the correct name.  

---

## 9) Day-to-day commands (cheat sheet)  

```powershell 

# Create cluster
kind create cluster --name aurynto

# Namespace & secret
kubectl create namespace aurynto
kubectl create secret docker-registry ghcr-cred `
  --namespace aurynto `
  --docker-server=ghcr.io `
  --docker-username=YOUR_GITHUB_USERNAME `
  --docker-password=YOUR_GHCR_PAT

# Deploy / update
helm upgrade --install aurynto .\helm\aurynto -n aurynto

# Check status
kubectl get pods -n aurynto
kubectl get svc  -n aurynto
kubectl logs deploy/frontend -n aurynto

# Port-forward for browser access
kubectl -n aurynto port-forward svc/frontend 8080:80

# Cleanup
helm uninstall aurynto -n aurynto
kind delete cluster --name aurynto

```

## 10) What to do next  

- **Backend:** enable backend deployment, add probes/env/config as needed.

- **Frontend:** replace placeholder with your real build (update the image).

- **CI:** keep `infrastructure-ci` required in repo rules.

- **(Later) Azure AKS:** reuse this Helm chart; add a deploy workflow and AKS credentials.

---

## TL;DR for the team  

1. Install Docker, VS Code, kind, kubectl, Helm.

2. `kind create cluster --name aurynto` → `kubectl create namespace aurynto`.

3. If images are private, create `ghcr-cred` and set `imagePullSecrets`.

4. `helm upgrade --install aurynto .\helm\aurynto -n aurynto`.

5. `kubectl -n aurynto port-forward svc/frontend 8080:80` → open http://localhost:8080
.

6. Changes go via **branch** → **PR** → **CI green** → **merge**.

---

Good Luck!