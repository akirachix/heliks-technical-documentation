# RICA Platform Overview

## 1. Platform Overview

RICA is a mineral supply chain traceability platform that brings together the web dashboard, backend API, and informational website.

The platform follows a client-server architecture where client applications communicate with the backend API, while the backend manages application logic, authentication, data processing, and communication with the PostgreSQL database.

The main platform components are:

- **RICA Web Dashboard** - Web-based interface used by Operations Managers, Supply Chain Managers, and System Administrators to manage mineral batches, track processing stages, perform value calculations, and monitor traceability.
- **RICA Backend API** - FastAPI-based backend responsible for application logic, data processing, authentication, role-based access control, value calculations using LME pricing, and communication with the database.
- **RICA Informational Website** - Public website that provides information about RICA, its purpose, features, and mineral supply chain traceability services.

These components communicate through APIs to provide a connected mineral traceability ecosystem.

## 2. Platform Components

### Web Dashboard
The RICA web dashboard provides role-specific functionality for:

- System Administrators
- Operations Managers
- Supply Chain Managers

The dashboard provides functionality including:

- User authentication and authorization
- Company and user management (System Administrators)
- Mineral batch creation and tracking
- Processing stage management
- Value calculations with LME copper pricing
- QR code generation and scanning
- Traceability views
- Scenario planning and analysis
- Audit logging
- Account settings

The web dashboard communicates with the RICA backend through the API layer.

### Backend API
The RICA backend provides the central application services used by the web dashboard.

The backend is implemented using FastAPI and provides functionality for:

- User management and authentication
- Role-based access control (RBAC)
- Company-level data scoping
- Mineral batch management
- Processing stage tracking and history
- Value calculations and pricing
- QR code generation
- Scenario planning
- Audit logging
- MFA and password reset
- Rate limiting
- API security and request handling

The backend communicates with the PostgreSQL database for persistent data storage.

### Informational Website
The RICA informational website provides publicly accessible information about the RICA platform, its purpose, features, and mineral supply chain traceability services.

The website serves as the public-facing information resource for RICA and is separate from the authenticated web dashboard.

[Visit the RICA Informational Website](https://heliks-informational-website.vercel.app)

## 3. User Roles

RICA supports three distinct user roles, each with specific permissions and access levels.

### Operations Manager (OPERATIONS_MANAGER)
Operations Managers handle day-to-day batch operations and processing.

They can:

- Create and manage mineral batches
- Track batches through processing stages
- Update batch information
- View batches within their company
- Generate QR codes for batches

### Supply Chain Manager (SUPPLY_CHAIN_MANAGER)
Supply Chain Managers focus on value analysis and pricing.

They can:

- Perform value calculations
- Access value analysis data
- View current and historical LME copper prices
- Run scenario planning
- Access value tables and reports
- View batches within their company

### System Administrator (SYSTEM_ADMINISTRATOR)
System Administrators manage and monitor the RICA platform.

They can:

- Manage all users across the platform
- Manage companies
- Enable/disable user accounts
- View all batches across companies (if no company assigned)
- Access audit logs
- Manage system settings

## 4. Core Platform Modules

The RICA platform is organized around several functional modules.

| Module | Description |
|---|---|
| Authentication | Handles user authentication, JWT tokens, MFA, and password reset |
| User Management | Manages RICA users, roles, and company associations |
| Company Management | Manages companies and their associated users |
| Mineral Batches | Manages copper batch registration, tracking, and ownership |
| Processing Stages | Manages workflow stages (Extraction, Processing, Smelting, Refining) |
| Batch History | Maintains immutable stage-by-stage snapshots for traceability |
| Value Calculations | Computes recoverable metal values using LME copper prices |
| QR Codes | Generates and provides QR codes for batch traceability |
| Scenarios | Provides what-if analysis and scenario planning |
| Audit Logging | Tracks and logs critical system actions |
## 5. Client-to-API Communication

The web dashboard communicates with the backend through HTTP API requests.

The general request flow is:

```
Web Dashboard
       │
       │ HTTP Request (Authorization: Bearer Token)
       │
   RICA Backend API
       │
       ├── Authentication & Authorization
       │
       ├── Validation (Pydantic)
       │
       ├── Business Logic
       │
       └── Database Operation
              │
          PostgreSQL
              │
       Backend Response
              │
       Web Dashboard
```

This architecture allows the client application to access centralized application functionality without directly communicating with the database.

## 6. Data Persistence

RICA uses PostgreSQL as its primary database.

Persistent application data includes information associated with:

- Users and roles
- Companies
- Mineral batches
- Processing stages
- Batch history
- Value calculations
- Prices (LME copper)
- Scenarios and scenario results
- MFA tokens and password reset tokens
- Audit logs

The backend is responsible for interacting with the database and exposing the required information through API endpoints.

The web dashboard does not directly access the PostgreSQL database.

## 7. API Integration

The web dashboard is integrated with the RICA backend API.

API integration allows the dashboard to:

- Authenticate users and manage sessions
- Create, read, update mineral batches
- Track processing stages
- Perform value calculations
- Retrieve current and historical pricing
- Generate and retrieve QR codes
- Run scenario analyses
- Manage users and companies (admin)
- Retrieve audit logs
- Access traceability information

The API serves as the primary communication layer between the web dashboard and backend services.

## 8. Platform Architecture

At a high level, RICA follows a client-server architecture consisting of:

### Presentation Layer
- **RICA Web Dashboard** - Role-based web interface
- **RICA Informational Website** - Public website

### Application Layer
- **RICA FastAPI Backend** - Core application logic
  - API routers
  - Services (business logic)
  - Repositories (data access)
  - Schemas (validation)
  - Core utilities (auth, config, security)
  - Middleware (audit, CORS)

### Data Layer
- **PostgreSQL Database** - Primary data store with relational schema

The separation of these layers allows the different parts of the platform to be developed and maintained independently while communicating through defined APIs.
