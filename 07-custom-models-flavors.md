# Topic 7: MLflow Custom Models & Flavors

## Kahani
Kabhi tumhara "model" sirf ek sklearn object nahi — usme pre-processing + model + post-processing sab hota. Isko MLflow me **custom pyfunc model** se wrap karte — ek deployable unit.

## Points
```python
class MyModel(mlflow.pyfunc.PythonModel):
    def predict(self, context, model_input):
        # custom logic: preprocess → model → postprocess
        return result
```
- **Custom flavor** — jab standard flavors (sklearn/pytorch) kaafi nahi, apna banao.
- Use case: ensemble models, custom preprocessing, business logic model ke saath.

## Interview one-liner
> "For models that aren't a plain sklearn/pytorch object — say preprocessing + model + postprocessing — you wrap them in a custom mlflow.pyfunc PythonModel, giving one deployable unit with your own predict logic."

## Q&A
**Q: Custom pyfunc kab?** — "Jab model + preprocessing + postprocessing sab ek saath deploy karna ho, ya ensemble/business logic — standard flavor kaafi nahi tab pyfunc wrap."
