# Topic 2: MLflow Tracking — Experiments & Runs

> Teacher-session style (flowing + lab). Learner: Rahul. Style: kahani → points → diagram → interview one-liner → lab (production-style) → Q&A.
> Prereq: MLflow ke 4 components (Tracking, Models, Registry, Projects).

---

## Kahani — Kya + Kyun

Socho ek data scientist roz 20-30 baar model train kar raha hai — har baar thoda alag setting (learning rate badla, data badla). Kuch din baad wo bhool jaata hai "kaunsi setting pe best accuracy aayi thi." Ye chaos har ML team me hota — koi record nahi. **Tracking** isi ko fix karta: har training **run** ko automatically record karta — "is run me lr 0.01 tha, 10 epochs, accuracy 92%." Agle din 50 runs ho chuke, MLflow UI me sab ek table me, sort/compare karke "best kaunsa" bol sakte ho.

Ek zaroori structure — **Experiment** aur **Run** ka rishta. Ek **Experiment** = ek project/problem ("fraud-detection-model"). Uske andar bahut **Runs** = har baar train karne pe ek naya run. Matlab **Experiment = folder, Run = us folder ke andar ek attempt**. Grouping isliye taaki ek project ke saare attempts ek jagah rahein, alag projects mix na hon.

---

## Points

### 1. Run = ek training execution
Ek run me log hota:
- **Parameters** — input settings (learning_rate, epochs, batch_size). Fixed inputs.
- **Metrics** — output results (accuracy, loss, f1). Time ke saath change ho sakte (per epoch).
- **Artifacts** — files (trained model, plots, confusion matrix).
- **Tags** — metadata (team, git-commit, data-version).

### 2. Experiment = runs ka group
- Experiment = ek problem/project. Run = us project ka ek attempt.
- Sab runs ek experiment ke andar → UI me compare.
- Production: **naam se experiment banao** (`set_experiment("...")`), default nahi.

### 3. Tracking URI (data kahan jaaye)
- `mlflow.set_tracking_uri(...)` — batata data kahan store ho.
- Local: files (`./mlruns`) ya SQLite.
- Production: **remote tracking server** (backend DB + artifact store), team share kare (Topic 5).

---

## Diagram

```
   EXPERIMENT: "wine-classifier"
   ├── Run 1: params{n_est:100, depth:5}  → metrics{acc:0.89} → artifact{model}
   ├── Run 2: params{n_est:200, depth:10} → metrics{acc:0.92} ← BEST
   ├── Run 3: params{n_est:50,  depth:3}  → metrics{acc:0.85}
   └── ... (UI me compare, sort by accuracy)

   Experiment = folder (project) | Run = ek attempt (training execution)
```

---

## Interview one-liner
> "Tracking logs each run's parameters, metrics, and artifacts under an experiment, which groups related runs — so an experiment is a project and each run is one attempt, compared side-by-side in the UI. The tracking URI decides where data goes: local files for dev, a remote tracking server with DB backend + artifact store for teams."

---

## 🧪 LAB (production-style)

### Setup (ek baar)
```bash
pip3 install mlflow scikit-learn pandas
mlflow --version
mkdir -p ~/mlflow-lab && cd ~/mlflow-lab
```

### train.py (production-grade: signature + input example)
```python
import mlflow
import mlflow.sklearn
from mlflow.models.signature import infer_signature
from sklearn.datasets import load_wine
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

mlflow.set_experiment("wine-classifier")   # naam se, default nahi

X, y = load_wine(return_X_y=True)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=42)

with mlflow.start_run(run_name="rf-baseline"):
    n_estimators = 100
    max_depth = 5

    mlflow.log_param("n_estimators", n_estimators)   # 1. params
    mlflow.log_param("max_depth", max_depth)

    model = RandomForestClassifier(n_estimators=n_estimators, max_depth=max_depth, random_state=42)
    model.fit(X_tr, y_tr)
    preds = model.predict(X_te)
    acc = accuracy_score(y_te, preds)

    mlflow.log_metric("accuracy", acc)               # 2. metric

    signature = infer_signature(X_te, preds)         # 3. artifact (production-grade)
    mlflow.sklearn.log_model(model, name="model", signature=signature, input_example=X_te[:2])

    print(f"Run done — accuracy: {acc:.3f}")
```

### Chalao (3 runs, alag settings)
```bash
python3 train.py                          # baseline
# edit: n_estimators=200, max_depth=10 → python3 train.py
# edit: n_estimators=50,  max_depth=3  → python3 train.py
```

### UI (⚠️ IMPORTANT — usi folder se jahan train chalा)
```bash
cd ~/mlflow-lab          # jis folder me mlruns bana, wahi se ui
mlflow ui                # http://127.0.0.1:5000
```

### Kya dekhna
- "wine-classifier" experiment → 3 runs → accuracy se sort → best
- Run kholo → Artifacts me model + **signature** (13 wine features in, class out)

---

## ⚠️ GOTCHAS (lab me pakdi)
1. **UI galat folder → sirf "Default" dikhta.** `mlflow ui` usi folder se chalao jahan `mlruns` bana. Warna alag khaali mlruns dekhta. (Production fix: remote tracking server / fixed SQLite backend — Topic 5.)
2. **`log_model(model, "model")` deprecated** → `name="model"` use karo.
3. **Signature na diya → warning.** Production me hamesha `signature` + `input_example` do (serving pe input schema validate hota).
4. **accuracy 1.000 = red flag** real world me (overfitting/leakage). Wine dataset chhota isliye theek, par production me check karo.
5. `NotOpenSSLWarning` (LibreSSL) = macOS system ka, harmless, ignore.

### Fixed-backend trick (production-style, folder se farak nahi)
```bash
mlflow server --backend-store-uri sqlite:///$HOME/mlflow.db \
              --default-artifact-root $HOME/mlartifacts \
              --host 127.0.0.1 --port 5000
# train.py me: mlflow.set_tracking_uri("http://127.0.0.1:5000")
```

---

## Q&A (interview)
**Q: Experiment vs Run?** — "Experiment = project/problem (group), Run = ek training attempt uske andar. Ek experiment me sau runs compare karte."

**Q: Ek run me kya log hota?** — "Parameters (input settings), Metrics (output results), Artifacts (model/plots), Tags (metadata). Params fixed, metrics change ho sakte per-step."

**Q: Tracking data kahan jaata?** — "Tracking URI decide karta — local ./mlruns files, ya remote tracking server (DB backend + artifact store) team ke liye."
