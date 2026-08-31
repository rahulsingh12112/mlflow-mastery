# MLflow Mastery — Rahul ka Complete Interview Reference

> **Target Role:** AI Infrastructure Architect / Senior ML Platform Engineer (₹1Cr)
> **Learner:** Rahul — 15 yrs infra (8 networking: BGP/OSPF/MPLS, 7 AWS CSE), code padh ke samajhta hai
> **Purpose:** Yeh mera FIRST aur LAST MLflow reference hai. Isi se interview dunga.
> **Source:** github.com/mxagar/mlflow_guide + official MLflow docs + production knowledge
> **Style:** Hinglish, Layer-by-Layer, har topic pe interview one-liner + networking analogy

---

## Padhne ka tarika (IMPORTANT)

Har file ka structure:
1. **One-Liner (Interview)** — yaad karne ke liye ek line
2. **Layer 1: Kya + Kyun** — concept, zero se
3. **Layer 2: Internally kaise** — mechanism
4. **Layer 3+: Deep dive** — code, config, production
5. **Networking Analogy** — tere background se connect
6. **Interview Q&A** — actual poochhe jaane wale sawaal
7. **Gotchas / Honesty notes** — kya galat ho sakta, kya over-claim na karna

---

## Complete Roadmap — 12 Files (koi kaam ka topic miss nahi)

| # | File | Topic | Interview Weight |
|---|------|-------|------------------|
| 00 | index-and-roadmap | Yeh file | — |
| 01 | mlops-and-mlflow-intro | MLOps kya, MLflow kyun, 4 components | ⭐⭐⭐ |
| 02 | tracking-experiments-runs | Experiments, Runs, Tracking basics + UI | ⭐⭐⭐ |
| 03 | logging-functions | Params, Metrics, Artifacts, Tags (deep) | ⭐⭐⭐ |
| 04 | autologging | mlflow.autolog(), framework-specific | ⭐⭐ |
| 05 | tracking-server-architecture | 4 scenarios, backend+artifact store, client-server | ⭐⭐⭐⭐ (Architect!) |
| 06 | model-component-signatures | Storage format, signatures, flavors, Model API | ⭐⭐⭐ |
| 07 | custom-models-flavors | pyfunc custom models, custom flavors | ⭐⭐ |
| 08 | model-evaluation | mlflow.evaluate(), SHAP, custom metrics, baseline | ⭐⭐⭐ |
| 09 | model-registry | Versions, stages→aliases, champion/production | ⭐⭐⭐⭐ |
| 10 | mlflow-projects | MLproject, entry points, reproducibility | ⭐⭐ |
| 11 | mlflow-client-cli | MlflowClient API, CLI commands | ⭐⭐ |
| 12 | aws-integration-production | EC2+S3+SageMaker deploy, remote arch, cost | ⭐⭐⭐⭐⭐ (TERA STACK) |

---

## Interview me MLflow ke top 10 sawaal (yeh doc inko cover karta)

1. MLflow ke 4 components kya hain? (F01)
2. Experiment vs Run ka farak? (F02)
3. Tracking server ka backend store vs artifact store? (F05) ← architect fav
4. Local se production tak MLflow architecture kaise scale karoge? (F05, F12)
5. Model signature kya hai, kyun zaroori? (F06)
6. Model registry me stages deprecate kyun hue, ab kya use hota? (F09)
7. Champion/Challenger deployment kaise? (F09)
8. MLflow model ko SageMaker pe kaise deploy karoge? (F12)
9. Autolog vs manual logging — kab kya? (F04)
10. Reproducibility MLflow kaise ensure karta? (F06, F10)

---

## Status Tracker — ALL COMPLETE ✅

- [x] F01 — MLOps & MLflow Intro
- [x] F02 — Tracking
- [x] F03 — Logging
- [x] F04 — Autologging
- [x] F05 — Tracking Server Architecture
- [x] F06 — Model Component & Signatures
- [x] F07 — Custom Models & Flavors
- [x] F08 — Model Evaluation
- [x] F09 — Model Registry
- [x] F10 — MLflow Projects
- [x] F11 — Client & CLI
- [x] F12 — AWS Integration & Production

---

## Revision Priority (interview se pehle)

**Tere role (AI Infra Architect) ke liye weight:**
1. **F05 (Server Architecture) + F12 (AWS Production)** — ⭐⭐⭐⭐⭐ sabse zyada padho
2. **F09 (Registry) + F06 (Models)** — deployment/governance
3. **F01 (components) + F02 (exp/run)** — foundation, opening questions
4. **F08 (eval) + F03 (logging)** — practical
5. **F04, F07, F10, F11** — supporting

**1-line pitch (interview opening):**
> "MLflow — 4 components: Tracking, Projects, Models, Registry. Production me main server on ECS/EKS, backend RDS PostgreSQL, artifacts S3, deploy via SageMaker endpoints — full AWS-native MLOps."
