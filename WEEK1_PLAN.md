# 🚀 AURYNTO PoC – Week 1 Master Plan

## 🎯 Objectives (Week 1)
- Set up **core foundation** for the PoC.
- Ensure all team members have their **local environments ready**.
- Establish **repo structure, CI/CD pipeline, observability, data pipeline, frontend & backend skeletons**.
- Deploy **first end-to-end dummy demo** by end of Week 1.
- Document everything in `docs/` for repeatability.

---

## 👥 Roles & Responsibilities

| Role | Name | Focus Area |
|------|------|-------------|
| Project Lead / Architect | **Talha** | Azure infra, GitHub repos, Helm umbrella, overall integration |
| Hashir | Infra & Observability | AKS (later), Helm umbrella, Prometheus, Grafana, Loki |
| Ubaid | Data & Messaging | TimescaleDB schema, MQTT broker, ingestion worker |
| Ayush | CI/CD & Automation | GitHub Actions, OIDC, Helm deploy pipeline |
| Fresher D | Frontend | Next.js dashboard, QR mobile pages |
| Muneeb | Backend Services | FastAPI services (Digital Twin, Dispatch, GeoSecure) |

---

## 🔑 Week 1 Access Policy
- **Azure Access:** Only **Talha (Lead)** will access Azure in Week 1.  
- **Freshers:** Work on **local setups** (k3d, Docker, Next.js, FastAPI).  
- Azure access for others will be introduced from **Week 2 onwards**.

---

## 🛠 Tools to Install (All Freshers)
- **Docker** (for local DBs, brokers, containers)  
- **kubectl + Helm** (for k3d cluster deployments)  
- **Python 3.11 + venv** (for backend, ingestion worker)  
- **Node.js 20+ + npm** (for Next.js frontend)  
- **VSCode** (recommended editor)  
- **Git + GitHub Desktop/CLI**  

---

## 📌 Tasks & How-To (Detailed)

### Project Lead / Architect (Talha)
1. **Azure Setup**
   ```bash
   az login
   az account set --subscription "<SUBSCRIPTION_ID>"
   az group create -n rg-aurynto-dev -l westeurope
   az acr create -n acrAuryntoDev -g rg-aurynto-dev --sku Basic
   az aks create -g rg-aurynto-dev -n aks-aurynto-dev --node-count 2 --enable-managed-identity --attach-acr acrAuryntoDev
   az aks get-credentials -g rg-aurynto-dev -n aks-aurynto-dev
   ```

Verify: kubectl get nodes  

2. **Helm Umbrella Chart**
    ```bash
    helm create aurynto
    rm -rf aurynto/templates/*
    ```  

Add subcharts: ```backend```, ```frontend```, ```observability```  

3. **GitHub Setup**  
- Repos: `aurynto-infra`, `aurynto-services`, `aurynto-frontend`

- Add branch protection rules (`main`)  

4. **Docs**  
- `ARCHITECTURE.md` (diagram + explanation)

- `TEAM_GUIDE.md` (branching, commits, PR workflow) 

---

### Hashir – Infra & Observability  

- Install kubectl + Helm.
- Deploy monitoring stack:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack -n observability --create-namespace
```  
- Access Grafana:

```bash
kubectl get secret --namespace observability monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
kubectl port-forward svc/monitoring-grafana -n observability 3000:80
```  

Open http://localhost:3000
 → login → create CPU/memory dashboard.

- Deliverable: Grafana dashboard showing CPU/memory usage.  

- Document in `docs/observability-setup.md`.  

---

### Ubaid – Data & Messaging  
1. Run **Mosquitto**:  

```bash
docker run -it -p 1883:1883 eclipse-mosquitto
```  

2. Install **PostgreSQL** + **TimescaleDB**:  

```bash
docker run -d --name timescaledb -p 5432:5432 -e POSTGRES_PASSWORD=aurynto timescale/timescaledb:latest-pg15
```  
3. Create schema:  

```bash
CREATE TABLE telemetry (
  id SERIAL PRIMARY KEY,
  asset_id VARCHAR(50),
  timestamp TIMESTAMPTZ DEFAULT now(),
  speed NUMERIC,
  temperature NUMERIC
);
SELECT create_hypertable('telemetry', 'timestamp');
```  

4. Write **MQTT publisher** (`publisher.py`) → publishes random speed/temp.  

5. Write **ingestion worker** (`worker.py`) → inserts into TimescaleDB.

6. Push code → `aurynto-services/data-pipeline/`.

---

### Ayush – CI/CD & Automation  

1. Create **dummy FastAPI app** with `/health`.  
2. Write Dockerfile.  
3. Add GitHub Actions workflow  
(`.github/workflows/deploy.yml`):  

```bash
    name: CI/CD
    on:
    push:
        branches: [ "main" ]
    pull_request:
    jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v3
        - name: Build Docker image
            run: docker build -t auryn:${{ github.sha }} .
```  

4. Deliverable: Pipeline runs on every push/PR.  
5. Document in `docs/ci-cd.md`.  

---

### Tech D – Frontend  
1. Bootstrap project: 

```bash
npx create-next-app@latest auryn-frontend
cd auryn-frontend
npm install tailwindcss shadcn-ui @tanstack/react-query maplibre-gl
npx tailwindcss init -p
```  

2. Create basic layout (sidebar + nav).  
3. Add placeholder pages:
    - `/dashboard` → Digital Twin
    - `/dispatch` → Smart Dispatch
    - `/geosecure` → GeoSecure
    - `/assetmax` → AssetMax  

4. Commit skeleton to `aurynto-frontend`.  

---

### Muneeb – Backend Services  
1. Init FastAPI project:  

```bash
mkdir auryn-backend && cd auryn-backend
python -m venv venv && source venv/bin/activate
pip install fastapi uvicorn sqlalchemy psycopg2 alembic
```  

2. Create `main.py`:  

```bash
from fastapi import FastAPI
app = FastAPI()

@app.get("/assets")
def get_assets():
    return [{"id": "crane1", "status": "active"}]
```

3. Add Dockerfile.  

4. Add Helm template (`backend-deployment.yaml`, `backend-service.yaml`).  

5. Push to `aurynto-services/backend/`.  

---

### Reporting & Demo  

- **Daily Standup (15 min)** → Yesterday’s work, today’s goal, blockers.  
- Documentation → Each fresher writes a 2–3 page markdown in `docs/`.
- Sunday Demo:
    - Hashir: Grafana dashboard
    - Ubaid: MQTT → Timescale ingestion
    - Ayush: CI/CD pipeline run
    - Tech D / Talha: Next.js dashboard skeleton
    - Muneed: FastAPI endpoints  

---

### Week 1 Definition of Done  
- Azure skeleton (AKS, ACR, Key Vault) ready.
- GitHub repos + branch protection in place.
- Helm umbrella initialized.
- Observability stack running locally (Grafana dashboard live).
- MQTT → TimescaleDB pipeline functional.
- CI/CD pipeline triggers on PR/push.
- Frontend skeleton with placeholder pages.
- Backend FastAPI service running in Docker.
- Team docs available in docs/.