# Topic 10: MLflow Projects (Complete Deep Dive)

> **Prerequisites:** File 02 (runs)
> **Source:** mxagar/mlflow_guide §12 + MLflow official docs

---

## 🎯 One-Liner (Interview)

> "MLflow Projects code ko ek standard format (MLproject YAML) me package karta jisme environment (conda/virtualenv/docker), entry points, aur parameters define hote — taaki koi bhi, kahin bhi (local ya remote git), same tarike se reproducibly chala sake. `mlflow run` se execute."

---

## Layer 1: Kya Aur Kyun?

Problem: tune training code likha, par doosra banda chalaye to:
- Kaunsa environment? Kaunse dependencies?
- Kaise chalaye? Kaunse parameters?

**MLflow Projects** yeh standardize karta — ek `MLproject` file me sab define. Phir ek command se koi bhi chala sake, same environment me.

**Yeh "Projects" component hai — File 01 ke 4 components me se ek (reproducibility).**

**Networking analogy:** MLproject = **Ansible playbook / Terraform module**. Standardized, parameterized, reproducible execution. Koi bhi run kare, same result.

---

## Layer 2: MLproject File

Project folder me `MLproject` (YAML, no extension):

```yaml
name: "Wine Quality Prediction"

# Environment (teen options — neeche)
conda_env: conda.yaml

# Entry points — kahan se execution shuru
entry_points:
  main:
    command: "python train.py --alpha={alpha} --l1_ratio={l1_ratio}"
    parameters:
      alpha:
        type: float
        default: 0.4
      l1_ratio:
        type: float
        default: 0.4
```

**3 parts:**
- **name** — project ka naam
- **environment** — dependencies (conda/virtualenv/docker)
- **entry_points** — commands + parameters

---

## Layer 3: Environment — 3 Options

```yaml
# 1. Virtualenv (MLflow prefer karta)
python_env: files/config/python_env.yaml

# 2. Conda
conda_env: files/config/conda.yaml

# 3. Docker (pre-built ya build)
docker_env:
  image: mlflow-example:1.0           # pre-built
  # image: 012345.dkr.ecr.us-west-2.amazonaws.com/mlflow-example:7.0  # ECR se
  volumes: ["/local/path:/container/path"]
  environment: [["NEW_VAR", "value"], "VAR_FROM_HOST"]
```

**Docker option tere liye kaam ka** — production/K8s me containerized reproducibility. ECR image reference kar sakte.

---

## Layer 4: Entry Points & Parameters

**Entry point** = execution start point:
- **name** (one entry = usually `main`)
- **command** — script call with `{placeholders}`
- **parameters** — name, default, type (string/float/path/uri — types validated)

Multiple entry points ho sakte (e.g., `data_prep`, `train`, `evaluate`).

---

## Layer 5: Running Projects

### CLI se:
```bash
# URI = local path (.) ya remote git repo
mlflow run . -P alpha=0.3 -P l1_ratio=0.3

# Remote git directly!
mlflow run https://github.com/user/repo.git -P alpha=0.5

# Options
mlflow run . \
  --entry-point main \
  --experiment-name "wine" \
  --env-manager conda \       # local/virtualenv/conda
  --backend local             # local/databricks/kubernetes
```

### Python API se:
```python
import mlflow

mlflow.projects.run(
    uri=".",
    entry_point="main",
    parameters={"alpha": 0.3, "l1_ratio": 0.3},
    experiment_name="wine",
    env_manager="conda",
)
```

**Killer feature (interview):** `mlflow run <git-url>` — **remote git repo directly chala sakte**, environment auto-setup. Ultimate reproducibility.

### Backend options:
- `local` — apni machine
- `databricks` — Databricks cluster
- `kubernetes` — K8s pe (tere liye relevant!)

---

## Layer 6: Environment Variables

```bash
MLFLOW_TRACKING_URI       # kahan track
MLFLOW_EXPERIMENT_NAME
MLFLOW_EXPERIMENT_ID
```
Set kar do to defaults use honge.

---

## 🎤 Interview Q&A

**Q1: MLflow Projects kya, kyun?**
> "Code ko MLproject YAML me package — environment + entry points + parameters. Reproducible execution: koi bhi, kahin bhi, same tarike se chala sake. mlflow run se execute, even remote git repo directly."

**Q2: MLproject file me kya?**
> "name, environment (conda/virtualenv/docker), entry_points (command + typed parameters). Standardized reproducible run definition."

**Q3: mlflow run ke backends?**
> "local (apni machine), databricks (cluster), kubernetes (K8s). Same project alag backends pe."

**Q4: Reproducibility kaise ensure karta?**
> "Environment (conda/docker) + entry points + typed params sab MLproject me. Remote git se bhi chala sakte with auto env setup — exact same execution guaranteed."

**Q5: Docker env kab?**
> "Production/K8s me — containerized reproducibility, ECR image reference. Consistent runtime across environments."

---

## ⚠️ Gotchas

1. **conda.yaml platform-generic rakho** — build numbers hata do (cross-platform issue)
2. **MLproject file me extension nahi** — plain `MLproject`
3. **Parameter types validated** — galat type = error (good)
4. **Remote git run** — repo public/accessible hona chahiye

---

## Next: File 11 — MLflow Client & CLI
