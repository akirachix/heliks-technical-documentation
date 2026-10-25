# External Integrations

This document describes all external service integrations used by the RICA Backend.

## 1. Metals-API (LME Copper Price Feed)

The system integrates with Metals-API to retrieve benchmark copper pricing from the London Metal Exchange (LME).

### Overview

The system retrieves copper pricing from Metals-API to compute batch valuations. All value calculations are performed exclusively for copper (LME-XCU) using the London Metal Exchange (LME) as the benchmark reference market.

### London Metal Exchange (LME) - Benchmark Reference

- **Exchange:** London Metal Exchange (LME) - the global benchmark market for non-ferrous industrial metals
- **Benchmark Role:** LME prices represent the industry-standard reference prices used for contract pricing, hedging, and mineral valuation. They are used here strictly as a benchmark/reference price, not as a realized transaction or sale price
- **Commodity Scope:** Only Grade A Copper Cathode on the LME is in scope. The system does not support valuation for other metals
- **Instrument/Symbol:** LME-XCU (LME Copper). This is the target instrument queried from the pricing feed

### External Price Fetching Routine

The calculation engine connects to Metals-API via synchronous HTTP requests to obtain copper price data.

### Integration Properties

| Property | Value |
|---|---|
- **API Endpoint Base:** [https://metals-api.com](https://metals-api.com/api)
| Target Symbol Token | XCU (maps to LME Copper / LME-XCU) |
| Full Symbol Reference | LME-XCU (Grade A Copper Cathode) |
| Authentication | API key passed via query parameter `access_key` (configured via `LME_API_KEY`) |
| Base Currency | USD |
| Price Units | Prices are normalized to USD/tonne for calculations (`price_unit`: "USD/tonne", `currency`: "USD") |
| Price Source | Stored as "Metals-API (LME copper)" in `value_calculations.price_source` |

### Endpoints

| Mode | Endpoint | Description |
|---|---|---|
| Live/Latest | `{API_BASE}/latest` | Fetches the most recent LME copper rate |
| Historical | `{API_BASE}/{price_date}` | Fetches the LME copper rate for a specific date (YYYY-MM-DD) |

Query parameters (both modes): `access_key`, `base=USD`, `symbols=XCU` (with fallback handling for alternative rate representations).

### Historical Price Validation

If an optional `price_date` parameter is supplied to value calculation endpoints/flows, the backend enforces the following:

- **Validation Window:** The requested date must be strictly within the last 60 days from the current date
- **Violation Response:** Requests with `price_date` older than 60 days are rejected with HTTP 422 Unprocessable Entity
- **Live Default:** When `price_date` is not provided, the system fetches the latest available LME price

### Calculation Usage

- **Storage:** Retrieved prices are persisted as both `reference_price` and `current_price` on each `value_calculation` record
- **Formulas:**
  - `contained_metal_tonnes = quantity × grade`
  - `recoverable_metal_tonnes = contained_metal_tonnes × recovery` (if recovery is NULL, `recoverable_metal_tonnes = contained_metal_tonnes`)
  - `calculated_value = recoverable_metal_tonnes × reference_price`
- **Price Date:** The `price_date` used for the fetched price is recorded on the calculation

### Configuration

| Setting | Required | Default | Purpose |
|---|---|---|---|
| LME_API_KEY | Yes (for live prices) | None | Metals-API access key |
| LME_API_URL | No | [https://metals-api.com/api](https://metals-api.com/api) | Metals-API base URL |
| LME_SYMBOL | No | XCU | Target symbol for LME copper |
| LME_UNIT_TO_TONNE | No | 1.0 | Conversion factor to normalize price to USD/tonne |

## 2. SMTP Email Service

The system uses SMTP for transactional emails related to authentication and security.

### Purpose

Send emails for Multi-Factor Authentication (MFA) setup/verification and password reset flows.

### Implementation

- **Service:** `services/email_service.py` using Python's `smtplib` with `EmailMessage`
- **Transport:** Configurable SMTP server (defaults to localhost for development)
- **Behavior:**
  - In test mode (`TESTING=1`), emails are recorded in-memory via `get_sent_emails()` instead of being sent
  - If SMTP credentials are provided, the service attempts STARTTLS and authentication
- **Usage:** Called by `services/mfa_service.py` and `services/password_reset_service.py`

### SMTP Configuration

| Setting | Required | Default | Purpose |
|---|---|---|---|
| SMTP_HOST | No | localhost | SMTP server hostname |
| SMTP_PORT | No | 1025 | SMTP server port |
| SMTP_FROM | No | noreply@localhost | Sender email address |
| SMTP_USER | No | None | SMTP username for authentication |
| SMTP_PASSWORD | No | None | SMTP password for authentication |

## Notes

Redis is listed as a dependency in `pyproject.toml` but is not actively used for external service integration in the current codebase.
