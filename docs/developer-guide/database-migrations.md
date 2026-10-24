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
