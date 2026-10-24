# Glossary & Reference Specifications

This glossary maps all technical terminology, framework tools, and mineral math equations used across the Heliks ecosystem.

| Term | Meaning |
| :--- | :--- |
| Heliks | The overarching software platform suite, consisting of the multi-tenant web application and the informational marketing portal. |
| RICA | Mineral Tracking, Value-Chain Analysis, and Scenario Modelling System - The specialized backend engine built to track and optimize copper value extraction loops in Zambia. |
| Group 5 Team | The group of 5 engineers who built the RICA platform: Samantha, Rose, Mailisha, Salvatrice, and Eyerusalem. |
| Operations Manager | An authenticated user role responsible for tracking physical mineral batches and stage chronology transitions. |
| Supply Chain Manager | An authenticated user role with exclusive authorization to access financial calculations and value-capture analytics. |
| System Administrator | An administrative user role with global privileges to register corporate companies, manage accounts, and modify user status filters. |
| Mineral Batch | A discrete batch of copper tracked through the supply chain, containing exact mass weight, grade, and recovery parameters. |
| Batch Code | A unique, system-generated alphanumeric identification string prefixed with a standard format: CU-[HEX]. |
| Contained Metal | The absolute theoretical weight mass of copper present inside an inventory batch. Formula: $\text{Quantity} \times \text{Grade}$ |
| Recoverable Metal | The estimated weight yield of copper expected after factoring node losses: $\text{Contained Metal} \times \text{Recovery}$ |
| Calculated Value | The final localized market valuation snapshot computed as: $\text{Recoverable Metal Tonnes} \times \text{Reference LME Spot Price}$ |
| Value Lost | The calculated financial deficit or revenue leakage resulting from stage processing inefficiencies: $(\text{Contained} - \text{Recoverable}) \times \text{Price}$ |
| Scenario Simulation | A speculative "what-if" workspace sandboxed from active tables, used to model prospective pricing or operational shifts. |
| Immutability Lock | The data security mechanism that permanently freezes a scenario's input parameters the instant its first child result is recorded. |
| Company Scoping | A row-level security isolation rule where users only see data rows matching their active company_name variable. |
| Assay Document | A physical laboratory certificate uploaded as a binary sequence to verify batch chemical grade parameters. |
| FastAPI | The asynchronous Python framework used to power the core RICA REST routing API paths and calculation handlers. |
| Next.js | The React-based web framework utilized to compile the multi-tenant Heliks operational web app and marketing portal. |
| PostgreSQL | The primary relational open-source database engine used to store persistent records, audit trails, and transactional snapshots. |
| SQLAlchemy | The Python SQL toolkit and Object Relational Mapper used by the backend to communicate cleanly with PostgreSQL. |
| Alembic | The database schema evolution engine used to track, provision, and deploy migration patch versions. |
| Pydantic | The data validation library used to enforce strict server-side formatting checks over incoming JSON payloads. |
| Redis | An in-memory data store cache utilized exclusively by backend security middleware to run endpoint rate-limiting rules. |
| JWT | JSON Web Token - A signed token stored in localStorage to authorize user sessions across protected routes. |
| MFA / TOTP | Multi-Factor Authentication - A secondary security gate using encrypted TOTP keys to protect account profiles. |
| Metals-API | The external third-party pricing provider integration used by the value calculation engine to pull live reference spot copper feeds. |
| Ruff | The automated code-quality scanning linter and formatting checker utilized inside the development workflow. |
| Figma | The UX/UI prototyping workspace tool used to design frontend interface components and match screens to backend APIs. |
