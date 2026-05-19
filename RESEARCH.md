# DriftCore — Research Summary

**DriftCore: A Governed Control Plane for AI-Assisted Data Pipeline Remediation**

---

## Abstract

Warehouses and pipelines fail in subtle ways: schema drift, bad backfills, row-level distribution shifts, downstream dbt breakage, stale lineage, and unexpected changes that quietly affect production analytics.

Manual remediation is slow. Unconstrained LLM remediation is unsafe. DriftCore explores a middle path: treating AI-generated fixes as governed proposals inside a deterministic control plane rather than as direct actions.

The private prototype captures drift through CDC-style change capture, deduplicates correlated symptoms into incidents, builds retrieval-augmented context, and generates structured proposals with a primary LLM. A second model scores the incident independently — without seeing the proposal — and a deterministic gate compares the signals before routing to human approval.

Persistent breakers, severity lanes, audit logs, and rollback-aware design surround the core flow.

The central claim is specific:

> An asymmetric proposer/scorer split combined with deterministic routing is a more credible foundation for LLM-assisted infrastructure operations than either single-model self-critique or unconstrained agentic execution.

---

## 1. Motivation

A single warehouse table can feed dashboards, ML features, executive reporting, dbt models, reverse ETL jobs, and customer-facing analytics.

When that table drifts, the blast radius is wide and the pressure to fix it quickly is real — which is exactly when bad decisions get made.

The interesting question is not:

> Can an LLM generate a fix?

It can.

The more important question is:

> How do we let an LLM contribute to remediation without letting it act unsafely?

DriftCore is built around that question.

---

## 2. Thesis

LLMs should sit **inside** a control plane, not **on top of** one.

In practice:

- AI generates proposals, summaries, and explanations.
- Deterministic logic compares signals and decides routing.
- Humans approve anything with real blast radius.
- Persistent state bounds automation when risk rises.
- Every decision is auditable.
- Every meaningful action should be reversible or at least rollback-aware.

The goal is bounded autonomy.

---

## 3. System Model

DriftCore models remediation as a controlled workflow:

```text
detect
→ deduplicate
→ retrieve context
→ propose
→ independently score
→ gate
→ approve
→ execute
→ audit
→ rollback if needed
```

The verified private prototype path currently covers:

```text
detect
→ deduplicate
→ retrieve context
→ propose
→ independently score
→ gate
→ approve
```

Execution, audit-at-scale, and rollback validation are partially built or designed, but not yet end-to-end verified.

---

## 4. Architecture

The system is a sequence of small components rather than a monolithic agent.

Each stage has a narrow contract. That makes failure modes easier to reason about and individual components easier to replace.

See:

```text
diagrams/architecture.png
diagrams/runtime-flow.png
diagrams/safety-model.png
```

---

## 5. Contributions

### 5.1 Blind Proposer / Scorer Split

The primary model generates a remediation proposal.

A second model scores the incident — severity, risk, downstream sensitivity, and confidence — without seeing the proposal.

This rules out one of the most common single-model failure modes: the model that wrote an answer is also the model asked to grade it.

**Open question:**  
Does a blind independent scorer measurably reduce unsafe or overconfident proposals compared with self-critique?

---

### 5.2 Deterministic Agreement Gate

The gate is deliberately non-intelligent.

It takes proposal confidence, independent score, severity tier, and policy thresholds, then returns a route.

Keeping this layer outside the model is the point.

The LLMs provide signals. The gate decides what is allowed to move forward.

**Open question:**  
Can deterministic gates retain enough useful AI signal without becoming so conservative that the system stops being useful?

---

### 5.3 Persistent Safety Breakers

Breakers are state machines that survive restarts.

They watch for risk signals such as rollback spikes, short-window failure bursts, detection-rate anomalies, LLM cost spikes, warehouse cost spikes, and repeated execution failures.

When thresholds trip, automation pauses or degrades.

**Open question:**  
Are persistent breakers a practical safety envelope for AI-assisted data operations, or do they collapse into noise under real workloads?

---

### 5.4 Human Approval as Part of the Data Model

Approval is not a feature bolted on at the end.

Severity lanes, dual approval for high-impact changes, modified-proposal tracking, and review records are first-class parts of the design.

**Open question:**  
What is the right approval structure when the object being approved is an LLM-generated infrastructure change?

---

### 5.5 Rollback-Aware Remediation

DriftCore treats remediation as work that should be reasoned about before and after execution.

The rollback model is designed around rollback manifests, intervening-change checks, rollback records, lineage refresh, and escalation paths for repeated failures.

**Open question:**  
How much operational risk from AI-assisted fixes can be reduced through rollback-aware design?

---

### 5.6 Lineage-Aware Context

dbt manifests and Snowflake lineage can extend the RAG context.

The hypothesis is that lineage-awareness reduces unnecessarily aggressive fixes and improves blast-radius reasoning.

**Open question:**  
Does lineage-aware prompting measurably improve proposal quality, or does it mostly add tokens?

---

## 6. Evaluation Plan

A defensible evaluation needs more than anecdote.

Planned experiments:

| Experiment | Goal | Metrics |
|---|---|---|
| Drift-to-Proposal Correctness | Test proposal quality on synthetic drift incidents | useful proposal rate, malformed proposal rate, human acceptance rate |
| Scorer Ablation | Compare primary-only, self-critique, and blind scorer designs | unsafe proposal rate, false confidence rate, gate precision/recall |
| Gate Calibration | Sweep thresholds across severity tiers | false pass rate, false reject rate, human-review rate |
| Breaker Effectiveness | Inject rollback bursts, detector spikes, cost spikes | trip time, missed unsafe events, false trips |
| Rollback Safety | Test rollback with and without intervening changes | rollback success rate, unsafe rollback prevention |
| Lineage-Aware Prompting | Compare proposals with and without lineage context | downstream awareness, unnecessary fix rate, approval rate |

None of these experiments have been run at publication scale yet.

---

## 7. Defensible Claims

What can be said honestly today:

- DriftCore implements a blind proposer/scorer architecture for data drift remediation.
- It uses deterministic gating over LLM signals rather than letting an LLM make routing decisions.
- It is designed around persistent safety breakers, severity-based approval, and rollback-aware execution.
- It treats auditability and human approval as part of the system model.
- It is research-oriented and explicitly bounded.
- Its verified private path currently reaches proposal generation, independent scoring, deterministic gating, persistence, and human approval.

---

## 8. Claims Not Being Made

DriftCore is not claiming:

- fully autonomous self-healing
- production readiness
- production validation against a real enterprise data stack
- proven superiority to commercial observability tools
- enterprise-grade multi-tenancy or hardening
- complete rollback safety under all conditions
- unrestricted automated execution

---

## 9. Related Work Categories

A future paper would need to engage with:

- data observability and quality monitoring
- schema drift detection
- data reliability platforms
- LLM agents for software and infrastructure
- AI governance and human-in-the-loop systems
- workflow orchestration and control-plane design
- database transaction safety
- rollback semantics
- incident management systems

---

## 10. Scope and Limitations

- The public repository is a sanitized showcase; the full implementation is private.
- The verified path ends at human approval.
- The execution loop is partially built but not validated end-to-end.
- Benchmarks are designed but not yet run at scale.
- Reported breaker thresholds are design defaults rather than measured calibrations.
- Enterprise concerns — RBAC, tenant isolation, immutable audit enforcement, disaster recovery — are scoped but not implemented.
- No comparative evaluation against commercial tools exists yet.

---

## 11. Future Work

- run the synthetic drift benchmark suite end to end
- complete the scorer/gate ablation with statistical analysis
- measure latency and cost under realistic load
- calibrate breaker thresholds against measured workloads
- harden the execution loop
- validate rollback under intervening-change scenarios
- strengthen RBAC, tenant isolation, and audit immutability
- draft an architecture paper with experimental results

---

## 12. Central Claim

LLM-assisted remediation becomes more credible — not just safer — when generated fixes are treated as proposals inside a deterministic control plane, with independent scoring, severity-based human approval, and persistent safety breakers.

That is the bet.

The rest of the work is evidence.

---

**Gnaneshwar Yadla**  
Data + AI Engineer  
[yadla.dev](https://yadla.dev)
