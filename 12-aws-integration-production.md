# Topic 12: MLflow AWS Integration & Production (⭐⭐⭐⭐⭐ TERA STACK)

> **HIGHEST interview weight for YOUR role — AWS + production deployment**
> **Prerequisites:** File 05 (server arch), File 06 (models), File 09 (registry)
> **Source:** mxagar/mlflow_guide §15 + MLflow official docs + AWS

---

## 🎯 One-Liner (Interview)

> "Production MLflow on AWS: MLflow tracking server EC2/ECS pe chalta, backend store PostgreSQL/RDS me (metadata), artifact store S3 me (models), code CodeCommit/GitHub pe. Training SageMaker pe, deployment ke liye MLflow model se Docker image banake ECR me push, phir SageMaker endpoint pe deploy — REST inference. Yeh end-to-end AWS-native MLOps."

---

## Layer 1: AWS Architecture (Full Picture)

```
┌──────────────┐
│  CodeCommit/ │  ← code repo
│  GitHub      │
└──────┬───────┘
       │ pull
       ▼
┌──────────────┐      log params/metrics      ┌─────────────────┐
│  SageMaker   │ ───────────────────────────▶ │  MLflow Server  │
│  Notebook    │                              │  (EC2)          │
│  (training)  │                              │                 │
└──────┬───────┘                              │  Backend:       │
       │ artifacts                            │  SQLite/RDS     │
       ▼                                      └────────┬────────┘
┌──────────────┐                                       │
│   S3 Bucket  │ ◀─────────────────────────────────────┘
│  (artifacts, │        artifact store
│   models)    │
└──────────────┘
       │ deploy
       ▼
┌──────────────┐     build image    ┌──────────┐    deploy   ┌──────────────┐
│ MLflow model │ ─────────────────▶ │   ECR    │ ──────────▶ │  SageMaker   │
│              │                    │ (image)  │             │  Endpoint    │
└──────────────┘                    └──────────┘             │  (inference) │
                                                             └──────────────┘
```

**AWS Services used:**
- **CodeCommit/GitHub** — code
- **EC2** — MLflow tracking server
- **RDS/PostgreSQL** (ya SQLite for demo) — backend store
- **S3** — artifact store (models)
- **SageMaker** — training + deployment
- **ECR** — Docker images for deployment

**Networking analogy:** Yeh bilkul **enterprise multi-tier architecture** — compute tier (SageMaker/EC2), data tier (RDS + S3), registry (ECR), serving tier (SageMaker endpoint). Tera infra background me yeh natural.

---

## Layer 2: MLflow Server on EC2 + S3 (Setup)

### EC2 pe MLflow server:
```bash
# EC2 instance pe (Ubuntu t2.micro demo, prod me bigger)
pip install mlflow boto3 psycopg2-binary

# Server start — backend SQLite (demo), artifacts S3
mlflow server \
  -h 0.0.0.0 \
  --backend-store-uri sqlite:///mlflow.db \
  --default-artifact-root s3://my-mlflow-artifacts

# Production: backend RDS PostgreSQL
mlflow server \
  -h 0.0.0.0 \
  --backend-store-uri postgresql://user:pass@rds-endpoint:5432/mlflowdb \
  --default-artifact-root s3://my-mlflow-artifacts
```

### EC2 security group:
- Port **5000** (MLflow UI/API) — inbound (par restricted IP, world-open NAHI in prod)
- Port **22** (SSH) — restricted

### Access:
```
http://<ec2-public-dns>:5000    # UI
```
Code me: `mlflow.set_tracking_uri("http://<ec2-dns>:5000")`

> **Cost tip:** EC2 stop karo jab use nahi (stopped = no compute cost). Restart pe server manually re-launch + DNS change (Elastic IP se fix karo). Artifacts S3 me persist, backend (SQLite local to EC2) bhi persist.

---

## Layer 3: IAM Setup (Security — tera strength)

### IAM User (local dev):
```
# ~/.aws credentials ya .env
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_DEFAULT_REGION=us-east-1
```
```python
from dotenv import load_dotenv
load_dotenv()   # AWS creds environment me
```

### IAM Role (SageMaker) — permissions:
- `AmazonSageMakerFullAccess`
- S3 access (artifacts)
- ECR access (deployment images)
- CodeCommit (if used)

> **Note:** AWS-hosted compute (SageMaker/EC2) me IAM role attached hota — local credentials ki zaroorat nahi wahan. Local dev me hi creds chahiye.

**Interview point (tera security background):** "Least privilege — SageMaker role ko sirf needed S3 bucket + ECR repo access, full access nahi. Credentials .env me (gitignored), production me IAM roles (no long-lived keys)."

---

## Layer 4: Training on SageMaker

```python
# SageMaker notebook terminal me
# 1. Code pull (CodeCommit/GitHub)
git pull

# 2. Tracking URI = EC2 MLflow server
mlflow.set_tracking_uri("http://<ec2-dns>:5000")

# 3. Train (MLproject se — File 10)
mlflow run . -P model=XGBRegressor
```

Artifacts S3 me jaayenge (`s3://bucket/...`), metadata EC2 backend me. UI me runs dikhenge with S3 artifact URIs.

---

## Layer 5: Deployment on SageMaker (KEY)

### Step 1: Model se Docker image banao + ECR push
```bash
mlflow sagemaker build-and-push-container \
  --container xgb \
  --env-manager conda
```
Yeh MLflow model ko deployable Docker image me pack karke **ECR** me push karta. (Deploy nahi karta — image prepare karta.)

### Step 2: SageMaker endpoint pe deploy
```python
from mlflow.deployments import get_deploy_client

client = get_deploy_client("sagemaker")

client.create_deployment(
    name="house-price-prod",           # endpoint name
    model_uri="s3://bucket/.../model", # registry/S3 se model
    flavor="python_function",
    config={
        "execution_role_arn": "arn:aws:iam::...",
        "bucket_name": "my-bucket",
        "image_url": "<ecr-image-uri>",
        "region_name": "us-east-1",
        "instance_type": "ml.m5.xlarge",
        "instance_count": 1,
    },
)
```
5-10 min me SageMaker endpoint live.

### Step 3: Inference
```python
import boto3, json

smrt = boto3.client("runtime.sagemaker", region_name="us-east-1")
response = smrt.invoke_endpoint(
    EndpointName="house-price-prod",
    Body=json.dumps({"instances": test_data.tolist()}),
    ContentType="application/json",
)
print(response["Body"].read().decode("ascii"))
```

---

## Layer 6: Production Best Practices (Architect thinking)

1. **HA:** MLflow server ECS/EKS pe (EC2 single point of failure); ALB peeche
2. **Backend:** RDS PostgreSQL (Multi-AZ) — not SQLite in prod
3. **Artifacts:** S3 with versioning + lifecycle (old → Glacier)
4. **Security:**
   - Server private subnet, ALB + OIDC/SSO auth (MLflow built-in auth basic)
   - S3 bucket policies, RDS/S3 encryption (KMS)
   - IAM least privilege, no long-lived keys
5. **Cost:**
   - EC2/SageMaker stop when idle
   - S3 lifecycle, right-size instances, spot for training
   - SageMaker endpoint auto-scaling / serverless inference for spiky traffic
6. **CI/CD:** GitHub Actions → train → evaluate (baseline gate, F08) → register (F09) → deploy
7. **Cleanup:** endpoints, notebooks, EC2 delete jab done (cost!)

> **Bedrock connection (tera stack):** MLflow LLM ke saath bhi — Bedrock models ko track/evaluate kar sakte MLflow se. GenAI evaluation (File 08 ke text/QA model_type) + MLflow Deployments for LLMs. Yeh AI Infra role me hot.

---

## 🎤 Interview Q&A (tere role ka CORE)

**Q1: Production MLflow on AWS architecture?**
> "Tracking server EC2/ECS pe, backend RDS PostgreSQL (metadata), artifacts S3 (models), code CodeCommit/GitHub. Training SageMaker, deploy ke liye MLflow model → Docker image → ECR → SageMaker endpoint (REST inference). HA ke liye ECS/EKS + ALB, security VPC + IAM + encryption."

**Q2: MLflow model SageMaker pe kaise deploy?**
> "mlflow sagemaker build-and-push-container se model ko Docker image bana ke ECR push. Phir get_deploy_client('sagemaker').create_deployment() se endpoint pe deploy with execution role, image URI, instance type. invoke_endpoint se inference."

**Q3: Backend aur artifact store production me?**
> "Backend RDS PostgreSQL (Multi-AZ, metadata, queryable, HA). Artifacts S3 (cheap, durable, scalable, versioning + lifecycle). SQLite/local sirf demo."

**Q4: MLflow production me secure kaise?**
> "Server private subnet, ALB + OIDC/SSO (built-in auth basic hai). S3 bucket policies + KMS encryption, RDS encryption. IAM least privilege, no long-lived keys — SageMaker/EC2 pe attached roles. Internet-facing avoid."

**Q5: Cost optimization?**
> "EC2/SageMaker idle pe stop, S3 lifecycle (Glacier), right-sizing, spot for training, SageMaker serverless/auto-scaling inference for spiky load. Endpoints cleanup when unused."

**Q6: End-to-end MLOps pipeline on AWS?**
> "GitHub Actions trigger → SageMaker training (MLflow tracked) → mlflow.evaluate baseline gate → register in MLflow registry (champion alias) → build/push ECR → SageMaker endpoint deploy → monitor. IaC (Terraform/CDK) for infra."

---

## ⚠️ Gotchas

1. **SQLite production me NAHI** — RDS. SQLite single-file, no concurrency/HA.
2. **EC2 restart → DNS change** — Elastic IP use karo, warna tracking URI toot jaayega
3. **S3 credentials client ke paas** — warna model download/log fail
4. **Security group world-open (0.0.0.0)** demo me theek, prod me NEVER — restrict
5. **Cleanup!** — endpoints/EC2/notebooks running = paisa jalta. Done pe delete.
6. **mlflow built-in auth basic** — production me reverse proxy + SSO

---

## 🎓 Tera Complete MLflow Doc DONE!

Ab tere paas 12 files hain — MLOps intro se AWS production tak, koi core topic miss nahi.

**Revision order (interview se pehle):**
1. F01 (components) + F02 (experiment/run) — foundation
2. F05 (server arch) + F12 (AWS) — **tere role ka core, sabse zyada padho**
3. F09 (registry) + F06 (models) — deployment/governance
4. F08 (eval) + F03 (logging) — practical
5. F04, F07, F10, F11 — supporting

**Top interview weight (tere role):** F05, F12, F09 — inhe rATTA + samajh.
