## Next Tasks – Ubaid (Data & Messaging)  

### Objective: Make the ingestion pipeline production-ready  

1. **Expand TimescaleDB schema**  
   - Add realistic fields for assets (`id`, `type`, `location`, `status`)  
   - Add `jobs` table (`job_id`, `asset_id`, `operator`, `status`, `eta`)  
   - Extend `telemetry` table with: `fuel_level`, `load_weight`, `engine_temp`, `coolant_temp`, `rmp`, `engine_hours`, `battery_v`, `speed`  
   - Add foreign keys to link `telemetry → assets` and `jobs → assets`.
  
2. **Enhance ingestion worker**  
   - Convert the simple script into a proper Python service with:  
     - Config via `.env` (DB creds, MQTT topic)  
     - Error handling & logging  
     - Batch inserts for efficiency  
   - Package with `requirements.txt` and a Dockerfile.
  
3. **API endpoint (FastAPI)**
   - Expose `/telemetry/recent/{asset_id}` → return last 10 readings from DB.  
   - This will feed Fresher D’s frontend later.
  
4. **Deliverables**
   - Push code into `aurynto-sandbox/aurynto-services/data-pipeline/`.  
   - Add DB schema as schema.sql.  
   - Add Dockerfile for ingestion worker.  
   - Document in docs/data-pipeline.md.
  
## Next Tasks – Hashir (Infra & Observability)  

### Objective: Go beyond cluster metrics → monitor apps & logs  

1. **Prometheus scraping for custom metrics**  
   - Deploy a dummy FastAPI service that exposes `/metrics` (use `prometheus-client`).  
   - Configure Prometheus to scrape this endpoint.  
   - Verify metrics (requests/sec, response time) show up in Grafana.
  
2. **Loki logging**
   - Install Loki + Promtail via Helm:
     
     ```bash
     helm repo add grafana https://grafana.github.io/helm-charts
     helm upgrade --install loki grafana/loki-stack -n observability
     ```
   - Forward logs from pods → Loki.  
   - Create Grafana dashboard panel showing logs from ingestion worker + backend.  

3. **Alerting**
   - Configure a Grafana alert (e.g., CPU > 80% or pod restart > 3).
   - Document alert setup.
  
4. **Deliverables**
   - Grafana dashboard with:
     - Cluster CPU/memory (already done)
     - Ingestion worker metrics (new)
     - Logs from Loki (new)
   - `docs/observability-setup.md` updated with new steps.
