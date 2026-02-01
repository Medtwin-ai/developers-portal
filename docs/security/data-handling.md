# Data Handling & Security

MedTWIN's security practices for handling your data through the Rubric Gates API.

## Data Processing Model

### What We Receive

When you call the `/v1/evaluate` endpoint, we receive:

| Data | Stored | Duration | Purpose |
|------|--------|----------|---------|
| Artifact metadata | Yes | 90 days | Audit trail |
| Artifact hash | Yes | 90 days | Verification |
| Context (optional) | Yes | 90 days | Evaluation |
| Evaluation results | Yes | 90 days | Certificate |
| Request metadata | Yes | 90 days | Debugging |

### What We Do NOT Receive

!!! success "Never Transmitted"
    - Raw artifact content (SQL, code, data)
    - Patient-level data
    - PHI/PII
    - Actual datasets

You send us **metadata about** artifacts, not the artifacts themselves.

## PHI/PII Policy

### Strict Prohibition

**Do NOT include PHI or PII in API requests.**

The following are prohibited in all fields:

- Patient names, IDs, or identifiers
- Dates of birth
- Social Security Numbers
- Medical record numbers
- Device identifiers
- Geographic data (more specific than state)
- Biometric identifiers
- Full face photographs
- Email addresses, phone numbers

### Automated Scanning

We automatically scan for PHI patterns:

```json
{
  "artifact": {
    "inputs_summary": "Patient John Doe, MRN 12345..."
  }
}
```

Response:
```json
{
  "error": "phi_detected",
  "message": "Potential PHI detected in inputs_summary. Please remove identifiable information.",
  "field": "artifact.inputs_summary"
}
```

### Safe Alternatives

Instead of:
```json
{"inputs_summary": "Patient John Doe, age 45, MRN 12345"}
```

Use:
```json
{"inputs_summary": "De-identified ICU cohort, n=5000, age 18-90"}
```

## Data Retention

| Data Type | Retention | Deletion |
|-----------|-----------|----------|
| Certificates | 90 days | Auto-purge |
| Request logs | 90 days | Auto-purge |
| Error logs | 30 days | Auto-purge |
| Audit trails | 1 year | Manual request |

### Data Deletion Request

To request data deletion:

1. Email privacy@medtwin.ai
2. Include your organization ID
3. Specify the data to delete
4. Allow 30 days for processing

## Encryption

### In Transit

All API traffic uses TLS 1.3:

- Certificate: RSA 2048-bit minimum
- Cipher suites: TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256
- Perfect forward secrecy: Enabled

Verify with:
```bash
curl -vI https://api.medtwin.ai/health 2>&1 | grep -i "ssl\|tls"
```

### At Rest

Stored data is encrypted:

- Algorithm: AES-256-GCM
- Key management: AWS KMS
- Key rotation: Automatic, 90 days

## Infrastructure Security

### Cloud Provider

MedTWIN runs on AWS with:

- SOC 2 Type II certified
- HIPAA eligible services
- Data residency: US East (Virginia)

### Network Security

- WAF (Web Application Firewall)
- DDoS protection (AWS Shield)
- VPC isolation
- No public database access

### Access Control

- Role-based access (RBAC)
- MFA required for all personnel
- Quarterly access reviews
- Just-in-time access for production

## Compliance

### HIPAA

MedTWIN maintains HIPAA compliance for covered entities:

- Business Associate Agreement (BAA) available
- Access controls and audit logging
- Encryption at rest and in transit
- Incident response procedures

!!! note "BAA Required"
    If you are a covered entity, contact enterprise@medtwin.ai to execute a BAA before transmitting any data that could be linked to PHI.

### SOC 2

SOC 2 Type II report available upon request for:

- Security
- Availability
- Confidentiality

### GDPR

For EU data subjects:

- Data processing agreement available
- Right to access, rectify, delete
- Data portability supported
- Standard contractual clauses available

## Incident Response

### Reporting Security Issues

Report security vulnerabilities to:

**security@medtwin.ai**

Include:

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Your contact information

We follow responsible disclosure:

1. Acknowledge within 24 hours
2. Investigate within 72 hours
3. Provide timeline for fix
4. Coordinate public disclosure

### Breach Notification

In the event of a data breach affecting your data:

1. Notification within 72 hours
2. Details of affected data
3. Remediation steps taken
4. Contact for questions

## Security Best Practices

### For API Users

1. **Use environment variables** for API keys
2. **Implement rate limiting** on your end
3. **Validate responses** before using
4. **Log API calls** for audit trails
5. **Rotate keys** periodically

### For Certificate Storage

1. **Store immutably** (append-only)
2. **Include in backups**
3. **Version with artifacts**
4. **Verify periodically**

### For Integration

1. **Use HTTPS only**
2. **Verify TLS certificates**
3. **Implement timeouts**
4. **Handle errors gracefully**

## Contact

| Topic | Contact |
|-------|---------|
| Security issues | security@medtwin.ai |
| Privacy/GDPR | privacy@medtwin.ai |
| Compliance (BAA, SOC 2) | compliance@medtwin.ai |
| General support | support@medtwin.ai |
