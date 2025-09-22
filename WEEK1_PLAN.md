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
| Project Lead / Architect | **Talha Najeeb** | Azure infra, GitHub repos, Helm umbrella, overall integration |
| Fresher A | Infra & Observability | AKS (later), Helm umbrella, Prometheus, Grafana, Loki |
| Fresher B | Data & Messaging | TimescaleDB schema, MQTT broker, ingestion worker |
| Fresher C | CI/CD & Automation | GitHub Actions, OIDC, Helm deploy pipeline |
| Fresher D | Frontend | Next.js dashboard, QR mobile pages |
| Fresher E | Backend Services | FastAPI services (Digital Twin, Dispatch, GeoSecure) |

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

### 🔹 Project Lead / Architect (Talha)
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
