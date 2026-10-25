# Local Workspace Onboarding Guide

Follow these steps to initialize your local development environment for the Python backend service framework.

##  System Prerequisites

Ensure your local machine has these foundational system components installed:

* **Python Engine:** Version `3.11+` exclusively.
* **Dependency Manager:** [uv](https://astral.sh) (v0.11.13+) installed via curl or shell script.
* **Local Caching:** Access to a local Redis server instance (optional for development).
* **Database:** PostgreSQL or SQLite for local development.

##  Execution Onboarding Sequence

### 1. Initialize Virtual Environment and Dependencies

Clone the backend repository, create the virtualenv via `uv`, and fetch every developer-group dependency asset:

```bash
# Initialize isolated virtual environment container matching .python-version
uv venv

# Sync and install all core project dependencies plus dev groups from pyproject.toml
uv pip install -r pyproject.toml --group dev
```

### 2. Formulating Local Configuration Variables

Generate a local `.env` configuration file inside your project backend root directory using these placeholders:

```ini
ENVIRONMENT="development"
DATABASE_URL="postgresql://user:password@localhost:5432/rica_dev"
JWT_SECRET="your-min-32-character-secret-key-here"
JWT_ALGORITHM="HS256"
ACCESS_TOKEN_EXPIRE_MINUTES="15"
MFA_REQUIRED="True"
MFA_ENCRYPTION_KEY="your-fernet-key-here"
REDIS_URL="redis://localhost:6379"
CORS_ORIGINS="http://localhost:3000"
LME_API_KEY="your-metals-api-key"
```
