# AURYNTO PoC – Week 2 Plan

## 🎯 Objectives (Week 2)
- Move beyond skeletons → build **production-ready pipeline & observability**.  
- UA & HN are ahead, so they advance into **Week 3-level tasks** (Digital Twin APIs + tracing/SLOs).  
- Ensure **CI/CD pipeline is deploying to AKS**.  
- Deliver first **vertical slice demo**:
  Telemetry published → Ingestion → DB → API → Frontend → Observability.

---

## 👥 Roles & Responsibilities

| Role | Name | Focus Area |
|------|------|-------------|
| Project Lead / Architect | **Talha Najeeb** | Azure infra, integration, docs |
| Infra & Observability | **HN** | Prometheus, Grafana, Loki, tracing, alerts |
| Data & Messaging | **UA** | Timescale + PostGIS schema, ingestion worker, Digital Twin API |
| CI/CD & Automation | **AM** | GitHub Actions → ACR → AKS |
| Frontend | **TN** | Next.js dashboard + QR generator |
| Backend Services | **MR** | Assets & Jobs FastAPI service |

---

## 🔑 Access Policy (Week 2)
- **Azure Access:** Talha + AM (CI/CD deployments).  
- **HN, UA, TN, MR:** Develop locally → deploy via CI/CD.

---

## 🛠 Tools to Install (all)
- Docker  
- kubectl + Helm  
- Python 3.11 + venv  
- Node.js 20+ + npm  
- Git + GitHub CLI  
- VSCode  

---

## 📌 Tasks & How-To

### 🔹 Talha (Lead / Architect)
1. **Azure Key Vault**
   ```bash
   az keyvault create -g rg-aurynto-dev -n kv-aurynto-dev
   az keyvault secret set --vault-name kv-aurynto-dev -n db-password --value <your_password>
   ```  
2. **Integrate with AKS** using CSI SecretProviderClass.  
3. **Vertical Slice Test:** Publish telemetry → verify ingestion → DB → API → frontend → Grafana.  
4. **Docs:** Update `ARCHITECTURE.md` (new APIs, observability, QR flow) + add executive diagram.

---

### 🔹 HN – Infra & Observability  
**🎯 Objective:** Full-stack observability & SLOs  
1. **Distributed Tracing (new)**
   - Add **OpenTelemetry** to FastAPI + ingestion worker.
   - Forward traces to Grafana Tempo.
2. **SLO Dashboards**
   - Define Service Level Objectives (SLOs):
     - Telemetry ingestion latency < 2s
     - API uptime 99%
   - Build Grafana dashboards with these metrics.
3. **Alert → Notification Integration)**
   - Hook Grafana alerts to SendGrid (email) or dummy webhook.
   - Document setup.
**📌 Deliverable:** Updated `docs/observability-setup.md` with **tracing, SLOs, alert-notification flow**.

---

### 🔹 UA – Data & Messaging  
**🎯 Objective: Prepare Digital Twin data layer**  
1. **Historical Queries**
   - API: `/telemetry/history/{asset_id}?from=…&to=…` → return windowed data.
   - Optimize with Timescale continuous aggregates.
2. **Geospatial (PostGIS)**
   - Extend schema: `location geometry(Point, 4326)`.  
   - Install PostGIS extension in Timescale.  
   - Insert mock GPS data into telemetry messages.  
3. **Digital Twin Sync**
   - API: `/assets/{asset_id}/state` → latest telemetry + location.
   - This powers TN’s frontend Digital Twin page.
**📌 Deliverables:**
- Updated schema with PostGIS.  
- FastAPI endpoints: history + state.
- Docs in `docs/data-pipeline.md`.

---

### 🔹 AM – CI/CD & Automation  
**🎯 Objective: CI/CD builds + deploys services to AKS**  
1. **Extend GitHub Actions workflow**
   - Build Docker images (backend + ingestion worker).
   - Push to ACR.
   - Deploy via Helm to AKS.  
2. **Add Helm test hook** → confirm pods healthy.  
3. **Add CI/CD status badge** to repo README.
📌 Deliverable: Commit → Actions builds → pushes → deploys to AKS.

---

### 🔹 TN – Frontend  
**🎯 Objective: Functional Dashboard + QR foundation**  
1. **Dashboard Home** → 4 cards: Digital Twin, Dispatch, GeoSecure, AssetMax.  
2. **Digital Twin Page:**  
   - Fetch from UA’s `/telemetry/{asset_id}/recent`.  
   - Display in table (shadcn/ui).  
3. **QR Generator** (qrcode.react):
   ```bash
   <QRCode value={`https://aurynto.com/asset/${asset.id}`} />
   ```
📌 Deliverable: Frontend showing live telemetry + per-asset QR.  

---

### 🔹 MR – Backend Services  
**🎯 Objective: Expand FastAPI backend**  
1. **Assets API**
   - `GET /assets` → list assets  
   - `POST /assets` → create new asset  
2. **Jobs API (mock for now)**  
   - `POST /jobs` → create job
   - `GET /jobs/{id}` → return job status
3. **Dockerize & Helm**
   - Build Dockerfile
   - Create Helm templates (`backend-deployment.yaml`, `backend-service.yaml`)
📌 Deliverable: Backend APIs live on AKS via CI/CD.

---
   
## 📅 Reporting & Demo  
- **Daily Standup (15 min)** → yesterday’s work, today’s goal, blockers.
- **Docs** → each fresher updates their guide in `docs/`.
- **Saturday/Sunday Demo:**
  - HN → Grafana dashboards (SLOs + traces).  
  - UA → Digital Twin API (history + state with GPS).
  - AM → CI/CD auto-deploy to AKS.
  - TN → Dashboard with telemetry + QR.
  - MR → Backend Assets & Jobs API.
  - Talha → Vertical slice demo + updated architecture doc.

--- 

## ✅ Week 2 Definition of Done  
- Telemetry pipeline supports **real-time + historical queries**.
- Digital Twin API returns **latest state + GPS**.
- Observability includes **metrics + logs + traces + SLOs + alerts**.
- CI/CD builds → pushes → deploys to AKS.
- Frontend fetches live telemetry + QR.
- Backend APIs for assets & jobs live.
- All docs updated in `docs/`.  
