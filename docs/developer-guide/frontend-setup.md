# Frontend Workspace Onboarding Guide

Follow these steps to initialize your local development environment for the `heliks-frontend` React application layer.

##  System Prerequisites

Ensure your local host machine features these foundational system engines prior to onboarding:

* **Node.js Environment:** Version `18.x` or `20.x` (LTS recommended).
* **Package Manager:** `npm` (v10+) natively mapped.
* **Environment Variable Resolution:** Verify that no global parameters conflict with target listening port `3000`.

##  Execution Onboarding Sequence

### 1. Extract Dependencies and Initialize Workspace

Clone the frontend application layer repository, access the workspace root, and trigger node dependency lookups:

```bash
# Extract application layer dependencies from registry maps
npm install
```

### 2. Configure Local Environment Variables

Create a localized environment file named `.env.local` inside the frontend workspace root to configure proxy hooks:

```ini
# Backend API Base URL destination proxy anchor
BACKEND_URL="http://127.0.0.1:8000"
NEXT_PUBLIC_API_URL="http://127.0.0.1:8000"
```

### 3. Spin Up Local Development Runtime

Execute the Webpack development compilation engine to execute local testing:

```bash
# Starts local development runtime server listening on http://localhost:3000
npm run dev
```
