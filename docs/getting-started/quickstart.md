<!--
PATH: docs/getting-started/quickstart.md
PURPOSE: Quickstart guide for calling MedTWIN APIs.
-->

## Quickstart

### 1) Get credentials
You will receive an API key from MedTWIN.

### 2) Call the health endpoint
```bash
curl -s https://api.medtwin.ai/health
```

### 3) Evaluate an artifact (Rubric Gates)
```bash
curl -s -X POST https://api.medtwin.ai/v1/evaluate \
  -H "Authorization: Bearer $MEDTWIN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "artifact": {
      "artifact_type": "cohort_spec",
      "artifact_version": "0.1.0",
      "artifact_hash": "..."
    }
  }'
```

You will receive a **decision** and a **certificate**.

