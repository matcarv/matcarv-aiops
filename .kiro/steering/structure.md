# MatCarv AIOps — Project Structure

## Repository Layout

```
matcarv-aiops/
├── .kiro/
│   └── steering/          # Kiro steering documents (this directory)
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
│   │   ├── alert-router/  # Enriches and routes incoming alerts
│   │   ├── anomaly-detector/  # Custom anomaly detection logic
│   │   ├── cost-analyzer/ # Cost Explorer integration
│   │   ├── remediation/   # Automated remediation handlers
│   │   └── shared/        # Shared utilities (logging, config, AWS clients)
│   └── stepfunctions/     # Step Functions definition files (ASL JSON/YAML)
├── scripts/
│   ├── ssm/               # SSM Automation and Run-Command scripts
│   └── local/             # Local development and testing helpers
├── .github/
│   └── workflows/         # GitHub Actions CI/CD pipelines
├── README.md
└── package.json           # Root workspace manifest (if using npm workspaces)
```

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| AWS resource logical IDs (CDK) | PascalCase | `AlertRouterFunction` |
| AWS resource physical names | kebab-case with env suffix | `alert-router-prod` |
| Lambda source directories | kebab-case | `alert-router/` |
| CDK Stack files | `<name>-stack.ts` | `monitoring-stack.ts` |
| CDK Construct files | `<name>-construct.ts` | `alert-bus-construct.ts` |
| Python modules | snake_case | `anomaly_detector.py` |
| Environment variables | SCREAMING_SNAKE_CASE | `LOG_LEVEL`, `ALERTS_TABLE_NAME` |

## Lambda Function Layout

Each Lambda in `src/lambdas/<name>/` follows this structure:

```
<name>/
├── index.ts (or handler.py)  # Handler entry point
├── handler.ts                # Business logic (TypeScript) — separated from entry point
├── types.ts                  # TypeScript type definitions
├── tests/
│   └── handler.test.ts       # Unit tests
└── package.json              # Lambda-specific dependencies (if any)
```

## Infrastructure Stack Layout

CDK stacks are organised by domain:

| Stack | Contents |
|-------|----------|
| `MonitoringStack` | CloudWatch dashboards, alarms, Anomaly Detectors, log groups |
| `AlertingStack` | SNS topics, SQS queues, alert-router Lambda, EventBridge rules |
| `RemediationStack` | SSM Automation docs, remediation Lambdas, Step Functions |
| `CostStack` | Cost analyzer Lambda, S3 bucket for reports, scheduled EventBridge rule |
| `SecurityStack` | Config rules, Security Hub subscription, CloudTrail configuration |
| `DataStack` | DynamoDB tables (alert deduplication, operational state) |
| `PipelineStack` | CDK Pipelines self-mutating pipeline |

## Configuration Management

- Environment-specific values (account IDs, region, thresholds) live in `infra/config/<env>.json`.
- Runtime configuration (non-secret) is passed as Lambda environment variables from CDK.
- Secrets are fetched at runtime from **AWS Secrets Manager** using the secret ARN stored in an environment variable.

## Testing Strategy

| Layer | Tool | Location |
|-------|------|---------|
| CDK unit tests | Jest (`@aws-cdk/assertions`) | `infra/test/` |
| Lambda unit tests | Jest (TypeScript) / Pytest (Python) | `src/lambdas/<name>/tests/` |
| Integration tests | AWS CDK Integ Tests or custom scripts | `scripts/local/` |

## Documentation Standards

- All `docs/runbooks/` files follow the template in `docs/runbooks/_template.md`.
- Architecture Decision Records (ADRs) are stored in `docs/architecture/adr-<number>-<title>.md`.
- Post-mortems follow the template in `docs/post-mortems/_template.md`.
