<!--
PATH: README.md
PURPOSE: Public developer portal for MedTWIN APIs (developers.medtwin.ai).

WHY:
- Separate product/API docs from the main application repo.
- Provide a clear, versioned “developer contract” for Rubric Gates as a service.

FLOW:
┌──────────────────────┐   ┌──────────────────────────┐   ┌──────────────────────────┐
│ Write docs in docs/  │──▶│ mkdocs build/serve        │──▶│ Deploy to developers.*    │
└──────────────────────┘   └──────────────────────────┘   └──────────────────────────┘

DEPENDENCIES:
- MkDocs Material (see requirements.txt)
-->

# MedTWIN Developers Portal

This repo powers `developers.medtwin.ai`.

## Local development
```bash
python -m venv .venv && source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
mkdocs serve
```

Open `http://localhost:8000`.

