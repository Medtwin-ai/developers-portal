# Authentication

All MedTWIN API endpoints (except `/health`) require authentication via API keys.

## API Keys

### Obtaining an API Key

Contact MedTWIN to request API access:

1. Email **api-access@medtwin.ai** with:
    - Organization name
    - Intended use case
    - Expected request volume

2. After approval, you'll receive:
    - API key (`MEDTWIN_API_KEY`)
    - Rate limit information
    - Support contact

### Key Format

API keys are prefixed strings:

```
mt_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx    # Production
mt_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx    # Testing/sandbox
```

!!! warning "Key Security"
    - Never commit API keys to version control
    - Use environment variables or secrets managers
    - Rotate keys if compromised
    - Use test keys for development

## Using Your API Key

### Bearer Token (Recommended)

Include your API key in the `Authorization` header:

```bash
curl https://api.medtwin.ai/v1/rubrics \
  -H "Authorization: Bearer $MEDTWIN_API_KEY"
```

### Python Example

```python
import os
import requests

response = requests.get(
    "https://api.medtwin.ai/v1/rubrics",
    headers={
        "Authorization": f"Bearer {os.environ['MEDTWIN_API_KEY']}"
    }
)
```

### JavaScript Example

```javascript
const response = await fetch('https://api.medtwin.ai/v1/rubrics', {
  headers: {
    'Authorization': `Bearer ${process.env.MEDTWIN_API_KEY}`
  }
});
```

## Rate Limits

| Plan | Requests/minute | Requests/day |
|------|-----------------|--------------|
| Free | 10 | 100 |
| Standard | 60 | 10,000 |
| Enterprise | Custom | Custom |

### Rate Limit Headers

Responses include rate limit information:

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1706799600
```

### Handling Rate Limits

When rate limited, you'll receive:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30

{
  "error": "rate_limit_exceeded",
  "message": "Too many requests. Retry after 30 seconds.",
  "retry_after": 30
}
```

Implement exponential backoff:

```python
import time
import requests

def call_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 30))
            time.sleep(retry_after)
            continue
            
        return response
    
    raise Exception("Max retries exceeded")
```

## Error Responses

### 401 Unauthorized

```json
{
  "error": "unauthorized",
  "message": "Invalid or missing API key"
}
```

Causes:

- Missing `Authorization` header
- Invalid API key
- Expired API key

### 403 Forbidden

```json
{
  "error": "forbidden",
  "message": "API key does not have permission for this resource"
}
```

Causes:

- Key restricted to certain endpoints
- Key restricted to certain artifact types

## Key Management

### Rotating Keys

1. Request a new key from MedTWIN
2. Update your applications to use the new key
3. Verify all systems are using the new key
4. Request revocation of the old key

### Multiple Keys

For different environments:

| Environment | Key Type | Purpose |
|-------------|----------|---------|
| Development | `mt_test_*` | Local testing |
| Staging | `mt_test_*` | Pre-production testing |
| Production | `mt_live_*` | Live traffic |

### Key Storage Best Practices

=== "Environment Variables"

    ```bash
    export MEDTWIN_API_KEY="mt_live_xxx"
    ```

=== "AWS Secrets Manager"

    ```python
    import boto3
    import json

    client = boto3.client('secretsmanager')
    secret = client.get_secret_value(SecretId='medtwin/api-key')
    api_key = json.loads(secret['SecretString'])['MEDTWIN_API_KEY']
    ```

=== "HashiCorp Vault"

    ```python
    import hvac

    client = hvac.Client(url='https://vault.example.com')
    secret = client.secrets.kv.read_secret_version(path='medtwin/api-key')
    api_key = secret['data']['data']['MEDTWIN_API_KEY']
    ```

=== "GitHub Secrets"

    ```yaml
    # .github/workflows/deploy.yml
    env:
      MEDTWIN_API_KEY: ${{ secrets.MEDTWIN_API_KEY }}
    ```

## Support

- **Lost key**: Email api-access@medtwin.ai
- **Compromised key**: Email security@medtwin.ai immediately
- **Rate limit increase**: Email enterprise@medtwin.ai
