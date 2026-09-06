# Topic 5: MLflow Tracking Server Architecture ⭐⭐⭐⭐⭐

> Architect ke liye HIGHEST-weight topic. Teacher-session style + production lab.
> Ye "UI galat folder / local mlruns" problem ka asli production ilaaj.

---

## Kahani — Kya + Kyun

Local MLflow = sab kuch laptop ke `mlruns/` folder me — solo dev ke liye theek. Par team (5 data scientists) me: har ek ka `mlruns` alag, koi kisi ka run nahi dekh sakta, "team ka best model kaunsa" ka single jawab nahi. Isiliye **production me central tracking server** — ek jagah jahan sab log karte, sab dekhte. MLflow "mere laptop ka tool" se "team ka platform" ban jaata.

**Critical architect concept:** MLflow do alag cheezein alag jagah store karta:
- **Metadata** (params, metrics, tags, run info) — chhota, structured, query-able → **database** (backend store)
- **Artifacts** (model files, plots — bade blobs) → **file/object store** (artifact store, S3)

Alag isliye kyunki nature alag: metadata chhota+query-able (DB best), artifacts bade blobs (object store best).

---

## Points

### 1. Do stores (interview me zaroor)
- **Backend store** — metadata (params/metrics/tags/run+experiment info). → SQLite (local), PostgreSQL/MySQL/RDS (production).
- **Artifact store** — actual files (model binaries, plots, datasets). → local folder ya **S3** (production).
- ⚠️ **Registry ke liye DB backend MANDATORY** — file store se registry nahi chalta.

### 2. 4 Deployment Scenarios
```
1. Local files    → backend + artifacts dono ./mlruns (solo dev)
2. Local + SQLite → backend SQLite DB, artifacts local (registry ke liye DB chahiye)
3. Remote server  → central mlflow server, team connect (DB backend + artifact store)
4. Full production→ mlflow server on EC2/ECS + PostgreSQL(RDS) + S3 artifacts
```

### 3. Client-Server model
- Client (DS code) → `set_tracking_uri("http://server:5000")` → server pe log.
- Server → metadata DB me, artifacts S3 me.
- Sab clients → ek server → single source of truth. UI-folder-mismatch khatam.

---

## Diagram

```
   [DS1] [DS2] [DS3]  → set_tracking_uri("http://mlflow-server:5000")
        \    |    /
     ┌──────────── MLflow Tracking Server (EC2/ECS) ────────────┐
     │  metadata (params/metrics/tags) → BACKEND (PostgreSQL/RDS)│
     │  artifacts (model/plots, bade)  → ARTIFACT STORE (S3)     │
     └───────────────────────────────────────────────────────────┘
   Backend = DB (chhota, query-able) | Artifact = S3 (bade blobs)
```

---

## AWS stack (production)
- Backend store = **RDS PostgreSQL** (metadata)
- Artifact store = **S3** (model blobs)
- Server = **ECS/EKS** (container) ya EC2
- Pitch: "Production MLflow: server on ECS, backend RDS PostgreSQL, artifacts S3 — full AWS-native."

---

## Interview one-liner
> "MLflow separates two stores: a backend store for metadata (params, metrics, tags) — structured and query-able, so a database like PostgreSQL/RDS in production; and an artifact store for large files like model binaries, so an object store like S3. In production a central tracking server (on ECS/EC2) is what all clients connect to via the tracking URI — single source of truth. Note: the model registry requires a DB backend, not a file store."

---

## 🧪 LAB (production-style) — Central server, DB backend + artifact store

Ye "UI folder mismatch" ka asli fix.

### Step 1 — Server chalao (SQLite backend + dedicated artifact root)
```bash
cd ~/mlflow-lab
mlflow server \
  --backend-store-uri sqlite:///$HOME/mlflow-lab/mlflow.db \
  --default-artifact-root $HOME/mlflow-lab/mlartifacts \
  --host 127.0.0.1 --port 5000
# terminal chhod do (server chal raha), naya terminal kholo
```

### Step 2 — train.py me server point karo (top pe)
```python
mlflow.set_tracking_uri("http://127.0.0.1:5000")   # server, local files nahi
mlflow.set_experiment("wine-classifier")
```

### Step 3 — naye terminal se train
```bash
cd ~/mlflow-lab
python3 train.py    # data server ke DB + artifact folder me, ./mlruns me nahi
```

### Step 4 — UI
`http://127.0.0.1:5000` — ab KISI BHI folder se train chalao, same data (folder-mismatch gone).

### Kya dekhna
- `mlflow.db` (backend — metadata, SQLite)
- `mlartifacts/` (artifacts — model files)
- Do alag jagah = backend vs artifact store LIVE
- Production: SQLite→RDS PostgreSQL, mlartifacts/→S3

---

## ⚠️ GOTCHAS
1. Registry ke liye DB backend MANDATORY (file store = no registry).
2. Central server = single source of truth → local mlruns folder-mismatch problem solved.
3. Backend = metadata (DB), Artifact = files (S3) — MAT mix, ye do alag.

---

## Q&A (interview)
**Q: Backend vs artifact store?** — "Backend = metadata (params/metrics/tags), structured/query-able → DB (RDS PostgreSQL prod). Artifact = large files (model binaries/plots) → object store (S3). Alag kyunki nature alag."

**Q: AWS production setup?** — "Server on ECS/EKS, backend RDS PostgreSQL, artifacts S3."

**Q: Local se production kaise scale?** — "4 scenarios: local files → local+SQLite → remote server → full prod (ECS server + RDS + S3). Team ke liye central server = single source of truth."
