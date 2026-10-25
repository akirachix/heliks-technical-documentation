# RICA Backend Documentation

## 1. Overview

The RICA backend is the central application layer that connects the RICA client applications (web frontend) to the PostgreSQL database and provides the RESTful API used by the platform.

The backend is responsible for:

- Exposing REST API endpoints
- Receiving and processing requests from client applications
- Validating incoming request data using Pydantic
- Applying application and business logic
- Authenticating and authorizing users with JWT tokens and role-based access control
- Managing user roles and company-scoped permissions
- Creating, reading, updating, and deleting application data
- Communicating with the PostgreSQL database via SQLAlchemy ORM
- Managing mineral batches, processing stages, batch history, and value calculations
- Computing recoverable metal values using live LME copper prices
- Supporting scenario planning and analysis
- Handling MFA, password reset, audit logging, and rate limiting
- Returning structured API responses to client applications

The backend acts as the single access point between client applications and PostgreSQL.

```
RICA Web Frontend
        |
        | HTTP / REST API requests (Authorization: Bearer Token)
        |
        v
   RICA Backend (FastAPI)
        |
        | SQLAlchemy ORM
        |
        v
   PostgreSQL Database
```

Client applications do not connect directly to PostgreSQL.

## 2. Technology Stack

### Core Technologies

| Technology | Version | Purpose |
|---|---|---|
| Python | 3.11+ | Primary backend programming language |
| FastAPI | Latest | Web framework for REST API development |
| Uvicorn | Latest | ASGI server for running FastAPI |
| Pydantic v2 | Latest | Request/response validation and serialization |
| pydantic-settings | Latest | Environment variable configuration |
| SQLAlchemy | 2.x | ORM for database communication |
| Alembic | Latest | Database migrations |
| PostgreSQL | 14+ | Primary relational database |
| psycopg | v3 | PostgreSQL adapter |

### Security & Authentication

| Technology | Purpose |
|---|---|
| PyJWT | JWT token generation and verification |
| cryptography (Fernet) | Encryption of MFA secrets |
| pyotp | TOTP-based MFA implementation |
| hashlib | Password hashing (salt$sha256) |

### Utilities & Integrations

| Technology | Purpose |
|---|---|
| requests | HTTP client for LME metals price API |
| qrcode + Pillow | QR code generation for mineral batches |
| redis | Rate limiting in production |
| emails | Email delivery for password reset/MFA |
| python-dotenv | Environment variable loading |

### Development & Testing

| Technology | Purpose |
|---|---|
| pytest | Unit and integration testing |
| pytest-cov | Code coverage reporting |
| ruff | Linting and code formatting |
| Postman | API testing |
| Swagger UI (/docs) | Interactive API documentation |

## 3. Python Configuration

### Setup

```bash
# Create virtual environment
python -m venv venv

# Activate (Linux/macOS)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate

# Install dependencies
uv pip install -r pyproject.toml --group dev
# or
pip install -r requirements.txt
```

### Running the Server

```bash
uv run --no-sync uvicorn main:app --reload
```

API documentation is available at:

- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc
- **Health Check:** http://localhost:8000/health

## 4. Architecture

### Layered Architecture

```
Client Application (RICA Frontend)
              │
              ▼
            Router Layer
              │
              ▼
            Schema Layer (Pydantic Validation)
              │
              ▼
            Service Layer (Business Logic)
              │
              ▼
           Repository Layer
              │
              ▼
            SQLAlchemy ORM
              │
              ▼
          PostgreSQL Database
```

### Layer Responsibilities

| Layer | Responsibility |
|---|---|
| Router | HTTP endpoint definitions, request/response handling, dependency injection |
| Schema | Pydantic models for request validation and response serialization |
| Service | Core business logic, rules enforcement, external API calls |
| Repository | Data access abstraction, database queries |
| Model | SQLAlchemy ORM models representing database tables |
| Core/Utils | Authentication, configuration, exceptions, middleware |

## 5. API Architecture & Endpoint Groups

All endpoints are RESTful and prefixed by their resource groups.

| Group | Prefix | Description |
|---|---|---|
| Auth | /auth | Login, logout, password reset, MFA (setup/verify/disable/login) |
| Users | /users | User CRUD, profile management, company assignment |
| Mineral Batches | /mineral-batches | Batch registration, tracking, search, QR codes, assay uploads |
| Processing Stages | /processing-stages | Stage management |
| Value Calculations | /values | Value computation, history, value tables with optional historical pricing |
| Scenarios | /scenarios | What-if scenario creation and results |
| Scenario Results | /scenario-results | Scenario result retrieval |
| Decision Support | /decision_support | Rule-based assessment of scenario results with findings and recommendations |

## 6. Authentication, Authorization & Security

### Authentication

- **JWT Tokens:** HS256 signed tokens with configurable expiry (default 15 minutes)
- **Token Blacklisting:** Revoked tokens on logout
- **Password Security:** `salt$sha256(salt + password)` hashing
- **MFA:** TOTP with Fernet-encrypted secrets
- **Password Reset:** Time-limited token-based reset flow
- **Rate Limiting:** Login attempt throttling with Redis support

### Authorization & RBAC

- **Roles:** SYSTEM_ADMINISTRATOR, SUPPLY_CHAIN_MANAGER, OPERATIONS_MANAGER
- **Route Protection:** Dependency-based guards
- **Value Access:** Restricted to SUPPLY_CHAIN_MANAGER only
- **Company Scoping:** Users see only their company's batches via company_name filtering

### Security Measures

- **CORS:** Configurable origin allowlist
- **Input Validation:** Pydantic schema validation on all requests
- **SQL Injection Prevention:** Parameterized queries via SQLAlchemy ORM
- **Audit Logging:** Comprehensive audit trail
- **Secure Headers:** Configurable via middleware
- **Environment Secrets:** Never committed to version control

## 7. Database Design

### Core Entities

- `users` - User accounts with roles and company association
- `processing_stages` - Workflow stages (Extraction, Processing, Smelting, Refining)
- `mineral_batches` - Copper batch records with ownership and tracking
- `batch_history` - Stage-by-stage snapshots of batch state
- `value_calculations` - Computed valuations with LME pricing
- `scenarios` / `scenario_results` - What-if analysis
- `mfa_tokens` / `password_reset_tokens` - Security tokens
- `audit_logs` - System audit trail

### Data Integrity

- Foreign key constraints maintain referential integrity
- Check constraints enforce business rules (grade 0-1, copper-only, non-negative quantities)
- Unique constraints on emails, batch codes, stage names/orders
- UUID primary keys for most entities
- Timezone-aware timestamps

Full schema details are available in [Database Schemas](../architecture/database-schemas.md).

## 8. Environment Configuration

Key environment variables (see .env.example):

| Variable | Required | Description |
|---|---|---|
| DATABASE_URL | Yes | PostgreSQL connection string (auto-normalized to psycopg3) |
| JWT_SECRET | Yes | JWT signing key (min 32 chars) |
| MFA_ENCRYPTION_KEY | Yes | Fernet key for MFA secret encryption |
| ENVIRONMENT | No | development or production |
| CORS_ORIGINS | No | Comma-separated allowed origins |
| REDIS_URL | Prod only | For rate limiting |
| LME_API_KEY/URL/SYMBOL | No | Metals-API configuration for pricing |

In production, strict validation enforces required secrets and non-localhost frontend URLs.

## 9. Development Workflow

```bash
# Install dependencies
uv pip install -r pyproject.toml --group dev

# Run migrations
alembic upgrade head

# Start dev server
uvicorn main:app --reload

# Run tests
pytest -q

# Lint and format
ruff check .
ruff format .
```

## 11. Decision Support Module

The Decision Support module provides rule-based assessment of scenario results to support data-driven decision making. It evaluates feasibility, risks, and recommendations based on technical, operational, regulatory, and financial factors.

| Component | File | Purpose |
|---|---|---|
| Model | models/decision_support.py | ScenarioDecision entity linking to scenario_results with assessment state, findings, recommendations, and audit metadata |
| Repository | repositories/decision_repository.py | Data access layer for decisions (queries, persistence, no business logic) |
| Schemas | schemas/decision_support.py | Request/response validation for assessment context and decision records |
| Service | services/decision_support_service.py | Orchestrates assessment flow, maps scenario data, and persists results |
| Rules Engine | services/decision_rules.py | Rule-based evaluation producing outcomes, findings, blockers, and recommendations |
| Router | routers/decision_support.py | REST endpoints under /decision_support |

**Endpoints:**

- `POST /decision_support/assess` – Assess or reassess an existing scenario result
- `GET /decision_support/{decision_id}` – Retrieve a decision by ID
- `GET /decision_support/scenario/{scenario_id}` – List all decisions for a scenario
- `DELETE /decision_support/{decision_id}` – Remove a decision

**Key Features:**

- One decision per scenario result (reassessing updates the existing record)
- Contextual assessment with optional operational inputs (ore type, mining method, capacity, regulatory/infrastructure status, etc.)
- Outcomes: VIABLE, CONDITIONALLY_VIABLE, NOT_VIABLE, INSUFFICIENT_INFORMATION
- Recommendations: PROCEED, PROCEED_AFTER_VALIDATION, REVISE_SCENARIO, REVIEW_REQUIRED
- Lifecycle tracking: CALCULATED → ASSESSED or BLOCKED
- Structured findings and actionable recommendations stored as JSON

