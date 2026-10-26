# Heliks Technical Documentation

This repository contains the technical documentation for the RICA (Resources and Inventory Control Application) mineral tracking ecosystem, built with [MkDocs](https://www.mkdocs.org/) and [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

## Project Overview

RICA is a mineral tracking and value calculation system designed to manage mineral batches, track their journey through processing stages, calculate recoverable values, and provide role-based access control for different stakeholders in the supply chain.

## Documentation Structure

```
heliks-technical-documentation/
├── docs/
│   ├── index.md                    # Home page
│   ├── architecture/               # System architecture
│   │   ├── system-context.md
│   │   ├── architecture-overview.md
│   │   └── database.md
│   ├── backend/                    # Backend documentation
│   │   └── backend.md
│   ├── frontend/                   # Frontend documentation
│   │   ├── frontend.md
│   │   └── frontend-info-architecture.md
│   ├── security/                   # Security documentation
│   │   └── security.md
│   ├── design/                     # Design documentation
│   │   └── wireframes.md
│   ├── stylesheets/                # Custom styling
│   └── assets/                     # Images and assets
├── mkdocs.yml                      # MkDocs configuration
└── requirements.txt                # Python dependencies
```

## Getting Started

### Prerequisites
- Python 3.8+

### Installation

```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Local Development

```bash
# Serve documentation locally
mkdocs serve
```

The documentation will be available at `http://127.0.0.1:8000`

### Build

```bash
# Build static site
mkdocs build
```

## Deployment

- **Backend**: Deployed on [Heroku](https://www.heroku.com/)
- **Frontend**: Deployed on [Vercel](https://vercel.com/)
- **Database**: PostgreSQL (managed via Heroku Postgres, inspected with DBeaver)

## Testing

- **Backend**: `pytest` for unit/integration tests, Swagger UI (`/docs`) for manual API testing, Postman for endpoint validation
- **Frontend**: Playwright (E2E) and Cypress (component/E2E) testing
- **Informational Website**: Tested using Playwright and Cypress

## Git Workflow

1. Pull latest from `main`: `git pull origin main`
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Make changes, test locally
4. Stage and commit: `git add . && git commit -m "descriptive message"`
5. Push: `git push origin feature/your-feature-name`
6. Open a Pull Request on GitHub targeting `main`
7. Request review from team members and resolve conflicts if any
8. Merge after approval

## Contributing

Follow the Git workflow above. All changes should go through pull requests with code review.

## Quick Links

- [Home](docs/index.md) - Project overview
- [Architecture](docs/architecture/architecture-overview.md) - System design
- [Backend Documentation](docs/backend/backend.md) - API, setup, testing & deployment
- [Frontend Documentation](docs/frontend/frontend.md) - App architecture, setup, testing & deployment
- [Database](docs/architecture/database.md) - Schemas and migrations
- [Security](docs/security/security.md) - Security features and controls
- [Glossary](docs/architecture/glossary.md) - Key terms and definitions
