# Topic 4: MLflow Autologging (Complete Deep Dive)

> **Prerequisites:** File 03 (Logging Functions)
> **Source:** mxagar/mlflow_guide §6 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "Autologging ek single call `mlflow.autolog()` hai jo automatically model ke parameters, metrics aur artifacts log kar deta hai — manually log_param/log_metric likhne ki zaroorat nahi. MLflow framework detect karke uske hisaab se sab capture karta."

---

## Layer 1: Kya Aur Kyun?

File 03 me humne **manually** har param/metric log kiya. Bada tedious — 20 hyperparameters ho to 20 log_param lines.

**Autolog** yeh automate karta. Ek line, model definition ke pehle:
```python
mlflow.autolog()
```
Bas — ab MLflow **khud** sab detect karke log karta: params, metrics, model, even training curves.

**Networking analogy:** Manual logging = har interface ka counter haath se note karna. Autolog = **SNMP auto-discovery** — device connect karo, sab metrics khud collect hone lage.

---

## Layer 2: Generic Autolog

```python
import mlflow

mlflow.autolog()   # framework auto-detect

# Ab normal training — sab auto-log hoga
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)   # params + metrics + model, sab log!
```

### Parameters of autolog():
```python
mlflow.autolog(
    log_models=True,              # model log kare ya nahi
    log_input_examples=False,     # input example log
    log_model_signatures=True,    # signature (schema)
    log_datasets=False,
    disable=False,                # sab auto-logging band
    exclusive=False,              # True: autolog content user-run me na jaaye
    disable_for_unsupported_versions=False,
    silent=False,                 # warnings suppress
)
```

---

## Layer 3: Framework-Specific Autolog

Generic ke alawa, specific library ka autolog bhi:
```python
mlflow.sklearn.autolog()
mlflow.keras.autolog()
mlflow.xgboost.autolog()
mlflow.pytorch.autolog()
mlflow.spark.autolog()
mlflow.statsmodels.autolog()
```

Framework-specific me **5 extra params**:
```python
mlflow.sklearn.autolog(
    max_tuning_runs=5,              # hyperparam search me max child runs
    log_post_training_metrics=True, # post-training metrics
    serialization_format='cloudpickle',
    registered_model_name=None,     # auto-register with this name
    pos_label=None,                 # binary classification positive class
)
```

**Killer feature:** `max_tuning_runs` — agar GridSearchCV use kar rahe, autolog har hyperparameter combination ka **child run** bana deta automatically. Manual me yeh nightmare hota.

---

## Layer 4: Autolog + Manual Mix

Kabhi autolog on rakhna par kuch manually control karna:
```python
mlflow.autolog(log_models=False)  # model manually log karunga

model.fit(X, y)                   # params/metrics auto

# Model apni marzi se log
mlflow.sklearn.log_model(model, "my_model", registered_model_name="prod-model")
```

---

## 🎤 Interview Q&A

**Q1: Autolog vs manual logging — kab kya?**
> "Autolog quick experimentation ke liye — ek line me sab capture. Manual jab fine control chahiye (specific metrics, custom artifacts, ya production pipeline me predictable logging). Production me often manual/hybrid — reproducibility aur clarity ke liye."

**Q2: Autolog kaise jaanta kya log karna?**
> "Framework detect karta (sklearn/PyTorch/XGBoost) aur uske known params/metrics ko hook karke capture karta. Har framework ka apna integration."

**Q3: GridSearchCV ke saath autolog?**
> "max_tuning_runs param se har hyperparameter combination ka child run auto-create hota — manual me har combo track karna painful, autolog handle karta."

**Q4: Autolog production me use karoge?**
> "Selectively. Quick experiments me haan. Production pipelines me main hybrid prefer karunga — explicit logging for critical metrics/reproducibility, autolog for convenience. Pure autolog production me kabhi unexpected cheezein log kar deta."

---

## ⚠️ Gotchas

1. **autolog() model definition ke PEHLE** — baad me lagaya to kaam nahi karega
2. **Unexpected content** — autolog kabhi zyada/kam log kar deta; production me verify karo
3. **exclusive=True** — agar chahte autolog content user-created runs me na mile
4. **Version mismatch** — purane framework versions me autolog fail ho sakta; `disable_for_unsupported_versions`

---

## Next: File 05 — Tracking Server Architecture (ARCHITECT-LEVEL, tere role ka core)
