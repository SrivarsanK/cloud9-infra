# Antigravity Execution Plan

---

## Phase 1 — Private Beta (Month 1–2)

**Goal:** One paying design partner on GCP with read-only recommendations (manual apply).

### Milestones
1. GCP discovery worker pulls resource inventory successfully.
2. Resource dashboard displays live data with filtering.
3. PSO generates nightly recommendations with confidence scores.
4. User can export Terraform snippets for manual apply.

### Engineering Tasks

#### Backend
- [ ] Set up FastAPI project structure with SQLAlchemy + Alembic.
- [ ] Implement GCP service account authentication (workload identity federation).
- [ ] Build Discovery Worker (Celery task) for GCP Compute Engine + Billing API.
- [ ] Design PostgreSQL schema: `tenants`, `resources`, `recommendations`, `audit_logs`.
- [ ] Implement row-level security for tenant isolation.
- [ ] Build PSO algorithm (particle swarm optimization) with fitness function.
- [ ] Create Celery cron for nightly PSO runs.
- [ ] Implement recommendation export endpoint (Terraform snippet generation).

#### Frontend
- [ ] Set up Next.js 16 project with App Router.
- [ ] Build resource inventory table (filter by provider, region, tag).
- [ ] Build recommendation dashboard (confidence score, estimated savings, export button).
- [ ] Implement dark mode + responsive design.

#### DevOps
- [ ] Set up PostgreSQL + TimescaleDB Docker compose for local dev.
- [ ] Configure Redis for Celery broker.
- [ ] Implement infrastructure-as-code (Terraform) for GCP deployment.
- [ ] Set up CI/CD pipeline (GitHub Actions → GCP Cloud Run).

### Definition of Done
- [ ] Resource inventory page shows 100+ GCP resources from design partner.
- [ ] PSO generates 20+ recommendations with confidence > 70.
- [ ] Design partner exports and manually applies 3+ recommendations.
- [ ] Actual savings within 20% of predicted savings (trust threshold).

---

## Phase 2 — GA (Month 3–5)

**Goal:** Multi-cloud (GCP + AWS), human-gated auto-remediation, RBAC, Terraform analyzer.

### Milestones
1. AWS discovery worker assumes cross-account roles and pulls EC2, RDS, Lambda inventory.
2. RBAC middleware enforces FinOps/SecOps/Admin permissions.
3. Approval workflow: recommendations require explicit approval before remediation.
4. Maintenance windows block execution outside allowed times.
5. Terraform analyzer parses HCL and outputs cost + security violations.
6. Audit log records all approvals + changes (immutable).

### Engineering Tasks

#### Backend
- [ ] Implement AWS STS assume-role flow for cross-account access.
- [ ] Build AWS Discovery Worker (EC2, RDS, Lambda, Cost Explorer).
- [ ] Add JWT authentication (Auth0 or AWS Cognito).
- [ ] Implement RBAC middleware: `@require_role("finops")` decorator.
- [ ] Build approval workflow endpoints: `POST /recommendations/{id}/approve`.
- [ ] Implement maintenance window checker (croniter integration).
- [ ] Build Remediation Worker with four hard properties (idempotent, atomic, blast-radius-limited, observable).
- [ ] Implement rollback logic: re-apply snapshot on failure.
- [ ] Build Terraform analyzer service (HCL parser + rule engine).
- [ ] Add feedback tracking: store accepted/rejected outcomes for PSO tuning.

#### Frontend
- [ ] Add user management page (invite users, assign roles).
- [ ] Build approval dashboard (pending recommendations with approve/reject buttons).
- [ ] Add maintenance window configuration UI.
- [ ] Build Terraform analyzer upload + violation display page.
- [ ] Add audit log viewer (filter by user, action, date range).

#### DevOps
- [ ] Set up Flower dashboard for Celery monitoring.
- [ ] Configure Prometheus + Grafana for metrics (recommendations_generated, applied, savings).
- [ ] Implement canary deployment for remediation worker (10% of tenants first).
- [ ] Add alerting on remediation failures (PagerDuty integration).

### Definition of Done
- [ ] Design partner has 50+ resources across GCP + AWS.
- [ ] 10+ recommendations approved and auto-applied via remediation agent.
- [ ] Zero production incidents from remediation (rollback works).
- [ ] SOC 2 audit log export passes compliance review.
- [ ] Terraform analyzer catches 5+ violations pre-deployment.

---

## Phase 3 — Scale (Month 6–9)

**Goal:** Azure, multi-cloud correlation, predictive forecasting, anomaly detection, NLP queries.

### Milestones
1. Azure discovery worker uses service principal authentication.
2. Multi-cloud correlation engine identifies cross-cloud inefficiencies.
3. TimescaleDB powers cost forecasting (6-month projections).
4. Anomaly detection alerts on spend spikes (>2σ from baseline).
5. Natural language query layer (LLM over resource graph).

### Engineering Tasks

#### Backend
- [ ] Implement Azure service principal authentication (MSAL library).
- [ ] Build Azure Discovery Worker (ARM resources, Cost Management API).
- [ ] Build multi-cloud correlation engine: match resources by tags, detect egress costs.
- [ ] Implement time-series forecasting: ARIMA or Prophet on TimescaleDB data.
- [ ] Build anomaly detection: statistical process control on cost metrics.
- [ ] Integrate LLM layer (Claude API) for natural language queries.
- [ ] Add priority queues in Celery (critical > high > medium > low).
- [ ] Implement rate limiting per tenant (prevent API quota exhaustion).

#### Frontend
- [ ] Add cost forecasting chart (6-month projection with confidence interval).
- [ ] Build anomaly alert dashboard (spike detection with drill-down).
- [ ] Add natural language query input ("show me underutilized resources").
- [ ] Build multi-cloud correlation view (egress cost Sankey diagram).

#### DevOps
- [ ] Migrate from Redis to Kafka if event volume exceeds 1M/day.
- [ ] Implement horizontal pod autoscaling for Celery workers.
- [ ] Add distributed tracing (OpenTelemetry) across API → worker → DB.
- [ ] Set up SLO dashboards (availability, latency, error rate).

### Definition of Done
- [ ] Design partner has resources in all three clouds (GCP, AWS, Azure).
- [ ] Multi-cloud correlation identifies 3+ cross-cloud optimization opportunities.
- [ ] Cost forecasting within 10% of actual spend at 3-month horizon.
- [ ] Anomaly detection catches 2+ spend spikes before billing review.
- [ ] NLP query accuracy > 90% (user confirms query interpretation).

---

## Team Structure

| Role | Owner | Responsibilities |
|------|-------|------------------|
| **Backend Engineer** | TBD | FastAPI, Celery, PSO engine, data models, worker layer |
| **Frontend Engineer** | TBD | Next.js dashboard, real-time updates, RBAC UI, Terraform analyzer UI |
| **Infra/DevOps Engineer** | TBD | Credential management, deployment pipeline, monitoring, SOC 2 compliance |
| **Architect/Product** | You | Multi-cloud integration design, scope management, stakeholder comms |
| **Terraform Analyzer Owner** | TBD (4th person) | Separate Python service, HCL parser, rule engine, no shared state |

**Ownership Areas:**
- Backend engineer owns `workers/`, `api/`, `models/`, `pso/`.
- Frontend engineer owns `app/`, `components/`, `hooks/`.
- DevOps owns `terraform/`, `.github/workflows/`, `docker/`, `monitoring/`.
- Terraform owner owns `terraform-analyzer/` as separate service.

---

## Risk Register

| Risk | Impact | Mitigation |
|------|--------|------------|
| **1. Credential management fails in production** | Critical: cannot access customer cloud data | - Implement workload identity federation from day one (no long-lived keys)<br>- Test failover: credential expiry → automatic refresh<br>- On-call runbook: manual credential rotation |
| **2. Remediation agent applies incorrect change** | Critical: customer data loss, trust destroyed | - Four hard properties enforced via unit + integration tests<br>- Canary deployment: 10% of tenants first<br>- Rollback tested in staging with chaos engineering |
| **3. PSO recommendations inaccurate** | High: design partner churns | - Phase 1 manual-only to build trust data<br>- Feedback loop: track actual vs. predicted savings<br>- A/B test fitness function weights |
| **4. Scope creep: multi-cloud too early** | High: Phase 1 slips, no paying customer | - Phase 1 GCP-only, AWS/ Azure explicitly deferred<br>- Product owner owns scope enforcement<br>- Design partner contract: GCP-only for 60 days |
| **5. SOC 2 audit fails** | High: enterprise deals blocked | - Immutable audit logs from Phase 1 (append-only table)<br>- Access logging all API requests<br>- Quarterly access reviews automated |

---

## Appendix: File Structure

```
antigravity/
├── backend/
│   ├── api/                 # FastAPI routes + RBAC middleware
│   ├── workers/             # Celery tasks (discovery, pso, remediation, terraform)
│   ├── models/              # SQLAlchemy models + migrations
│   ├── pso/                 # PSO algorithm + fitness functions
│   ├── credentials/         # Workload identity federation, STS assume-role
│   └── celery_config.py
├── frontend/
│   ├── app/                 # Next.js App Router pages
│   ├── components/          # Reusable UI components
│   ├── hooks/               # React hooks for API calls
│   └── lib/                 # API client, auth utils
├── terraform-analyzer/      # Separate Python service
│   ├── parser.py            # HCL AST builder
│   ├── rules/               # Cost + security rules
│   └── main.py              # FastAPI entrypoint
├── terraform/               # Infra for antigravity itself
│   ├── gcp/                 # GCP resources (Cloud Run, PostgreSQL, Redis)
│   ├── aws/                 # AWS resources (ECS, RDS, ElastiCache)
│   └── monitoring/          # Prometheus, Grafana, Flower
├── .github/workflows/       # CI/CD pipelines
└── docs/                    # This plan, requirements, design
```
