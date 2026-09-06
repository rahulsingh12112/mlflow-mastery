# Topic 3: MLflow Logging Functions

## Kahani
Tracking me dekha run me params/metrics/artifacts log hote — ab exactly **kaise**. Har cheez ka apna function, simple.

## Points
```python
mlflow.log_param("lr", 0.01)          # ek parameter (input setting, fixed)
mlflow.log_metric("acc", 0.92)        # ek metric (result)
mlflow.log_metric("loss", 0.3, step=5)# step = epoch/iteration (metric over time → UI graph)
mlflow.log_artifact("plot.png")       # ek file (plot, data)
mlflow.set_tag("team", "ml-plat")     # metadata label
mlflow.log_params({...})              # batch (dict)
mlflow.log_metrics({...})             # batch metrics
```
**Key:** params = fixed input (ek baar). metrics = result (step ke saath change ho sakte, isliye UI me graph banta).

## Interview one-liner
> "log_param for fixed inputs, log_metric for results (optional step for per-epoch curves), log_artifact for files, set_tag for metadata; batch versions log dicts at once."

## Q&A
**Q: param vs metric?** — "Param = fixed input setting (lr, epochs). Metric = result (accuracy/loss), can have a step for per-epoch curves."
