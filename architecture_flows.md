# Architecture Flow Definitions

Use these definitions to create professional architecture diagrams (draw.io, Lucidchart, or similar).

---

## 1. ML Inference Pipeline (AWS Cloud Architecture)

This is the production version of the local AstroLens pipeline, deployed on AWS managed services.

### Flow

```
                                    +-------------------+
                                    |   CloudWatch      |
                                    |   (Monitoring &   |
                                    |    Dashboards)    |
                                    +--------^----------+
                                             |
+------------+     +-----------------+     +-----------+     +-------------+     +-----------+
|  S3 Bucket | --> | Step Functions  | --> | SageMaker | --> | DynamoDB    | --> | API GW /  |
|  (Survey   |     | (Pipeline       |     | Endpoint  |     | (Anomaly    |     | AppSync   |
|   Images)  |     |  Orchestrator)  |     | (ViT+OOD  |     |  Candidates |     | (Results  |
|            |     |                 |     |  +YOLO)   |     |  +Metadata) |     |  API)     |
+-----+------+     +---+----+-------+     +-----+-----+     +------+------+     +-----------+
      |                 |    |                   |                   |
      |                 v    |                   v                   v
      |           +----------+--+         +----------+        +----------+
      |           | Lambda      |         | Bedrock  |        | SNS      |
      |           | (Preprocess,|         | (NL      |        | (Alert   |
      |           |  Validate,  |         |  Summary |        |  High-   |
      |           |  Resize)    |         |  of Find |        |  Conf.   |
      |           +-------------+         |  -ings)  |        |  Detect) |
      |                                   +----------+        +-----+----+
      |                                                             |
      +-- EventBridge (Scheduled/Event-driven trigger) <------------+
```

### Components Detail

| Component | AWS Service | Input | Output | Purpose |
|-----------|-------------|-------|--------|---------|
| Data Lake | S3 | Raw survey images (FITS, JPEG, PNG) | Validated images in standardised format | Central storage for all survey data and model artifacts |
| Trigger | EventBridge | Schedule (cron) or S3 event | Step Functions execution | Initiates pipeline runs on schedule or new data arrival |
| Orchestrator | Step Functions | Pipeline configuration | State machine execution | Coordinates all stages, handles retries and branching |
| Preprocess | Lambda | Raw images from S3 | Resized, normalised images back to S3 | Validate, resize to 224x224, reject corrupted files |
| Inference | SageMaker Endpoint | Preprocessed images | Classification + OOD scores + YOLO detections | Runs ViT classifier, OOD ensemble (MSP/Energy/Mahalanobis), YOLOv8 |
| Enhanced Analysis | Bedrock | Detection metadata | Natural language summaries | Generate readable descriptions of findings |
| Results Store | DynamoDB | Detection records | Queryable candidate database | Low-latency store for anomaly candidates with scores and metadata |
| Cross-Reference | Lambda | Candidate coordinates | Match/no-match with catalog objects | Query SIMBAD/NED APIs, classify as known/unknown |
| Alerts | SNS | High-confidence detections | Email/SMS/webhook notifications | Notify on significant discoveries |
| Monitoring | CloudWatch | All service metrics | Dashboards and alarms | Pipeline health, throughput, error rates, model drift |
| Results API | API Gateway / AppSync | Query parameters | JSON candidate lists | REST/GraphQL API for external consumers |

---

## 2. DevOps / CI/CD Pipeline

End-to-end delivery pipeline from code commit to production deployment with full observability.

### Flow

```
+----------+     +---------------+     +-----------+     +----------+     +------------+
| Developer| --> | GitHub Actions| --> | Amazon    | --> | Amazon   | --> | Production |
| Git Push |     | (Build, Test, |     | ECR       |     | ECS/EKS  |     | (Blue-     |
|          |     |  Lint, Scan)  |     | (Container|     | (Deploy) |     |  Green)    |
+----------+     +-------+-------+     |  Registry)|     +-----+----+     +------+-----+
                         |             +-----------+           |                |
                         v                                     v                v
                  +------+-------+                     +-------+------+  +------+------+
                  | Quality Gates|                     | ArgoCD       |  | CloudWatch  |
                  | - Unit Tests |                     | (GitOps      |  | X-Ray       |
                  | - Lint (ruff)|                     |  Sync &      |  | Prometheus  |
                  | - SAST Scan  |                     |  Rollback)   |  | Grafana     |
                  | - Container  |                     +--------------+  +-------------+
                  |   Scan       |
                  +--------------+
```

### Stages Detail

| Stage | Tool | What Happens | Failure Action |
|-------|------|-------------|----------------|
| Source | Git + GitHub | Developer pushes to feature branch | -- |
| Build | GitHub Actions | Install deps, compile Rust core (astroSETI), build Python wheel | Fail fast, notify |
| Test | pytest, ruff | Unit tests, integration tests, linting, type checks | Block merge |
| Security Scan | Trivy / Snyk | Container vulnerability scan, dependency audit | Block if critical CVE |
| Package | Docker | Build multi-stage Docker image, tag with git SHA + semver | -- |
| Push | Amazon ECR | Push container image to private registry | -- |
| Deploy (staging) | ECS/EKS + ArgoCD | Blue-green deployment to staging environment | Auto-rollback |
| Smoke Test | Custom Lambda | Health check, API validation, inference test | Rollback staging |
| Deploy (production) | ECS/EKS + ArgoCD | Blue-green deployment to production | Auto-rollback |
| Monitor | CloudWatch + X-Ray | Error rate, latency, model accuracy metrics | Alert on anomaly |

---

## 3. Infrastructure Architecture (Platform View)

High-level view of the platform showing networking, compute, and data layers.

```
+------------------------------------------------------------------+
|                         AWS Account                               |
|                                                                   |
|  +--------------------+    +----------------------------------+   |
|  |   Route 53 (DNS)   |    |         CloudFront (CDN)         |   |
|  +--------+-----------+    +----------------+-----------------+   |
|           |                                 |                     |
|  +--------v---------------------------------v-----------------+   |
|  |                      VPC (Multi-AZ)                        |   |
|  |                                                            |   |
|  |  +-- Public Subnets ----+   +-- Private Subnets --------+ |   |
|  |  | ALB / API Gateway    |   | ECS/EKS (Inference)       | |   |
|  |  | NAT Gateway          |   | SageMaker Endpoints       | |   |
|  |  +----------------------+   | Lambda Functions           | |   |
|  |                             | RDS / DynamoDB             | |   |
|  |                             +-----------------------------+ |   |
|  |                                                            |   |
|  +------------------------------------------------------------+   |
|                                                                   |
|  +-- Data Layer --------------------------------------------------+
|  | S3 (Images, Models, Reports)                                   |
|  | DynamoDB (Candidates, Metadata)                                |
|  | ElastiCache (Inference Cache)                                  |
|  +----------------------------------------------------------------+
|                                                                   |
|  +-- Observability -----------------------------------------------+
|  | CloudWatch (Logs, Metrics, Alarms)                             |
|  | X-Ray (Distributed Tracing)                                    |
|  | CloudTrail (Audit)                                             |
|  +----------------------------------------------------------------+
+------------------------------------------------------------------+
```

---

## 4. Diagram Creation Guide

### Recommended Tools (for professional diagrams)
1. **draw.io / diagrams.net** (free) -- Export as SVG for the website
2. **Lucidchart** -- Professional, good AWS icon library
3. **AWS Architecture Icons** -- Download from https://aws.amazon.com/architecture/icons/

### Colour Scheme (match website)
- Background: #0f1525
- Storage nodes: #4da6ff (blue)
- Compute nodes: #f97316 (orange)
- Orchestration: #7c3aed (purple)
- Database: #22c55e (green)
- AI/ML: #ec4899 (pink)
- Monitoring: #6366f1 (indigo)
- Text: #e8edf5 (light)

### What to Include in Each Diagram
- AWS service icons (official icon set)
- Data flow arrows with labels
- Input/output annotations
- Service names + brief purpose
- Colour coding by function (storage, compute, orchestration, etc.)
