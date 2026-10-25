# Security

This page explains how security works in the RICA backend. It covers login, passwords, roles, MFA, password reset, rate limiting, audit logging and startup checks.


## Summary

| Control | What it does | Where in the code |
| --- | --- | --- |
| Login with JWT | Checks email and password, then returns a signed access token | `routers/auth.py`, `services/user_service.py`, `core/auth.py` |
| Logout | Revokes the token by saving its ID (`jti`) in a blacklist | `services/user_service.py`, `core/auth.py` |
| Password hashing | Salted SHA-256 | `services/user_service.py` |
| Roles | Three roles with helper checks for each | `dependencies/rbac.py` |
| Account status | A `DISABLED` account cannot log in or use a token | `services/user_service.py`, `core/auth.py` |
| MFA | Time-based one-time codes (TOTP), secret stored encrypted | `routers/mfa.py`, `services/mfa_service.py` |
| Password reset | One-time reset link, token stored as a hash, valid for 1 hour | `routers/password_reset.py`, `services/password_reset_service.py` |
| Login rate limit | 5 attempts per 15 minutes per IP address | `middleware/rate_limit.py` |
| Audit logging | One database row for every request | `middleware/audit.py`, `models/audit_log.py` |
| Startup checks | The app refuses to start with weak or missing secrets | `core/config.py` |

---

## 1. Authentication

### Login flow

1. The client sends `POST /auth/login` with `email` and `password`.
2. The service looks up the user by email and checks the password.
3. If the email or the password is wrong, it returns `401` with one message for both cases: `Invalid email or password`. This does not tell an attacker which part was wrong.
4. If the account status is `DISABLED`, it returns `401` with `Account is disabled`.
5. Otherwise it creates a JWT and returns it.

```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "YourPassword"}'
```

```json
{ "access_token": "<jwt>", "token_type": "bearer" }
```

### The token

| Claim | Meaning |
| --- | --- |
| `user_id` | ID of the user |
| `email` | Email of the user |
| `role` | One of the three roles |
| `jti` | Unique ID of this token, used for logout |
| `exp` | Expiry time |

- Algorithm: `HS256` (setting `JWT_ALGORITHM`).
- Lifetime: 15 minutes by default (setting `ACCESS_TOKEN_EXPIRE_MINUTES`).
- Signed with `JWT_SECRET`, which must be at least 32 characters.

### Sending the token

Protected endpoints need this header:

```text
Authorization: Bearer <access_token>
```

### What happens on every protected request

`get_current_user` in `core/auth.py` runs these checks in order:

1. The `Authorization` header must be present.
2. The token must have a valid signature and must not be expired.
3. The token's `jti` must not be in the blacklist.
4. The user must exist.
5. The account status must not be `DISABLED`.

| Status | Message | When |
| --- | --- | --- |
| `401` | `Missing or invalid authorization header` | No bearer token |
| `401` | `Token has expired` | Token past its `exp` |
| `401` | `Invalid token` | Bad signature or broken token |
| `401` | `Token has been revoked` | Token was logged out |
| `401` | `Account is disabled` | Account status is `DISABLED` |

### Logout

`POST /auth/logout` reads the token, takes its `jti` and `exp`, and saves the `jti` in the blacklist until the token's own expiry time. After that, every request with this token returns `401 Token has been revoked`. Expired entries are removed from the blacklist automatically.

---

## 2. Password storage

Passwords are never stored in plain text. The stored value in `users.password_hash` has this format:

```text
<salt>$<sha256(salt + password)>
```

- The salt is 16 random bytes (32 hex characters), created for each password with `os.urandom`.
- Two users with the same password get different hashes.
- `verify_password` splits the stored value at `$`, hashes the typed password with the same salt, and compares the result.
- `password_changed_at` records when the password was last reset.

---

## 3. Roles and access control

### Roles

| Role | Meaning |
| --- | --- |
| `SYSTEM_ADMINISTRATOR` | Full access, manages users |
| `SUPPLY_CHAIN_MANAGER` | Value calculations and supply chain data |
| `OPERATIONS_MANAGER` | Operations data; default role for a new user |

### Role checks

`dependencies/rbac.py` has three ready-made checks. Each one returns `403` when the role is not allowed.

| Check | Allowed roles | `403` message |
| --- | --- | --- |
| `require_admin` | `SYSTEM_ADMINISTRATOR` | `Admin access required` |
| `require_supply_chain_manager` | `SYSTEM_ADMINISTRATOR`, `SUPPLY_CHAIN_MANAGER` | `Supply chain manager access required` |
| `require_operations_manager` | All three roles | `Operations manager access required` |

Higher roles can do everything a lower role can do.

### Where access is enforced today

| Area | How |
| --- | --- |
| Value calculation | The service allows only `SUPPLY_CHAIN_MANAGER` (`403 Only supply chain managers can access value calculations`). It also blocks access to a batch created by a user from a different company (`403 Cannot access batches from a different company`). |
| Mineral batch list | `GET /mineral-batches` returns only batches from the logged-in user's `company_name` |
| Other endpoints | Require a valid login (`get_current_user`) |

### Account status

`users.account_status` is `ACTIVE` or `DISABLED`. A user is disabled with `PATCH /users/{user_id}/disable` or `DELETE /users/{user_id}`. Users are disabled, not removed, so their history stays in the database.

---

## 4. Multi-factor authentication (MFA)

MFA uses time-based one-time codes (TOTP), the kind used by Google Authenticator and similar apps.

### Endpoints

| Endpoint | Needs login | Purpose |
| --- | --- | --- |
| `POST /auth/mfa/setup` | Yes | Start MFA setup |
| `POST /auth/mfa/verify` | Yes | Confirm setup with a code |
| `POST /auth/mfa/disable` | Yes | Turn MFA off with a code |
| `POST /auth/mfa/login` | No | Log in with email, password and code |

### Setup

1. The user calls `POST /auth/mfa/setup`. If MFA is already on, it returns `400 MFA is already enabled`.
2. The server creates a random TOTP secret and encrypts it with Fernet using `MFA_ENCRYPTION_KEY`.
3. It saves a setup token (64 hex characters, valid for 10 minutes) with the encrypted secret in `mfa_tokens`.
4. It returns the `secret`, a QR code as a `qr_code_url` image (issuer name `RICA`) and the setup `token`, and it also emails the setup code.
5. The user scans the QR code, then calls `POST /auth/mfa/verify` with the setup `token` and a 6-digit `code`.
6. If the token is valid, not expired and not used, and the code is correct, the server marks the token as used. It then sets `mfa_enabled = true` and stores the encrypted secret in `users.mfa_secret_encrypted`.

Codes are accepted within 2 time steps before or after the current one (about 60 seconds either side).

### Login with MFA

`POST /auth/mfa/login` takes `email`, `password` and `code`. It checks the password in the same way as normal login, then checks that MFA is on and that the code is correct. It returns an access token.

| Message | When |
| --- | --- |
| `MFA is not enabled for this user` | The user has not set up MFA |
| `Invalid MFA code` | The code is wrong |

### Turning MFA off

`POST /auth/mfa/disable` needs a correct current code. It clears `mfa_enabled` and removes the stored secret.

---

## 5. Password reset

### Forgot password

`POST /auth/forgot-password` takes an `email`.

- It always returns the message `If the email exists, a reset link has been sent`, so the message does not show whether the email is registered.
- For a registered email, it creates a random token (32 bytes), saves only its SHA-256 hash in `password_reset_tokens`, sets the expiry to 1 hour, and emails the link `FRONTEND_URL/reset-password?token=...`.

### Reset password

`POST /auth/reset-password` takes the `token` and a `new_password` (at least 8 characters).

| Message | When |
| --- | --- |
| `Invalid or expired reset token` | The token hash is not found |
| `Reset token has expired` | More than 1 hour has passed |
| `Reset token has already been used` | The token was used before |
| `Password has been reset successfully` | Success |

On success, the token is marked as used, the new password is hashed with a new salt, and `password_changed_at` is updated. Each token works only once.

---

## 6. Login rate limiting

A middleware limits attempts on `POST /auth/login`.

| Setting | Default | Meaning |
| --- | --- | --- |
| `LOGIN_MAX_ATTEMPTS` | `5` | Attempts allowed in one window |
| `LOGIN_WINDOW_SECONDS` | `900` | Window length (15 minutes) |

- The counter is kept per client IP address.
- After 5 attempts in 15 minutes, the next request returns `429` with `Too many login attempts. Please try again later.`
- Old attempts drop out of the window as time passes.
- The limiter is switched off when `TESTING=1`.

---

## 7. Audit logging

A middleware writes one row to the `audit_logs` table for every request, after the response is ready.

| Column | Value |
| --- | --- |
| `audit_id` | Unique ID |
| `user_id` | The user from the bearer token, or empty if there is no valid token |
| `action` | Method and path, for example `POST /auth/login` |
| `resource` | Request path |
| `ip_address` | Client IP address |
| `user_agent` | Browser or client name |
| `timestamp` | Time in UTC |

- Request bodies and query strings are not stored, so passwords and tokens do not end up in the log.
- If writing the log fails, the error is ignored and the request still succeeds.
- If a user is deleted from the database, `user_id` in their old rows is set to empty.
- The audit log is switched off when `TESTING=1`.

---

## 8. Startup configuration checks

The settings are checked when the app starts. If a rule fails, the app does not start.

| Rule | Setting |
| --- | --- |
| At least 32 characters | `JWT_SECRET` |
| Must be a valid Fernet key | `MFA_ENCRYPTION_KEY` |
| Must be set in production | `REDIS_URL` |
| Must not be `http://localhost:3000` in production | `FRONTEND_URL` |
| Must be a valid comma-separated list | `CORS_ORIGINS` |

### Environment variables for security

| Variable | Required | Notes |
| --- | --- | --- |
| `JWT_SECRET` | Yes | At least 32 characters |
| `MFA_ENCRYPTION_KEY` | Yes | Generate with `Fernet.generate_key()` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | No | Default `15` |
| `LOGIN_MAX_ATTEMPTS` | No | Default `5` |
| `LOGIN_WINDOW_SECONDS` | No | Default `900` |
| `ENVIRONMENT` | No | `development` or `production` |
| `REDIS_URL` | Production | Checked at startup |
| `FRONTEND_URL` | Production | Used in the reset link |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_FROM` | For email | Used for MFA setup and password reset emails |

---

## 9. CORS

Only the origins in `CORS_ORIGINS` can call the API from a browser.

- Allowed methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`
- Allowed headers: `Authorization`, `Content-Type`, `Accept`, `Origin`
- Credentials are allowed.

---

## 10. Error handling

Business errors use small classes in `core/exceptions.py`. Each class has a status code, and a handler turns it into a JSON response: `{"detail": "..."}`.

| Class | Status |
| --- | --- |
| `AppError` | `400` |
| `NotFoundError` | `404` |
| `ConflictError` | `409` |
| `BusinessRuleError` | `422` |
| `InvalidCredentialsError` | `401` |
| `AccountDisabledError` | `401` |

---

## 11. Security data model

| Table | Security columns |
| --- | --- |
| `users` | `password_hash`, `password_changed_at`, `mfa_enabled`, `mfa_secret_encrypted`, `role`, `account_status`, `company_name` |
| `mfa_tokens` | `token`, `user_id`, `secret_encrypted`, `purpose` (`setup` or `verify`), `expires_at`, `used` |
| `password_reset_tokens` | `token_hash`, `user_id`, `purpose`, `expires_at`, `used` |
| `audit_logs` | `user_id`, `action`, `resource`, `ip_address`, `user_agent`, `timestamp` |

`mfa_tokens` and `password_reset_tokens` rows are deleted when the user is deleted.

---

## 12. Endpoint reference

| Method | Path | Login needed | Success | Main errors |
| --- | --- | --- | --- | --- |
| `POST` | `/auth/login` | No | `200` with token | `401`, `429` |
| `POST` | `/auth/logout` | Token | `200` | `401` |
| `POST` | `/auth/forgot-password` | No | `200` | |
| `POST` | `/auth/reset-password` | No | `200` | `400`, `422` |
| `POST` | `/auth/mfa/setup` | Yes | `200` | `400`, `401` |
| `POST` | `/auth/mfa/verify` | Yes | `200` | `400`, `401` |
| `POST` | `/auth/mfa/disable` | Yes | `200` | `400`, `401` |
| `POST` | `/auth/mfa/login` | No | `200` with token | `400`, `401` |

---