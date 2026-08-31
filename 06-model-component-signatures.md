# Topic 6: MLflow Model Component & Signatures (Complete Deep Dive)

> **Prerequisites:** File 02, 05
> **Source:** mxagar/mlflow_guide §8 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "MLflow Model Component model ko ek standard, self-contained format me package karta — model binary + dependencies + signature (input/output schema) + flavor (serialization method). Isse model kisi bhi environment me reproducibly deploy hota — REST API, Docker, batch. Signature schema enforce karta (Pydantic jaisa), flavor batata kaunse framework se model bana."

---

## Layer 1: Model Component — Kya Aur Kyun?

Problem: tune model train kiya (`model.pkl`). Par deploy karne ko sirf pkl kaafi nahi:
- Kaunsa Python version? Kaunsi library versions?
- Input kya format me? Output kya?
- Kaunse framework se bana (sklearn? PyTorch?)?

MLflow Model = **model + yeh saari info ek package me** (ONNX jaisa idea). Reproducible + portable + deployable.

**Networking analogy:** Model package = **firmware image with manifest**. Sirf binary nahi — version, dependencies, compatibility info bhi. Kisi bhi compatible device pe deploy ho.

---

## Layer 2: 4 Cheezein Model Component Me

```
MLflow Model
├── Storage Format  → kaise package/save (directory/file/docker)
├── Signature       → input/output types & shapes
├── API             → standard REST interface (online/batch)
└── Flavor          → serialization method (framework-specific)
```

---

## Layer 3: Storage Format — Files

Jab `mlflow.log_model()` karo, ek folder banta with:

```
wine_model/
├── MLmodel              # ★ YAML — most important, model describe karta
├── model.pkl            # serialized model binary
├── conda.yaml           # conda environment
├── python_env.yaml      # virtualenv requirements
├── requirements.txt     # pip dependencies
└── input_example.json   # sample input rows
```

**`MLmodel` file = dil** — yeh batata kaunsa flavor, signature kya, kaise load karna. MLflow isko padhke model reconstruct karta.

**Key insight (interview):** Yeh sab files ensure karti hain ki **model + uska environment dono reproducible** hain. Naya machine pe same env bana ke model chala sakte.

---

## Layer 4: Signature — Input/Output Schema (IMPORTANT)

Signature = model ke input/output ki **schema** (types + shapes). Pydantic jaisa validation.

### Kyun zaroori?
- Deploy pe galat input aaye to **pehle hi pakad le** (production me crash na ho)
- Documentation — koi bhi dekhe model kya expect karta

### Do tarike:

**1. Manual (usually recommended nahi):**
```python
from mlflow.models.signature import ModelSignature
from mlflow.types.schema import Schema, ColSpec

input_schema = Schema([
    ColSpec("double", "fixed acidity"),
    ColSpec("double", "alcohol"),
])
output_schema = Schema([ColSpec("long")])
signature = ModelSignature(inputs=input_schema, outputs=output_schema)
```

**2. Inferred (PREFERRED — recommended):**
```python
from mlflow.models import infer_signature

signature = infer_signature(X_test, model.predict(X_test))
mlflow.sklearn.log_model(model, "model", signature=signature)
```
Auto-detect input/output schema — kam galti, easy.

### Signature Enforcement Levels:
- **Signature enforcement:** type + name check
- **Name-ordering:** column order fix
- **Input-type:** types check + cast if needed

**Tensors:** deep learning me shape ka ek dimension `-1` hota (batch size — arbitrary).

**Networking analogy:** Signature = **interface schema / API contract**. Jaise ek API endpoint define karta kya payload aayega (types, required fields) — galat payload reject. Model signature same — galat input shape/type reject.

---

## Layer 5: Model API — Save, Log, Load

```python
# SAVE — sirf local directory me (do flavors: sklearn, pyfunc)
mlflow.save_model(sk_model, path, signature, input_example, ...)

# LOG — server ke artifacts me (local/remote), optionally register
mlflow.log_model(
    sk_model,
    artifact_path,               # artifact folder name
    registered_model_name=None,  # agar diya to register bhi
    signature=signature,
    input_example=input_example,
)

# LOAD — saved/logged model wapas
model = mlflow.pyfunc.load_model(model_uri)
# model_uri: runs:/<run_id>/model, s3://..., models:/<name>/<version>
```

**save vs log farak (interview):**
- `save_model` = hamesha **local** save
- `log_model` = **server** ke artifacts me (local ya remote S3), aur register bhi kar sakta

---

## Layer 6: Flavors — Serialization Methods

**Flavor** = model kaise serialize/store hua (framework-specific method).

Built-in flavors:
```python
mlflow.sklearn.log_model(...)
mlflow.pytorch.log_model(...)
mlflow.keras.log_model(...)
mlflow.xgboost.log_model(...)
```

**Special: `pyfunc` flavor** — universal Python function flavor. Koi bhi model ko generic Python function ki tarah load/use kar sakta:
```python
model = mlflow.pyfunc.load_model(uri)
model.predict(data)   # kisi bhi framework ka model, same interface
```

**Kyun pyfunc important:** deployment me tujhe framework ki fikar nahi — sab `pyfunc` interface se `.predict()`. Uniform.

**Networking analogy:** Flavor = **device driver**. Har hardware ka apna driver (serialization), par OS ek uniform interface deta (pyfunc). App ko underlying hardware ki fikar nahi.

---

## 🎤 Interview Q&A

**Q1: MLflow Model kya package karta?**
> "Model binary + dependencies (conda/pip) + signature (I/O schema) + flavor (serialization). Ek self-contained, reproducible, deployable package — REST/Docker/batch me chal sake."

**Q2: Signature kya, kyun zaroori?**
> "Input/output ki schema (types + shapes), Pydantic jaisa. Deploy pe galat input pehle pakad leta, production crash se bachata. infer_signature se auto-detect (recommended)."

**Q3: MLmodel file kya?**
> "Model package ki YAML — flavor, signature, load instructions describe karta. MLflow isse model reconstruct karta. Package ka dil."

**Q4: pyfunc flavor kyun special?**
> "Universal Python function interface — koi bhi framework ka model same .predict() se use hota. Deployment uniform, framework-agnostic."

**Q5: save_model vs log_model?**
> "save = local only. log = server artifacts (local/remote S3) + optional register. Production me log_model."

**Q6: Reproducibility MLflow kaise ensure karta?**
> "Model package me environment (conda.yaml, requirements.txt) + signature + model binary sab hota. Naya machine pe same env bana ke exact model chala sakte."

---

## ⚠️ Gotchas

1. **Signature optional par HIGHLY recommended** — bina signature production me silent wrong-input bugs
2. **infer_signature use karo** manual ke bajaye — kam galti
3. **input_example bhi do** — documentation + validation
4. **pyfunc = deployment ka default** — uniform interface

---

## Next: File 07 — Custom Models & Flavors
