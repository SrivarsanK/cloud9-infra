# Antigravity Requirements

## Product Overview

Antigravity is a multi-cloud infrastructure cost optimizer that uses Particle Swarm Optimization (PSO) to generate resource right-sizing recommendations. It provides auto-remediation with human approval workflows and Terraform static analysis to catch cost + security issues before deployment.

---

## Functional Requirements

### Phase 1 — Private Beta (Month 1–2)

1. **GCP Discovery Worker**: Authenticate via service account credentials (workload identity federation) and pull resource inventory from GCP Compute Engine, GKE, and Cloud Billing APIs.
2. **Resource Inventory Dashboard**: Display discovered resources with metadata (type, region, current cost, utilization metrics) in a Next.js table view with filtering + sorting.
3. **PSO Recommendation Engine**: Run nightly cron job that evaluates each resource against fitness functions (cost vs. utilization) and outputs recommendations with confidence scores (0–100) and estimated monthly savings.
4. **Manual Apply Workflow**: Generate Terraform snippets or `gcloud` CLI commands for each recommendation; user copies and executes manually. No auto-apply.
5. **Confidence Scoring**: Each recommendation includes a breakdown of factors (CPU avg < 30%, memory < 40%, 30-day window) and a confidence score based on data completeness.
6. **Single Tenant Onboarding**: Support one design partner tenant with isolated data (row-level security in PostgreSQL).

### Phase 2 — GA (Month 3–5)

1. **AWS Discovery Worker**: Cross-account IAM role assumption with read-only permissions (SecurityAudit + ReadOnlyAccess) for EC2, RDS, Lambda, and Cost Explorer.
2. **Multi-User RBAC**: Three roles:
   - **FinOps**: View cost data, approve cost-related recommendations
   - **SecOps**: View security recommendations, approve security-related changes
   - **Admin**: Full access, can modify maintenance windows, approve all changes
3. **Human-Gated Auto-Remediation**: Approval workflow where recommendations require explicit approval before execution; notification via email + in-dashboard alert.
4. **Maintenance Windows**: Per-tenant configuration defining allowed execution windows (e.g., "Saturdays 02:00–06:00 UTC"); remediation jobs blocked outside windows.
5. **Terraform Analyzer**: Parse HCL files, run cost + security rules (e.g., "no unencrypted S3 buckets", "reserved instance coverage < 50%"), output violations with line numbers and suggested fixes.
6. **Audit Log**: Immutable append-only table recording who approved what, when, with what confidence score and expected savings.
7. **Feedback Loop**: Track accepted/rejected recommendations; store outcome data (actual savings vs. predicted) for PSO weight tuning.

### Phase 3 — Scale (Month 6–9)

1. **Azure Discovery Worker**: Service principal authentication with read access to ARM resources, Cost Management API.
2. **Multi-Cloud Correlation**: Detect cross-cloud inefficiencies (e.g., "data egress from GCP to AWS costs $X/mo — consider VPC endpoints or CDN").
3. **Predictive Cost Forecasting**: Use TimescaleDB time-series data to project 6-month cost trends; surface anomalies ("spend increased 40% WoW").
4. **Anomaly Detection**: Alert on spend spikes (>2σ from baseline) with drill-down to resource level.
5. **Natural Language Query**: LLM layer over resource graph — parse queries like "show me resources under 20% CPU for 30+ days without reserved tagging" into structured queries.
6. **Priority Queues**: Celery priority routing for remediation jobs (critical > high > medium > low).

---

## Non-Functional Requirements

### Security
- All credentials stored encrypted at rest (AES-256) using cloud KMS (GCP CMEK, AWS KMS).
- No long-lived credentials in code — use workload identity federation (GCP → GCE/AWS) or IAM role assumption (AWS).
- Row-level security in PostgreSQL: tenants cannot query other tenants' data.
- SOC 2 compliance: immutable audit logs, access logging, quarterly access reviews.

### Reliability
- Remediation agent must be idempotent: re-running same job produces identical state.
- Atomic changes with rollback: snapshot state before apply; rollback re-applies snapshot.
- Blast radius limits: max 10 changes per tenant per hour; skip resources tagged `do-not-automate`.
- Retry logic with exponential backoff for transient API failures (max 3 retries).

### Observability
- Structured logging (JSON) with correlation IDs tracing request → queue → worker → DB.
- Flower dashboard for Celery task monitoring (queue depth, task duration, failure rate).
- Prometheus metrics: recommendations_generated, recommendations_applied, mean_savings_per_recommendation.
- Distributed tracing (OpenTelemetry) across API → queue → worker.

### Compliance
- GDPR: data export + deletion endpoints per tenant.
- SOC 2: audit logs immutable for 7 years; access logs retained for 1 year.
- Change management: all production changes require approval ticket reference.

---

## Constraints and Non-Goals

### Constraints
- Phase 1 supports GCP only; AWS and Azure deferred to Phase 2 and 3.
- PSO runs as nightly batch, not real-time (Phase 1); on-demand triggered runs added in Phase 2.
- No auto-remediation in Phase 1 — manual apply only to build trust.

### Non-Goals
- No support for on-premises infrastructure (cloud-only: GCP, AWS, Azure).
- No Kubernetes workload tuning (only node-level right-sizing in Phase 1–2).
- No multi-account aggregation for MSP use cases (single tenant per deployment in Phase 1).
- No native mobile app (web dashboard only).
