# 🧩 DriftCore

## Governed AI-Assisted Remediation for Data Pipeline Drift

DriftCore is a research prototype exploring a specific question:

**How do you let a language model help fix broken data pipelines without trusting it to act on production?**

DriftCore treats AI-generated fixes as structured proposals that must pass through blind independent scoring, deterministic gating, human approval, persistent safety breakers, and rollback-aware control logic.

> This is the public research and architecture repository. The full implementation is private because it contains integration code, deployment configuration, internal automation logic, credentials-sensitive setup, and environment-specific infrastructure.

---

# 🚀 Features

📡 **Warehouse drift capture**  
Models how warehouse and pipeline signals can be captured as structured change events.

🧾 **CDC-style change events**  
Represents row-level drift and schema-related changes as incident signals.

🧩 **Incident deduplication**  
Groups correlated symptoms into deduplicated incident batches.

🧠 **RAG-based proposal context**  
Builds retrieval-augmented context before generating remediation proposals.

🤖 **Primary LLM proposer**  
Uses a primary LLM to generate structured remediation proposals.

🧪 **Blind independent scorer**  
Scores the incident independently without seeing the primary model’s proposal.

🧱 **Deterministic agreement gate**  
Routes proposals using deterministic thresholds instead of letting an LLM decide.

🧑‍⚖️ **Human approval workflow**  
Keeps human review as a first-class part of higher-risk remediation.

🛑 **Persistent safety breakers**  
Uses durable breaker state to pause or degrade automation when risk increases.

🔁 **Rollback-aware control model**  
Treats remediation as work that should be auditable, reversible, or rollback-aware.

📊 **Observability-aware design**  
Frames remediation as an auditable control-plane workflow, not a black-box agent.

❄️ **Snowflake / dbt lineage direction**  
Includes optional design direction for warehouse and lineage-aware context.

---

# 🧩 System Architecture

The following diagram shows DriftCore’s governed remediation flow: warehouse drift is captured, deduplicated into incidents, converted into RAG context, proposed by a primary LLM, independently scored, gated deterministically, routed through approval, and then passed into the control-plane / rollback model.

![Architecture overview](diagrams/architecture.png)

---

# 🔄 Runtime Flow

The strongest verified private prototype path currently reaches proposal generation, blind scoring, deterministic gating, proposal persistence, and human approval.

![Runtime flow](diagrams/runtime-flow.png)

Verified private prototype flow:

```text
warehouse row drift
→ CDC capture
→ deduplication
→ RAG context construction
→ primary LLM proposal
→ blind independent score
→ deterministic agreement gate
→ proposal persistence
→ human approval
```

---

# 🛡️ Safety Model

DriftCore is designed around bounded autonomy.

The LLM path is only one part of the system. Safety is enforced through deterministic routing, persistent breaker state, human approval, rollback design, and audit records.

![Safety model](diagrams/safety-model.png)

Safety boundaries include:

- prompt sanitization for warehouse-derived text
- isolated untrusted prompt sections
- blind independent scoring
- agreement thresholds
- severity-based routing
- human approval for higher-risk changes
- persistent cost and rollback breakers
- audit logging for decisions and state transitions
- rollback manifests and notification outbox

---

# 🛠️ Tech Stack

This public repository documents the architecture and research direction. The private prototype uses or targets the following stack:

| Area | Technology |
|---|---|
| Language | Python 3.11 |
| Database | Postgres |
| ORM / Models | SQLAlchemy |
| Orchestration | Apache Airflow |
| API | FastAPI |
| LLM Provider | Anthropic |
| Evaluation / Tests | Pytest |
| Observability | Prometheus, Grafana, Alertmanager |
| Tracing | OpenTelemetry / Jaeger |
| Warehouse Extension | Snowflake |
| Lineage Extension | dbt manifest ingestion |
| Notifications | Slack webhook support |
| Deployment Scaffolding | Docker, Docker Compose |

---

# ✅ Validation Snapshot

The private prototype has been validated through a mix of live integration tests and targeted deterministic test suites.

The honest boundary:

> DriftCore has a verified governed proposal path, but not yet a proven fully autonomous self-healing apply loop.

| Area | Result | What It Validates | Notes |
|---|---:|---|---|
| Targeted Non-Live Suite | `26 passed, 1 warning` | Deterministic/local behavior across selected core tests | Private repo validation |
| Harness Canary | `2 passed` | Live integration harness configuration | Phase 8 LLM spend: `$0.0000 / $5.00` |
| Live Pipeline Path | `1 passed` | Real Postgres + live Anthropic proposal/scoring path | Runtime: `11.47s`, spend: `$0.0137 / $5.00` |
| Live Pipeline Repeat | `1 passed` | Repeatability of live proposal/scoring path | Runtime: `11.34s`, spend: `$0.0107 / $5.00` |

---

# 📦 Public Repository Contents

```text
README.md                 Public project overview
RESEARCH.md               Research framing and paper-style summary
docs/ARCHITECTURE.md      Architecture notes
docs/EVALUATION_PLAN.md   Planned benchmark and validation strategy
docs/ROADMAP.md           Future implementation and hardening plan
diagrams/                 Hand-drawn architecture diagrams
examples/                 Sanitized mock incidents and gate decisions
screenshots/              Placeholder for public-safe screenshots
LICENSE                   Public repository license
```

This public repo does **not** include:

- private implementation code
- deployment configuration
- credentials
- private schemas
- secrets
- environment-specific infrastructure
- internal automation logic

---

# 🧪 Example Artifacts

## Mock incident

```json
{
  "incident_id": "mock-incident-001",
  "source": "warehouse.orders",
  "type": "row_data_drift",
  "severity": "P2",
  "summary": "Unexpected spike in null values for payment_status",
  "affected_columns": ["payment_status"],
  "detected_at": "2026-05-17T12:00:00Z"
}
```

## Mock gate decision

```json
{
  "proposal_id": "mock-proposal-001",
  "primary_confidence": 0.82,
  "independent_score": 0.78,
  "severity": "P2",
  "gate_status": "HUMAN_REVIEW_REQUIRED",
  "reason": "Confidence is acceptable, but affected table has downstream consumers."
}
```

More examples live in `examples/`.

---

# ⚠️ What DriftCore Is Not

DriftCore is not claiming:

- fully autonomous self-healing
- production readiness
- production validation against a real enterprise data stack
- proven superiority to commercial observability tools
- enterprise-grade multi-tenancy or hardening
- complete rollback safety under all conditions
- unrestricted automated execution

This project is stronger when the boundary is clear.

---

# 🧠 Research Positioning

DriftCore is best understood as a systems prototype for **bounded AI remediation**, not a finished autonomous agent.

Its main contribution is a governance-centric architecture that surrounds LLM-generated fixes with:

- deterministic gates
- blind independent scoring
- persistent safety state
- human approval
- audit logs
- rollback-aware control logic

Safe claim:

> DriftCore is a governed AI-assisted remediation prototype for data pipeline drift.

Avoid claiming:

> DriftCore is a fully autonomous production self-healing data platform.

---

# 🧪 CI / Validation Scope

Default CI should validate deterministic unit tests and static checks only.

Live Anthropic, Snowflake, Postgres, and chaos tests should be excluded from default CI and run locally or through a manual workflow with explicit secrets.

---

# 🗺️ Roadmap

## Now — Public Research Showcase

- architecture write-up
- hand-drawn diagrams
- research summary
- sanitized examples
- evaluation plan

## Next — Demo Artifacts

- approval-flow screenshots
- public-safe dashboard screenshots
- test result captures
- mock incident walkthrough

## Later — Research Validation

- synthetic drift benchmark suite
- scorer/gate ablation study
- latency and cost measurement
- breaker-threshold calibration
- rollback safety experiments

## Eventually — Hardening

- complete approved-proposal execution path
- validate rollback under intervening-change scenarios
- RBAC
- tenant isolation
- append-only audit enforcement
- secrets management
- disaster-recovery planning

---

# 👤 Author

**Gnaneshwar Yadla**  
Data + AI Engineer  
[yadla.dev](https://yadla.dev)
