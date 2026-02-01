<!--
PATH: docs/api/rubric-gates.md
PURPOSE: API reference for Rubric Gates evaluation.
-->

## Rubric Gates API (v1)

### `GET /health`
Returns service health.

### `GET /v1/rubrics`
Returns available rubric suites and versions.

### `POST /v1/evaluate`
Evaluate an artifact and return a decision + certificate.

#### Request (example)
```json
{
  "artifact": {
    "artifact_type": "cohort_spec",
    "artifact_version": "0.1.0",
    "artifact_hash": "sha256..."
  }
}
```

#### Response (example)
```json
{
  "decision": "revise",
  "certificate": {
    "certificate_id": "uuid",
    "created_at": "iso8601",
    "artifact": { "type": "cohort_spec", "version": "0.1.0", "hash": "sha256..." },
    "rubrics": { "tier_1": { "pass": true, "checks": [] }, "tier_2": { "pass": true, "checks": [] }, "tier_3": { "pass": false, "checks": [] } },
    "gate_decision": { "decision": "revise", "blocking_reasons": [], "required_fixes": [] },
    "provenance": { "audit_trace_id": "trace_id", "run_manifest_id": "manifest_id", "rubric_versions": {} }
  }
}
```

### Certificate verification
Use the public verifier from:
- `https://github.com/Medtwin-ai/rubric-gates`

