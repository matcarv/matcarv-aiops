# MatCarv AIOps — Monitoring & Observability

## Principles

1. **Everything is an alarm** — every AWS resource that can produce metrics must have at least one CloudWatch Alarm.
2. **Structured logs everywhere** — all Lambda functions emit JSON logs via AWS Lambda Powertools Logger.
3. **Trace every request** — X-Ray active tracing is mandatory for Lambda and API Gateway.
4. **Dashboards are living documents** — CloudWatch dashboards are managed as CDK code, not manually.
5. **Alert on symptoms, not causes** — customer-facing SLIs drive alerting; internal metrics are for diagnostics.

## CloudWatch Alarms

### Alarm Naming Convention

```
<environment>-<service>-<metric>-<severity>
```

Examples:
- `prod-order-api-error-rate-critical`
- `prod-ec2-cpu-utilization-warning`

### Standard Alarm Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Lambda error rate | ≥ 1 % | ≥ 5 % |
| Lambda P99 duration | ≥ 80 % of timeout | ≥ 95 % of timeout |
| SQS ApproximateAgeOfOldestMessage | ≥ 5 min | ≥ 15 min |
| DynamoDB ThrottledRequests | ≥ 1 / min | ≥ 10 / min |
| EC2 CPUUtilization | ≥ 70 % | ≥ 90 % |
| RDS FreeStorageSpace | ≤ 20 % | ≤ 10 % |
| ALB HTTPCode_Target_5XX_Count | ≥ 10 / min | ≥ 50 / min |

### Alarm Actions

- **Warning alarms** → SNS topic `<env>-aiops-warning` → Slack `#ops-warnings`.
- **Critical alarms** → SNS topic `<env>-aiops-critical` → PagerDuty + Slack `#ops-incidents`.
- All alarm state changes also fan-out to EventBridge for automated triage.

### Anomaly Detection

- Use **CloudWatch Anomaly Detection** for metrics with seasonal patterns (e.g., request rates, error rates).
- Anomaly detection bands are configured with a standard deviation of `2` for warnings, `3` for critical.
- Custom ML-based anomaly detection Lambda is triggered on schedule (every 5 minutes) for metrics not covered by CloudWatch Anomaly Detection.

## CloudWatch Logs

### Log Group Naming

```
/aws/lambda/<function-name>
/aws/ecs/<cluster-name>/<service-name>
/aws/rds/instance/<db-instance-id>/error
/aiops/<custom-app-name>
```

### Log Retention

| Environment | Retention |
|-------------|-----------|
| prod | 90 days |
| staging | 30 days |
| dev | 7 days |

All log groups export to S3 (Log Archive account) for long-term retention (1 year).

### Metric Filters

Define CloudWatch Metric Filters for key log patterns:

| Pattern | Metric Name |
|---------|------------|
| `{ $.level = "ERROR" }` | `ApplicationErrors` |
| `{ $.level = "WARN" }` | `ApplicationWarnings` |
| `{ $.statusCode >= 500 }` | `Http5xxErrors` |
| `{ $.coldStart = true }` | `LambdaColdStarts` |

### Log Insights Queries

Save standard queries in `docs/runbooks/cloudwatch-insights-queries.md` for:
- Top errors by Lambda function in the last hour.
- Slowest requests by P99 latency.
- Cold start frequency per function.

## AWS X-Ray Tracing

- **Active tracing** enabled on all Lambda functions via `Tracing: Active` (CDK: `tracing: Tracing.ACTIVE`).
- **X-Ray groups** created per service to isolate trace sampling.
- All downstream HTTP calls (via AWS SDK or `axios`) use X-Ray segments automatically via Powertools Tracer.
- Sampling rule: 5 % default; 100 % for requests that result in errors.

## CloudWatch Dashboards

### Standard Dashboards

| Dashboard | Contents |
|-----------|----------|
| `AIOps-Overview` | SLO burn rates, open incidents, alert volume trend |
| `AIOps-Lambda` | Error rates, durations, throttles, cold starts per function |
| `AIOps-Infrastructure` | EC2, RDS, ECS, EKS resource utilisation |
| `AIOps-Cost` | Daily spend, top cost drivers, savings recommendations |
| `AIOps-Security` | Security Hub findings by severity, Config compliance score |

### Dashboard Conventions

- Use **Alarm Status** widgets at the top of each dashboard for at-a-glance health.
- All time series graphs use a **1-hour default period** with `SampleCount` statistics for count metrics.
- Period is set in CDK; never hardcoded in CloudWatch console.

## AWS Health

- **AWS Health API** events are consumed via EventBridge and enriched into the alert pipeline.
- Health events with `eventTypeCategory = "issue"` are treated as Warning severity or above.

## Tagging for Observability

All AWS resources must be tagged to enable metric filtering:

| Tag Key | Example Value |
|---------|--------------|
| `Environment` | `prod` |
| `Service` | `order-api` |
| `Team` | `platform` |
| `CostCentre` | `CC-1234` |
