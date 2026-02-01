<!--
PATH: docs/api/rubric-gates.md
PURPOSE: Complete API reference for Rubric Gates evaluation service.
-->

# Rubric Gates API (v1)

The Rubric Gates API evaluates artifacts against hierarchical rubrics and returns verifiable certificates.

## Base URL

```
https://api.medtwin.ai/v1
```

## Authentication

All endpoints require Bearer token authentication.

```bash
Authorization: Bearer $MEDTWIN_API_KEY
```

---

## Endpoints

### `GET /health`

Health check endpoint.

**Response**

```json
{
  "status": "ok"
}
```

---

### `GET /v1/rubrics`

List all available rubric suites.

**Response**

```json
{
  "tiers": {
    "tier1": [
      {
        "id": "tier1.constitution",
        "tier": 1,
        "version": "1.0.0",
        "purpose": "Prevent non-auditable, non-reproducible, unsafe outputs.",
        "check_count": 4
      }
    ],
    "tier2": [...],
    "tier3": [...]
  },
  "total_suites": 3
}
```

---

### `POST /v1/evaluate`

Evaluate an artifact against all rubric tiers.

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `artifact` | object | Yes | Artifact metadata |
| `artifact.artifact_type` | string | Yes | Type: `cohort_spec`, `mapping_spec`, `analysis_spec` |
| `artifact.artifact_version` | string | Yes | Semantic version |
| `artifact.artifact_hash` | string | Yes | SHA-256 hash |
| `artifact.deterministic_executor` | string | No | Executor used (e.g., `duckdb+sql`) |
| `artifact.inputs_summary` | string | No | Summary (no PHI) |
| `context` | object | No | Evaluation context |
| `context.provenance` | object | No | Audit/manifest IDs |
| `context.features` | object | No | Feature definitions with units |
| `context.index_time` | string | No | ISO timestamp for temporal checks |
| `context.sql_executed` | boolean | No | SQL execution result |
| `context.cohort_jaccard` | number | No | Jaccard overlap (0.0-1.0) |

**Example Request**

```bash
curl -X POST https://api.medtwin.ai/v1/evaluate \
  -H "Authorization: Bearer $MEDTWIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "artifact": {
      "artifact_type": "cohort_spec",
      "artifact_version": "1.0.0",
      "artifact_hash": "abc123...",
      "deterministic_executor": "duckdb+sql"
    },
    "context": {
      "provenance": {
        "audit_trace_id": "trace_001",
        "run_manifest_id": "manifest_001"
      },
      "index_time": "2024-01-01T00:00:00Z",
      "sql_executed": true,
      "cohort_jaccard": 0.85
    }
  }'
```

**Response**

```json
{
  "decision": "approve",
  "certificate": {
    "certificate_id": "550e8400-e29b-41d4-a716-446655440000",
    "created_at": "2026-02-01T00:00:00Z",
    "artifact": {
      "type": "cohort_spec",
      "version": "1.0.0",
      "hash": "abc123...",
      "deterministic_executor": "duckdb+sql"
    },
    "rubrics": {
      "tier_1": { "pass": true, "checks": [...] },
      "tier_2": { "pass": true, "checks": [...] },
      "tier_3": { "pass": true, "checks": [...] }
    },
    "gate_decision": {
      "decision": "approve",
      "blocking_reasons": [],
      "required_fixes": [],
      "deferral": { "recommended": false, "to": "human_review" }
    },
    "provenance": {
      "audit_trace_id": "trace_001",
      "run_manifest_id": "manifest_001",
      "rubric_versions": { "tier1": "1.0.0", "tier2": "1.0.0", "tier3": "1.0.0" }
    }
  },
  "request_id": "req_abc123"
}
```

**Decision Values**

| Decision | Meaning |
|----------|---------|
| `approve` | All checks passed; artifact is ready |
| `revise` | Tier 3 issues; can be fixed and re-submitted |
| `block` | Tier 1 or 2 violations; requires human review |

---

### `POST /v1/verify`

Verify a certificate's schema validity.

**Request Body**

```json
{
  "certificate": { ... }
}
```

**Response**

```json
{
  "is_valid": true,
  "errors": []
}
```

---

## Rubric Tiers

### Tier 1: Constitution (Non-negotiable)

| Check ID | Description |
|----------|-------------|
| `tier1.determinism_required` | Outputs must be reproducible from specs + seeds |
| `tier1.audit_trace_complete` | All decisions must have an audit trace |
| `tier1.no_phi_in_artifacts` | No PHI in certificates or reasoning logs |
| `tier1.no_outcome_claims_without_validation` | No clinical claims without prospective validation |

### Tier 2: Clinical Invariants (Transferable)

| Check ID | Description |
|----------|-------------|
| `tier2.unit_consistency` | All numeric features must have declared units |
| `tier2.plausible_ranges` | Values must fall within physiological bounds |
| `tier2.temporal_coherence` | No post-outcome information in pre-index features |
| `tier2.outcome_leakage_prevention` | No label leakage via features |

### Tier 3: Task Benchmarks (Dataset-specific)

| Check ID | Description |
|----------|-------------|
| `tier3.sql_executes` | SQL/spec must execute without errors |
| `tier3.cohort_overlap_jaccard` | Overlap with reference ≥ threshold (default 0.7) |

---

## Certificate Verification (Local)

Use the public `rubric-gates` CLI to verify certificates locally:

```bash
pip install git+https://github.com/Medtwin-ai/rubric-gates.git
rubric-gates verify certificate.json
```

---

## Error Codes

| Status | Meaning |
|--------|---------|
| 200 | Success |
| 400 | Bad request (invalid payload) |
| 401 | Unauthorized (invalid/missing API key) |
| 500 | Internal server error |

---

## Rate Limits

Contact MedTWIN for rate limit information based on your plan.
