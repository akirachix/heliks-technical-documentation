# Security Technical

# Security Technical Documentation

## Overview

RICA implements layered security controls covering authentication, authorization, session/token management, input validation, auditability, and secure configuration to protect user accounts, sensitive data, API endpoints, and administrative functionality.

## Authentication

- **JWT Access Tokens**: Issued on login via `POST /auth/login`. Signed with `HS256` using `JWT_SECRET` (min 32 characters). Default expiry `ACCESS_TOKEN_EXPIRE_MINUTES=15`. Payload includes `user_id`, `email`, `role`, `jti`, `exp`.
- **Token Blacklisting**: `core/auth.py` maintains a blacklist for revoked tokens (used on logout). Tokens are checked against the blacklist on authenticated requests.
- **Password Hashing**: Passwords hashed as `salt$sha256(salt + password)`. Salt is 16 random bytes hex-encoded. Verification splits on `$` and compares digests.
- **Login Protection**: Rate limiting and lockout via `login_max_attempts` (5), `login_window_seconds` (900), `lockout_duration_seconds` (900). `redis_url` required in production for distributed rate limiting.
- **MFA (TOTP)**: Optional/enabled via `mfa_required`. MFA secrets encrypted with Fernet using `MFA_ENCRYPTION_KEY`. Endpoints under `POST /auth/mfa/*` (setup, verify, disable, login). Models: `MFAToken`.
- **OTP Verification**: One-Time Password (OTP) verification provides an additional identity verification mechanism.
- **Password Reset**: Token-based password reset flow (`POST /auth/forgot-password`, `POST /auth/reset-password`). Tokens tracked in `PasswordResetToken`.

## Authorization

- **Role-Based Access Control (RBAC)**: Roles `SYSTEM_ADMINISTRATOR`, `SUPPLY_CHAIN_MANAGER`, `OPERATIONS_MANAGER`. Enforced via dependencies (`core/auth.py`, `dependencies/rbac.py`).
- **Value Calculations**: Restricted to `SUPPLY_CHAIN_MANAGER` via `ValueCalculationService.require_supply_chain_manager()` (401 if unauthenticated, 403 if wrong role).
- **Company Scoping**: 
  - `GET /mineral-batches` filters by `current_user.company_name` joined through batch creator
  - `_check_company_access()` blocks cross-company access to batches with 403
  - Users with `company_name = NULL` see all batches (fallback for legacy/admin use)
- **Least Privilege**: Users and system components receive only the permissions required for their intended tasks.
- **IDOR Protection**: Insecure Direct Object Reference (IDOR) vulnerabilities are mitigated by checking whether the authenticated user is authorized to access the requested resource. Users cannot access another user's private information by manipulating IDs.
- **Endpoint Protection**: Sensitive operations require `get_current_user` dependency.

## Input Validation & Data Integrity

- **Pydantic Schemas**: All request/response bodies validated via Pydantic v2 schemas
- **Database Constraints**: 
  - Copper-only batches (`ck_batches_mineral_copper`)
  - Range checks: `quantity_tonnes >= 0`, `grade` 0–1, `recovery` 0–1 (nullable)
  - Stage/recovery constraints on `batch_history`
  - Unique constraints on emails, batch codes, stage names/orders
- **Value Calculation Guards**: Validates quantity/grade/recovery ranges, requires quantity and grade, rejects invalid values with 422

## Secure Configuration

- **Secrets Management**: All secrets via environment variables; never committed to source control
- **Settings Validation**:
  - `JWT_SECRET` >= 32 characters
  - `MFA_ENCRYPTION_KEY` must be valid Fernet key
  - Production requires `REDIS_URL`
  - Production requires non-localhost `FRONTEND_URL`
- **Test Isolation**: `TESTING=1` forces SQLite in-memory database
- **CORS**: Configured via `CORS_ORIGINS` (comma-separated), defaults to `http://localhost:3000`. Middleware allows credentials and specific headers/methods
- **Proxy Headers**: `TRUST_PROXY_HEADERS` configurable for deployments behind proxies

## Logging & Auditing

- **Audit Logging**: `AuditLog` model and `middleware/audit.py` track actions
- **Error Handling**: Centralized exception handlers (`core/exceptions.py`)
- **Logging**: Application logging via Python `logging` (no secrets logged)

## Network & Transport

- **HTTPS**: Required in production (Heroku terminates TLS)
- **CORS Policy**: Restrictive origin allowlist
- **Rate Limiting**: Applied to auth endpoints to mitigate brute force (15-minute window)
- **Health Endpoint**: `GET /health` public for uptime checks

## Secure Coding Practices

- **Password Storage**: Never store plaintext passwords. Passwords are hashed using `salt$sha256(salt + password)`.
- **SQL Injection**: Prevented by SQLAlchemy ORM parameterized queries
- **Dependency Pinning**: Managed via `pyproject.toml`
- **Secret Rotation**: Rotate `JWT_SECRET`, `MFA_ENCRYPTION_KEY` if compromised
- **Least Privilege**: Role checks on sensitive operations
- **Error Messages**: Generic auth errors to avoid user enumeration where appropriate

## Data Protection

- **Sensitive Data Protection**: Sensitive information such as maternal health information, user credentials, authentication information, and private user data are protected.
- **Encryption in Transit**: Information is encrypted while being transferred between the client, API, and backend.
- **Encryption at Rest**: Sensitive information is protected while stored. Passwords use hashing rather than reversible encryption.
- **Secrets Management**: Sensitive credentials and secrets are stored in environment configuration and should not be committed to the repository.

## Administrative Access

Administrative functionality is restricted to authorized System Administrator accounts.

Administrators can:

- Manage users
- Manage companies
- Manage maternal health content and tips
- View platform growth and engagement metrics
- Monitor safety alerts
- Manage administrator account settings
- Access audit logs

## OWASP Top 10 Considerations

| Risk | Mitigation |
|---|---|
| A01: Broken Access Control | RBAC, company scoping, IDOR protection, least privilege, route guards |
| A02: Cryptographic Failures | Password hashing (salt$sha256), Fernet encryption for MFA secrets, HTTPS in transit |
| A03: Injection | Pydantic validation, SQLAlchemy ORM parameterized queries |
| A04: Insecure Design | Layered architecture, secure defaults, audit logging |
| A05: Security Misconfiguration | Environment-based config, CORS restrictions, secure headers |
| A06: Vulnerable Components | Dependency management, regular updates |
| A07: Authentication Failures | JWT with blacklisting, MFA/TOTP, rate limiting, strong password policies |
| A08: Software/Data Integrity Failures | Input validation, audit trail, secure CI/CD |
| A09: Security Logging/Monitoring | Comprehensive audit logging with user, IP, and action tracking |
| A10: Server-Side Request Forgery | Input validation, restricted external API calls with proper configuration |

## Security Architecture

```
Authentication
     │
     ├── JWT Tokens
     ├── OTP Verification
     └── MFA (TOTP)
     │
     ▼
Authorization
     │
     ├── Role-Based Access Control (RBAC)
     ├── Least Privilege
     └── IDOR Protection
     │
     ▼
API Protection
     │
     ├── Rate Limiting (15-minute window)
     └── Protected Endpoints
     │
     ▼
Data Protection
     │
     ├── Sensitive Data Protection
     ├── Encryption in Transit & At Rest
     └── Password Hashing (salt$sha256)
     │
     ▼
RICA Backend (FastAPI)
     │
     ▼
PostgreSQL Database
```

## Incident Response Notes

- Revoke tokens via blacklist on logout
- Disable accounts via `account_status`
- Use password reset flow for credential recovery
- Review audit logs for suspicious activity
- Rotate compromised secrets immediately

## Compliance Considerations

- Store minimal PII (name, email, company)
- Encrypt MFA secrets at rest (Fernet)
- Use secure defaults in production configuration validation
- Audit trail for sensitive operations
- Implement data protection controls for sensitive information


---

# Threat Model

# Threat Mitigation Model

This documentation tracks critical application threat vectors and outlines the software controls built into the codebase to secure them.

## Operational Threat Vector Mitigations

### 1. Privilege Escalation & Cross-Tenant Data Leaks
* **Vulnerability Sensitivity Tier:** ️ **HIGH**
* **Target Vulnerability Component:** Authorization services, endpoint routers, backend database drivers.
* **Code Implementation Control:**

    !!! info "Compound Parameter Authorization Checks"
        System access scopes are evaluated through a multi-dimensional criteria check, rather than role attributes alone:
        $$\text{Authorization Verification} = \text{User} + \text{Role} + \text{Company} + \text{Site} + \text{Resource}$$
        Any ambiguous authorization decisions must fail closed, denying access and returning a `403 Forbidden` response.

### 2. Unauthorized Data Alterations & Audit Trails Erasure
* **Vulnerability Sensitivity Tier:**  **CRITICAL**
* **Target Vulnerability Component:** Batch registry logs, transaction ledgers, database storage permissions.
* **Code Implementation Control:**
    * Software routing blocks intercept and drop direct `DELETE` or `UPDATE` queries targeting timeline history tables.
    * Corrections must be processed on an **append-only model** by inserting a new event log that captures the `previous_state`, `new_state`, `user_id`, `timestamp`, and an explicit `reason` string.

### 3. Automated Account Enumeration & Brute-Force Probing
* **Vulnerability Sensitivity Tier:** ️ **HIGH**
* **Target Vulnerability Component:** User login, password recovery services.
* **Code Implementation Control:**
    * Background rate-limiting middleware limits request frequencies on sensitive authentication entry points.
    * Authentication failure handlers return a single generic error message to fully hide whether the email or password parameter was invalid.


---

# Compliance Audit

# Statutory Compliance & Audit Policies

##  Zambian Data Protection Act (2021) Alignment
RICA implements specific backend storage rules to comply with **Zambia's Data Protection Act of 2021** regarding the processing, isolation boundaries, and long-term archiving of customer data records.

### Enforced Compliance Invariants:
* **Statutory Archiving Window:** All active tracking ledgers and administrative logs are kept for a **minimum mandatory retention threshold of 1 year** before entering archival storage routines.
* **Anonymization Invariants:** When custom profiles are deactivated or users execute their legal right to data erasure, all personal identifiers are thoroughly anonymized. The transactional asset tracking loops remain intact to prevent structural dependency breaks.
* **Geographic Data Sovereignty Constraints:** Database clusters and cloud computing boundaries run within geographical parameters that comply with regional cloud data privacy guidelines.

##  Permanent System Log Capture Events
RICA middleware automatically captures these operational modifications, recording them to an unalterable auditing stream containing user metadata, timestamp parameters, resource hashes, and source IP headers:

### Access & Security Audits
* Elevation of workspace permissions, custom rule modifications, or security group resets.
* User status adjustments (e.g., locking an account into a `DISABLED` state).

### Inventory & Value Audits
* Initialization of a new mineral batch record.
* Stage transition event triggers.
* Adjustments to financial baseline snapshots or passport data extractions.


---

# Data Classification

# Sensitive Data Classification Registry

This central registry profiles all data assets processed by the platform, detailing their structural importance, encryption baselines, and retention constraints.

## Information Asset Security Mapping

| Information Asset Target | Sensitivity Tier | In-Transit Encryption | At-Rest Security Controls | Storage Engine Location |
| :--- | :--- | :--- | :--- | :--- |
| **System Signing Keys & Master Variables** <br>*(PostgreSQL access strings, JWT secret tokens)* |  **CRITICAL** | Encrypted via TLS 1.3 loops | Abstracted out of the codebase entirely. Injected as runtime environment variables. | Host Runtime Memory Cache |
| **User Access Credentials** <br>*(Active login pass strings, session tokens, reset hashes)* |  **HIGH** | Encrypted via TLS 1.3 loops | Hashed using secure cryptographic algorithms. Excluded from all system logs. | PostgreSQL (`users`) Table |
| **Mineral Batch Specifications** <br>*(Mass weights, laboratory assay inputs, moisture balances)* |  **HIGH** | Encrypted via TLS 1.3 loops | Isolated behind multi-tenant row security filters. Append-only change logging. | PostgreSQL (`mineral_batches`) Table |
| **Traceability & Timeline Ledgers** <br>*(Custody transition records, processing timelines)* |  **HIGH** | Encrypted via TLS 1.3 loops | Blocked database write operations (no `UPDATE`/`DELETE` authority). | PostgreSQL (`batch_history`) Table |
| **System Event Audit Logs** <br>*(Administrative records, transaction tracking records)* |  **HIGH** | Encrypted via TLS 1.3 loops | Permanent read-only rows. 1-year statutory archiving window. | PostgreSQL (`audit_events`) Table |
| **User Identity Profiles** <br>*(Real names, employee emails, authorization roles)* |  **MEDIUM** | Encrypted via TLS 1.3 loops | Restricted visibility. Data fields are masked during export loops. | PostgreSQL (`users`) Table |
| **Market Reference Data** <br>*(LME pricing logs, fetch date parameters)* |  **MEDIUM** | Encrypted via TLS 1.3 loops | Read-only cache layers flagged with temporal provenance stamps. | PostgreSQL (`market_feeds`) Table |


---

