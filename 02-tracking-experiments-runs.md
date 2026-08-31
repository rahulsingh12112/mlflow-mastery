# Topic 2: MLflow Tracking — Experiments & Runs (Complete Deep Dive)

> **Target Role:** AI Infrastructure Architect / Senior ML Platform Engineer
> **Prerequisites:** File 01 (MLflow 4 components)
> **Source:** mxagar/mlflow_guide §3 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "MLflow Tracking ek API + UI hai jo har ML training run ke parameters, metrics, aur artifacts ko log karta hai, taaki hum experiments compare karke best model choose kar sakein. Hierarchy: ek Experiment ke andar multiple Runs, har Run ek code execution."

---

## Layer 1: Kya Aur Kyun?

File 01 ki problem yaad kar: 50 experiments chalaye, bhool gaye konsa best. **Tracking yeh solve karta.**

Har training run pe MLflow record karta:
- **Parameters** — alpha, learning rate (jo tune set kiye)
- **Metrics** — accuracy, RMSE (jo results aaye)
- **Artifacts** — model file, plots, datasets
- **Metadata** — time, code version, status

Phir UI me sab compare karke best chunte.

**Networking analogy:** Tracking = **syslog + monitoring dashboard** for ML experiments. Har run log hota timestamp ke saath, baad me analyze/compare.

---

## Layer 2: Experiment vs Run (Interview me GUARANTEED)

### Experiment = logical group (folder)
Ek project ka container. "Wine Quality Prediction" = ek experiment.

### Run = ek single execution (folder ke andar entry)
Ek experiment me **multiple runs**. Har run = ek baar code chalaya, ek params set ke saath.

```
Experiment: "Wine Quality Prediction"
├── Run 1: alpha=0.7 → RMSE=0.82
├── Run 2: alpha=0.5 → RMSE=0.75   ← best!
└── Run 3: alpha=0.1 → RMSE=0.79
```

**Networking analogy:** Experiment = ek project (e.g., "BGP migration"). Run = us project ke andar har individual config-push with different settings.

---

## Layer 3: Basic Tracking Code (Hands-on)

```python
import mlflow
import mlflow.sklearn
from mlflow.models import infer_signature
from sklearn.linear_model import ElasticNet
from sklearn.metrics import mean_squared_error

# 1. Train (with-block ke BAHAR — best practice)
lr = ElasticNet(alpha=alpha, l1_ratio=l1_ratio, random_state=42)
lr.fit(train_x, train_y)

# 2. Evaluate
predicted = lr.predict(test_x)
rmse = mean_squared_error(test_y, predicted, squared=False)

# 3. Experiment set (nahi hai to banega)
exp = mlflow.set_experiment(experiment_name="wine_quality")

# 4. Signature (input/output schema)
signature = infer_signature(train_x, lr.predict(train_x))

# 5. Run (with-block = auto start/end)
with mlflow.start_run(experiment_id=exp.experiment_id):
    mlflow.log_param("alpha", alpha)
    mlflow.log_param("l1_ratio", l1_ratio)
    mlflow.log_metric("rmse", rmse)
    mlflow.sklearn.log_model(
        sk_model=lr,
        artifact_path="wine_model",
        signature=signature,
        input_example=train_x[:2],
    )
```

| Code | Kya karta |
|------|-----------|
| `set_experiment(name)` | Experiment banao/select |
| `start_run()` | Run shuru (auto-end in with) |
| `log_param(k, v)` | Ek hyperparameter |
| `log_metric(k, v)` | Ek result metric |
| `log_model(...)` | Model artifact save |
| `infer_signature(...)` | Schema auto-detect |

> **Best practice:** `fit()` with-block ke bahar — fail hone pe corrupt run na bane.

---

## Layer 4: mlruns Folder (Local Store)

```
mlruns/
├── 0/                    # default exp (ignore)
├── 99xxx/                # tera experiment
│   ├── meta.yaml         # exp info
│   ├── 8c3xxx/           # ek RUN
│   │   ├── meta.yaml     # run info
│   │   ├── artifacts/    # model.pkl, MLmodel, conda.yaml, requirements.txt
│   │   ├── metrics/      # 1 file per metric
│   │   ├── params/       # 1 file per param
│   │   └── tags/         # metadata
│   └── 6bdxxx/           # doosra run
└── models/               # registry (agar register kiya)
```

**Insights:**
- Har run = ek folder
- `artifacts/` me sab kuch jo model re-create ko chahiye
- Local hai — production me S3/DB remote (File 05)
- log ≠ register (registry alag, File 09)

> **Gotcha:** `mlruns/` ko `.gitignore` me daalo. Data hai, code nahi.

---

## Layer 5: MLflow UI

```bash
mlflow ui   # http://127.0.0.1:5000
```

Tabs: **Experiments** (runs) + **Models** (registered).

Kar sakta:
- Runs ke params/metrics dekhna, filter/sort
- 2+ runs "Compare" → parallel/scatter/box/contour plots
- CSV download
- Run kholke artifacts + register

**Analogy:** UI = **Grafana** for experiments.

---

## Layer 6: Server Teaser

Abhi server nahi start kiya — library ne local files banaye. Production me:
```bash
mlflow server --host 127.0.0.1 --port 8080
```
```python
mlflow.set_tracking_uri("http://127.0.0.1:8080")
```

- **Bina server:** library files banata (local)
- **Server ke saath:** library REST se server se baat karta

Poori detail File 05 me (architect-level).

---

## 🎤 Interview Q&A

**Q1: Experiment vs Run?**
> "Experiment logical group/container; Run ek single execution us experiment me apne params/metrics ke saath. Ek exp me multiple runs — compare karke best."

**Q2: Kya log hota?**
> "Parameters (hyperparams), Metrics (results), Artifacts (model, plots), plus tags/metadata."

**Q3: mlruns folder?**
> "Local store — har exp ek folder, har run sub-folder with params/metrics/tags/artifacts. Production me S3/DB remote."

**Q4: log vs register?**
> "log_model = artifacts me save. register = central Registry me version. Alag cheezein."

**Q5: with start_run() kyun?**
> "Context manager — auto start/end, exception-safe. Training bahar rakhte taaki corrupt run na bane."

---

## ⚠️ Gotchas

1. `mlruns/` local by default — team/prod me remote chahiye (F05)
2. Training with-block ke andar + crash = corrupt run. Bahar rakho.
3. Same experiment name = existing use hoga (naya nahi)
4. `log_param` ek baar/run; `log_metric` multiple (step ke saath, epochs)

---

## Next: File 03 — Logging Functions (Deep)
