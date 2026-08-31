# Topic 5: MLflow Tracking Server Architecture (ARCHITECT-LEVEL Deep Dive)

> **⭐⭐⭐⭐ Interview weight — yeh TERA role ka core hai (Infra Architect)**
> **Prerequisites:** File 02 (Tracking)
> **Source:** mxagar/mlflow_guide §7 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "MLflow client-server architecture pe chalta — client (tera training code) REST se tracking server se baat karta. Server ke 2 storage components hain: Backend Store (metadata: params, metrics, tags — DB ya file) aur Artifact Store (models, plots, large files — S3/local). Deployment 4 scenarios me scale hota: pure local → local+SQLite → local server → fully remote+distributed (S3 + PostgreSQL)."

---

## Layer 1: Client-Server Model (Fundamental)

MLflow do parts me:

```
┌─────────────┐         REST API          ┌──────────────────┐
│   CLIENT    │  ────────────────────────▶│  TRACKING SERVER │
│ (tera code, │  ◀────────────────────────│                  │
│  Python API)│                            │  ┌─────────────┐ │
└─────────────┘                            │  │Backend Store│ │ ← metadata
                                           │  │(DB/file)    │ │   (params,
                                           │  └─────────────┘ │    metrics,
                                           │  ┌─────────────┐ │    tags)
                                           │  │Artifact     │ │ ← models,
                                           │  │Store(S3/dir)│ │   plots,
                                           │  └─────────────┘ │   files
                                           └──────────────────┘
```

**Client** = jahan training chalti (tera laptop/EC2/notebook). Python API call karta.
**Server** = data manage karta, 2 stores ke saath.

**Networking analogy:** Bilkul **client-server** jaise. Client = network device jo SNMP traps bhejta. Server = NMS (Network Management System) jo do jagah store karta — metadata DB me, aur bulk data (pcaps/logs) file storage me.

---

## Layer 2: 2 Storage Components (INTERVIEW GOLD)

Yeh farak interview me **guaranteed** poochha jaata:

### 1. Backend Store — metadata
Kya store karta: **params, metrics, tags, run info, experiment info** (structured, small data).

Do types:
- **DB Stores:** SQLite, MySQL, **PostgreSQL**, MS SQL
- **File Stores:** local filesystem, etc.

### 2. Artifact Store — bulk files
Kya store karta: **models, images, plots, datasets** (large, binary files).

Location: local folder, **Amazon S3**, Azure Blob, GCS, etc.

**Kyun 2 alag?**
- Metadata = chhota, structured → DB fast queries/filtering ke liye
- Artifacts = bade binary files → object storage (S3) cheap + scalable

**Networking analogy:** Backend store = **config database** (structured, queryable — jaise IPAM/CMDB). Artifact store = **bulk file storage** (jaise TFTP/S3 pe firmware images, backups). Do alag kyunki nature alag — ek queryable metadata, doosra bulk blobs.

---

## Layer 3: Networking (Communication) — 3 Types

Client aur server kaise baat karte:
1. **REST API (HTTP)** — most common, default
2. **RPC (gRPC)** — high-performance
3. **Proxy access** — restricted, role-based access

---

## Layer 4: 4 DEPLOYMENT SCENARIOS (Zero → Enterprise)

Yeh tere liye **most important section**. Interview me "local se production tak kaise scale karoge" — yeh answer.

### Scenario 1: Pure Local (Development)
```
Client: tera laptop
Server: NAHI (koi `mlflow server` nahi)
Backend: ./mlruns folder
Artifacts: ./mlruns folder
```
- Sabse simple, laptop pe experiment
- `mlflow ui` se dekho
- **Problem:** team share nahi kar sakti, scale nahi

### Scenario 2: Local + SQLite (Slightly better)
```
Client: laptop
Server: NAHI
Backend: SQLite DB (local)  ← ab DB me metadata
Artifacts: ./mlruns folder
```
- Metadata DB me → better queries, model registry enable hota (registry ko DB chahiye!)
- **Note:** Model Registry ke liye DB backend MANDATORY (file store se registry nahi chalta)

### Scenario 3: Local Server (Team, single machine)
```
Client: laptop → REST → Server
Server: `mlflow server` launched (localhost:5000)
Backend: ./mlruns ya SQLite
Artifacts: ./mlruns
```
```bash
mlflow server \
  --backend-store-uri sqlite:///mlflow.db \
  --default-artifact-root ./mlflow-artifacts \
  --host 127.0.0.1 --port 5000
```
- Ab REST se connect, dedicated server process
- Code me: `mlflow.set_tracking_uri("http://127.0.0.1:5000")`

### Scenario 4: Remote & Distributed (PRODUCTION — enterprise)
```
Client: laptop/EC2/notebook → REST → Remote Server
Server: `mlflow server` on remote host (EC2), ports exposed
Backend: PostgreSQL (on separate DB node/RDS)   ← metadata
Artifacts: Amazon S3 bucket                       ← models/files
```
```bash
mlflow server \
  --backend-store-uri postgresql://user:pass@postgres-host:5432/mlflowdb \
  --default-artifact-root s3://my-mlflow-bucket \
  --host 0.0.0.0 --port 5000
```

**Yeh production architecture hai — TERE role ka answer:**
- **Backend:** PostgreSQL (RDS) — metadata, HA, queryable
- **Artifacts:** S3 — cheap, scalable, durable
- **Server:** EC2/ECS/EKS — client REST se connect
- Team-wide, scalable, durable

**Networking analogy:** Scenario 1-2 = ek router pe local logging. Scenario 4 = **centralized enterprise logging** — dedicated syslog server (EC2), structured DB (PostgreSQL/RDS), bulk storage (S3). Distributed, HA, team-wide — bilkul enterprise NMS setup.

---

## Layer 5: Scenario Comparison Table

| Scenario | Backend | Artifacts | Server? | Use-case |
|----------|---------|-----------|---------|----------|
| 1. Pure Local | ./mlruns | ./mlruns | No | Solo dev |
| 2. Local+SQLite | SQLite | ./mlruns | No | Registry enabled |
| 3. Local Server | SQLite/file | local | Yes | Small team, 1 machine |
| 4. Remote Distributed | **PostgreSQL** | **S3** | Yes (remote) | **Production/enterprise** |

---

## Layer 6: Production Considerations (Architect thinking)

Interview me yeh bolna depth dikhata:

1. **HA (High Availability):** MLflow server ko ECS/EKS pe multiple replicas, load balancer ke peeche
2. **Security:** Server ko VPC me, private subnet; auth (basic auth / reverse proxy with OIDC); S3 bucket policies; RDS encryption
3. **Cost:** S3 lifecycle policies (purane artifacts Glacier), RDS right-sizing
4. **Backup:** RDS automated backups, S3 versioning
5. **Networking:** Server ko internet-facing mat rakho; VPN/PrivateLink se access

> **Honesty note:** MLflow ka built-in auth basic hai. Production me **reverse proxy (nginx/ALB) + OIDC/SSO** laga ke secure karte. Yeh bolna — tera security background chamkega.

---

## 🎤 Interview Q&A (yeh section RATTA + samajh — tere role ka core)

**Q1: Backend store vs artifact store?**
> "Backend store metadata rakhta — params, metrics, tags, run/experiment info — DB ya file (production me PostgreSQL). Artifact store bulk files rakhta — models, plots, datasets — S3/local. Do alag kyunki metadata structured+queryable, artifacts large binary blobs."

**Q2: Local se production tak MLflow kaise scale karoge?**
> "4 scenarios: (1) pure local mlruns, (2) local+SQLite for registry, (3) local dedicated server, (4) production — remote mlflow server on EC2/EKS, backend PostgreSQL/RDS, artifacts S3. Client REST se connect. HA ke liye multiple replicas + ALB, VPC + auth for security."

**Q3: Model registry ke liye kya chahiye?**
> "DB backend mandatory — SQLite/PostgreSQL/MySQL. File store se registry nahi chalta. Isliye Scenario 2+ (DB backend) chahiye registry ke liye."

**Q4: Client-server communication?**
> "REST API (HTTP) default; gRPC bhi possible. Client (training code) set_tracking_uri se server ko point karta, phir sab logging REST se server ko jaata."

**Q5: Production MLflow secure kaise?**
> "Server private subnet me (VPC), reverse proxy + OIDC/SSO for auth (built-in auth basic hai), S3 bucket policies + encryption, RDS encryption + backups. Internet-facing avoid."

---

## ⚠️ Gotchas

1. **Registry needs DB** — file store se registry nahi. SQLite minimum.
2. **`--host 0.0.0.0`** production me — par tab firewall/security group se protect karo (warna world-open)
3. **Artifact store client se accessible hona chahiye** — S3 credentials client ke paas ho (warna model download fail)
4. **Public DNS badalta** — EC2 restart pe IP/DNS change, tracking URI update karna pad sakta (Elastic IP use karo)

---

## Next: File 06 — Model Component & Signatures
