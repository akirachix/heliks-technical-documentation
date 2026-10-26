
# Database Entities Specifications

This specification maps the production table structures, constraint targets, and column rules used by the data layer.

## Core Schema Matrices

### users table
Tracks core credential maps and roles for system entities.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| user_id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| email | VARCHAR(255) | NOT NULL, UNIQUE | Case-insensitive login profile key. |
| password_hash | VARCHAR(255) | NOT NULL | Hashed password stored as salt$sha256(salt + password). |
| first_name | VARCHAR(100) | NOT NULL | User's first name. |
| last_name | VARCHAR(100) | NOT NULL | User's last name. |
| role | ENUM (user_role) | NOT NULL | Limits: SYSTEM_ADMINISTRATOR, SUPPLY_CHAIN_MANAGER, OPERATIONS_MANAGER. |
| company_name | VARCHAR(255) | NULLABLE | Multi-tenant isolation filter used for batch scoping. |
| account_status | ENUM (account_status) | NOT NULL | Default: ACTIVE. Limits: ACTIVE, DISABLED. |
| mfa_enabled | BOOLEAN | NOT NULL | Default: FALSE. |
| mfa_secret_encrypted | VARCHAR(512) | NULLABLE | Fernet-encrypted TOTP secret. |
| password_changed_at | TIMESTAMP WITH TIME ZONE | NULLABLE | Timestamp of last password change. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### processing_stages table
Defines the processing workflow stages.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| stage_id | SMALLINT | PRIMARY KEY | Unique stage identifier. |
| stage_name | VARCHAR(100) | NOT NULL, UNIQUE | Stage name: Extraction, Processing, Smelting, Refining. |
| stage_order | SMALLINT | NOT NULL, UNIQUE | Sequential order for processing flow. Check: stage_order > 0. |

### mineral_batches table
Central registry map for inventory records.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| batch_id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| created_by | UUID | NOT NULL, FK | References users(user_id). |
| batch_code | VARCHAR(100) | NOT NULL, UNIQUE | System token formatting output: CU-[HEX]. |
| mineral_type | VARCHAR(100) | NOT NULL | Default: COPPER. Check constraint: mineral_type = 'COPPER'. |
| source_type | ENUM (source_type) | NOT NULL | Limits: SELF_EXTRACTED, PURCHASED. |
| quantity_tonnes | NUMERIC(18,6) | NOT NULL | Tonnage bounds rule: >= 0. |
| grade | NUMERIC(8,6) | NOT NULL | Fractional bounds rule: >= 0.0 AND <= 1.0. |
| recovery | NUMERIC(8,6) | NULLABLE | Expected yield limit fraction: >= 0.0 AND <= 1.0. |
| stage_id | SMALLINT | NOT NULL, FK | References processing_stages(stage_id). |
| status | ENUM (batch_status) | NOT NULL | Default: ACTIVE. Limits: ACTIVE, CLOSED. |
| assay_file_data | BYTEA | NULLABLE | Uploaded assay document binary data. |
| assay_original_filename | VARCHAR(255) | NULLABLE | Original filename of uploaded assay document. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### batch_history table
Tracks stage-by-stage progression snapshots for each mineral batch.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| history_id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| batch_id | UUID | NOT NULL, FK | References mineral_batches(batch_id). |
| current_stage_id | SMALLINT | NOT NULL, FK | References processing_stages(stage_id). |
| recorded_by | UUID | NOT NULL, FK | References users(user_id). |
| quantity_tonnes | NUMERIC(18,6) | NULLABLE | Tonnage bounds rule: >= 0 if not null. |
| grade | NUMERIC(8,6) | NULLABLE | Fractional bounds rule: >= 0.0 AND <= 1.0 if not null. |
| recovery | NUMERIC(8,6) | NULLABLE | Fractional bounds rule: >= 0.0 AND <= 1.0 if not null. |
| reason | TEXT | NULLABLE | Reason for batch movement. |
| occurred_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Time the event occurred. |
| recorded_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### value_calculations table
Stores historical snapshots of metal valuation transactions.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| calculation_id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| batch_id | UUID | NOT NULL, FK | References mineral_batches(batch_id). |
| batch_history_id | UUID | NOT NULL, FK | References batch_history(history_id). |
| stage_id | SMALLINT | NOT NULL, FK | References processing_stages(stage_id). |
| calculated_by | UUID | NOT NULL, FK | References users(user_id). |
| quantity | NUMERIC(18,6) | NOT NULL | Quantity used in calculation. |
| quantity_unit | VARCHAR(20) | NOT NULL | Unit of measure (e.g., tonnes). |
| grade | NUMERIC(8,6) | NOT NULL | Grade used in calculation. |
| recovery | NUMERIC(8,6) | NULLABLE | Recovery used in calculation. |
| reference_price | NUMERIC(18,6) | NOT NULL | LME reference price used (USD/tonne). |
| current_price | NUMERIC(18,6) | NOT NULL | Current price used (USD/tonne). |
| price_unit | VARCHAR(20) | NOT NULL | Price unit (e.g., USD/tonne). |
| currency | VARCHAR(10) | NOT NULL | Currency code (e.g., USD). |
| price_source | VARCHAR(100) | NOT NULL | Price source (e.g., Metals-API (LME copper)). |
| price_date | DATE | NOT NULL | Date the price was fetched for. |
| contained_metal_tonnes | NUMERIC(18,6) | NOT NULL | Computed: quantity × grade. |
| recoverable_metal_tonnes | NUMERIC(18,6) | NOT NULL | Computed: contained × recovery or contained. |
| calculated_value | NUMERIC(24,6) | NOT NULL | Computed: recoverable_metal_tonnes × reference_price. |
| calculated_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### scenarios table
Stores what-if analysis scenarios.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| scenario_id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| batch_id | UUID | NOT NULL, FK | References mineral_batches(batch_id). |
| created_by | UUID | NOT NULL, FK | References users(user_id). |
| assumed_stage_id | SMALLINT | NOT NULL, FK | References processing_stages(stage_id). |
| name | VARCHAR(150) | NOT NULL | Scenario name. |
| description | TEXT | NULLABLE | Scenario description. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### scenario_results table
Stores computed results for scenarios.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| result_id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| scenario_id | UUID | NOT NULL, FK | References scenarios(scenario_id). |
| total_value | NUMERIC(24,6) | NOT NULL | Total scenario value. |
| contained_metal_tonnes | NUMERIC(18,6) | NOT NULL | Contained metal in scenario. |
| recoverable_metal_tonnes | NUMERIC(18,6) | NOT NULL | Recoverable metal in scenario. |
| value_lost | NUMERIC(24,6) | NOT NULL | Value lost in scenario. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### mfa_tokens table
Stores MFA verification tokens.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| user_id | UUID | NOT NULL, FK | References users(user_id). |
| token_hash | VARCHAR(255) | NOT NULL | Hashed MFA token. |
| expires_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Token expiration time. |
| used_at | TIMESTAMP WITH TIME ZONE | NULLABLE | Timestamp when token was used. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### password_reset_tokens table
Stores password reset tokens.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| user_id | UUID | NOT NULL, FK | References users(user_id). |
| token_hash | VARCHAR(255) | NOT NULL | Hashed reset token. |
| expires_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Token expiration time. |
| used_at | TIMESTAMP WITH TIME ZONE | NULLABLE | Timestamp when token was used. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

### audit_logs table
Stores audit trail events.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| user_id | UUID | NULLABLE, FK | References users(user_id). |
| action | VARCHAR(100) | NOT NULL | Action performed. |
| resource | VARCHAR(100) | NOT NULL | Resource affected. |
| ip_address | VARCHAR(45) | NULLABLE | Client IP address. |
| user_agent | TEXT | NULLABLE | Client user agent. |
| details | JSON | NULLABLE | Additional event details. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |

## Enums

| Enum Name | Values |
| :--- | :--- |
| user_role | SYSTEM_ADMINISTRATOR, SUPPLY_CHAIN_MANAGER, OPERATIONS_MANAGER |
| account_status | ACTIVE, DISABLED |
| source_type | SELF_EXTRACTED, PURCHASED |
| batch_status | ACTIVE, CLOSED |

## Check Constraints

| Table | Constraint | Rule |
| :--- | :--- | :--- |
| mineral_batches | ck_batches_mineral_copper | mineral_type = 'COPPER' |
| mineral_batches | ck_batches_quantity_nonneg | quantity_tonnes >= 0 |
| mineral_batches | ck_batches_grade_range | grade >= 0 AND grade <= 1 |
| mineral_batches | ck_batches_recovery_range | recovery IS NULL OR (recovery >= 0 AND recovery <= 1) |
| batch_history | ck_history_quantity_nonneg | quantity_tonnes IS NULL OR quantity_tonnes >= 0 |
| batch_history | ck_history_grade_range | grade IS NULL OR (grade >= 0 AND grade <= 1) |
| batch_history | ck_history_recovery_range | recovery IS NULL OR (recovery >= 0 AND recovery <= 1) |
| processing_stages | ck_processing_stages_order_positive | stage_order > 0 |

### scenario_decisions table
Stores rule-based decision support assessments for scenario results.

| Column Name | Storage Type | Modifiers | Constraints / Defaults |
| :--- | :--- | :--- | :--- |
| id | UUID | PRIMARY KEY | Generated via uuid.uuid4(). |
| scenario_result_id | UUID | NOT NULL, UNIQUE, FK, INDEX | References scenario_results(result_id). ON DELETE CASCADE. |
| lifecycle_state | VARCHAR(20) | NOT NULL | Default: CALCULATED. Limits: CALCULATED, ASSESSED, BLOCKED. |
| assessment_outcome | VARCHAR(32) | NOT NULL | Default: INSUFFICIENT_INFORMATION. Limits: VIABLE, CONDITIONALLY_VIABLE, NOT_VIABLE, INSUFFICIENT_INFORMATION. |
| recommendation | VARCHAR(40) | NOT NULL | Default: REVIEW_REQUIRED. Limits: PROCEED, PROCEED_AFTER_VALIDATION, REVISE_SCENARIO, REVIEW_REQUIRED. |
| assessment_summary | TEXT | NOT NULL | Default: "". Summary of the assessment. |
| findings | JSON | NOT NULL | Default: []. List of rule evaluation findings. |
| recommendations | JSON | NOT NULL | Default: []. List of actionable recommendations. |
| assessed_by | VARCHAR(100) | NOT NULL | Default: SYSTEM. Identifier of assessor. |
| assessment_method | VARCHAR(40) | NOT NULL | Default: RULE_BASED_V1. Assessment engine version/method. |
| assessed_at | TIMESTAMP WITH TIME ZONE | NULLABLE | Timestamp when assessment was completed. |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). |
| updated_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Default: NOW(). Auto-updated on changes. |



## Database Setup & Management
- **Migration Tool:** Alembic is used for database schema migrations
- **Database Type:** PostgreSQL
- **Management Tool:** DBeaver (used for database inspection, querying, and management)
- **Migrations:** Managed through Alembic in the backend project


# Database Migrations & Data Seeding

Database versions, migrations, and structural modifications are managed through **Alembic**.

##  Executing Migration Routines

To provision or upgrade your active local database layer schema to the latest configuration target, execute:

```bash
# Run every migration patch version straight to the head version
alembic upgrade head
```

##  Provisioning Seed Data

Your repository provides automated seeding options to populate your local database workspace with realistic data layers (including multi-tenant companies, matching user roles, copper batch entries, and historical processing stages):

=== "Option A: Python Execution"
    Run the standalone database seed pipeline script directly inside your active environment:
    ```bash
    # Seed users, batches, and timeline data
    python scripts/seed_demo_data.py
    ```

=== "Option B: Direct SQL Injection"
    Inject the raw transaction script directly into your target PostgreSQL instance container:
    ```bash
    # Feed SQL transactions straight to your database url destination
    psql "$DATABASE_URL" -f scripts/seed_demo_data.sql
    ```

=== "Provisioning Admin Session Access"
    The seed dataset purposefully excludes superuser credentials. Generate your localized administrator profile separately:
    ```bash
    # Inject initial system administrator account profile directly
    python scripts/create_admin.py admin@example.com AdminPassword123!
    ```


## Database Usage
- **Database Type:** PostgreSQL
- **Management Tool:** DBeaver (used for database inspection, querying, and management)
- **Migrations:** Managed through the backend migration system

## Testing
Database-related functionality is tested as part of the backend test suite using pytest.

## Deployment
The PostgreSQL database is hosted as part of the backend deployment on **Heroku** (Heroku Postgres addon).

## Git Workflow & Collaboration
1. **Start from main**: `git checkout main && git pull origin main`
2. **Create feature branch**: `git checkout -b feature/your-feature-name`
3. **Make changes and test**: Update schemas/migrations, test locally
4. **Stage and commit**: `git add . && git commit -m "descriptive message"`
5. **Push**: `git push origin feature/your-feature-name`
6. **Create PR**: Open Pull Request on GitHub targeting `main`
7. **Review**: Team members review and provide feedback
8. **Resolve conflicts**: Pull latest `main`, resolve conflicts, commit, push
9. **Merge**: After approval, merge into `main`

**Branch Strategy:** `main` (production) and `develop` branches are used. New features are developed in `feature/*` branches off `develop` or `main` as per workflow. Always pull before creating a new branch and resolve conflicts when they arise.
