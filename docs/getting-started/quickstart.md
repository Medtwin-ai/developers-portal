# Quickstart Guide

Get up and running with the MedTWIN Rubric Gates API in 5 minutes.

## Prerequisites

- An API key from MedTWIN
- `curl` or any HTTP client
- (Optional) Python 3.11+ for SDK usage

## Get Credentials

### Request Access

Contact MedTWIN to request API access:

1. Email **api-access@medtwin.ai** with:
    - Your organization name
    - Intended use case
    - Expected volume (requests/month)

2. You'll receive:
    - An API key (`MEDTWIN_API_KEY`)
    - Rate limit information
    - Onboarding documentation

### Set Up Your Environment

```bash
# Store your API key securely
export MEDTWIN_API_KEY="your_api_key_here"
```

!!! danger "Security"
    Never commit API keys to version control. Use environment variables or a secrets manager.

## First API Call

### Health Check

Verify connectivity:

```bash
curl https://api.medtwin.ai/health
```

Expected response:
```json
{"status": "ok"}
```

### List Available Rubrics

See what rubric checks are available:

```bash
curl -H "Authorization: Bearer $MEDTWIN_API_KEY" \
  https://api.medtwin.ai/v1/rubrics
```

Response:
```json
{
  "tiers": {
    "tier1": [
      {
        "id": "tier1.constitution",
        "version": "1.0.0",
        "check_count": 4
      }
    ],
    "tier2": [...],
    "tier3": [...]
  },
  "total_suites": 3
}
```

## Evaluate an Artifact

### Basic Evaluation

Send an artifact for evaluation:

```bash
curl -X POST https://api.medtwin.ai/v1/evaluate \
  -H "Authorization: Bearer $MEDTWIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "artifact": {
      "artifact_type": "cohort_spec",
      "artifact_version": "1.0.0",
      "artifact_hash": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    }
  }'
```

### With Full Context

For best results, include context:

```bash
curl -X POST https://api.medtwin.ai/v1/evaluate \
  -H "Authorization: Bearer $MEDTWIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "artifact": {
      "artifact_type": "cohort_spec",
      "artifact_version": "1.0.0",
      "artifact_hash": "sha256:e3b0c44...",
      "deterministic_executor": "duckdb+sql",
      "inputs_summary": "De-identified MIMIC-IV data"
    },
    "context": {
      "provenance": {
        "audit_trace_id": "trace_001",
        "run_manifest_id": "manifest_001"
      },
      "features": {
        "age": {"unit": "years"},
        "weight": {"unit": "kg"}
      },
      "index_time": "2024-01-01T00:00:00Z",
      "sql_executed": true,
      "cohort_jaccard": 0.85
    }
  }'
```

### Understanding the Response

```json
{
  "decision": "approve",
  "certificate": {
    "certificate_id": "550e8400-e29b-41d4-a716-446655440000",
    "created_at": "2026-02-01T12:00:00Z",
    "artifact": {
      "type": "cohort_spec",
      "version": "1.0.0",
      "hash": "sha256:e3b0c44..."
    },
    "rubrics": {
      "tier_1": {
        "pass": true,
        "checks": [
          {"id": "tier1.determinism_required", "pass": true},
          {"id": "tier1.audit_trace_complete", "pass": true},
          {"id": "tier1.no_phi_in_artifacts", "pass": true},
          {"id": "tier1.no_outcome_claims", "pass": true}
        ]
      },
      "tier_2": {"pass": true, "checks": [...]},
      "tier_3": {"pass": true, "checks": [...]}
    },
    "gate_decision": {
      "decision": "approve",
      "blocking_reasons": [],
      "required_fixes": [],
      "deferral": {"recommended": false}
    },
    "provenance": {
      "audit_trace_id": "trace_001",
      "rubric_versions": {"tier1": "1.0.0", "tier2": "1.0.0", "tier3": "1.0.0"}
    }
  },
  "request_id": "req_abc123"
}
```

## Gate Decisions

| Decision | Meaning | Action |
|----------|---------|--------|
| `approve` | All tiers passed | Use the artifact |
| `revise` | Tier 3 failures | Fix issues and resubmit |
| `block` | Tier 1 or 2 failures | Requires human review |

## Verify Certificates Locally

Install the CLI:

```bash
pip install git+https://github.com/Medtwin-ai/rubric-gates.git
```

Save and verify a certificate:

```bash
# Save the certificate from the API response
echo '$CERTIFICATE_JSON' > certificate.json

# Verify locally (no API call)
rubric-gates verify certificate.json
```

Output:
```
✅ Certificate is valid
```

## Next Steps

- **[Certificate Schema](../rubric-gates/certificates.md)** — Detailed certificate structure
- **[Rubric Tiers](../rubric-gates/index.md)** — What each tier checks
- **[Integration Guide](integration.md)** — CI/CD, webhooks, SDKs
- **[API Reference](../api/rubric-gates.md)** — Full endpoint documentation

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Unauthorized` | Invalid or missing API key | Check `MEDTWIN_API_KEY` |
| `400 Bad Request` | Malformed request body | Validate JSON structure |
| `429 Too Many Requests` | Rate limit exceeded | Implement backoff |
| `500 Internal Error` | Server issue | Retry with exponential backoff |

### Getting Help

- Check the [API Reference](../api/rubric-gates.md)
- Open an issue on [GitHub](https://github.com/Medtwin-ai/developers-portal/issues)
- Email support@medtwin.ai
