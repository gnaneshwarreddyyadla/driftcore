# DriftCore Evaluation Plan

A publication-grade DriftCore evaluation should test both usefulness and safety.

## Evaluation Matrix

| Experiment | Goal | Metrics |
|---|---|---|
| E1 — Drift-to-Proposal Correctness | Evaluate proposal quality across synthetic incidents | useful proposal rate, malformed proposal rate, human acceptance rate |
| E2 — Scorer Ablation | Compare primary-only, self-critique, and blind scorer designs | unsafe proposal rate, false confidence rate, gate precision/recall |
| E3 — Gate Calibration | Tune deterministic thresholds across severity tiers | false pass rate, false reject rate, human-review rate |
| E4 — Breaker Effectiveness | Validate safety breakers under injected failures | trip time, missed unsafe events, false trips |
| E5 — Rollback Safety | Test rollback under intervening changes | rollback success rate, unsafe rollback prevention |
| E6 — Lineage-Aware Prompting | Measure lineage context value | downstream awareness, unnecessary fix rate, approval rate |

## Evaluation Boundary

Current private validation supports the governed proposal path. It does not yet prove full autonomous remediation or production-scale reliability.
