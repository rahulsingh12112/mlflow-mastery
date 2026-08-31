# Topic 11: MLflow Client & CLI (Complete Deep Dive)

> **Prerequisites:** File 02, 09
> **Source:** mxagar/mlflow_guide §13-14 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "MlflowClient ek programmatic API hai jo UI ke saare operations code se karne deta — experiments/runs/models manage, search, delete, restore. MLflow CLI terminal se same cheezein + server management, artifacts, DB migration. Dono automation/scripting ke liye — CI/CD pipelines me kaam ke."

---

## Layer 1: MlflowClient — Kya Aur Kyun?

Ab tak humne `mlflow.log_param()` jaise **high-level** functions dekhe. `MlflowClient` **low-level, fine control** deta — UI ka sab kaam code se.

```python
from mlflow import MlflowClient

client = MlflowClient(tracking_uri="http://127.0.0.1:5000")
```

**Kyun (interview):** Automation. CI/CD pipeline me — model auto-register, best run search, purane experiments cleanup. UI manual hai; client programmatic.

**Networking analogy:** High-level API = router ke basic commands. MlflowClient = **full config API / NETCONF** — programmatic, granular control for automation scripts.

---

## Layer 2: Experiment Management

```python
# Create
exp_id = client.create_experiment("my_exp", tags={"version": "v1"})

# Get (by name ya id)
exp = client.get_experiment_by_name("my_exp")
exp = client.get_experiment(exp_id)

# Rename, tag
client.rename_experiment(exp_id, "new_name")
client.set_experiment_tag(exp_id, "framework", "sklearn")

# Delete + restore
client.delete_experiment(exp_id)
client.restore_experiment(exp_id)

# Search (powerful — filter strings)
from mlflow.entities import ViewType
exps = client.search_experiments(
    view_type=ViewType.ALL,   # ACTIVE_ONLY/DELETED_ONLY/ALL
    filter_string="tags.version = 'v1' AND tags.framework = 'sklearn'",
    order_by=["experiment_id ASC"],
)
```

---

## Layer 3: Run Management

```python
# Create run (start_run se alag — sirf object banata, code nahi chalta)
run = client.create_run(experiment_id=exp_id, tags={"Priority": "P1"},
                        run_name="my_run")

# Log via client (run_id explicitly dena)
client.log_param(run.info.run_id, "alpha", 0.3)
client.log_metric(run.info.run_id, "r2", 0.9)

# Metric history (time-series)
history = client.get_metric_history(run.info.run_id, "r2")

# Status change
client.set_terminated(run.info.run_id, status="FINISHED")

# Search runs
runs = client.search_runs(
    experiment_ids=["6", "10"],
    run_view_type=ViewType.ACTIVE_ONLY,
    filter_string="metrics.r2 > 0.8",
    order_by=["metrics.r2 DESC"],
)
```

**Killer use-case:** `search_runs` with filter — best model programmatically dhundo:
```python
best = client.search_runs(
    experiment_ids=[exp_id],
    order_by=["metrics.rmse ASC"],   # lowest rmse
    max_results=1,
)[0]
# Phir best ko register/deploy — CI/CD automation
```

---

## Layer 4: Model Registry via Client

```python
# Registered model create
client.create_registered_model("lr-model", tags={"framework": "sklearn"},
                               description="Elastic Net wine model")

# Version add
client.create_model_version(
    name="lr-model",
    source=f"runs:/{run_id}/model",
    tags={"framework": "sklearn"},
)

# Version tag
client.set_model_version_tag("lr-model", "1", "validated", "true")

# Get versions
mv = client.get_model_version("lr-model", "1")
latest = client.get_latest_versions("lr-model")

# Alias (champion — File 09)
client.set_registered_model_alias("lr-model", "champion", "1")
mv = client.get_model_version_by_alias("lr-model", "champion")

# Search + delete
mvs = client.search_model_versions("tags.framework = 'sklearn'")
client.delete_model_version("lr-model", "1")
```

---

## Layer 5: MLflow CLI

Terminal se — pehle server:
```bash
mlflow server
```

Useful CLI commands:
```bash
# Health check config (Python/MLflow version, URIs)
mlflow doctor
mlflow doctor --mask-envs        # secrets hide

# Artifacts
mlflow artifacts list --run-id <id>
mlflow artifacts download --run-id <id> --dst-path ./local
mlflow artifacts log-artifacts --local-dir ./data --run-id <id>

# DB schema upgrade (version migration)
mlflow db upgrade sqlite:///mlflow.db

# Experiments
mlflow experiments create --experiment-name my_exp
mlflow experiments csv --experiment-id <id> --filename out.csv   # export runs
mlflow experiments delete/restore/rename/search

# Runs
mlflow runs list --experiment-id <id> --view all
mlflow runs describe --run-id <id>       # JSON with all info
mlflow runs delete/restore --run-id <id>
```

**CI/CD relevant:** `mlflow experiments csv` (export for reporting), `mlflow db upgrade` (migration in deploy), `mlflow doctor` (health check in pipeline).

---

## 🎤 Interview Q&A

**Q1: MlflowClient vs high-level API?**
> "High-level (mlflow.log_*) quick logging. MlflowClient low-level, granular — UI ke saare ops code se: experiment/run/model manage, search, delete. Automation/CI-CD ke liye."

**Q2: Best model programmatically kaise dhundo?**
> "client.search_runs with filter_string (metrics.rmse < X) + order_by (rmse ASC) + max_results=1. Best run mil gaya, phir register/deploy — automated pipeline."

**Q3: create_run vs start_run?**
> "start_run active run banata (code chalta). client.create_run sirf run object banata (UNFINISHED), koi code nahi — jab run prepare karna par ML abhi nahi chalana. Status manually set karte."

**Q4: CLI CI/CD me kaise?**
> "mlflow doctor (health check), mlflow db upgrade (schema migration in deploy), mlflow experiments csv (export/reporting), artifacts commands (download/upload). Pipeline automation."

---

## ⚠️ Gotchas

1. **Client me run_id explicitly dena** — high-level me implicit active run, client me manual
2. **create_run != start_run** — client wala code nahi chalata
3. **filter_string SQL-like** — `metrics.x > 0.8`, `tags.env = 'prod'` (backticks for special chars)
4. **mlflow db upgrade** — DB schema version mismatch pe zaroori (deploy me)

---

## Next: File 12 — AWS Integration & Production (⭐⭐⭐⭐⭐ TERA STACK)
