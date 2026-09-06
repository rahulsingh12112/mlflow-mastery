# Topic 9: MLflow Model Registry ⭐⭐⭐⭐

> Governance ka dil. Teacher-session style + production lab. Interview: deployment/governance sawaal yahan se.

---

## Kahani — Kya + Kyun

Tracking me models log kiye. Par 50 runs ke baad: kaunsa production me jaana chahiye? kaunsa live hai? kisne approve kiya? naya version aaya to purane ka kya? Ye governance problem **Model Registry** solve karta — ek **central catalog** (library ke register jaisa) jahan har model ka naam, version, status likha hai.

**Key insight:** Registry model ko **MOVE nahi karta** — actual model file jahan hai (S3) wahin rehti, registry sirf **reference + metadata + version + alias/tag** rakhta. (Networking analogy: IPAM/CMDB — central record, actual device jahan hai wahin.)

**Recent change (interview trap):** Pehle 4 stages the — None/Staging/Production/Archived. **DEPRECATED.** Ab **Tags** (manual labels) + **Aliases** (named pointer jaise "champion"). "Staging/Production stages" bologe = purana knowledge signal.

---

## Points

### 1. Registry = central version-controlled catalog
- Versions auto-increment (v1, v2, v3...) har register pe.
- Metadata: version, tags, alias, description, registered date, kisne (audit trail — compliance).
- Model ko move nahi karta — artifacts S3 me, registry sirf reference+metadata.
- ⚠️ DB backend MANDATORY (file store se registry nahi chalta).

### 2. Stages DEPRECATED → Tags + Aliases
- Purana (mat bolo): None/Staging/Production/Archived.
- Naya: **Tags** (`production`, `validated`) + **Aliases** (`champion`, `challenger`).
- Alias = named pointer to a version, fetch `models:/wine-classifier@champion`.
- Alias = DNS CNAME jaisa: indirection, code me alias use, version hardcode nahi.

### 3. Champion/Challenger (production pattern)
- Champion = current prod (alias). Challenger = naya candidate (alias).
- A/B: 90% champion, 10% challenger → challenger better → **alias swap** (promote). Code change nahi.
- Rollback = champion alias purane version pe → instant, plug-and-play.

---

## Diagram

```
   Runs (50 models) → REGISTRY (central catalog)
   ┌────────────────────────────────────────────┐
   │ Model: "wine-classifier"                     │
   │   v1  [tag: archived]                        │
   │   v2  ← alias: champion   (production)       │
   │   v3  ← alias: challenger (A/B candidate)    │
   └────────────────────────────────────────────┘
        │ registry = reference + metadata (NOT the file)
        └─→ actual model files S3 me (registry move nahi karta)

   Deploy: code "models:/wine-classifier@champion"
   Promote: challenger better → alias swap (v3=champion), code same
   Rollback: champion alias v2 pe wapas (instant)
   Alias = DNS CNAME | stages DEPRECATED
```

---

## AI infra se jodo
- Registry = DB backend → RDS PostgreSQL (production).
- Champion/challenger = 2.6 A/B deployment + 5F GitOps (alias swap = deploy).
- Rollback via alias = 4.4 HA/DR instant model rollback.
- SageMaker Model Registry = AWS native; MLflow = open-source alternative.

---

## Interview one-liner
> "The Model Registry is a central version-controlled catalog — it stores references, metadata, versions, but the actual model files stay in S3, and it needs a DB backend. Old Staging/Production stages are deprecated; now tags and aliases. An alias like 'champion' is a named pointer to a version, like a DNS CNAME — for champion/challenger you A/B test and swap the alias to promote, with instant rollback by re-pointing, no code change."

---

## 🧪 LAB (production-style) — Register + version + alias

Server chal raha ho (Topic 5, DB backend — registry ke liye zaroori).

### Step 1 — train.py me register add
```python
mlflow.sklearn.log_model(
    model, name="model", signature=signature, input_example=X_te[:2],
    registered_model_name="wine-classifier"   # auto-register
)
```
`python3 train.py` 2-3 baar → registry me v1, v2, v3.

### Step 2 — champion alias (MlflowClient)
```python
# set_alias.py
import mlflow
from mlflow import MlflowClient
mlflow.set_tracking_uri("http://127.0.0.1:5000")
client = MlflowClient()
client.set_registered_model_alias("wine-classifier", "champion", version=2)
print("champion = v2")
```

### Step 3 — champion load (jaise production)
```python
# load_champion.py
import mlflow
mlflow.set_tracking_uri("http://127.0.0.1:5000")
model = mlflow.pyfunc.load_model("models:/wine-classifier@champion")
print("Loaded champion:", model)
```

### Step 4 — UI: Models tab → wine-classifier → versions + champion alias

### Kya dekhna
- Multiple versions, v2 pe champion alias.
- `@champion` se load (version hardcode nahi = indirection).
- Alias badlo (v3 champion) → code same. Rollback = wapas v2.

---

## ⚠️ GOTCHAS
1. Registry = metadata only; actual model S3 me (move nahi karta).
2. Stages DEPRECATED → tags + aliases (interview trap).
3. DB backend mandatory (file store = no registry).
4. Register ≠ deploy (registry catalog hai, serving alag).
5. Alias = indirection → code me alias use, version hardcode mat karo.

---

## Q&A (interview)
**Q: Registry file store karta ya reference?** — "Sirf reference + metadata (versions/tags/aliases/description); actual model S3 me. Move nahi karta."

**Q: Stages ki jagah ab kya?** — "Deprecated stages → tags (labels) + aliases (named pointers like champion). Alias = DNS CNAME jaisa indirection."

**Q: Champion/challenger + rollback?** — "Champion=prod, challenger=candidate; A/B test → alias swap to promote (code same); rollback = champion alias purane version pe, instant."
