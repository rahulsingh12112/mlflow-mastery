# Topic 11: MLflow Client & CLI

## Kahani
Ab tak `mlflow.log_*` (high-level) use kiya. **MlflowClient** = low-level API jab fine control chahiye (registry manage, runs query, alias set). Aur **CLI** = terminal se same kaam — jo CI/CD me wire hota.

## Points
```python
# MlflowClient (Python) — programmatic control
client.set_registered_model_alias(...)   # alias (registry lab me kiya)
client.search_runs(...)                  # runs query/filter
client.create_registered_model(...)
```
```bash
# CLI (terminal)
mlflow ui / mlflow server        # UI/server
mlflow runs list
mlflow models serve -m ...       # serve (lab me kiya)
mlflow run .                     # project run
```
CLI CI/CD pipeline me wire hota (deploy/rollback scripts).

## Interview one-liner
> "The high-level mlflow.* functions cover logging, but MlflowClient is the low-level API for programmatic control — managing the registry, setting aliases, querying runs — and the CLI (mlflow server, mlflow models serve, mlflow run) does the same from the terminal, which is what you wire into CI/CD."

## Q&A
**Q: mlflow.* vs MlflowClient?** — "mlflow.* = high-level logging (log_param/metric). MlflowClient = low-level control (registry, aliases, run queries). CLI = terminal, CI/CD me wire hota."
