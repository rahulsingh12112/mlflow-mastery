# Topic 8: MLflow Model Evaluation

## Kahani
Model train hone ke baad, **kitna acha hai** ye systematically check karna. MLflow ka `evaluate()` ye automate karta — metrics + plots + explanations ek saath.

## Points
```python
mlflow.evaluate(model, data, targets, model_type="classifier")
# auto: accuracy, f1, precision, recall, confusion matrix, ROC
# SHAP (feature importance — kaunsa feature kitna matter karta)
# custom metrics bhi add kar sakte
# BASELINE comparison — naya model purane se behtar? (validation gate)
```
- **Baseline compare** = deploy gate: naya model baseline se behtar tabhi promote (2.5 eval gate).
- ⚠️ Ye **offline/training time** hota — live monitoring NAHI (distinct).

## Interview one-liner
> "mlflow.evaluate() runs a full evaluation — classification/regression metrics, confusion matrix, ROC, even SHAP feature importance — and can compare against a baseline to act as a promotion gate. This is offline evaluation at training time, distinct from live monitoring."

## Q&A
**Q: evaluate() live monitoring hai?** — "Nahi — ye offline/training time evaluation (test set pe). Live monitoring alag tools (CloudWatch/Evidently)."
**Q: Baseline comparison kyun?** — "Deploy gate — naya model baseline se behtar tabhi promote, warna regression prod me na jaye."
