# MedTWIN Developers Portal

[![Deploy Docs](https://github.com/Medtwin-ai/developers-portal/actions/workflows/deploy.yml/badge.svg)](https://github.com/Medtwin-ai/developers-portal/actions/workflows/deploy.yml)

**Documentation for MedTWIN APIs and developer tools.**

This repository contains the source for [developers.medtwin.ai](https://developers.medtwin.ai).

## Local Development

### Prerequisites

- Python 3.11+

### Setup

```bash
# Clone the repository
git clone https://github.com/Medtwin-ai/developers-portal.git
cd developers-portal

# Install dependencies
pip install -r requirements.txt

# Start local server
mkdocs serve

# Open http://localhost:8000
```

### Build

```bash
mkdocs build --strict
```

The built site will be in the `site/` directory.

## Structure

```
developers-portal/
├── docs/
│   ├── index.md                    # Landing page
│   ├── getting-started/
│   │   └── quickstart.md           # Quickstart guide
│   ├── rubric-gates/
│   │   └── index.md                # Rubric Gates overview
│   ├── api/
│   │   ├── authentication.md       # Auth documentation
│   │   └── rubric-gates.md         # Rubric Gates API reference
│   └── security/
│       └── data-handling.md        # Data handling policies
├── mkdocs.yml                      # MkDocs configuration
├── requirements.txt                # Python dependencies
└── README.md
```

## Adding Documentation

1. Create a new `.md` file in the appropriate `docs/` subdirectory
2. Add the page to the `nav` section in `mkdocs.yml`
3. Run `mkdocs serve` to preview
4. Submit a pull request

## Deployment

The site is automatically deployed to GitHub Pages when changes are merged to `main`.

## Related Repositories

- [`rubric-gates`](https://github.com/Medtwin-ai/rubric-gates) - Public rubric package
- [`rubric-gates-service`](https://github.com/Medtwin-ai/rubric-gates-service) - API service (private)

## License

MIT License
