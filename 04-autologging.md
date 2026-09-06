# Topic 4: MLflow Autologging

## Kahani
Har param/metric haath se log karna tedious. **Autologging** = ek line, MLflow khud framework ke sab params/metrics/model automatically log kar deta.

## Points
```python
mlflow.autolog()            # universal (framework detect karke sab log)
mlflow.sklearn.autolog()    # framework-specific
mlflow.pytorch.autolog()
```
- Khud pakadta: hyperparameters, training metrics, model, even plots.
- Faayda: kam code, consistency. Nuksan: kabhi zyada log karta (control kam).

## Interview one-liner
> "mlflow.autolog() automatically logs params, metrics, and the model for supported frameworks with one line — less boilerplate, but you trade some control over exactly what's logged."

## Q&A
**Q: Autolog vs manual?** — "Autolog = one line, auto-captures everything (less control). Manual log_param/metric = explicit, full control. Autolog for speed, manual when you need precise control."
