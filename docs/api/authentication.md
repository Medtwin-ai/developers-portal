<!--
PATH: docs/api/authentication.md
PURPOSE: Authentication model for MedTWIN APIs.
-->

## Authentication

MedTWIN APIs use **Bearer token authentication**.

### Request header
```bash
Authorization: Bearer $MEDTWIN_API_KEY
```

### Security note
Never commit API keys into repositories or client-side bundles.

