# Topic 10: MLflow Projects

## Kahani
"Mere laptop pe chala tha" problem. **Projects** = code + environment ko package karo taaki koi bhi, kahin bhi, same run reproduce kare.

## Points
```yaml
# MLproject file (YAML):
#   entry points  — kaunsa script chalाना
#   parameters    — run ke inputs
#   environment   — conda/docker/venv (dependencies)
```
```bash
mlflow run . -P alpha=0.5     # reproducible run (env auto setup)
mlflow run git://...          # seedha GitHub se run
```
- Reproducibility = 2.1 wala code+data+model versioning ka **code+env** side.

## Interview one-liner
> "An MLflow Project packages code, its entry points, parameters, and environment (conda/docker) via an MLproject file, so anyone can reproduce a run with `mlflow run` — even directly from a Git URL — solving the 'works on my machine' problem."

## Q&A
**Q: Projects kya solve karta?** — "Reproducibility — code + env package, koi bhi `mlflow run` se same run reproduce kare, chahe Git URL se."
