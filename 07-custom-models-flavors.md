# Topic 7: MLflow Custom Models & Flavors (Complete Deep Dive)

> **Prerequisites:** File 06 (Model Component, pyfunc, flavors)
> **Source:** mxagar/mlflow_guide §9 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "Custom models tab use karte jab MLflow tumhare framework ko support nahi karta ya tumhara apna custom Python logic hai — hum mlflow.pyfunc.PythonModel se ek wrapper class banate. Custom flavors tab jab apni serialization method chahiye. Dono MLflow ko extend karte non-standard models ke liye."

---

## Layer 1: Kya Aur Kyun?

File 06 me built-in flavors dekhe (sklearn, pytorch). Par kabhi:
- **Framework unsupported** — MLflow tumhari library nahi jaanta
- **Custom logic** — model ke saath pre/post-processing, multiple models, business rules

Tab **custom model** banate — ek Python wrapper jo MLflow samajh le.

**Do concepts:**
- **Custom model** = apni model library/logic wrap karna
- **Custom flavor** = apni serialization method (advanced, rarely needed)

**Networking analogy:** Built-in flavor = standard protocol (BGP, OSPF) — device jaanta hai. Custom model = **custom/proprietary protocol** — tumhe khud encapsulation/handling define karni padti taaki system samajh le.

---

## Layer 2: Custom Python Model (pyfunc)

Steps:
1. Artifacts (model, data) locally dump karo
2. Unke paths ek `artifacts` dict me
3. `mlflow.pyfunc.PythonModel` se derive karke wrapper class
4. Conda env dict
5. `mlflow.pyfunc.log_model()` se log

```python
import mlflow
import joblib

# 1. Model artifact dump
model_path = "models/model.pkl"
joblib.dump(lr, model_path)

# 2. Artifacts dict (paths)
artifacts = {"model": model_path, "data": "data"}

# 3. Wrapper class — MLflow ka PythonModel extend
class ModelWrapper(mlflow.pyfunc.PythonModel):
    def load_context(self, context):
        # Model load — context.artifacts se path milta
        self.model = joblib.load(context.artifacts["model"])

    def predict(self, context, model_input):
        # Custom predict logic (yahan pre/post-processing daal sakte)
        return self.model.predict(model_input.values)

# 4. Conda env
conda_env = {
    "channels": ["conda-forge"],
    "dependencies": [
        f"python=3.10",
        "pip",
        {"pip": ["mlflow", "scikit-learn", "cloudpickle"]},
    ],
    "name": "my_env",
}

# 5. Log custom model
mlflow.pyfunc.log_model(
    artifact_path="custom_model",
    python_model=ModelWrapper(),
    artifacts=artifacts,
    code_path=[__file__],   # code files (local dir me hone chahiye)
    conda_env=conda_env,
)
```

### 2 must-know methods:
- **`load_context(self, context)`** — model/artifacts load karna (deploy pe ek baar chalta)
- **`predict(self, context, model_input)`** — prediction logic (har request)

### Loading + prediction:
```python
loaded = mlflow.pyfunc.load_model(model_uri=f"runs:/{run_id}/custom_model")
predictions = loaded.predict(test_x)
```

**Key:** `mlflow.log_param/metric` use karo (NOT `mlflow.sklearn.*`) — kyunki custom model hai, sklearn flavor nahi.

---

## Layer 3: Custom Flavors (Advanced — awareness)

Custom flavor = apni serialization/deserialization method. **Rarely needed** — advanced topic, MLflow extend karna padta.

Steps (high-level):
- Serialization + deserialization logic define
- Flavor directory structure banao
- Custom flavor register karo
- Custom `save_model` / `load_model` implement

> **Honesty note (interview):** Custom flavors rarely use hota. Interview me "aware hoon, pyfunc custom model 95% cases cover karta, custom flavor sirf jab bahut specific serialization chahiye" — yeh bolo. Deep mat jao unless expert role.

---

## Layer 4: Use-Cases (Interview me example dena)

Custom model kab real me:
1. **Multiple models ek saath** — ensemble, ek wrapper me kai models
2. **Pre/post-processing** — input transform, output formatting model ke saath bundle
3. **Business logic** — threshold, rules prediction ke saath
4. **Unsupported library** — koi niche framework

**Example:** RAG pipeline ko ek custom pyfunc model bana sakte — retrieval + LLM call ek `predict()` me wrap. (Tera RAG knowledge se connect!)

---

## 🎤 Interview Q&A

**Q1: Custom model kab banate?**
> "Jab MLflow framework support nahi karta, ya custom logic chahiye — pre/post-processing, multiple models, business rules. mlflow.pyfunc.PythonModel wrapper banate load_context + predict methods ke saath."

**Q2: load_context vs predict?**
> "load_context deploy pe ek baar chalta — model/artifacts load. predict har request pe — actual prediction logic (custom transforms yahan)."

**Q3: Custom model me sklearn.log_model kyun nahi?**
> "Custom model sklearn flavor nahi — pyfunc.log_model use karte. Aur mlflow.log_param/metric (generic), mlflow.sklearn.* nahi."

**Q4: Custom flavor vs custom model?**
> "Custom model = apni model logic wrap (common, pyfunc). Custom flavor = apni serialization method (rare, advanced). 95% cases custom pyfunc model kaafi."

---

## ⚠️ Gotchas

1. **code_path files local dir me** — warna log fail
2. **Generic mlflow.log_* use karo** custom model me, framework-specific nahi
3. **conda_env accurate rakho** — deploy pe yahi env banega, galat versions = crash
4. **Custom flavor avoid** unless truly needed — pyfunc model prefer

---

## Next: File 08 — Model Evaluation
