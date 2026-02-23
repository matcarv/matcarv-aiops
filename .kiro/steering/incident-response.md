# MatCarv AIOps — Incident Response

## Incident Severity Levels

| Level | Name | Customer Impact | Initial Response | Example |
|-------|------|----------------|-----------------|---------|
| SEV-1 | Critical | Complete outage or data loss | Immediate (< 5 min) | Production API returning 5xx for all users |
| SEV-2 | High | Significant degradation | < 15 min | Latency P99 > 3× normal |
| SEV-3 | Medium | Partial degradation | < 1 hour | Single AZ issue, failover active |
| SEV-4 | Low | Minor or no customer impact | Next business day | Non-critical batch job failure |

## Incident Lifecycle

```
Detection → Triage → Acknowledgment → Investigation → Remediation → Resolution → Post-Mortem
```

### 1. Detection

- **Automated:** CloudWatch Alarm transitions to `ALARM` → SNS → alert-router Lambda.
- **Manual:** Engineer observes anomaly and creates an incident via Slack `/incident` command.
- All detections produce an **Alert Event** on the EventBridge `aiops-alerts` bus.

### 2. Triage (Automated)

The `alert-router` Lambda performs automated triage:

1. **Deduplication** — checks DynamoDB `AlertDeduplication` table (TTL = 10 min) to suppress duplicate alerts.
2. **Enrichment** — adds service owner, runbook URL, SLO impact, and AWS account context.
3. **Severity mapping** — maps CloudWatch alarm names to SEV levels using a configuration table.
4. **Routing** — sends enriched alert to the appropriate SNS topic and Slack channel.
5. **Auto-remediation check** — if an approved runbook exists, triggers `RemediationOrchestrator` Step Function.

### 3. Acknowledgment

- On-call engineer acknowledges the incident in PagerDuty or Slack within the SLA window.
- If not acknowledged within the SLA, PagerDuty escalates to the secondary on-call.
- Acknowledgment creates an incident record in DynamoDB `Incidents` table.

### 4. Investigation

Standard investigation steps (automate where possible):

1. Open the relevant **CloudWatch Dashboard** for the affected service.
2. Run the relevant **CloudWatch Insights** query from `docs/runbooks/cloudwatch-insights-queries.md`.
3. Check **X-Ray** for trace anomalies in the affected time window.
4. Review **CloudTrail** for recent API changes (past 1 hour).
5. Check **AWS Health Dashboard** for service events.
6. Correlate with recent deployments via the `aiops-deployments` EventBridge bus.

### 5. Remediation

#### Automated Runbooks

Automated runbooks are AWS Systems Manager Automation documents stored in `scripts/ssm/`.

| Runbook | Trigger Condition | Actions |
|---------|------------------|---------|
| `AIOps-RestartECSService` | ECS service task count = 0 | Forces new deployment |
| `AIOps-ScaleOutEC2ASG` | EC2 CPU > 90 % for 5 min | Increases desired capacity by 25 % |
| `AIOps-PurgeSQSDLQ` | DLQ message count > 100 | Moves messages to source queue |
| `AIOps-RotateLambdaAlias` | Lambda error rate > 10 % | Rolls back alias to previous version |
| `AIOps-RDSFailover` | RDS multi-AZ event detected | Confirms failover and notifies |
| `AIOps-InvalidateCFCache` | ALB 5xx spike | Triggers CloudFront invalidation |

All automated runbooks require:
- An approved IAM role with least-privilege permissions.
- An entry in the `AutoRemediationApproval` DynamoDB table.
- A maximum execution time of 15 minutes (Step Functions timeout).

#### Manual Escalation

If automated remediation fails or no runbook exists:
1. Incident commander is paged.
2. Incident bridge (Chime/Zoom) is opened.
3. Engineers follow the relevant manual runbook in `docs/runbooks/`.

### 6. Resolution

- Incident is resolved when CloudWatch Alarm returns to `OK`.
- `alert-router` Lambda automatically closes the incident record in DynamoDB.
- Resolution message posted to Slack `#ops-incidents`.

### 7. Post-Mortem

A post-mortem is **mandatory** for SEV-1 and SEV-2 incidents.

- Use the template at `docs/post-mortems/_template.md`.
- Must be filed within **48 hours** of resolution.
- Includes: timeline, root cause, impact, contributing factors, action items with owners and due dates.
- Action items are tracked as GitHub Issues labelled `incident-action-item`.

## EventBridge Rules for Incident Automation

| Rule | Source | Detail Type | Target |
|------|--------|-------------|--------|
| Route critical alerts | `aiops.alerts` | `AlertCreated` | `alert-router` Lambda |
| Auto-remediate | `aiops.alerts` | `AlertEnriched` | `RemediationOrchestrator` Step Function |
| Create incident record | `aiops.alerts` | `AlertAcknowledged` | `incident-manager` Lambda |
| Send weekly digest | `aws.scheduler` | Scheduled | `incident-digest` Lambda |

## On-Call Conventions

- On-call rotation managed in **PagerDuty**.
- Primary on-call handles all SEV-1 and SEV-2.
- Secondary on-call is auto-paged if primary does not acknowledge within 5 minutes (SEV-1) or 15 minutes (SEV-2).
- Business hours (09:00–18:00 local time): SEV-3 alerts go to Slack only.
- After hours: SEV-3 alerts go to Slack and email; no page.

## Communication Templates

### Incident Declared (Slack)

```
🔴 *INCIDENT DECLARED — SEV-<N>*
*Service:* <service-name>
*Impact:* <brief description>
*Incident ID:* INC-<id>
*Commander:* <@user>
*Bridge:* <link>
*Status page:* Being updated
```

### Incident Resolved (Slack)

```
✅ *INCIDENT RESOLVED — SEV-<N>*
*Service:* <service-name>
*Duration:* <HH:MM>
*Incident ID:* INC-<id>
*Post-mortem due:* <date>
```
