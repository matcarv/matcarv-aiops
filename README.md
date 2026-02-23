# MatCarv AIOps

An AI-driven operations platform for AWS environments. MatCarv AIOps provides automated monitoring, intelligent alerting, self-healing remediation, and continuous cost optimisation by applying machine learning and rule-based automation to operational data produced by AWS services.

## Table of Contents

- [Purpose & Goals](#purpose--goals)
- [Key Features](#key-features)
- [Target Users](#target-users)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Monitoring & Observability](#monitoring--observability)
- [Incident Response](#incident-response)
- [Automation & Remediation](#automation--remediation)
- [Documentation](#documentation)

---

## Purpose & Goals

| Goal | Description |
|------|-------------|
| **Reduce MTTR** | Detect anomalies and trigger automated runbooks before human intervention is required |
| **Improve observability** | Aggregate metrics, logs, and traces from all AWS services into a unified view |
| **Automate routine operations** | Replace manual, repetitive tasks (scaling, patching, log rotation, snapshot management) with event-driven workflows |
| **Optimise costs** | Identify idle or over-provisioned resources and recommend or execute right-sizing actions |
| **Increase security posture** | Continuously audit IAM policies, Security Group rules, and CloudTrail events for anomalies |

> **Out of scope:** On-premises or multi-cloud environments, application-level code changes as part of remediation, and capacity planning beyond AWS resource right-sizing.

---

## Key Features

1. **Anomaly Detection** — CloudWatch Anomaly Detection and custom ML models identify unusual resource behaviour.
2. **Automated Runbooks** — AWS Systems Manager Automation documents execute pre-approved remediation steps.
3. **Intelligent Alerting** — SNS + Lambda-based alert routing with suppression, grouping, and severity enrichment.
4. **Cost Intelligence** — AWS Cost Explorer API integration surfaces savings recommendations.
5. **Compliance Scanning** — AWS Config rules and Security Hub standards provide continuous compliance checks.
6. **Incident Management** — EventBridge-driven incident lifecycle: detection → triage → remediation → post-mortem.
7. **SLO Tracking** — CloudWatch dashboards exposing error-budget burn rates per service.

---

## Target Users

| Role | Primary Use |
|------|-------------|
| Cloud Operations Engineer | Monitors dashboards, reviews alerts, approves automated remediations |
| SRE / DevOps Engineer | Authors runbooks, configures automation, tunes alert thresholds |
| FinOps Analyst | Reviews cost reports, approves right-sizing recommendations |
| Security Engineer | Reviews compliance findings, investigates security events |
| Engineering Manager | Reviews SLO dashboards and incident post-mortems |

---

## Technology Stack

**Cloud provider:** Amazon Web Services (AWS) — all services, SDKs, and IAM conventions are AWS-native.

### Core AWS Services

| Category | Service |
|----------|---------|
| Metrics | Amazon CloudWatch Metrics |
| Logs | Amazon CloudWatch Logs |
| Traces | AWS X-Ray |
| Events | Amazon EventBridge |
| Alerting | Amazon SNS |
| Automation | AWS Systems Manager (SSM) Automation, AWS Lambda |
| Orchestration | AWS Step Functions |
| Containers | Amazon ECS / Amazon EKS |
| Storage | Amazon S3 |
| Database | Amazon DynamoDB |
| Queue | Amazon SQS |
| IaC | AWS CDK (TypeScript) / AWS CloudFormation |
| Cost | AWS Cost Explorer API |
| Security | AWS Security Hub, AWS Config, AWS CloudTrail |
| Secrets | AWS Secrets Manager |

### Languages & Runtimes

| Language | Version | Use |
|----------|---------|-----|
| TypeScript | ≥ 5.x | CDK stacks, Lambda functions |
| Python | ≥ 3.12 | ML models, data-processing Lambda, CLI tooling |
| Bash | POSIX | SSM run-command scripts, local helper scripts |

### Key Libraries

- **AWS SDK for JavaScript v3** (`@aws-sdk/*`) — Lambda and CDK TypeScript code.
- **AWS SDK for Python (Boto3)** — Python Lambda and tooling.
- **AWS CDK v2** — Infrastructure definitions.
- **Powertools for AWS Lambda (TypeScript / Python)** — Structured logging, tracing, metrics, and idempotency.
- **Pandas / NumPy** — Data processing in Python Lambdas.

### CI/CD

- **GitHub Actions** for build, lint, test, and deployment pipelines.
- Environments: `dev` → `staging` → `prod`, each in a separate AWS account.
- Deployment uses CDK context variables per environment.

---

## Project Structure

```
matcarv-aiops/
├── .kiro/
│   └── steering/          # Kiro steering documents
├── docs/
│   ├── architecture/      # Architecture decision records (ADRs) and diagrams
│   ├── runbooks/          # Human-readable operational runbooks
│   └── post-mortems/      # Incident post-mortem reports
├── infra/
│   ├── bin/               # CDK app entry point(s)
│   ├── lib/
│   │   ├── stacks/        # CDK Stack definitions (one file per stack)
│   │   └── constructs/    # Reusable CDK constructs
│   ├── config/            # Per-environment CDK context files (dev.json, prod.json)
│   └── test/              # CDK unit tests (Jest)
├── src/
│   ├── lambdas/           # Lambda function source code
│   │   ├── alert-router/
│   │   ├── anomaly-detector/
│   │   ├── cost-analyzer/
│   │   ├── remediation/
│   │   └── shared/        # Shared utilities (logging, config, AWS clients)
│   └── stepfunctions/     # Step Functions definition files (ASL JSON/YAML)
├── scripts/
│   ├── ssm/               # SSM Automation and Run-Command scripts
│   └── local/             # Local development and testing helpers
├── .github/
│   └── workflows/         # GitHub Actions CI/CD pipelines
├── README.md
└── package.json
```

### CDK Stacks

| Stack | Contents |
|-------|----------|
| `MonitoringStack` | CloudWatch dashboards, alarms, Anomaly Detectors, log groups |
| `AlertingStack` | SNS topics, SQS queues, alert-router Lambda, EventBridge rules |
| `RemediationStack` | SSM Automation docs, remediation Lambdas, Step Functions |
| `CostStack` | Cost analyzer Lambda, S3 bucket for reports, scheduled EventBridge rule |
| `SecurityStack` | Config rules, Security Hub subscription, CloudTrail configuration |
| `DataStack` | DynamoDB tables (alert deduplication, operational state) |
| `PipelineStack` | CDK Pipelines self-mutating pipeline |

### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| AWS resource logical IDs (CDK) | PascalCase | `AlertRouterFunction` |
| AWS resource physical names | kebab-case with env suffix | `alert-router-prod` |
| Lambda source directories | kebab-case | `alert-router/` |
| CDK Stack files | `<name>-stack.ts` | `monitoring-stack.ts` |
| Python modules | snake_case | `anomaly_detector.py` |
| Environment variables | SCREAMING_SNAKE_CASE | `LOG_LEVEL`, `ALERTS_TABLE_NAME` |

---

## Getting Started

1. **Prerequisites:** AWS CLI v2, Node.js ≥ 18, Python ≥ 3.12, AWS CDK v2.
2. **Install dependencies:** `npm install` at the repo root.
3. **Configure environment:** Copy `infra/config/dev.json.example` to `infra/config/dev.json` and fill in your account/region.
4. **Bootstrap CDK:** `npx cdk bootstrap aws://<account>/<region>` for each target environment.
5. **Deploy to dev:** `npx cdk deploy --all --context env=dev` from the `infra/` directory.

### Testing

| Layer | Tool | Location |
|-------|------|---------|
| CDK unit tests | Jest (`@aws-cdk/assertions`) | `infra/test/` |
| Lambda unit tests | Jest (TypeScript) / Pytest (Python) | `src/lambdas/<name>/tests/` |
| Integration tests | AWS CDK Integ Tests or custom scripts | `scripts/local/` |

---

## Monitoring & Observability

### Principles

- **Everything is an alarm** — every AWS resource that can produce metrics must have at least one CloudWatch Alarm.
- **Structured logs everywhere** — all Lambda functions emit JSON logs via AWS Lambda Powertools Logger.
- **Trace every request** — X-Ray active tracing is mandatory for Lambda and API Gateway.
- **Dashboards are living documents** — CloudWatch dashboards are managed as CDK code, not manually.
- **Alert on symptoms, not causes** — customer-facing SLIs drive alerting; internal metrics are for diagnostics.

### Standard Dashboards

| Dashboard | Contents |
|-----------|----------|
| `AIOps-Overview` | SLO burn rates, open incidents, alert volume trend |
| `AIOps-Lambda` | Error rates, durations, throttles, cold starts per function |
| `AIOps-Infrastructure` | EC2, RDS, ECS, EKS resource utilisation |
| `AIOps-Cost` | Daily spend, top cost drivers, savings recommendations |
| `AIOps-Security` | Security Hub findings by severity, Config compliance score |

### Alarm Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Lambda error rate | ≥ 1 % | ≥ 5 % |
| Lambda P99 duration | ≥ 80 % of timeout | ≥ 95 % of timeout |
| SQS ApproximateAgeOfOldestMessage | ≥ 5 min | ≥ 15 min |
| DynamoDB ThrottledRequests | ≥ 1 / min | ≥ 10 / min |
| EC2 CPUUtilization | ≥ 70 % | ≥ 90 % |
| ALB HTTPCode_Target_5XX_Count | ≥ 10 / min | ≥ 50 / min |

Alarm naming convention: `<environment>-<service>-<metric>-<severity>` (e.g. `prod-order-api-error-rate-critical`).

---

## Incident Response

### Severity Levels

| Level | Name | Customer Impact | Initial Response |
|-------|------|----------------|-----------------|
| SEV-1 | Critical | Complete outage or data loss | < 5 min |
| SEV-2 | High | Significant degradation | < 15 min |
| SEV-3 | Medium | Partial degradation | < 1 hour |
| SEV-4 | Low | Minor or no customer impact | Next business day |

### Incident Lifecycle

```
Detection → Triage → Acknowledgment → Investigation → Remediation → Resolution → Post-Mortem
```

- **Detection:** CloudWatch Alarm → SNS → `alert-router` Lambda → EventBridge `aiops-alerts` bus.
- **Triage (automated):** Deduplication, enrichment, severity mapping, routing, and auto-remediation check.
- **Post-mortem:** Mandatory for SEV-1 and SEV-2; must be filed within 48 hours using the template at `docs/post-mortems/_template.md`. Action items tracked as GitHub Issues labelled `incident-action-item`.

### Automated Runbooks

| Runbook | Trigger Condition | Action |
|---------|------------------|--------|
| `AIOps-RestartECSService` | ECS service task count = 0 | Forces new deployment |
| `AIOps-ScaleOutEC2ASG` | EC2 CPU > 90 % for 5 min | Increases desired capacity by 25 % |
| `AIOps-PurgeSQSDLQ` | DLQ message count > 100 | Moves messages to source queue |
| `AIOps-RotateLambdaAlias` | Lambda error rate > 10 % | Rolls back alias to previous version |
| `AIOps-RDSFailover` | RDS multi-AZ event detected | Confirms failover and notifies |
| `AIOps-InvalidateCFCache` | ALB 5xx spike | Triggers CloudFront invalidation |

---

## Automation & Remediation

### Automation Principles

1. **Automate toil first** — prioritise automation of tasks performed more than once a week.
2. **Safe by default** — every automated action must be reversible or have an explicit rollback step.
3. **Observe before acting** — collect diagnostic data before any state-changing action.
4. **Minimal blast radius** — automated changes must target the smallest possible scope.
5. **Audit everything** — every automated action writes an audit log to CloudTrail and DynamoDB.
6. **Human in the loop for irreversible actions** — deleting data, terminating instances, or production changes require async human approval.

### Automation Patterns

| Pattern | Use Case | Flow |
|---------|----------|------|
| Event-Driven Lambda | Fast, stateless, single-step remediations (< 15 s) | EventBridge → Lambda → AWS API |
| SSM Automation | Multi-step, auditable remediations on EC2/ECS/RDS | EventBridge → Lambda → SSM |
| Step Functions | Complex, stateful, multi-resource or approval-required remediations | EventBridge → Step Functions |
| Scheduled | Periodic maintenance tasks | EventBridge Scheduler → Lambda |

### Scheduled Maintenance

| Schedule | Task |
|----------|------|
| Daily 02:00 UTC | Delete EBS snapshots older than retention period (non-prod) |
| Daily 03:00 UTC | Generate cost anomaly report and post to Slack |
| Weekly Sunday 01:00 UTC | Rotate non-prod IAM access keys |
| Every 5 minutes | Run custom anomaly detection model |
| Every 15 minutes | Sync Security Hub findings to DynamoDB |

---

## Documentation

- **Architecture Decision Records:** `docs/architecture/adr-<number>-<title>.md`
- **Operational Runbooks:** `docs/runbooks/` (template: `docs/runbooks/_template.md`)
- **Post-Mortems:** `docs/post-mortems/` (template: `docs/post-mortems/_template.md`)
- **CloudWatch Insights Queries:** `docs/runbooks/cloudwatch-insights-queries.md`
- **Steering Documents:** `.kiro/steering/`
