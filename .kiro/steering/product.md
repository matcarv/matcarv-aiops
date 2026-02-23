# MatCarv AIOps — Product Overview

## Purpose

MatCarv AIOps is an AI-driven operations platform for AWS environments. It provides automated monitoring, intelligent alerting, self-healing remediation, and continuous cost optimisation by applying machine learning and rule-based automation to operational data produced by AWS services.

## Goals

- **Reduce MTTR (Mean Time To Recover):** Detect anomalies and trigger automated runbooks before human intervention is required.
- **Improve observability:** Aggregate metrics, logs, and traces from all AWS services into a unified view.
- **Automate routine operations:** Replace manual, repetitive tasks (scaling, patching, log rotation, snapshot management) with event-driven workflows.
- **Optimise costs:** Identify idle or over-provisioned resources and recommend or execute right-sizing actions.
- **Increase security posture:** Continuously audit IAM policies, Security Group rules, and CloudTrail events for anomalies.

## Target Users

| Role | Primary Use |
|------|-------------|
| Cloud Operations Engineer | Monitors dashboards, reviews alerts, approves automated remediations |
| SRE / DevOps Engineer | Authors runbooks, configures automation, tunes alert thresholds |
| FinOps Analyst | Reviews cost reports, approves right-sizing recommendations |
| Security Engineer | Reviews compliance findings, investigates security events |
| Engineering Manager | Reviews SLO dashboards and incident post-mortems |

## Key Features

1. **Anomaly Detection** — CloudWatch Anomaly Detection and custom ML models identify unusual resource behaviour.
2. **Automated Runbooks** — AWS Systems Manager Automation documents execute pre-approved remediation steps.
3. **Intelligent Alerting** — SNS + Lambda-based alert routing with suppression, grouping, and severity enrichment.
4. **Cost Intelligence** — AWS Cost Explorer API integration surfaces savings recommendations.
5. **Compliance Scanning** — AWS Config rules and Security Hub standards provide continuous compliance checks.
6. **Incident Management** — EventBridge-driven incident lifecycle: detection → triage → remediation → post-mortem.
7. **SLO Tracking** — CloudWatch dashboards exposing error-budget burn rates per service.

## Out of Scope

- On-premises or multi-cloud environments (future roadmap).
- Application-level code changes as part of remediation.
- Capacity planning beyond AWS resource right-sizing.
