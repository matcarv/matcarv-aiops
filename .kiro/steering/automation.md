# MatCarv AIOps — Automation & Remediation

## Principles

1. **Automate toil first** — prioritise automation of tasks performed more than once a week.
2. **Safe by default** — every automated action must be reversible or have an explicit rollback step.
3. **Observe before acting** — collect diagnostic data before any state-changing action.
4. **Minimal blast radius** — automated changes must target the smallest possible scope (single resource, not fleet-wide).
5. **Audit everything** — every automated action writes an audit log to CloudTrail and DynamoDB.
6. **Human in the loop for irreversible actions** — deleting data, terminating instances, or changes in production require an async approval step via SNS + human confirmation.

## Automation Patterns

### Pattern 1 — Event-Driven Lambda Remediation

Use for **fast, stateless, single-step** remediations (< 15 seconds).

```
EventBridge Rule → Lambda (remediation handler) → AWS API call → CloudWatch metric/log
```

Example: Automatically re-enable a disabled CloudWatch Alarm when an EventBridge Config rule detects it was disabled.

### Pattern 2 — SSM Automation Document

Use for **multi-step, auditable** remediations on EC2, ECS, or RDS.

```
EventBridge Rule → Lambda (trigger) → SSM StartAutomationExecution → SSM Automation Document
```

SSM documents are stored in `scripts/ssm/` as YAML files and deployed via CDK (`aws-ssm.CfnDocument`).

### Pattern 3 — Step Functions Orchestration

Use for **complex, stateful, multi-resource** remediations or those requiring approval.

```
EventBridge Rule → Step Functions (RemediationOrchestrator)
  ├── Task: Collect diagnostics (Lambda)
  ├── Choice: Is auto-approve eligible?
  │   ├── Yes → Task: Execute remediation (Lambda / SSM)
  │   └── No  → Task: Request approval (SNS → human) → Wait for callback
  ├── Task: Validate remediation (Lambda)
  └── Task: Update incident record (Lambda)
```

The `RemediationOrchestrator` state machine definition is in `src/stepfunctions/remediation-orchestrator.asl.yaml`.

### Pattern 4 — Scheduled Automation

Use for **periodic maintenance** tasks.

```
EventBridge Scheduler → Lambda (maintenance handler) → AWS API calls
```

| Schedule | Task |
|----------|------|
| Daily 02:00 UTC | Delete EBS snapshots older than the configured retention period (default: 30 days, non-prod only) |
| Daily 03:00 UTC | Generate cost anomaly report and post to Slack |
| Weekly Sunday 01:00 UTC | Rotate non-prod IAM access keys |
| Every 5 minutes | Run custom anomaly detection model |
| Every 15 minutes | Sync Security Hub findings to DynamoDB |

## Auto-Remediation Catalogue

### Compute

| Condition | Action | Pattern | Reversible |
|-----------|--------|---------|-----------|
| EC2 CPU > 90 % for 10 min | Scale-out EC2 Auto Scaling Group | Lambda | Yes (scale-in) |
| ECS service task count < desired | Force new ECS deployment | Lambda | Yes |
| Lambda throttles > 100 / min | Increase reserved concurrency by 25 % | Lambda | Yes |
| EC2 instance status check failed | Stop and start instance (reboot) | SSM | N/A |

### Storage & Data

| Condition | Action | Pattern | Reversible |
|-----------|--------|---------|-----------|
| S3 bucket public access enabled | Re-enable block public access | Lambda | No (approval required) |
| EBS volume utilisation > 85 % | Trigger snapshot + alert for manual resize | Lambda | N/A |
| DynamoDB throttled reads | Enable auto-scaling or increase RCU | Lambda | Yes |

### Networking

| Condition | Action | Pattern | Reversible |
|-----------|--------|---------|-----------|
| Security Group with 0.0.0.0/0 SSH | Revoke rule + create Config finding | Lambda | No (approval required) |
| ALB 5xx spike | Invalidate CloudFront cache | Lambda | N/A |
| Route53 health check failed | Failover DNS to secondary region | Step Functions | Yes |

### Queues & Messaging

| Condition | Action | Pattern | Reversible |
|-----------|--------|---------|-----------|
| SQS DLQ message count > 100 | Redrive messages to source queue | Lambda | Yes |
| SQS queue depth > 10,000 | Trigger scaling of consumer Lambda | Lambda | Yes |
| SNS delivery failure rate > 5 % | Alert on-call and disable failing subscription | Lambda | Yes |

### Security & Compliance

| Condition | Action | Pattern | Reversible |
|-----------|--------|---------|-----------|
| CloudTrail disabled | Re-enable CloudTrail + create SEV-2 incident | Lambda | No (approval required) |
| IAM access key age > 90 days | Disable key and notify owner | Lambda | Yes (re-enable) |
| Config rule non-compliant | Create Security Hub finding + alert | Lambda | N/A |
| GuardDuty HIGH/CRITICAL finding | Quarantine IAM principal (deny policy) | Step Functions | Yes (remove deny) |

## Runbook Authoring Guidelines

All SSM Automation documents must:

1. Start with a **`CollectDiagnostics`** step that captures current state to S3.
2. Include an **`EligibilityCheck`** step that validates preconditions and aborts gracefully if not met.
3. Define explicit `onFailure: Abort` and `onCancel: Abort` for destructive steps.
4. End with a **`ValidateRemediation`** step that confirms the target metric has improved.
5. Emit a custom CloudWatch metric `RemediationExecuted` (namespace `AIOps/Automation`) on completion.
6. Include `maxAttempts: 3` and `timeoutSeconds: 300` per step.

## IAM Roles for Automation

| Role | Used By | Key Permissions |
|------|---------|----------------|
| `AIOpsRemediationLambdaRole` | Remediation Lambda functions | Per-action least-privilege; never `*` |
| `AIOpsSSMAutomationRole` | SSM Automation documents | EC2, ECS, RDS specific actions |
| `AIOpsStepFunctionsRole` | Step Functions state machines | `lambda:InvokeFunction`, `ssm:StartAutomationExecution` |
| `AIOpsEventBridgeRole` | EventBridge targets | `lambda:InvokeFunction`, `states:StartExecution` |

## Automation Audit Trail

Every automated action must record the following to the `AutomationAuditLog` DynamoDB table:

| Field | Description |
|-------|-------------|
| `executionId` | Unique execution ID (UUID) |
| `timestamp` | ISO 8601 UTC timestamp |
| `trigger` | EventBridge event that triggered the action |
| `runbook` | Runbook / Lambda name |
| `targetResource` | ARN of the affected resource |
| `action` | Description of the action taken |
| `outcome` | `SUCCESS` / `FAILURE` / `SKIPPED` |
| `operator` | `automated` or IAM principal if human-approved |
| `rollbackPlan` | Description or ARN of rollback runbook |
