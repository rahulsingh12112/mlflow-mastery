# Topic 3: MLflow Logging Functions (Complete Deep Dive)

> **Prerequisites:** File 02 (Tracking, Experiments, Runs)
> **Source:** mxagar/mlflow_guide §4 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "MLflow logging functions se hum ek run ke andar parameters, metrics, artifacts aur tags record karte hain — log_param/params (hyperparameters), log_metric/metrics (results, step ke saath time-series bhi), log_artifact/artifacts (files/models), aur set_tag/tags (metadata for grouping/filtering)."

---

## Layer 1: 4 Cheezein Log Hoti Hain

| Type | Function | Kya | Example |
|------|----------|-----|---------|
| **Parameters** | `log_param(s)` | Input hyperparameters (fixed) | alpha=0.5 |
| **Metrics** | `log_metric(s)` | Output results (numeric, changeable) | rmse=0.75 |
| **Artifacts** | `log_artifact(s)` | Files: model, plots, data | model.pkl |
| **Tags** | `set_tag(s)` | Metadata for organizing | version="v1" |

**Core farak (interview):**
- **Param** = INPUT jo tune diya (immutable in a run)
- **Metric** = OUTPUT jo aaya (mutable, time-series possible)

---

## Layer 2: Tracking URI — Data Kahan Jaaye

Logging ke pehle — decide karo data kahan store ho:

```python
# Set: kahan store karna
mlflow.set_tracking_uri(uri="")   # empty = local ./mlruns

# Get: current location
print(mlflow.get_tracking_uri())
```

### Possible URI values:
| Value | Matlab |
|-------|--------|
| `""` (empty) | Local `./mlruns` (default) |
| `"./my_folder"` | Custom local folder |
| `"file:/Users/.../path"` | Absolute file path |
| `"http://localhost:5000"` | Local server |
| `"https://my-server:5000"` | Remote server |
| `"databricks://"` | Databricks workspace |

**Networking analogy:** Tracking URI = **syslog server address**. Local file, ya remote collector — jaha logs bhejne hain woh point karta.

---

## Layer 3: Parameters Logging

```python
# Single param → returns logged value
mlflow.log_param("alpha", 0.5)

# Multiple params → no return
mlflow.log_params({"alpha": 0.5, "l1_ratio": 0.1})
```

- **Params immutable** ek run me — ek baar set, dobara nahi (error)
- Hyperparameters, config values yahan

---

## Layer 4: Metrics Logging

```python
# Single metric
mlflow.log_metric("rmse", 0.75)

# Multiple
mlflow.log_metrics({"mae": 0.5, "r2": 0.9})

# With STEP (time-series — deep learning epochs!)
for epoch in range(100):
    loss = train_one_epoch()
    mlflow.log_metric("loss", loss, step=epoch)
```

**Key: `step` parameter** — yeh metrics ko time-series banata. Deep learning me har epoch ka loss log karke UI me **curve** dikhta. Yeh param me nahi hota.

**Analogy:** Metric with step = **interface counter over time** (bandwidth graph). Param = static config value.

---

## Layer 5: Artifacts Logging

```python
# Single file
mlflow.log_artifact("data/train.csv", artifact_path="datasets")

# Multiple (poora directory)
mlflow.log_artifacts("data/", artifact_path="datasets")

# Artifact URI nikalna
uri = mlflow.get_artifact_uri()                    # current artifact dir
uri = mlflow.get_artifact_uri("datasets/train.csv") # specific file
```

Artifacts = koi bhi file: datasets, plots (confusion matrix, SHAP), images, configs.

> **Note:** Model log karne ko `log_artifact` nahi, `log_model` use karo (File 06) — woh signature/flavor bhi handle karta.

---

## Layer 6: Tags Logging

```python
mlflow.set_tag("version", "1.0")
mlflow.set_tags({"environment": "dev", "team": "ml-platform"})
```

Tags = metadata to **group/filter/search** runs. MLflow khud kuch internal tags auto-set karta (run name, git commit hash, filename).

**Use-cases:**
- `environment: prod/dev`
- `data_version: v3`
- `git_commit: abc123`

**Analogy:** Tags = **resource tags in AWS** (Environment, Team, CostCenter). Filtering/organizing ke liye.

---

## Layer 7: Full Example (Sab Ek Saath)

```python
import mlflow
import pandas as pd
from sklearn.model_selection import train_test_split

mlflow.set_tracking_uri("")   # local
exp = mlflow.set_experiment("wine_quality")

with mlflow.start_run(run_name="run_baseline"):
    # Params
    mlflow.log_params({"alpha": 0.5, "l1_ratio": 0.1})

    # Metrics
    mlflow.log_metrics({"rmse": 0.75, "r2": 0.85})

    # Artifacts — dataset log
    data = pd.read_csv("data/red-wine-quality.csv")
    train, test = train_test_split(data)
    train.to_csv("data/train.csv")
    test.to_csv("data/test.csv")
    mlflow.log_artifacts("data/")

    # Tags
    mlflow.set_tags({"version": "1.0", "environment": "dev"})
```

---

## 🎤 Interview Q&A

**Q1: Param vs Metric?**
> "Param = input hyperparameter, immutable in a run (alpha). Metric = output result, numeric, can be logged over steps for time-series (loss per epoch)."

**Q2: step parameter kya karta metric me?**
> "Metric ko time-series banata. Har epoch/iteration ka value step ke saath log karke UI me curve milta — training progress visualize."

**Q3: Artifact aur Model log me farak?**
> "log_artifact koi bhi file save karta. log_model model ko flavor + signature + environment ke saath package karta — deployment-ready. Model ke liye hamesha log_model."

**Q4: Tags kis kaam ke?**
> "Metadata for grouping/filtering runs — environment, data version, git commit. AWS resource tags jaise. Search/organize easy."

**Q5: log_param dobara same key?**
> "Error — params ek run me immutable. Metric dobara chalega (step ke saath)."

---

## ⚠️ Gotchas

1. **Param immutable, metric mutable** — yeh farak yaad rakh
2. **Model ke liye log_model, log_artifact nahi** — warna signature/flavor miss
3. **log_artifacts (plural) = directory**, log_artifact (singular) = file
4. **Tracking URI pehle set karo** — warna default local jaayega

---

## Next: File 04 — Autologging
