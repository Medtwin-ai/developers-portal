# Certificate Schema

Certificates are the core unit of trust in Rubric Gates. This page documents the complete certificate schema.

## Overview

A **certificate** is a JSON document that proves an artifact passed rubric gates. Certificates are:

- **Immutable**: Once created, certificates cannot be modified
- **Verifiable**: Anyone can verify a certificate using the open-source CLI
- **Versioned**: Certificates include the rubric versions used for evaluation
- **Traceable**: Full provenance chain from artifact to decision

## Schema Structure

```json
{
  "$schema": "https://github.com/Medtwin-ai/rubric-gates/schemas/certificate.schema.json",
  "certificate_id": "string (UUID)",
  "created_at": "string (ISO 8601 datetime)",
  "artifact": { ... },
  "rubrics": { ... },
  "gate_decision": { ... },
  "provenance": { ... }
}
```

## Field Reference

### `certificate_id`

Unique identifier for the certificate.

| Property | Value |
|----------|-------|
| Type | `string` |
| Format | UUID v4 |
| Example | `"550e8400-e29b-41d4-a716-446655440000"` |

### `created_at`

Timestamp when the certificate was created.

| Property | Value |
|----------|-------|
| Type | `string` |
| Format | ISO 8601 datetime with timezone |
| Example | `"2026-02-01T12:00:00+00:00"` |

### `artifact`

Metadata about the evaluated artifact.

```json
{
  "artifact": {
    "type": "cohort_spec",
    "version": "1.0.0",
    "hash": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "deterministic_executor": "duckdb+sql",
    "inputs_summary": "De-identified clinical data"
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | Yes | Artifact type (`cohort_spec`, `mapping_spec`, `analysis_spec`) |
| `version` | string | Yes | Semantic version |
| `hash` | string | Yes | SHA-256 hash of artifact content |
| `deterministic_executor` | string | No | Executor used (e.g., `duckdb+sql`, `python`) |
| `inputs_summary` | string | No | Human-readable summary (no PHI) |

### `rubrics`

Results from each rubric tier.

```json
{
  "rubrics": {
    "tier_1": {
      "pass": true,
      "checks": [
        {
          "id": "tier1.determinism_required",
          "pass": true
        },
        {
          "id": "tier1.audit_trace_complete",
          "pass": true
        }
      ]
    },
    "tier_2": {
      "pass": true,
      "checks": [
        {
          "id": "tier2.unit_consistency",
          "pass": true,
          "score": 1.0,
          "threshold": 1.0
        }
      ]
    },
    "tier_3": {
      "pass": true,
      "checks": [
        {
          "id": "tier3.cohort_overlap_jaccard",
          "pass": true,
          "score": 0.85,
          "threshold": 0.7,
          "message": "Jaccard 0.85 ≥ 0.7"
        }
      ]
    }
  }
}
```

#### Check Result Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Check identifier |
| `pass` | boolean | Yes | Whether the check passed |
| `score` | number | No | Numeric score (for scored checks) |
| `threshold` | number | No | Required threshold |
| `message` | string | No | Human-readable message |

### `gate_decision`

The final gate decision based on all tier results.

```json
{
  "gate_decision": {
    "decision": "approve",
    "blocking_reasons": [],
    "required_fixes": [],
    "deferral": {
      "recommended": false,
      "to": "human_review"
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `decision` | string | `"approve"`, `"revise"`, or `"block"` |
| `blocking_reasons` | array | List of reasons for non-approval |
| `required_fixes` | array | Specific fixes needed |
| `deferral.recommended` | boolean | Whether human review is recommended |
| `deferral.to` | string | Deferral target (e.g., `"human_review"`) |

### `provenance`

Audit and reproducibility information.

```json
{
  "provenance": {
    "audit_trace_id": "trace_001",
    "run_manifest_id": "manifest_001",
    "rubric_versions": {
      "tier1": "1.0.0",
      "tier2": "1.0.0",
      "tier3": "1.0.0"
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `audit_trace_id` | string | Unique trace ID for audit |
| `run_manifest_id` | string | ID of the run manifest |
| `rubric_versions` | object | Versions of rubrics used |

## Complete Example

```json
{
  "$schema": "https://github.com/Medtwin-ai/rubric-gates/schemas/certificate.schema.json",
  "certificate_id": "550e8400-e29b-41d4-a716-446655440000",
  "created_at": "2026-02-01T12:00:00+00:00",
  "artifact": {
    "type": "cohort_spec",
    "version": "1.0.0",
    "hash": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "deterministic_executor": "duckdb+sql",
    "inputs_summary": "De-identified MIMIC-IV data for sepsis cohort"
  },
  "rubrics": {
    "tier_1": {
      "pass": true,
      "checks": [
        {"id": "tier1.determinism_required", "pass": true},
        {"id": "tier1.audit_trace_complete", "pass": true},
        {"id": "tier1.no_phi_in_artifacts", "pass": true},
        {"id": "tier1.no_outcome_claims_without_validation", "pass": true, "message": "Policy check - requires human review"}
      ]
    },
    "tier_2": {
      "pass": true,
      "checks": [
        {"id": "tier2.unit_consistency", "pass": true, "score": 1.0, "threshold": 1.0},
        {"id": "tier2.plausible_ranges", "pass": true, "score": 1.0, "threshold": 0.95},
        {"id": "tier2.temporal_coherence", "pass": true},
        {"id": "tier2.outcome_leakage_prevention", "pass": true}
      ]
    },
    "tier_3": {
      "pass": true,
      "checks": [
        {"id": "tier3.sql_executes", "pass": true},
        {"id": "tier3.cohort_overlap_jaccard", "pass": true, "score": 0.85, "threshold": 0.7, "message": "Jaccard 0.85 ≥ 0.7"}
      ]
    }
  },
  "gate_decision": {
    "decision": "approve",
    "blocking_reasons": [],
    "required_fixes": [],
    "deferral": {
      "recommended": false,
      "to": "human_review"
    }
  },
  "provenance": {
    "audit_trace_id": "trace_sepsis_cohort_001",
    "run_manifest_id": "manifest_2026_02_01",
    "rubric_versions": {
      "tier1": "1.0.0",
      "tier2": "1.0.0",
      "tier3": "1.0.0"
    }
  }
}
```

## Verification

Certificates can be verified without an API call:

```bash
# Install the CLI
pip install git+https://github.com/Medtwin-ai/rubric-gates.git

# Verify schema compliance
rubric-gates verify certificate.json
```

The verifier checks:

1. **Schema compliance**: JSON structure matches the certificate schema
2. **Hash integrity**: Artifact hash is well-formed
3. **Tier consistency**: Tier pass/fail matches individual check results
4. **Decision consistency**: Gate decision matches tier results

## JSON Schema

The full JSON Schema is available at:

[`https://github.com/Medtwin-ai/rubric-gates/schemas/certificate.schema.json`](https://github.com/Medtwin-ai/rubric-gates/blob/main/schemas/certificate.schema.json)

## Next Steps

- [Rubric Tiers](index.md) — What each tier checks
- [Local Verification](verification.md) — Verify certificates offline
- [API Reference](../api/rubric-gates.md) — Endpoint documentation
