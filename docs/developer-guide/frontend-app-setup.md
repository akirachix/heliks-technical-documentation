# Heliks Web Application Onboarding Guide

Follow these steps to initialize your local development workspace for the multi-tenant React operations platform.

##  System Prerequisites
* **Node.js Environment:** Version `18.x` or `20.x` (LTS branches)
* **Package Manager:** Standard `npm` framework

##  Execution Onboarding Sequence

### 1. Extract Dependencies
Access the frontend directory containing `package.json` and initialize core node modules:
```bash
npm install
```

### 2. Configure Local Environment Variables
Create a localized environment file named `.env.local` inside the frontend workspace root to map variables:
```ini
BACKEND_URL="http://127.0.0.1:8000"
NEXT_PUBLIC_API_URL="http://127.0.0.1:8000"
```

### 3. Spin Up Local Development Runtime
```bash
npm run dev
```
