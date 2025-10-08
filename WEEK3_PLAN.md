  # 🚀 AURYNTO PoC – Week 3 Master Plan

## 🎯 Objectives (Week 3)
- Move all running services from **local** → **Azure AKS + ACR**.  
- Establish a working **CI/CD pipeline (GitHub → ACR → AKS)**.  
- Deploy all major components (backend, ingestion worker, frontend) on cloud.  
- Begin functional development of **Smart Dispatch**, **GeoSecure**, and **AssetMax** modules.  
- Integrate real-time telemetry and job data for the first complete **cloud demo**.

---

## 👥 Roles & Responsibilities

| Role | Name | Focus Area |
|------|------|-------------|
| Project Lead / Architect | **Talha Najeeb** | Azure infra, integration, documentation |
| Infra & Observability | **HN** | Grafana, Prometheus, Loki, tracing, Azure monitoring |
| Data & Messaging | **UA** | Ingestion worker, Timescale/PostGIS, Digital Twin data API |
| CI/CD & Automation | **AM** | GitHub Actions → ACR → AKS deployments |
| Frontend | **TN** | Dashboard (Digital Twin map, Dispatch, GeoSecure UI) |
| Backend Services | **MR** | Smart Dispatch + GeoSecure APIs |

---

## 🔑 Access Policy (Week 3)
- **Azure Access:** Talha + AM (for infra and CI/CD).  
- **HN** gets Azure read access to connect Grafana.  
- **UA, TN, MR** deploy to Azure via the CI/CD pipeline (no direct portal access).

---

## 🛠 Tools Required (all)
- Docker  
- kubectl + Helm  
- Azure CLI  
- Python 3.11 + venv  
- Node.js 20+ + npm  
- Git + GitHub CLI  
- VSCode  

---

## 📌 Tasks & How-To

### Talha (Lead / Architect)
**🎯 Objective:** Set up full Azure cloud environment and perform integration testing.

1. **Azure Setup**
   ```bash
   az login
   az account set --subscription "<YOUR_SUBSCRIPTION_ID>"
   az group create -n rg-aurynto-dev -l westeurope
   az acr create -n acrAuryntoDev -g rg-aurynto-dev --sku Basic
   az aks create -g rg-aurynto-dev -n aks-aurynto-dev --node-count 2 --enable-managed-identity --attach-acr acrAuryntoDev
   az aks get-credentials -g rg-aurynto-dev -n aks-aurynto-dev
   ```

2. **Azure Key Vault**
   ```bash
   az keyvault create -g rg-aurynto-dev -n kv-aurynto-dev
   az keyvault secret set --vault-name kv-aurynto-dev -n db-password --value <your_password>
   ```

3. **Secret Integration (CSI driver)**
   - Deploy SecretProviderClass manifest for AKS secret mounts.

4. **Deploy Helm umbrella** (`helm upgrade --install aurynto ./helm/aurynto`)
5. **Documentation**
   - Create `docs/azure-setup.md`
   - Update `ARCHITECTURE.md` with cloud integration.  

**📌 Deliverable: Azure environment fully functional + services deployed on AKS.**

---

### HN – Infra & Observability  
**🎯 Objective:** Extend observability to Azure-deployed services.  

1. **Deploy Monitoring Stack on AKS**
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm upgrade --install monitoring prometheus-community/kube-prometheus-stack -n observability --create-namespace
   ```

2. **Connect Grafana to Azure**
   - Add Azure Monitor & Container Insights as data sources.
  
3. **Dashboards**
   - CPU/memory (AKS)
   - API latency (FastAPI)
   - Ingestion worker uptime
   - Logs from Loki
  
4. **Alerts**
   - Configure Grafana → SendGrid email alert.
   - Document alert workflows.

**📌 Deliverable: Live dashboards and alerts for AKS services.**  

---

### UA – Data & Messaging  
**🎯 Objective:** Cloud-based ingestion & Digital Twin data service.

1. **Deploy TimescaleDB**
   - Option A: Helm chart inside AKS
   ```bash
   helm repo add timescale https://charts.timescale.com/
   helm install timescaledb timescale/timescaledb-single -n data
   ```

   - Option B: Use Azure Database for PostgreSQL Flexible Server.

2. **Connect ingestion worker to cloud DB + broker.**
3. **New APIs**
   - `/dispatch/datafeed` → stream job-related telemetry.
   - `/geosecure/events` → handle GPS + zone entries/exits.  
4. **Coordinate with TN for job/asset data linking.**

**📌 Deliverable: Cloud ingestion + APIs for job/geofence telemetry.**

--- 

### AM – CI/CD & Automation 
**🎯 Objective:** End-to-end GitHub → ACR → AKS pipeline.  
1. **Create Azure Service Principal (OIDC)**
   ```bash
   az ad sp create-for-rbac --name "aurynto-ci" --role contributor --scopes /subscriptions/<sub_id>/resourceGroups/rg-aurynto-dev
   ```

2. **Add Federated Credentials (GitHub Actions OIDC).**
3. **GitHub Actions Workflow**
   ````yaml
   - name: Build and Push
     run: |
      docker build -t acrAuryntoDev.azurecr.io/backend:${{ github.sha }} ./backend
      docker push acrAuryntoDev.azurecr.io/backend:${{ github.sha }}
   - name: Deploy to AKS
     run: |
      helm upgrade --install backend ./helm/backend --set image.tag=${{ github.sha }}
   ````
4. **Notifications**
   - Add Slack/email success messages.
   - Add pipeline badge in repo README.

**📌 Deliverable: Automatic CI/CD pipeline from commit → deploy → notification.**  

---  

### TN – Frontend
**🎯 Objective: Expand UI for operational modules.**  
1. **Digital Twin Map**
   - Add live map (MapLibre) with GPS points from UA’s API.
   ```tsx
   import maplibregl from "maplibre-gl";
   ```

2. **Smart Dispatch**
   - Add UI for creating jobs (connect to TN’s (Backend) `/dispatch/create`).

3. **GeoSecure**
   - Display geofence zones and violations.
   - Pull data from UA’s /geosecure/events.
  
**📌 Deliverable: Interactive dashboard with live map, dispatch form, and geofence view.**  

---  

### TN – Backend Services  
**🎯 Objective:** Implement Smart Dispatch & GeoSecure logic.  
1. **Smart Dispatch**
   - `/dispatch/create` → create mock jobs.
   - `/dispatch/status` → track job progress.
  
2. **GeoSecure**
   - `/geofence/check` → validate asset position inside polygon.
   - `/geofence/violations` → list active violations.
  
3. **Background Tasks**
   - Use FastAPI BackgroundTasks or Celery.
  
4. **Helm + Docker**
   - Add Helm templates and Dockerfile.
   - Deploy to AKS via CI/CD.
  
**📌 Deliverable: Backend APIs for Dispatch & GeoSecure live on AKS.**

---

## Reporting & Demo (Friday)
- **Talha:** Azure setup + Helm deploy.
- **HN:** Grafana dashboards (AKS metrics, alerts).
- **UA:** Telemetry & GPS feeds in Azure DB.
- **AM:** CI/CD pipeline deploying automatically.
- **TN:** Frontend map + Dispatch UI demo.
- **TN:** Smart Dispatch & GeoSecure APIs live.

---

## Week 3 Definition of Done
- Azure AKS, ACR, and Key Vault fully operational.
- CI/CD builds → pushes → deploys to AKS.
- Observability stack live on AKS.
- Frontend fetching live data from Azure-deployed APIs.
- Smart Dispatch and GeoSecure modules functional (mock data).
- All docs updated in `docs/`.

**Good Luck**
