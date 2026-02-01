# Rubric Gates Overview

Rubric Gates is MedTWIN's certificate-first validation framework for AI-generated clinical research artifacts.

## The Problem

AI agents generating clinical artifacts (cohort definitions, statistical analyses, manuscript drafts) face a fundamental trust problem:

- **No verifiable evidence** that outputs are correct
- **No audit trail** for how decisions were made
- **No standardized checks** for clinical validity
- **No reproducibility** guarantees

## The Solution

Rubric Gates provides **certificates**—machine-checkable proof that every artifact passed a hierarchy of quality gates.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         RUBRIC GATES PIPELINE                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────────┐  │
│   │ Artifact │ ──▶ │  Tier 1  │ ──▶ │  Tier 2  │ ──▶ │    Tier 3    │  │
│   │  Input   │     │Constitution    │ Clinical │     │  Benchmarks  │  │
│   └──────────┘     └──────────┘     └──────────┘     └──────────────┘  │
│                           │               │                │            │
│                           ▼               ▼                ▼            │
│                    ┌─────────────────────────────────────────────┐     │
│                    │              GATE DECISION                   │     │
│                    │         approve / revise / block            │     │
│                    └─────────────────────────────────────────────┘     │
│                                        │                                │
│                                        ▼                                │
│                              ┌──────────────────┐                      │
│                              │   CERTIFICATE    │                      │
│                              │  (verifiable)    │                      │
│                              └──────────────────┘                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## Hierarchical Rubrics

Rubrics are organized into three tiers with increasing specificity:

### Tier 1: Constitution (Universal)

Non-negotiable requirements that apply to **all** outputs:

| Check | Description | Gate |
|-------|-------------|------|
| `determinism_required` | Same spec + seed → same output | Block |
| `audit_trace_complete` | Every decision has a trace ID | Block |
| `no_phi_in_artifacts` | No identifiable information in certificates | Block |
| `no_outcome_claims` | No clinical claims without validation | Block |

!!! danger "Tier 1 Failures"
    Tier 1 failures result in **BLOCK** — the artifact cannot be used and requires human review.

### Tier 2: Clinical Invariants (Domain)

Domain knowledge that transfers across datasets:

| Check | Description | Gate |
|-------|-------------|------|
| `unit_consistency` | All features have declared units | Block |
| `plausible_ranges` | Values fall within physiological bounds | Revise |
| `temporal_coherence` | No future-leakage in retrospective studies | Block |
| `outcome_leakage` | Labels don't leak via features | Block |

!!! warning "Tier 2 Failures"
    Critical Tier 2 failures result in **BLOCK**. Minor failures may result in **REVISE**.

### Tier 3: Task Benchmarks (Dataset-Specific)

Empirical checks against known-good references:

| Check | Description | Gate |
|-------|-------------|------|
| `sql_executes` | Cohort specs run without errors | Revise |
| `cohort_overlap` | Jaccard similarity ≥ 0.7 with reference | Revise |

!!! info "Tier 3 Failures"
    Tier 3 failures result in **REVISE** — the artifact can be fixed and resubmitted.

## Gate Decisions

| Decision | Meaning | Action |
|----------|---------|--------|
| **Approve** | All tiers passed | Use the artifact, store the certificate |
| **Revise** | Tier 3 failures | Fix issues based on `required_fixes`, resubmit |
| **Block** | Tier 1 or 2 failures | Escalate to human review, investigate root cause |

### Decision Flow

```
                    ┌─────────────┐
                    │  Evaluate   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
              ┌─────│  Tier 1 OK? │─────┐
              │ No  └─────────────┘ Yes │
              │                         │
              ▼                         ▼
        ┌───────────┐           ┌──────────────┐
        │   BLOCK   │     ┌─────│  Tier 2 OK?  │─────┐
        │ (human)   │     │ No  └──────────────┘ Yes │
        └───────────┘     │                          │
                          ▼                          ▼
                    ┌───────────┐          ┌──────────────┐
                    │   BLOCK   │    ┌─────│  Tier 3 OK?  │─────┐
                    │ (human)   │    │ No  └──────────────┘ Yes │
                    └───────────┘    │                          │
                                     ▼                          ▼
                               ┌───────────┐            ┌───────────┐
                               │  REVISE   │            │  APPROVE  │
                               │ (auto)    │            │ (ship it) │
                               └───────────┘            └───────────┘
```

## Deferral to Humans

When the system is uncertain or encounters blocking issues, it defers to human review:

```json
{
  "gate_decision": {
    "decision": "block",
    "blocking_reasons": ["Tier 2 violation: outcome leakage detected"],
    "required_fixes": ["Remove feature X from pre-index time window"],
    "deferral": {
      "recommended": true,
      "to": "human_review"
    }
  }
}
```

Deferral is recommended when:

- Critical safety checks fail
- Uncertainty is high
- Novel edge cases are detected
- Audit trail is incomplete

## Rubric Versioning

Rubrics are versioned for reproducibility:

```json
{
  "provenance": {
    "rubric_versions": {
      "tier1": "1.0.0",
      "tier2": "1.0.0",
      "tier3": "1.0.0"
    }
  }
}
```

Version changes follow semantic versioning:

- **MAJOR**: Breaking changes to check logic
- **MINOR**: New checks added
- **PATCH**: Bug fixes, threshold adjustments

## Adding Custom Rubrics

Organizations can extend rubrics for their specific needs:

```yaml
# custom_rubrics/tier3/my_org_checks.yaml
rubric_suite:
  id: tier3.my_org_custom
  tier: 3
  version: "1.0.0"
  purpose: "Organization-specific validation checks"
  checks:
    - id: tier3.internal_review_approved
      description: "Internal review process completed"
      check_type: policy
      severity: major
      gate: revise
```

## Open Source Components

| Component | Repository | Purpose |
|-----------|------------|---------|
| Rubric definitions | [rubric-gates](https://github.com/Medtwin-ai/rubric-gates) | YAML rubric specs |
| Certificate schema | [rubric-gates](https://github.com/Medtwin-ai/rubric-gates) | JSON schema |
| Verifier CLI | [rubric-gates](https://github.com/Medtwin-ai/rubric-gates) | Local verification |
| Benchmark harness | [rubric-gates](https://github.com/Medtwin-ai/rubric-gates) | Run evaluations |

## Next Steps

- [Certificate Schema](certificates.md) — Detailed certificate structure
- [Local Verification](verification.md) — Verify certificates offline
- [API Reference](../api/rubric-gates.md) — Evaluate via API
- [Integration Guide](../getting-started/integration.md) — CI/CD patterns
