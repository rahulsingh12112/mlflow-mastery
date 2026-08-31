# Topic 8: MLflow Model Evaluation (Complete Deep Dive)

> **Prerequisites:** File 06 (pyfunc), File 03 (metrics)
> **Source:** mxagar/mlflow_guide §10 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "mlflow.evaluate() ek single call hai jo MLflow-packaged model pe automatically performance metrics, plots, aur explanations (SHAP feature importance) generate karke log kar deta — bina alag evaluation tools. Custom metrics/artifacts aur baseline comparison bhi support karta. pyfunc flavor pe kaam karta."

---

## Layer 1: Kya Aur Kyun?

Model train kiya — ab evaluate karna hai. Manually metrics compute karna tedious. `mlflow.evaluate()` yeh automate:
- **Metrics** — accuracy, RMSE, F1, etc. (model type ke hisaab se)
- **Plots** — confusion matrix, ROC, etc.
- **Explanations** — SHAP (feature importance — kaunsa feature kitna matter karta)

Sab automatically compute + log.

**Networking analogy:** mlflow.evaluate = **automated health-check suite**. Ek command, poora device diagnostic (latency, throughput, errors) + report. Manually har test nahi chalana.

---

## Layer 2: Basic Evaluate

```python
mlflow.evaluate(
    model,          # pyfunc model ya model URI
    data,           # eval data (DataFrame/np/Spark)
    targets="quality",  # target column
    model_type="regressor",  # 'regressor'/'classifier'/'question-answering'/'text'
    evaluators=["default"],  # default evaluator (SHAP use karta)
)
```

### model_type options:
- `regressor` — RMSE, MAE, R2
- `classifier` — accuracy, F1, precision, recall, ROC
- `question-answering`, `text`, `text-summarization` — LLM/NLP metrics

> **Note:** `default` evaluator SHAP use karta → `pip install shap` chahiye.

### Full example:
```python
model_uri = mlflow.get_artifact_uri("model")
mlflow.evaluate(
    model_uri,
    test,               # test data with target
    targets="quality",
    model_type="regressor",
    evaluators=["default"],
)
# Result: metrics + SHAP plots artifacts me auto-log
```

---

## Layer 3: Custom Metrics & Artifacts

Apni evaluation metrics/plots define kar sakte:

```python
from mlflow.models import make_metric
import numpy as np

# Custom metric function
# eval_df: data, builtin_metrics: mlflow ke built-in metrics dict
def squared_diff_plus_one(eval_df, _builtin_metrics):
    return np.sum(np.abs(eval_df["prediction"] - eval_df["target"] + 1) ** 2)

# Metric object banao
custom_metric = make_metric(
    eval_fn=squared_diff_plus_one,
    greater_is_better=False,   # lower = better
    name="squared_diff_plus_one",
)

# Custom artifact (plot) function
def prediction_scatter(eval_df, _builtin_metrics, artifacts_dir):
    import matplotlib.pyplot as plt
    plt.scatter(eval_df["target"], eval_df["prediction"])
    path = f"{artifacts_dir}/scatter.png"
    plt.savefig(path)
    return {"scatter_plot": path}

# Evaluate with custom
mlflow.evaluate(
    model_uri, test, targets="quality", model_type="regressor",
    evaluators=["default"],
    custom_metrics=[custom_metric],
    custom_artifacts=[prediction_scatter],
)
```

---

## Layer 4: Baseline Comparison (Production-critical)

Naya model **baseline se accha hai ya nahi** — thresholds se check:

```python
from mlflow.models import MetricThreshold

thresholds = {
    "mean_squared_error": MetricThreshold(
        threshold=0.6,           # MSE < 0.6 chahiye (max acceptable)
        min_absolute_change=0.1, # baseline se min 0.1 improvement
        min_relative_change=0.05,# min 5% relative improvement
        greater_is_better=False,
    )
}

mlflow.evaluate(
    model_uri, test, targets="quality", model_type="regressor",
    evaluators=["default"],
    validation_thresholds=thresholds,
    baseline_model=baseline_model_uri,   # compare against this
)
```

**Kyun important (interview):** Production me naya model deploy karne se pehle **automated gate** — agar baseline se better nahi ya threshold miss, deploy **reject**. Yeh CI/CD me model quality gate.

**Networking analogy:** Baseline comparison = **regression testing before deploy**. Naya config purane se better/equal na ho to rollout block. Quality gate.

---

## 🎤 Interview Q&A

**Q1: mlflow.evaluate() kya karta?**
> "Ek call me model pe metrics, plots, aur SHAP explanations auto-generate + log karta. model_type (regressor/classifier/text) ke hisaab se relevant metrics. Custom metrics/artifacts aur baseline comparison support."

**Q2: SHAP kya, kyun?**
> "SHAP feature importance batata — kaunsa input feature prediction me kitna contribute karta. Explainability/debugging ke liye. default evaluator SHAP use karta (pip install shap)."

**Q3: Baseline comparison kaise, kyun?**
> "validation_thresholds + baseline_model pass karte. Naya model baseline se defined threshold (absolute/relative improvement) meet kare tabhi accept. CI/CD me automated quality gate — bad model deploy na ho."

**Q4: Kaunse model par evaluate chalta?**
> "pyfunc flavor pe. Model URI ya pyfunc model pass karo with eval data + targets + model_type."

---

## ⚠️ Gotchas

1. **`pip install shap`** — default evaluator ko chahiye, warna fail
2. **pyfunc flavor** pe kaam karta — ensure model pyfunc compatible
3. **targets column** eval data me hona chahiye
4. **Baseline model bhi log/pyfunc** hona chahiye comparison ke liye

---

## Next: File 09 — Model Registry (⭐⭐⭐⭐ governance)
