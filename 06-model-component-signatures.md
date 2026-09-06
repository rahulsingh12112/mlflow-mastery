# Topic 6: MLflow Models & Signatures ⭐⭐⭐

## Kahani
Model log karte waqt MLflow use ek **standard format** me pack karta taaki kahin bhi deploy ho. Aur ek important cheez — **signature** — model ka input/output schema. Lab me signature na dene pe warning aayi thi.

## Points
```
1. MLflow Model format — standard packaging (MLmodel file + weights + env)
                         kisi bhi framework, same tarika deploy
2. Signature — input/output schema (columns, types)
   infer_signature(X, preds)  # auto nikal leta
   → serving pe galat input → clean error (validation)
3. Flavors — ek model multiple flavors (sklearn flavor + pyfunc flavor)
   pyfunc = universal flavor, kahin bhi load
4. input_example — sample input (documentation + auto-signature)
```
**Kyun (production):** signature = contract. Bina iske galat input pe model random behave karega. Isliye lab me warning aayi thi.

## Interview one-liner
> "MLflow packages models in a standard format (flavors like sklearn and the universal pyfunc) so they deploy anywhere. A signature captures the input/output schema so serving validates inputs and fails cleanly on mismatch — always log with a signature and input_example in production."

## Q&A
**Q: Signature kya, kyun?** — "Model ka input/output schema (kya columns in, kya out). Serving pe input validate hota, galat input pe clean error. Production best practice."
**Q: Flavor kya?** — "Ek model multiple formats me save — sklearn flavor + universal pyfunc. pyfunc kahin bhi load hota."
