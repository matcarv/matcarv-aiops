# MatCarv AIOps — Technology Stack

## Cloud Provider

**Amazon Web Services (AWS)** is the sole cloud target. All services, SDKs, and IAM conventions are AWS-native.

## Core AWS Services

| Category | Service | Purpose |
|----------|---------|---------|
| Metrics | Amazon CloudWatch Metrics | Time-series resource and custom metrics |
| Logs | Amazon CloudWatch Logs | Centralised log ingestion and querying |
| Traces | AWS X-Ray | Distributed request tracing |
| Events | Amazon EventBridge | Event bus for operational and custom events |
| Alerting | Amazon SNS | Notification fan-out for alerts |
| Automation | AWS Systems Manager (SSM) Automation | Runbook execution |
| Automation | AWS Lambda | Serverless remediation functions |
| Orchestration | AWS Step Functions | Multi-step remediation workflows |
| Container | Amazon ECS / Amazon EKS | Containerised workload monitoring |
| Serverless | AWS Lambda | Application tier and glue logic |
| Storage | Amazon S3 | Audit logs, runbook artefacts, cost reports |
| Database | Amazon DynamoDB | Operational state, alert deduplication table |
| Queue | Amazon SQS | Decoupled alert ingestion pipeline |
| IaC | AWS CloudFormation / AWS CDK | Infrastructure as code |
| Cost | AWS Cost Explorer API | Cost analysis and recommendations |
| Security | AWS Security Hub | Aggregated security findings |
| Security | AWS Config | Compliance rules and resource snapshots |
| Security | AWS CloudTrail | API audit trail |
| Secrets | AWS Secrets Manager | Credentials and API keys |
| IAM | AWS IAM | Least-privilege access control |

## Infrastructure as Code

- **AWS CDK (TypeScript)** is the preferred IaC tool for new resources.
- **CloudFormation** templates are used for legacy or service-catalogue stacks.
- All infrastructure changes go through a CI/CD pipeline (GitHub Actions → CDK deploy).

## Languages & Runtimes

| Language | Version | Use |
|----------|---------|-----|
| TypeScript | ≥ 5.x | CDK stacks, Lambda functions |
| Python | ≥ 3.12 | ML models, data-processing Lambda, CLI tooling |
| Bash | POSIX | SSM run-command scripts, local helper scripts |

## Key Libraries & SDKs

- **AWS SDK for JavaScript v3** (`@aws-sdk/*`) — Lambda and CDK TypeScript code.
- **AWS SDK for Python (Boto3)** — Python Lambda and tooling.
- **AWS CDK v2** — Infrastructure definitions.
- **Powertools for AWS Lambda (TypeScript / Python)** — Structured logging, tracing, metrics, and idempotency.
- **Pandas / NumPy** — Data processing in Python Lambdas.

## CI/CD

- **GitHub Actions** for build, lint, test, and deployment pipelines.
- Environments: `dev` → `staging` → `prod`, each in a separate AWS account.
- Deployment uses CDK context variables per environment.

## Observability Standards

- **Structured JSON logging** using AWS Lambda Powertools Logger in all Lambda functions.
- **Custom CloudWatch metrics** emitted via `aws-embedded-metrics` or Powertools Metrics.
- **X-Ray active tracing** enabled on all Lambda functions and API Gateway stages.
- Logs forwarded to a central **Log Archive** account via CloudWatch cross-account subscriptions.

## Security Standards

- All Lambda execution roles follow **least-privilege IAM**.
- Secrets stored in **AWS Secrets Manager**; never in environment variables or source code.
- All S3 buckets have **server-side encryption (SSE-S3 or SSE-KMS)** and block public access.
- VPC-based resources use **Security Groups** with minimal ingress rules.
