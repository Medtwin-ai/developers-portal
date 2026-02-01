# MedTWIN Developer Portal

Welcome to the MedTWIN developer documentation. Build trustworthy clinical AI applications with verifiable outputs.

## What is MedTWIN?

MedTWIN provides **Rubric Gates**—a certificate-first validation framework for AI-generated clinical research artifacts. Every output ships with machine-checkable proof that it passed rigorous quality gates.

<div class="grid cards" markdown>

-   :material-shield-check:{ .lg .middle } **Verifiable Outputs**

    ---

    Every artifact comes with a certificate proving it passed hierarchical rubric checks.

    [:octicons-arrow-right-24: Learn about certificates](#certificates)

-   :material-layers-triple:{ .lg .middle } **Hierarchical Rubrics**

    ---

    Three-tier validation: constitutional → clinical → task-specific benchmarks.

    [:octicons-arrow-right-24: View rubric tiers](rubric-gates/index.md)

-   :material-api:{ .lg .middle } **Simple API**

    ---

    One endpoint to evaluate artifacts and receive decisions with full provenance.

    [:octicons-arrow-right-24: API Reference](api/rubric-gates.md)

-   :material-open-source-initiative:{ .lg .middle } **Open Verification**

    ---

    Verify certificates locally with our open-source CLI—no API calls needed.

    [:octicons-arrow-right-24: Local verification](rubric-gates/verification.md)

</div>

## Quick Example

```bash
# Evaluate an artifact
curl -X POST https://api.medtwin.ai/v1/evaluate \
  -H "Authorization: Bearer $MEDTWIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "artifact": {
      "artifact_type": "cohort_spec",
      "artifact_version": "1.0.0",
      "artifact_hash": "sha256:abc123..."
    }
  }'
```

Response:
```json
{
  "decision": "approve",
  "certificate": {
    "certificate_id": "cert_abc123",
    "rubrics": {
      "tier_1": {"pass": true},
      "tier_2": {"pass": true},
      "tier_3": {"pass": true}
    },
    "gate_decision": {"decision": "approve"}
  }
}
```

## Certificates

A **certificate** is a JSON document that proves an artifact passed rubric gates:

```
┌────────────────────────────────────────────────────────────────┐
│                        CERTIFICATE                              │
├────────────────────────────────────────────────────────────────┤
│  certificate_id: "cert_abc123"                                  │
│  created_at: "2026-02-01T00:00:00Z"                            │
│                                                                 │
│  artifact:                                                      │
│    type: "cohort_spec"                                         │
│    hash: "sha256:abc123..."                                    │
│                                                                 │
│  rubrics:                                                       │
│    tier_1: ✓ Constitution (determinism, audit, PHI)            │
│    tier_2: ✓ Clinical (units, ranges, temporal)                │
│    tier_3: ✓ Benchmark (SQL executes, Jaccard ≥ 0.7)          │
│                                                                 │
│  gate_decision: APPROVE                                         │
│                                                                 │
│  provenance:                                                    │
│    audit_trace_id: "trace_xyz"                                 │
│    rubric_versions: {tier1: "1.0.0", ...}                      │
└────────────────────────────────────────────────────────────────┘
```

Certificates can be **independently verified** using our open-source CLI:

```bash
pip install rubric-gates
rubric-gates verify certificate.json
# ✅ Certificate is valid
```

## What We Do NOT Claim

!!! warning "Important Disclaimers"

    - This API is **not medical advice** and does not provide patient-level treatment recommendations
    - We do **not claim clinical outcome improvements** (mortality, LOS, treatment benefit) without prospective validation
    - Rubric Gates validates **process correctness**, not clinical efficacy

## Getting Started

1. **[Get API credentials](getting-started/quickstart.md#get-credentials)** — Request access from MedTWIN
2. **[Make your first call](getting-started/quickstart.md#first-api-call)** — Evaluate an artifact
3. **[Understand certificates](rubric-gates/certificates.md)** — Learn the certificate schema
4. **[Integrate in your workflow](getting-started/integration.md)** — CI/CD, SDKs, webhooks

## Support

- **Documentation issues**: [GitHub Issues](https://github.com/Medtwin-ai/developers-portal/issues)
- **API support**: support@medtwin.ai
- **Enterprise inquiries**: enterprise@medtwin.ai
