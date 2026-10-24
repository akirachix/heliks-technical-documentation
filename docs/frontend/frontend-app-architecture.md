# Heliks Frontend Documentation

## 1. Overview

Heliks is a web-based mineral supply chain management platform frontend built with Next.js 16 and React 19. It serves as the client interface for three distinct user roles, each with role-specific functionality and access controls. The frontend communicates with the Heliks backend via RESTful API calls proxied through Next.js rewrites, providing a secure and role-based user experience.

The application manages the complete mineral supply chain workflow including:

- Company and user administration
- Mineral batch tracking and management
- Processing stage monitoring
- Value calculation and analysis
- Scenario planning and results
- User account management across all roles

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Client Layer (Browser)                          │
├─────────────────────────────────────────────────────────────────────────┤
│  System Admin    │  Operations Manager  │  Supply Chain Manager        │
│  (/admin/*)      │  (/(main)/*, /home)  │  (/value, /supply_chain_*)   │
└────────────────────────────┬────────────────────────────────────────────┘
                             │ HTTP/HTTPS Requests
                             │ (Authorization: Bearer Token)
                             ▼
                    ┌─────────────────────────┐
                    │   Next.js Frontend      │
                    │   (App Router, SSR/CSR) │
                    │   - Route Guards        │
                    │   - API Proxy/Rewrites  │
                    └─────────────┬───────────┘
                                  │ Proxied API Calls
                                  ▼
                        ┌─────────────────────────┐
                        │    Heliks Backend API   │
                        │      (RESTful API)      │
                        └─────────────┬───────────┘
                                      │ SQLAlchemy ORM
                                      ▼
                              ┌─────────────────┐
                              │   PostgreSQL DB │
                              └─────────────────┘
```

## 2. Technology Stack

### Core Framework

| Technology | Version | Purpose |
|---|---|---|
| Next.js | 16.4.2 | React framework with App Router, server components, and API rewrites |
| React | 19.0.0 | Component-based UI library with latest features |
| TypeScript | ^5.7.2 | Static type checking and enhanced developer experience |

### UI & Styling

| Technology | Version | Purpose |
|---|---|---|
| Tailwind CSS | ^4.3.3 | Utility-first CSS framework |
| CSS Modules | Built-in | Component-scoped styling |
| @fontsource/instrument-sans | ^5.3.0 | Custom font family |
| Lucide React | ^1.61.0 | Modern icon library |

### Forms & Utilities

| Technology | Version | Purpose |
|---|---|---|
| React Hook Form | ^7.90.0 | Performant form handling with validation |
| PostCSS | Via Next.js | CSS processing |
| @tailwindcss/postcss | ^4.3.3 | Tailwind PostCSS integration |

### Development & Testing

| Technology | Version | Purpose |
|---|---|---|
| ESLint | ^9.17.0 | Code linting with Next.js config |
| Prettier | ^3.4.2 | Code formatting |
| Husky | ^9.1.7 | Git hooks |
| lint-staged | ^16.2.4 | Pre-commit linting |
| Playwright | ^1.49.0 | End-to-end testing framework |

## 3. Architecture

The Heliks frontend follows a modern Next.js App Router architecture with clear separation of concerns:

### Architectural Principles

- **Component-Based Architecture**: Reusable, composable React components
- **Route-Based Code Splitting**: Automatic code splitting per route
- **Role-Based Access Control (RBAC)**: Granular permissions per user role
- **API Abstraction**: Centralized API client for all backend communication
- **Type Safety**: Full TypeScript coverage
- **Responsive Design**: Mobile-first approach with CSS breakpoints
- **Progressive Enhancement**: Works across modern browsers

### Application Flow

1. User visits application
2. Check authentication token in localStorage
3. If not authenticated → redirect to /login
4. If authenticated → fetch user profile (/users/me)
5. Determine user role → route to appropriate dashboard
6. Apply role-based guards on protected routes
7. Render role-specific UI with appropriate navigation

## 4. User Roles and Access Control

The application supports three distinct user roles, each with specific permissions and interfaces:

### 4.1 System Administrator (SYSTEM_ADMINISTRATOR)

**Access Level:** Full system administration

**Permissions:**

- Manage all companies (view, register new companies)
- Manage all users across the platform (view, enable/disable users)
- Access admin account settings
- View system-wide data

**Default Route:** /admin/companies (as per login routing logic)

**Key Features:**

- Company registry with search functionality
- User management table with role and status filters
- Ability to enable/disable user accounts
- Company registration with supply chain manager assignment

### 4.2 Operations Manager (OPERATIONS_MANAGER)

**Access Level:** Operations and batch management

**Permissions:**

- Manage mineral batches
- Track batch processing stages
- View operations dashboard
- Access settings

**Default Route:** /home

**Key Features:**

- Mineral batch creation and tracking
- Processing stage management
- Batch status monitoring
- Operations dashboard with relevant metrics

### 4.3 Supply Chain Manager (SUPPLY_CHAIN_MANAGER)

**Access Level:** Value analysis and supply chain metrics

**Permissions:**

- Perform value calculations
- View value analysis data
- Access value dashboard
- Manage supply chain settings

**Default Route:** /value or /supply_chain_manager/value_caclulations

**Key Features:**

- Value calculation interface
- Batch value analysis
- Price data input
- Export functionality
- Mobile-responsive value tables

### 4.4 Access Control Implementation

**Route Guards**

- `RequireAuth` (components/RequireAuth.tsx): Wrapper component ensuring user authentication
- `OperationsManagerGuard` (components/OperationsManagerGuard.tsx): Restricts access to operations managers
- `SystemAdminGuard` (components/SystemAdminGuard.tsx): Restricts access to system administrators

**Login Routing Logic**

In `app/login/page.tsx`, after successful authentication:

```typescript
if (role === "SYSTEM_ADMINISTRATOR") router.push("/admin/companies");
else if (role === "OPERATIONS_MANAGER") router.push("/home");
else if (role === "SUPPLY_CHAIN_MANAGER") router.push("/value");
```

All active registered roles are granted access to their respective designated pages.

## 5. Project Structure

```
heliks-frontend/
│
├── app/                              # Next.js App Router (Pages & Layouts)
│   ├── layout.tsx                    # Root layout with fonts, global styles
│   ├── globals.css                   # Global CSS and CSS variables
│   ├── page.tsx                      # Root page (redirects)
│   ├── page.module.css               # Root page styles
│   ├── favicon.ico                   # Favicon
│   │
│   ├── (main)/                       # Route group for authenticated main routes
│   │   ├── layout.tsx                # Main layout with Sidebar
│   │   ├── home/                     # Operations Manager dashboard
│   │   ├── batches/                  # Mineral batch management
│   │   ├── batch-stages/             # Processing stage tracking
│   │   └── settings/                 # User settings
│   │
│   ├── admin/                        # System Administrator routes
│   │   ├── home/                     # Admin dashboard
│   │   ├── companies/                # Company management
│   │   ├── users/                    # User management
│   │   └── account/                  # Admin account settings
│   │
│   ├── supply_chain_manager/         # Supply Chain Manager routes
│   │   ├── settings/                 # SCM settings
│   │   └── value_caclulations/       # Value calculation
│   │
│   ├── login/                        # Authentication
│   ├── forgot-password/              # Password recovery
│   ├── reset-password/               # Password reset
│   ├── reset-verify/                 # Reset verification
│   ├── activate/                     # Account activation
│   ├── verify/                       # Email verification
│   └── value/                        # Alternative value route
│
├── components/                       # Reusable UI components
│   ├── Sidebar.tsx                   # Main navigation sidebar
│   ├── AdminSidebar.tsx              # Admin navigation sidebar
│   ├── RequireAuth.tsx               # Authentication guard
│   ├── OperationsManagerGuard.tsx    # OM role guard
│   └── SystemAdminGuard.tsx          # Admin role guard
│
├── lib/                              # Utilities and helper functions
│   ├── api.ts                        # API client with auth handling
│   ├── admin.ts                      # Admin-specific utilities
│   └── useSessionValue.ts            # Session state hook
│
├── tests/                            # Testing
│   └── e2e/                          # Playwright E2E tests
│
├── public/                           # Static assets
├── next.config.ts                    # Next.js configuration
├── tsconfig.json                     # TypeScript configuration
├── package.json                      # Dependencies and scripts
└── playwright.config.ts              # Playwright configuration
```

## 6. Routing Architecture

The application uses Next.js App Router with a combination of route groups and dynamic routing.

### Public Routes (No Authentication Required)

| Route | Purpose | Description |
|---|---|---|
| `/login` | User login | Email/password authentication |
| `/forgot-password` | Password recovery | Initiate password reset |
| `/reset-password` | Password reset | Set new password |
| `/reset-verify` | Reset verification | Verify reset token |
| `/activate` | Account activation | Activate new account |
| `/verify` | Email verification | Verify email address |

### Protected Routes - System Administrator

| Route | Purpose | Access |
|---|---|---|
| `/admin/home` | Admin dashboard | SYSTEM_ADMINISTRATOR |
| `/admin/companies` | Company listing | SYSTEM_ADMINISTRATOR |
| `/admin/companies/new` | Register company | SYSTEM_ADMINISTRATOR |
| `/admin/users` | User management | SYSTEM_ADMINISTRATOR |
| `/admin/account` | Admin account settings | SYSTEM_ADMINISTRATOR |

### Protected Routes - Operations Manager

| Route | Purpose | Access |
|---|---|---|
| `/home` | Operations dashboard | OPERATIONS_MANAGER |
| `/batches` | Mineral batch management | OPERATIONS_MANAGER |
| `/batch-stages` | Processing stage tracking | OPERATIONS_MANAGER |
| `/settings` | User settings | OPERATIONS_MANAGER |

### Protected Routes - Supply Chain Manager

| Route | Purpose | Access |
|---|---|---|
| `/value` | Value calculation dashboard | SUPPLY_CHAIN_MANAGER |
| `/supply_chain_manager/value_caclulations` | Value calculations (alt route) | SUPPLY_CHAIN_MANAGER |
| `/supply_chain_manager/settings` | SCM settings | SUPPLY_CHAIN_MANAGER |

**Route Groups:** `(main)` groups operations manager routes under a shared layout with Sidebar.

## 7. Authentication and Session Management

### Token Management

- **Storage:** JWT token stored in localStorage as `rica_token`
- **Injection:** Automatically added to all API requests via `Authorization: Bearer <token>` header in `lib/api.ts`
- **Persistence:** Token persists across browser sessions

### Authentication Flow

1. User submits credentials
2. POST to `/auth/login` (proxied to backend)
3. Receive token and user data
4. Store token in localStorage
5. Fetch user details from `/users/me`
6. Route based on user role

### Session Hooks

`useSessionValue` (lib/useSessionValue.ts): Custom hook for managing session-related data

### Protected Route Implementation

Components use guards like `RequireAuth` to protect routes:

```tsx
<RequireAuth>
  <div className={styles.layout}>
    <Sidebar />
    {/* page content */}
  </div>
</RequireAuth>
```

## 8. API Communication

### API Client (lib/api.ts)

The centralized API client provides typed methods for all HTTP operations:

```typescript
export async function apiGet<T>(path: string): Promise<T>
export async function apiPost<T>(path: string, body?: unknown): Promise<T>
export async function apiPut<T>(path: string, body?: unknown): Promise<T>
export async function apiPatch<T>(path: string, body?: unknown): Promise<T>
export async function apiDelete<T>(path: string): Promise<T>
```

**Features:**

- Automatic Authentication: Injects `rica_token` from localStorage into headers
- Error Handling: Custom `ApiError` class with status codes
- JSON Serialization: Automatic JSON handling for requests/responses
- Type Safety: Generic type parameters for response typing
- 401 Handling: Components handle 401 responses (often redirect to login)

### Next.js Rewrites (API Proxying)

Configured in `next.config.ts` to proxy API requests to backend:

```typescript
rewrites() {
  return [
    { source: "/auth/:path*", destination: `${BACKEND_URL}/auth/:path*` },
    { source: "/users/:path*", destination: `${BACKEND_URL}/users/:path*` },
    { source: "/mineral-batches/:path*", destination: `${BACKEND_URL}/mineral-batches/:path*` },
    { source: "/values/:path*", destination: `${BACKEND_URL}/values/:path*` },
    { source: "/processing-stages/:path*", destination: `${BACKEND_URL}/processing-stages/:path*` },
    { source: "/scenarios/:path*", destination: `${BACKEND_URL}/scenarios/:path*` },
  ];
}
```

### Common API Patterns

- GET requests for data fetching
- POST for resource creation
- PATCH for partial updates (e.g., enable/disable users)
- PUT for full updates
- DELETE for resource deletion

## 9. Component Architecture

### Layout Components

| Component | Location | Purpose |
|---|---|---|
| RootLayout | app/layout.tsx | Global layout with fonts (Instrument Sans), metadata, global styles |
| MainLayout | app/(main)/layout.tsx | Layout for Operations Manager with Sidebar |
| Sidebar | components/Sidebar.tsx | Main navigation for OM and SCM |
| AdminSidebar | components/AdminSidebar.tsx | Navigation for System Administrator |

### Guard Components

| Component | Purpose | Usage |
|---|---|---|
| RequireAuth | Ensures user is authenticated before rendering | Wraps protected pages |
| OperationsManagerGuard | Restricts access to OPERATIONS_MANAGER role | Protects OM-specific routes |
| SystemAdminGuard | Restricts access to SYSTEM_ADMINISTRATOR role | Protects admin routes |

### Page Components

All pages are React Server Components by default (Next.js App Router), with `"use client"` directive added where needed for interactivity.

## 10. State Management

### Local State

- `useState`: Component-level state (forms, UI toggles, loading states)
- `useReducer`: Complex state logic where needed
- `useMemo`: Memoized computed values (filtering, sorting)
- `useCallback`: Memoized callback functions

### Data Fetching Pattern

```typescript
const [data, setData] = useState([]);
const [loading, setLoading] = useState(true);
const [error, setError] = useState("");

useEffect(() => {
  fetchData()
    .then(setData)
    .catch(handleError)
    .finally(() => setLoading(false));
}, []);
```

### Session State

- `localStorage`: Stores authentication token
- `useSessionValue`: Custom hook for session-related values
- User profile: Fetched on app load and stored in component state

## 11. Styling Architecture

### CSS Strategy

- **Tailwind CSS v4:** Utility classes for rapid styling
- **CSS Modules:** Component-scoped styles for complex components
- **Global CSS:** CSS variables and base styles in `app/globals.css`

### CSS Variables

```css
:root {
  --navy: #262262;
  --sidebar-bg: #25205f;
  --teal: #0ac297;
  --green: #0ac297;
  --muted: #333333;
  --text: #111111;
  --error: #d93025;
}
```

### Responsive Design

- Mobile-first approach: Base styles for mobile, enhanced for larger screens
- Breakpoints: Standard CSS media queries (e.g., 375px for mobile overflow fixes)
- Flexible layouts: CSS Grid and Flexbox
- Horizontal scrolling: Added for tables on mobile devices

### Design System

- **Primary Color:** Navy/Indigo (#25225f, #26215f)
- **Accent Color:** Teal/Green (#0cc39b, #0ac297)
- **Typography:** Instrument Sans font family
- **Spacing:** Consistent padding/margin scale
- **Border Radius:** Rounded corners for modern UI
- **Shadows:** Subtle shadows for depth

## 12. Forms and Validation

### React Hook Form

Used throughout the application for form management:

- **Performance:** Uncontrolled components for better performance
- **Validation:** Built-in and custom validation
- **Error Handling:** Easy error display
- **Type Safety:** TypeScript integration

### Validation Patterns

- Client-side validation using React Hook Form
- Server-side validation (backend responsibility)
- Error messages displayed inline
- Accessible form labels and inputs

## 13. Data Flow

### User Management Flow

1. Admin navigates to `/admin/users`
2. UsersPage component mounts
3. `useEffect` triggers `fetchUsers()` and `apiGet('/users/me')`
4. API calls go through `lib/api.ts` with auth token
5. Next.js rewrites proxy to backend
6. Backend returns user data
7. Component updates state and renders table
8. Admin clicks "Disable" button
9. `toggle()` calls `apiPatch('/users/{id}/disable')`
10. Optimistically update UI or refetch
11. Display updated status

### Value Calculation Flow

1. SCM navigates to value calculation page
2. Fetch user profile and company info
3. Fetch mineral batches filtered by company (defensive filtering)
4. User selects batch
5. Fetch value calculations for batch
6. Handle 404/empty states gracefully
7. Display calculations in table
8. Export functionality available

## 14. Environment Configuration

### Required Environment Variables

| Variable | Required | Description | Example |
|---|---|---|---|
| BACKEND_URL | Yes | Backend API base URL (no trailing slash) | https://api.example.com |
| NEXT_PUBLIC_API_URL | No | Public API URL if different | Same as BACKEND_URL |

### Configuration Files

- `.env.local`: Local development environment variables (gitignored)
- `next.config.ts`: Reads `BACKEND_URL` at build/runtime, enforces configuration
- `playwright.config.ts`: Test environment configuration

**Note:** The application requires `BACKEND_URL` to be configured. `next.config.ts` will use empty string if not set, which would break API rewrites.

## 15. Testing

### End-to-End Testing with Playwright

- **Configuration:** `playwright.config.ts`
- **Test Directory:** `./tests/e2e`
- **Browser:** Chromium (Desktop Chrome)
- **Base URL:** `http://localhost:3000` (or from `BASE_URL` env)
- **Web Server:** Automatically starts `npm run dev`
- **Parallel Execution:** Enabled
- **Retries:** 0 in dev, 2 in CI
- **Reporting:** HTML report

### Running Tests

```bash
npm run test:e2e      # Run all E2E tests
npm run test:e2e:ui   # Run in interactive UI mode
```

### Testing Approach

- **API Mocking:** Uses Playwright's route interception for isolated testing
- **Authentication Simulation:** Sets tokens via localStorage
- **Role-Based Testing:** Tests for each user role
- **Responsive Testing:** Viewport testing for mobile breakpoints
- **Navigation Testing:** Sidebar and routing verification

## 16. Build and Deployment

### Development

```bash
npm run dev
```
Starts Next.js dev server with webpack on port 3000, with hot reloading.

### Production Build

```bash
npm run build
```
Creates optimized production build with code splitting, minification, image optimization, static optimization, Cache Components enabled, and partial prefetching enabled.

### Production Start

```bash
npm run start
```
Starts production server on port 3000.

### Code Quality Scripts

```bash
npm run lint          # ESLint checking
npm run format        # Prettier formatting
npm run format:check  # Check formatting without changes
```

### Git Hooks

- **Husky:** Manages git hooks
- **lint-staged:** Runs linters on staged files (JS/TS/JSX/TSX files: ESLint + Prettier; CSS/JSON/MD/YAML/HTML files: Prettier)

### Deployment Considerations

- Set `BACKEND_URL` environment variable
- Ensure backend API is accessible
- Configure allowed dev origins if needed
- Use production build for deployment
- Consider CDN for static assets

## 17. Performance Considerations

### Next.js Optimizations

- **App Router:** Automatic code splitting and streaming
- **Cache Components:** Enabled (`cacheComponents: true`)
- **Partial Prefetching:** Enabled for faster navigation
- **Turbopack:** Used in dev mode
- **Webpack:** Used in dev with `--webpack` flag

### Frontend Optimizations

- **React 19:** Latest performance improvements
- **CSS Modules:** Scoped styles reduce CSS bloat
- **Component Lazy Loading:** Automatic via Next.js
- **Image Optimization:** Next.js Image component where used
- **Memoization:** useMemo and useCallback for expensive operations

### Bundle Optimization

- Tree shaking enabled
- Dynamic imports where appropriate
- Minimal dependencies

## 18. Security Considerations

### Authentication Security

- **JWT Tokens:** Stored in localStorage (consider httpOnly cookies for enhanced security)
- **Token Injection:** Automatic in API client
- **Route Protection:** Guards on all protected routes
- **Role-Based Access:** Enforced at component level

### API Security

- **Proxied Requests:** All API calls go through Next.js rewrites
- **Authorization Headers:** Bearer token on every request
- **401 Handling:** Redirect to login on unauthorized
- **CORS:** Handled by backend, proxied through Next.js

### Client-Side Security

- **No Direct DB Access:** All data through API
- **Environment Variables:** Sensitive values in `.env.local` (gitignored)
- **Input Validation:** React Hook Form validation
- **XSS Protection:** React's built-in escaping
- **Type Safety:** TypeScript prevents many runtime errors

### Best Practices

- Never commit `.env.local` or secrets
- Validate user permissions on both client and server
- Use HTTPS in production
- Sanitize user inputs
- Keep dependencies updated

## 19. Development Workflow

### Getting Started

```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your BACKEND_URL

# Run development server
npm run dev
```

### Development Guidelines

- **Use TypeScript:** All new files should be TypeScript
- **Follow Conventions:** Match existing code style
- **Component Structure:** Use functional components with hooks
- **Styling:** Prefer CSS Modules for component styles, Tailwind for utilities
- **API Calls:** Always use `lib/api.ts` client
- **Route Protection:** Wrap protected pages with appropriate guards
- **Forms:** Use React Hook Form for complex forms
- **Testing:** Write E2E tests for critical flows

### Git Workflow

- Feature branches from main/develop
- Descriptive commit messages
- Pre-commit hooks run automatically
- Pull requests for code review

## 20. Troubleshooting

### Common Issues

**Build Errors**

- **Issue:** BACKEND_URL not configured  
  **Solution:** Set BACKEND_URL in environment variables. Check `next.config.ts` - it requires this value.
- **Issue:** TypeScript errors  
  **Solution:** Run `npm run lint` to see errors, fix type issues

**Development Issues**

- **Issue:** API calls failing  
  **Solution:** Check BACKEND_URL configuration, ensure backend is running, check browser network tab
- **Issue:** Authentication not working  
  **Solution:** Check localStorage for `rica_token`, verify login flow, check API responses
- **Issue:** Route guards blocking access  
  **Solution:** Verify user role in `/users/me` response matches expected role for route

**Testing Issues**

- **Issue:** Playwright tests failing  
  **Solution:** Ensure dev server can start, check mocks in `helpers.ts`, verify selectors match UI
- **Issue:** Port already in use  
  **Solution:** Kill process on port 3000 or use different port

### Debugging Tips

- Use browser DevTools Network tab for API debugging
- Check React DevTools for component state
- Review Next.js console for server-side errors
- Use Playwright UI mode for test debugging
- Check localStorage for auth token and user data

**Document Version:** 1.0  
**Last Updated:** October 2024  
**Maintained By:** Heliks Development Team
