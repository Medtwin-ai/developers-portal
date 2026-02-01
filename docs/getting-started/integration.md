# Integration Guide

Integrate Rubric Gates into your workflows, CI/CD pipelines, and applications.

## Integration Patterns

### 1. CI/CD Pipeline Integration

Add certificate verification to your deployment pipeline:

=== "GitHub Actions"

    ```yaml
    name: Validate Artifacts

    on:
      push:
        branches: [main]

    jobs:
      validate:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4

          - name: Install Rubric Gates CLI
            run: pip install git+https://github.com/Medtwin-ai/rubric-gates.git

          - name: Evaluate artifact
            env:
              MEDTWIN_API_KEY: ${{ secrets.MEDTWIN_API_KEY }}
            run: |
              # Create artifact metadata
              echo '{
                "artifact_type": "cohort_spec",
                "artifact_version": "1.0.0",
                "artifact_hash": "'$(sha256sum cohort.sql | cut -d' ' -f1)'"
              }' > artifact.json

              # Evaluate via API
              curl -X POST https://api.medtwin.ai/v1/evaluate \
                -H "Authorization: Bearer $MEDTWIN_API_KEY" \
                -H "Content-Type: application/json" \
                -d @artifact.json \
                -o certificate.json

              # Verify locally
              rubric-gates verify certificate.json

          - name: Store certificate
            uses: actions/upload-artifact@v4
            with:
              name: certificate
              path: certificate.json
    ```

=== "GitLab CI"

    ```yaml
    validate-artifacts:
      image: python:3.11
      stage: validate
      script:
        - pip install git+https://github.com/Medtwin-ai/rubric-gates.git
        - |
          curl -X POST https://api.medtwin.ai/v1/evaluate \
            -H "Authorization: Bearer $MEDTWIN_API_KEY" \
            -H "Content-Type: application/json" \
            -d @artifact.json \
            -o certificate.json
        - rubric-gates verify certificate.json
      artifacts:
        paths:
          - certificate.json
    ```

### 2. Python SDK

Use the `rubric-gates` package directly in Python:

```python
from rubric_gates import RubricEvaluator, create_certificate

# Local evaluation (no API call)
evaluator = RubricEvaluator()

artifact = {
    "type": "cohort_spec",
    "version": "1.0.0",
    "hash": "sha256:abc123",
    "deterministic_executor": "duckdb+sql",
}

context = {
    "provenance": {"audit_trace_id": "trace_001"},
    "index_time": "2024-01-01T00:00:00Z",
    "sql_executed": True,
    "cohort_jaccard": 0.85,
}

# Evaluate
result = evaluator.evaluate(artifact, context)

# Check decision
if result.gate_decision.decision == "approve":
    certificate = create_certificate(artifact, result, context["provenance"])
    print("✅ Artifact approved")
else:
    print(f"❌ Decision: {result.gate_decision.decision}")
    print(f"   Reasons: {result.gate_decision.blocking_reasons}")
```

### 3. Pre-commit Hook

Validate artifacts before committing:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: rubric-gates-check
        name: Rubric Gates Validation
        entry: bash -c 'rubric-gates evaluate artifact.json --context context.json'
        language: system
        files: '\.sql$'
        pass_filenames: false
```

### 4. REST API Integration

For custom integrations, use the REST API directly:

```python
import requests
import os

def evaluate_artifact(artifact: dict, context: dict = None) -> dict:
    """Evaluate an artifact via the MedTWIN API."""
    response = requests.post(
        "https://api.medtwin.ai/v1/evaluate",
        headers={
            "Authorization": f"Bearer {os.environ['MEDTWIN_API_KEY']}",
            "Content-Type": "application/json",
        },
        json={
            "artifact": artifact,
            "context": context or {},
        },
    )
    response.raise_for_status()
    return response.json()


# Example usage
result = evaluate_artifact(
    artifact={
        "artifact_type": "cohort_spec",
        "artifact_version": "1.0.0",
        "artifact_hash": "sha256:abc123",
    },
    context={
        "sql_executed": True,
        "cohort_jaccard": 0.85,
    },
)

if result["decision"] == "approve":
    # Store the certificate
    with open("certificate.json", "w") as f:
        import json
        json.dump(result["certificate"], f, indent=2)
```

## Storing Certificates

Certificates should be stored alongside the artifacts they validate:

```
project/
├── artifacts/
│   ├── cohort_v1.sql
│   └── cohort_v1.certificate.json   # Certificate for cohort_v1.sql
├── data/
│   └── results.parquet
└── manifests/
    └── run_manifest.json            # Links artifacts to certificates
```

### Certificate Naming Convention

```
<artifact_name>.<artifact_version>.certificate.json
```

Examples:
- `sepsis_cohort.v1.0.0.certificate.json`
- `mortality_analysis.v2.1.0.certificate.json`

## Error Handling

### Retry Logic

Implement exponential backoff for transient failures:

```python
import time
import requests
from typing import Optional

def evaluate_with_retry(
    artifact: dict,
    context: dict = None,
    max_retries: int = 3,
    base_delay: float = 1.0,
) -> Optional[dict]:
    """Evaluate with exponential backoff retry."""
    for attempt in range(max_retries):
        try:
            response = requests.post(
                "https://api.medtwin.ai/v1/evaluate",
                headers={
                    "Authorization": f"Bearer {os.environ['MEDTWIN_API_KEY']}",
                    "Content-Type": "application/json",
                },
                json={"artifact": artifact, "context": context or {}},
                timeout=30,
            )
            
            if response.status_code == 429:
                # Rate limited - wait and retry
                delay = base_delay * (2 ** attempt)
                time.sleep(delay)
                continue
                
            response.raise_for_status()
            return response.json()
            
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise
            delay = base_delay * (2 ** attempt)
            time.sleep(delay)
    
    return None
```

### Decision Handling

Handle all three gate decisions:

```python
def handle_evaluation_result(result: dict) -> bool:
    """Handle the evaluation result appropriately."""
    decision = result["decision"]
    certificate = result["certificate"]
    
    if decision == "approve":
        # Artifact is good to use
        save_certificate(certificate)
        return True
        
    elif decision == "revise":
        # Fixable issues - log and retry
        print("Revisions needed:")
        for fix in certificate["gate_decision"]["required_fixes"]:
            print(f"  - {fix}")
        return False
        
    else:  # block
        # Critical issues - escalate to human
        print("BLOCKED - Human review required:")
        for reason in certificate["gate_decision"]["blocking_reasons"]:
            print(f"  - {reason}")
        notify_human_reviewer(certificate)
        return False
```

## Monitoring & Observability

### Logging

Log all evaluations for audit:

```python
import logging
import json

logger = logging.getLogger("rubric_gates")

def log_evaluation(artifact: dict, result: dict):
    """Log evaluation for audit trail."""
    logger.info(
        "Rubric Gates evaluation",
        extra={
            "artifact_type": artifact.get("artifact_type"),
            "artifact_hash": artifact.get("artifact_hash"),
            "decision": result["decision"],
            "certificate_id": result["certificate"]["certificate_id"],
            "request_id": result["request_id"],
        },
    )
```

### Metrics

Track key metrics:

- **Evaluation latency**: Time to receive response
- **Pass rate**: % of artifacts approved
- **Failure distribution**: Which tiers/checks fail most
- **Certificate verification rate**: How often certificates are verified

## Security Best Practices

1. **API Key Management**
    - Use environment variables or secrets managers
    - Rotate keys periodically
    - Use separate keys for dev/staging/prod

2. **Certificate Storage**
    - Store certificates immutably (append-only)
    - Include certificates in backups
    - Use content-addressed storage when possible

3. **Network Security**
    - Always use HTTPS
    - Implement request signing for sensitive workflows
    - Validate TLS certificates

## Next Steps

- [API Reference](../api/rubric-gates.md) — Full endpoint documentation
- [Certificate Schema](../rubric-gates/certificates.md) — Certificate structure details
- [Troubleshooting](troubleshooting.md) — Common issues and solutions
