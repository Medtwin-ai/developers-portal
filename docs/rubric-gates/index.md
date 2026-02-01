<!--
PATH: docs/rubric-gates/index.md
PURPOSE: Conceptual overview of Rubric Gates for developers.
-->

# Rubric Gates Overview

Rubric Gates is MedTWIN's certificate-first validation framework for AI-generated clinical research artifacts.

## What Problem Does It Solve?

AI agents generating clinical cohorts, mappings, and analyses need **auditable evidence** that their outputs are:

1. **Deterministic** — reproducible from specs + seeds
2. **Safe** — no PHI leakage, no unsupported clinical claims
3. **Clinically Sound** — physiologically plausible, temporally coherent
4. **Benchmark-aligned** — matches reference implementations on known datasets

## The Certificate Model

Every artifact produced by MedTWIN's agentic system comes with a **certificate** — a JSON document proving:

```
┌──────────────────────────────────────────────────────────────────────┐
│                           CERTIFICATE                                 │
├──────────────────────────────────────────────────────────────────────┤
│ artifact: {type, version, hash}                                       │
│ rubrics:                                                              │
│   tier_1: {pass: bool, checks: [...]}  ← Constitution                │
│   tier_2: {pass: bool, checks: [...]}  ← Clinical invariants         │
│   tier_3: {pass: bool, checks: [...]}  ← Task benchmarks             │
│ gate_decision: {decision, blocking_reasons, required_fixes}          │
│ provenance: {audit_trace_id, run_manifest_id, rubric_versions}       │
└──────────────────────────────────────────────────────────────────────┘
```

## Hierarchical Rubrics

### Tier 1: Constitution (Non-negotiable)

These are **universal principles** that apply to ALL outputs:

- **Determinism** — Same spec + seed → same output
- **Audit completeness** — Every decision has a trace
- **PHI protection** — No identifiable information in artifacts
- **Outcome claim discipline** — No clinical claims without validation

Tier 1 failures → **BLOCK** (human review required)

### Tier 2: Clinical Invariants (Domain-transferable)

These encode **clinical domain knowledge**:

- **Unit consistency** — All features have declared units
- **Plausible ranges** — Age ∈ [0, 120], HR ∈ [20, 300], etc.
- **Temporal coherence** — No future-leakage in retrospective studies
- **Outcome leakage prevention** — Labels don't leak via features

Tier 2 failures → **BLOCK** (critical) or **REVISE** (fixable)

### Tier 3: Task Benchmarks (Dataset-specific)

These are **empirical checks** against known-good references:

- **SQL executes** — Cohort specs run without errors
- **Cohort overlap** — Jaccard similarity ≥ threshold with reference
- **Metric agreement** — Statistical results within tolerance

Tier 3 failures → **REVISE** (agent can retry)

## Gate Decisions

| Decision | Meaning | Action |
|----------|---------|--------|
| `approve` | All tiers passed | Ship the artifact |
| `revise` | Tier 3 issues | Auto-retry or manual fix |
| `block` | Tier 1/2 violations | Human review required |

## Public vs. Private Components

| Component | Visibility | Purpose |
|-----------|------------|---------|
| **Rubric definitions** (YAML) | Public | Anyone can read what's being checked |
| **Certificate schema** | Public | Anyone can verify certificate structure |
| **Verifier CLI** | Public | Anyone can validate certificates locally |
| **Evaluation API** | Private | MedTWIN runs the evaluation logic |
| **Agentic generation** | Private | How MedTWIN agents produce artifacts |
| **Metacognition/refinement** | Private | How agents self-correct based on rubric failures |

## Integration Paths

### 1. Verify Certificates Locally

```bash
pip install git+https://github.com/Medtwin-ai/rubric-gates.git
rubric-gates verify certificate.json
```

### 2. Call the Evaluation API

```bash
curl -X POST https://api.medtwin.ai/v1/evaluate \
  -H "Authorization: Bearer $MEDTWIN_API_KEY" \
  -d '{"artifact": {...}, "context": {...}}'
```

### 3. Integrate in CI/CD

```yaml
# GitHub Actions example
- name: Verify artifact certificate
  run: |
    rubric-gates verify ./artifacts/cohort_certificate.json
```

## Learn More

- [API Reference](../api/rubric-gates.md)
- [Public Repo: rubric-gates](https://github.com/Medtwin-ai/rubric-gates)
- [Research Paper Blueprint](https://research.medtwin.ai/research/)
