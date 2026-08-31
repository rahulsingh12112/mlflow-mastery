# Topic 9: MLflow Model Registry (⭐⭐⭐⭐ Governance Deep Dive)

> **Interview weight HIGH — governance/deployment ke sawaal yahan se**
> **Prerequisites:** File 05 (server/DB backend), File 06 (models)
> **Source:** mxagar/mlflow_guide §11 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "Model Registry ek central database hai jahan model versions unke metadata ke saath store hote — model artifacts jahan hain wahin rehte, sirf reference + metadata registry me. Registry version management, aur deployment lifecycle (staging → production → archive, ab tags/aliases se) handle karta. DB backend mandatory."

---

## Layer 1: Kya Aur Kyun?

File 02 me humne model **log** kiya (artifacts me save). Par:
- Kaunsa model version production me hai?
- Kisne approve kiya?
- Naya version aaya to purane ka kya?

**Registry yeh solve karta** — models ka **central version-controlled catalog** with governance.

**Key insight:** Registry model ko **move nahi karta** — artifacts jahan (S3) wahin rehte. Registry sirf **reference + metadata + version + stage** store karta.

**Networking analogy:** Registry = **IPAM / CMDB for models**. Central catalog — kaunsa model version kahan, kis stage me, kisne approve kiya. Actual model (device) jahan hai wahin, registry uska authoritative record rakhta.

---

## Layer 2: Pre-requisites

Registry chalane ko:
1. **DB backend** — `mlflow server` with SQLite/PostgreSQL (file store se registry NAHI chalta!)
2. Model **log** kiya hua

```python
mlflow.set_tracking_uri("http://127.0.0.1:5000")  # server with DB
```

> **Critical (interview):** Registry ke liye DB backend MANDATORY. Yeh File 05 ke Scenario 2+ se connect.

---

## Layer 3: Model Register Karna — 3 Tarike

### 1. UI se
Run → Artifacts → model select → **"Register Model"** → naam do. Same naam dobara = nayi version.

### 2. log_model me register
```python
mlflow.sklearn.log_model(
    model, "model",
    registered_model_name="elasticnet-wine",  # yeh diya = auto-register
)
```

### 3. register_model() se (alag call)
```python
mlflow.sklearn.log_model(lr, "model")  # pehle log (no name)

mlflow.register_model(
    model_uri=f"runs:/{run.info.run_id}/model",
    name="elasticnet-wine",
    tags={"stage": "staging"},
)
```

**Har register = version auto-increment** (v1, v2, v3...).

### Registered model load:
```python
model = mlflow.pyfunc.load_model(model_uri="models:/elasticnet-wine/1")
# models:/<name>/<version>  ya  models:/<name>@<alias>
```

---

## Layer 4: Stages vs Aliases (IMPORTANT — recent change)

### PURANA tarika (DEPRECATED now):
Model 4 stages me hota tha:
- **None**
- **Staging** — production candidate
- **Production** — live
- **Archived** — retired

> **Interview trap:** Agar tu "Staging/Production stages" bolega, interviewer soch sakta tu purana knowledge rakhta. **Yeh ab deprecated hai.** Naya jaanna zaroori.

### NAYA tarika (current):
1. **Tags** — manually version ko tag karo: `staging`, `production`, `archive`
2. **Aliases** — named reference to a specific version:
   - `champion` alias ek version ko
   - Fetch: `models:/<name>@champion`
   - API: `get_model_version_by_alias()`

```python
# Alias set (champion = production model)
client.set_registered_model_alias("elasticnet-wine", "champion", version=3)

# Load by alias
model = mlflow.pyfunc.load_model("models:/elasticnet-wine@champion")
```

**Kyun aliases better:** flexible — `champion`/`challenger` A/B testing, code me alias reference (version hardcode nahi). Deploy pe alias badlo, code same.

**Networking analogy:** Alias = **DNS CNAME / floating IP**. `champion` ek CNAME jaise — kis actual version (server) pe point karta woh badal sakta, par clients `champion` hi use karte. Version hardcode nahi — indirection.

---

## Layer 5: Champion/Challenger Deployment (Production pattern)

Interview me yeh bolna advanced dikhata:
- **Champion** = current production model (alias `champion`)
- **Challenger** = naya candidate (alias `challenger`)
- Traffic split: 90% champion, 10% challenger (A/B test)
- Challenger better perform kare → promote to champion (alias swap)
- Code change nahi — sirf alias update

**Rollback:** champion alias purane version pe point kar do — instant rollback.

---

## Layer 6: Metadata & Descriptions

Registry me **model level** aur **version level** dono pe metadata:
- **Descriptions** — model kya karta, kab train hua
- **Tags** — framework, data slice, production-ready?

```python
# Model level tag
client.set_registered_model_tag("elasticnet-wine", "team", "ml-platform")

# Version level tag
client.set_model_version_tag("elasticnet-wine", "1", "validated", "true")
```

**Governance value:** audit trail — kaunsa version kab, kisne, kis data pe. Compliance ke liye critical.

---

## 🎤 Interview Q&A

**Q1: Model Registry kya, kyun?**
> "Central version-controlled catalog for models with metadata + deployment lifecycle. Artifacts jahan (S3) wahin, registry reference/metadata/version/stage rakhta. Governance — kaunsa version live, kisne approve. DB backend mandatory."

**Q2: Stages deprecate kyun, ab kya?**
> "Purane None/Staging/Production/Archived stages deprecated. Ab Tags (manual labels) aur Aliases (named refs like champion). Aliases flexible — champion/challenger, code me alias reference, deploy pe swap, rollback easy."

**Q3: Champion/Challenger kaise?**
> "Champion = current prod (alias), challenger = candidate. A/B traffic split, challenger better → alias swap to promote. Code change nahi, sirf alias. Rollback = champion alias purane version pe."

**Q4: log_model vs register_model?**
> "log_model model artifacts me save. register_model (ya registered_model_name param) usko registry me version banata. log = save, register = catalog + govern."

**Q5: Registry ke liye kya chahiye?**
> "DB backend (SQLite/PostgreSQL) mandatory — file store se registry nahi chalta. Server with --backend-store-uri as DB."

---

## ⚠️ Gotchas

1. **DB backend mandatory** — file store = no registry
2. **Stages DEPRECATED** — interview me aliases/tags bolo, na ki Staging/Production stages
3. **Register ≠ deploy** — registry catalog hai, actual serving alag (File 12)
4. **Alias = indirection** — code me alias use karo, version hardcode mat karo (flexibility)

---

## Next: File 10 — MLflow Projects
