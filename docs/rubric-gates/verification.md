# Local Verification

Verify Rubric Gates certificates locally without API calls using the open-source CLI.

## Why Local Verification?

Local verification provides:

- **Independence**: Verify without MedTWIN API access
- **Speed**: No network latency
- **Privacy**: Certificate contents never leave your machine
- **Auditability**: Third parties can verify your certificates

## Installation

=== "pip"

    ```bash
    pip install git+https://github.com/Medtwin-ai/rubric-gates.git
    ```

=== "pipx (isolated)"

    ```bash
    pipx install git+https://github.com/Medtwin-ai/rubric-gates.git
    ```

=== "From source"

    ```bash
    git clone https://github.com/Medtwin-ai/rubric-gates.git
    cd rubric-gates
    pip install -e .
    ```

## Basic Verification

```bash
rubric-gates verify certificate.json
```

Output (valid):
```
✅ Certificate is valid
```

Output (invalid):
```
❌ Certificate is invalid
  - Missing required field: artifact.hash
  - Tier 1 pass status inconsistent with checks
```

## Verification with Artifact Hash

Verify that the certificate matches a specific artifact:

```bash
# Compute artifact hash
sha256sum cohort.sql
# e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855  cohort.sql

# Verify certificate matches artifact
rubric-gates verify certificate.json --artifact cohort.sql
```

Output:
```
✅ Certificate is valid
✅ Artifact hash matches
```

## What Gets Verified

The verifier checks:

### 1. Schema Compliance

- All required fields present
- Field types correct
- Values in allowed ranges

### 2. Internal Consistency

- `tier_*.pass` matches individual check results
- `gate_decision.decision` matches tier results
- Dates are valid ISO 8601

### 3. Cryptographic Integrity (with `--artifact`)

- Artifact hash matches certificate hash
- Hash algorithm is recognized (SHA-256)

## Programmatic Verification

Use the Python API for integration:

```python
from rubric_gates import verify_certificate, verify_certificate_file

# From file
result = verify_certificate_file("certificate.json")
print(f"Valid: {result.is_valid}")
print(f"Errors: {result.errors}")

# From dict
import json
with open("certificate.json") as f:
    cert = json.load(f)

result = verify_certificate(cert)
if not result.is_valid:
    for error in result.errors:
        print(f"Error: {error}")
```

### With Artifact Verification

```python
from rubric_gates import verify_certificate_file

result = verify_certificate_file(
    certificate_path="certificate.json",
    artifact_path="cohort.sql",
)

if result.is_valid:
    print("Certificate and artifact verified")
else:
    print("Verification failed:")
    for error in result.errors:
        print(f"  - {error}")
```

## Batch Verification

Verify multiple certificates:

```bash
# Using find and xargs
find ./certificates -name "*.json" | xargs -I {} rubric-gates verify {}

# Using a shell loop
for cert in certificates/*.json; do
    echo "Verifying $cert..."
    rubric-gates verify "$cert"
done
```

Python batch verification:

```python
from pathlib import Path
from rubric_gates import verify_certificate_file

def verify_all_certificates(cert_dir: str) -> dict:
    """Verify all certificates in a directory."""
    results = {"valid": [], "invalid": []}
    
    for cert_path in Path(cert_dir).glob("*.json"):
        result = verify_certificate_file(str(cert_path))
        if result.is_valid:
            results["valid"].append(str(cert_path))
        else:
            results["invalid"].append({
                "path": str(cert_path),
                "errors": result.errors,
            })
    
    return results

# Usage
results = verify_all_certificates("./certificates")
print(f"Valid: {len(results['valid'])}")
print(f"Invalid: {len(results['invalid'])}")
```

## CI/CD Integration

Add verification to your pipeline:

=== "GitHub Actions"

    ```yaml
    - name: Verify certificates
      run: |
        pip install git+https://github.com/Medtwin-ai/rubric-gates.git
        for cert in artifacts/*.certificate.json; do
          rubric-gates verify "$cert" || exit 1
        done
    ```

=== "GitLab CI"

    ```yaml
    verify:
      script:
        - pip install git+https://github.com/Medtwin-ai/rubric-gates.git
        - |
          for cert in artifacts/*.certificate.json; do
            rubric-gates verify "$cert" || exit 1
          done
    ```

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Missing required field: X` | Certificate incomplete | Re-generate certificate |
| `Invalid date format` | Malformed timestamp | Use ISO 8601 format |
| `Tier pass inconsistent` | Check results don't match tier.pass | Re-evaluate artifact |
| `Artifact hash mismatch` | Certificate is for different artifact | Verify correct file |

### Debug Mode

Get detailed output:

```bash
rubric-gates verify certificate.json --verbose
```

## Security Considerations

!!! warning "Verification Scope"
    
    Local verification confirms:
    
    - Certificate **structure** is valid
    - Certificate **internal logic** is consistent
    - Artifact **hash** matches (if provided)
    
    It does NOT confirm:
    
    - Certificate was issued by MedTWIN
    - Artifact actually passed rubric checks
    - Certificate hasn't been tampered with

For high-security applications, consider:

1. **Signature verification**: Future feature to verify MedTWIN signatures
2. **Certificate registry**: Check certificate ID against a trusted registry
3. **Provenance chain**: Verify the full audit trace

## Next Steps

- [Certificate Schema](certificates.md) — Full schema reference
- [Integration Guide](../getting-started/integration.md) — CI/CD patterns
- [API Reference](../api/rubric-gates.md) — Generate new certificates
