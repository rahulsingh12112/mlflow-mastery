# Topic 12: MLflow on AWS — Production Architecture ⭐⭐⭐⭐⭐

> Highest-weight topic for AWS Infra Architect. Teacher-session style + production lab + full deployment flow.

---

## Kahani — Kya + Kyun

Local/chhote server ke baad, production me MLflow AWS pe proper architecture me deploy karte (tera strongest answer — 7yr AWS). 3 hisson me (Topic 5), ab AWS services ke saath: tracking server container pe (ECS/EKS/EC2), backend store managed DB (RDS PostgreSQL), artifact store S3. Teeno alag services, jud ke platform.

Deploy ke baad: serving (registry @champion → endpoint, aksar SageMaker managed), aur security+cost.

---

## Points

### 1. 3-component AWS mapping
```
Tracking server → ECS/EKS (container) ya EC2
Backend store   → RDS PostgreSQL (metadata: params/metrics/tags)
Artifact store  → S3 (model binaries)
```

### 2. Serving (registry → endpoint)
- `models:/name@champion` → deploy.
- SageMaker endpoint (production — managed HTTPS URL, autoscaling, HA, AWS ops sambhaale).
- MLflow serve (chhota scale) ya container (ECS/EKS).
- Live inference endpoint handle karta; MLflow NAHI per-request path me.

### 3. Security + Cost
- Security: private VPC subnet, IAM/IRSA (S3/RDS no keys), Secrets Manager, HTTPS.
- Cost: RDS small (metadata chhota), S3 lifecycle, server right-size.
- HA: RDS Multi-AZ, server multiple tasks.

---

## Diagram

```
   DS clients → log (tracking URI)
        ▼
   ┌──── MLflow Server (ECS/EKS/EC2) ────┐
   │ backend → RDS PostgreSQL (metadata)  │
   │ artifacts → S3 (model files)         │
   └───────────────────────────────────────┘
        │ deploy (registry @champion)
        ▼
   SageMaker Endpoint (managed, autoscale) ← live inference
   Security: VPC private + IAM/IRSA + Secrets Mgr
```

---

## SageMaker Endpoint kya (clarity)
= managed live HTTPS URL jahan model baitha, predictions deta. Tum model do, AWS scaling/HA/patching/health sab manage kare (jaise Bedrock managed vs self-host). User → URL → prediction.

MLflow vs SageMaker roles:
```
MLflow = RECORD + CATALOG (tracking + registry) — "kaunsa model/version/champion"
SageMaker endpoint = SERVING (live prediction) — managed, AWS ops
Deploy time: registry se model → SageMaker endpoint. Uske baad live traffic SageMaker.
```

---

## Poora Production Flow
```
1. TRAIN + EVALUATE (offline) — MLflow track (params/metrics/model/version)
2. REGISTER — registry, champion alias
3. DEPLOY — registry se → SageMaker/ECS/EKS endpoint
4. LIVE INFERENCE — user → endpoint → prediction (MLflow NAHI beech me)
5. MONITORING (live) — drift/accuracy/latency — CloudWatch/Evidently (MLflow NAHI)
6. Issue? → TRIGGER (human ya automation) → CI/CD-GitOps pipeline →
   ROLLBACK: MLflow alias swap + redeploy (MLflow EXECUTE karta)
   Drift? → RETRAIN naya version (rollback nahi — purana bhi drift face karega)
```

### DETECT / TRIGGER / EXECUTE (teen alag)
```
DETECT  → monitoring (CloudWatch/Evidently) — "kuch galat"
TRIGGER → human (manual) ya automation (Lambda/pipeline) — "rollback karo"
EXECUTE → MLflow registry alias swap — "purana version live"
   (aksar CI/CD-GitOps pipeline me configured; GitOps: Git me version pointer badlo)
```

---

## Model deploy patterns (bake vs runtime-load)
```
Bake in image  → model image ke andar; har version alag image;
                 rollback = purane image deploy (slow, bada pull)
Runtime load   → ek image, model registry/S3 se load;
                 rollback = alias swap, SAME image (FAST — no rebuild/re-pull)
```
Model = weights FILE, Docker image = package (code+env+maybe model). Alag cheezein.

---

## Key clarities (Q&A se)
- Live me MLflow = registry (version control + rollback), per-request inference me NAHI.
- Live monitoring = alag tools (CloudWatch/Evidently), MLflow ka kaam NAHI.
- Drift → RETRAIN; version-bug/regression → ROLLBACK. (Alag!)
- Git me model FILE nahi, sirf pointer (version/alias); actual model S3/registry me.
- Registry vs direct-S3: registry abstraction deta (@champion), S3 path hardcode nahi (cleaner).

---

## Interview one-liner
> "Production MLflow is AWS-native: tracking server on ECS/EKS, backend on RDS PostgreSQL, artifacts on S3. Clients log via the tracking URI; for deployment I pull the champion from the registry onto a SageMaker endpoint (managed, autoscaling). MLflow isn't in the per-request path — it controls deploy and rollback via the registry, wired into the CI/CD or GitOps pipeline. Monitoring (CloudWatch) detects, a human or automation triggers, MLflow executes the alias swap. Server sits in a private VPC subnet with IAM/IRSA to S3/RDS."

---

## 🧪 LAB
```bash
# Option A — S3 artifacts (agar AWS sandbox)
mlflow server --backend-store-uri sqlite:///$HOME/mlflow-lab/mlflow.db \
  --default-artifact-root s3://YOUR-BUCKET/mlflow-artifacts --host 127.0.0.1 --port 5000

# Option B — serve model as REST endpoint
mlflow models serve -m "models:/wine-classifier@champion" --port 5001 --no-conda
curl -X POST http://127.0.0.1:5001/invocations -H "Content-Type: application/json" \
  -d '{"inputs": [[13.0,2.0,2.3,15.0,100.0,2.8,3.0,0.3,2.0,5.0,1.0,3.0,1000.0]]}'
# → prediction wapas (class)
```

---

## Q&A (interview)
**Q: AWS production setup?** — "Server ECS/EKS, backend RDS PostgreSQL, artifacts S3. Private VPC subnet, IAM/IRSA."

**Q: SageMaker endpoint kyun?** — "Managed serving URL — AWS autoscaling/HA/patching sambhaale, ops burden kam. MLflow serve chhote scale ke liye."

**Q: Live me MLflow ka role?** — "Registry (kaunsa version live + rollback control), per-request inference me nahi. Monitoring detect, human/automation trigger, MLflow (via pipeline) execute alias swap."

**Q: Drift pe rollback?** — "Nahi — drift = retrain (naya version, purana bhi drift face karega). Rollback = version-bug/regression ke liye."
