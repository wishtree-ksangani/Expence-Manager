# Phase 1 — MVP Task Breakdown

**Timeline:** Months 1–3  
**Goal:** Deliver a working Expense Manager with authentication, expense CRUD, default categories, budgets, a basic dashboard, and CSV export.

Each task below maps to one GitHub Issue. Labels and dependencies are noted.

---

## Epic 1: Project Setup & Infrastructure

### Issue 1.1 — Initialize monorepo structure
**Labels:** `setup`, `backend`, `frontend`  
**Description:**
- Create `frontend/` (React + TypeScript via Vite or CRA)
- Create `backend/` (Node.js + Express + TypeScript)
- Configure root-level `package.json` with workspaces or a `Makefile`
- Add `.gitignore`, `.editorconfig`, `.nvmrc`

### Issue 1.2 — Configure linting and formatting
**Labels:** `setup`, `chore`  
**Description:**
- ESLint + Prettier for both `frontend/` and `backend/`
- Husky pre-commit hooks to enforce lint on staged files
- Shared TypeScript `tsconfig.base.json`

### Issue 1.3 — Docker Compose for local development
**Labels:** `setup`, `devops`  
**Description:**
- `docker-compose.yml` with services: `postgres`, `redis`, `backend`, `frontend`
- Environment variable setup via `.env.example`
- Health checks for postgres and redis

### Issue 1.4 — Database schema and migration setup
**Labels:** `setup`, `backend`, `database`  
**Description:**
- Choose and configure a migration tool (Knex.js recommended)
- Create initial migration files for Phase 1 entities:
  - `users`
  - `categories`
  - `expenses`
  - `budgets`
- Write a `seeds/` script to insert the 15+ default categories

### Issue 1.5 — CI/CD pipeline (GitHub Actions)
**Labels:** `setup`, `devops`  
**Description:**
- Workflow: lint → test → build on every PR
- Separate workflow for deployment (staging/production) on merge to `main`
- Environment secrets management

---

## Epic 2: Authentication Module

> Covers: AUTH-1, AUTH-2, AUTH-3, AUTH-4, AUTH-5 from PRD

### Issue 2.1 — User registration (email + password)
**Labels:** `feature`, `backend`, `auth`  
**Dependencies:** Issue 1.4  
**Description:**
- `POST /api/v1/auth/register` endpoint
- Input validation: email format, password strength (min 8 chars)
- Password hashing with bcrypt (salt rounds: 12)
- Return JWT access token + refresh token on success
- Store user in `users` table

### Issue 2.2 — User login (email + password)
**Labels:** `feature`, `backend`, `auth`  
**Dependencies:** Issue 2.1  
**Description:**
- `POST /api/v1/auth/login` endpoint
- Validate credentials, return JWT access + refresh tokens
- Lock account after 5 failed attempts (use Redis for attempt counter)

### Issue 2.3 — JWT session management (access + refresh tokens)
**Labels:** `feature`, `backend`, `auth`  
**Dependencies:** Issue 2.1  
**Description:**
- Access token: 15-minute expiry
- Refresh token: 7-day expiry, stored in `refresh_tokens` table
- `POST /api/v1/auth/refresh` to issue new access token
- `POST /api/v1/auth/logout` to blacklist refresh token
- Auth middleware to validate access token on protected routes

### Issue 2.4 — Password reset via email OTP
**Labels:** `feature`, `backend`, `auth`  
**Dependencies:** Issue 2.1  
**Description:**
- `POST /api/v1/auth/forgot-password` — generate 6-digit OTP, store in Redis with 15-min TTL, send via email (use SendGrid or Nodemailer)
- `POST /api/v1/auth/verify-otp` — validate OTP
- `POST /api/v1/auth/reset-password` — update password hash

### Issue 2.5 — Google OAuth integration
**Labels:** `feature`, `backend`, `auth`  
**Dependencies:** Issue 2.3  
**Description:**
- Configure Passport.js Google strategy (OAuth2)
- `GET /api/v1/auth/google` and `/api/v1/auth/google/callback`
- On success: create user if new, issue JWT tokens
- Link Google account to existing user if email matches

### Issue 2.6 — Role-based access control (RBAC)
**Labels:** `feature`, `backend`, `auth`  
**Dependencies:** Issue 2.3  
**Description:**
- Roles: `admin`, `member`, `viewer`
- Role stored on `users` table (`role` column)
- Middleware factory: `requireRole('admin')` etc.
- For Phase 1, most users are `member`; `admin` role reserved for future group approval flow

### Issue 2.7 — Frontend: Registration and Login pages
**Labels:** `feature`, `frontend`, `auth`  
**Dependencies:** Issue 2.1, 2.2  
**Description:**
- `/register` page: email, name, password, confirm password fields
- `/login` page: email, password, "Forgot password?" link, "Sign in with Google" button
- Form validation via `react-hook-form` + `zod`
- Store access token in memory; refresh token in `httpOnly` cookie
- Redirect to dashboard on success

### Issue 2.8 — Frontend: Forgot password flow
**Labels:** `feature`, `frontend`, `auth`  
**Dependencies:** Issue 2.4, 2.7  
**Description:**
- `/forgot-password` page: email input
- `/reset-password` page: OTP input + new password
- Show success/error states clearly

### Issue 2.9 — Frontend: Protected routes and auth context
**Labels:** `feature`, `frontend`, `auth`  
**Dependencies:** Issue 2.7  
**Description:**
- React Context (or Zustand) for auth state (user, token)
- `<PrivateRoute>` component — redirects to `/login` if unauthenticated
- Auto-refresh access token before expiry using refresh token

---

## Epic 3: Category Management

> Covers: CAT-1 from PRD (Phase 1 scope)

### Issue 3.1 — Seed default categories
**Labels:** `feature`, `backend`, `database`  
**Dependencies:** Issue 1.4  
**Description:**
- 15+ default categories with name, icon (emoji or icon slug), and color:
  - Food & Dining, Transport, Housing/Rent, Utilities, Healthcare, Entertainment, Shopping, Education, Travel, Personal Care, Fitness, Gifts & Donations, Business, Subscriptions, Other
- `GET /api/v1/categories` endpoint — returns all categories available to the user (defaults + user-created in Phase 2)

### Issue 3.2 — Frontend: Category picker component
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 3.1  
**Description:**
- Reusable `<CategoryPicker>` component for use in Add/Edit Expense form
- Display icon + color + name
- Searchable dropdown

---

## Epic 4: Expense Management

> Covers: EXP-1, EXP-3 from PRD (Phase 1 scope)

### Issue 4.1 — Backend: Expense CRUD API
**Labels:** `feature`, `backend`  
**Dependencies:** Issue 1.4, 2.3, 3.1  
**Description:**
- `POST /api/v1/expenses` — create expense
  - Fields: `amount`, `currency` (default: user's currency preference), `category_id`, `date`, `description`, `payment_method`
- `GET /api/v1/expenses` — list expenses (paginated, filterable by date range, category)
- `GET /api/v1/expenses/:id` — get single expense
- `PUT /api/v1/expenses/:id` — edit expense
- `DELETE /api/v1/expenses/:id` — soft-delete (set `status = 'deleted'`, keep for 30 days)
- All routes require authentication; users can only access their own expenses

### Issue 4.2 — Frontend: Add Expense form
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 4.1, 3.2  
**Description:**
- Modal or dedicated `/expenses/new` page
- Fields: amount, category (picker), date (date picker), description, payment method (cash, card, bank transfer, other)
- Validation: amount > 0, category required, date required
- Submit → optimistic update → show in expense list

### Issue 4.3 — Frontend: Expense list page
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 4.1  
**Description:**
- `/expenses` route
- Paginated table/list of expenses sorted by date (newest first)
- Show: date, description, category (icon + name), amount, payment method
- Filter bar: date range picker, category multi-select
- Click row → open Edit Expense form
- Delete button with confirmation dialog (soft-delete)

### Issue 4.4 — Frontend: Edit Expense form
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 4.2, 4.3  
**Description:**
- Reuse Add Expense form pre-filled with existing expense data
- On submit → `PUT /api/v1/expenses/:id`

---

## Epic 5: Budget Module

> Covers: BUD-1, BUD-3 from PRD (Phase 1 scope)

### Issue 5.1 — Backend: Budget CRUD API
**Labels:** `feature`, `backend`  
**Dependencies:** Issue 1.4, 2.3, 3.1  
**Description:**
- `POST /api/v1/budgets` — create/update budget for a category + period
  - Fields: `category_id` (nullable for overall budget), `amount`, `period_type` (`monthly`), `period_start`
- `GET /api/v1/budgets` — list all budgets for current user
- `PUT /api/v1/budgets/:id` — update budget amount
- `DELETE /api/v1/budgets/:id` — remove a budget
- `GET /api/v1/budgets/utilization` — return current spend vs. budget per category for the current period

### Issue 5.2 — Frontend: Budget setup page
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 5.1, 3.2  
**Description:**
- `/budgets` route
- List all categories with budget input field next to each
- Visual progress bar: actual spend / budget amount (color: green < 80%, yellow 80–99%, red ≥ 100%)
- "Save budgets" action
- Show current month's spending totals inline

---

## Epic 6: Dashboard

> Covers: RPT-1 from PRD

### Issue 6.1 — Backend: Dashboard summary API
**Labels:** `feature`, `backend`  
**Dependencies:** Issue 4.1, 5.1  
**Description:**
- `GET /api/v1/dashboard/summary` — return for current month:
  - `total_spent`: sum of all expenses
  - `budget_utilization`: array of `{category, budget, spent, percentage}`
  - `top_categories`: top 5 categories by spend
  - `recent_transactions`: last 5 expenses

### Issue 6.2 — Frontend: Dashboard page
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 6.1  
**Description:**
- `/dashboard` route (default landing after login)
- **Summary cards:** Total Spent this month, Number of transactions, Remaining overall budget
- **Budget progress bars:** one per category with a set budget (from BUD-1)
- **Top categories:** ranked list with spend amount
- **Recent transactions:** last 5 expenses with category icon, description, amount, date
- **Quick-add button:** floating action button → opens Add Expense modal

---

## Epic 7: CSV Export

> Covers: RPT-5 (CSV only) from PRD

### Issue 7.1 — Backend: CSV export endpoint
**Labels:** `feature`, `backend`  
**Dependencies:** Issue 4.1  
**Description:**
- `GET /api/v1/expenses/export?format=csv&from=YYYY-MM-DD&to=YYYY-MM-DD`
- Returns a downloadable CSV file with columns: `Date, Description, Category, Amount, Currency, Payment Method`
- Filter by date range (default: current month)

### Issue 7.2 — Frontend: Export button on Expense List
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 7.1, 4.3  
**Description:**
- "Export CSV" button on the Expense List page
- Passes current filter state (date range, category) to the export endpoint
- Triggers browser file download

---

## Epic 8: User Settings & Profile

### Issue 8.1 — Backend: User profile API
**Labels:** `feature`, `backend`  
**Dependencies:** Issue 2.3  
**Description:**
- `GET /api/v1/users/me` — return current user profile
- `PUT /api/v1/users/me` — update name, avatar URL, currency preference, locale

### Issue 8.2 — Frontend: Settings / Profile page
**Labels:** `feature`, `frontend`  
**Dependencies:** Issue 8.1  
**Description:**
- `/settings` route
- Edit name, currency preference (dropdown of major currencies)
- Change password section (old + new + confirm)
- Danger zone: "Delete my account" (soft-delete for now)

---

## Summary Table

| Epic | Issues | Milestone |
|------|--------|-----------|
| 1. Project Setup | 1.1 – 1.5 | Sprint 0 (Week 1) |
| 2. Authentication | 2.1 – 2.9 | Sprint 1–2 (Weeks 2–4) |
| 3. Categories | 3.1 – 3.2 | Sprint 1 (Week 2) |
| 4. Expense Management | 4.1 – 4.4 | Sprint 2–3 (Weeks 3–6) |
| 5. Budget Module | 5.1 – 5.2 | Sprint 3–4 (Weeks 5–8) |
| 6. Dashboard | 6.1 – 6.2 | Sprint 4 (Weeks 7–9) |
| 7. CSV Export | 7.1 – 7.2 | Sprint 4 (Week 9) |
| 8. Settings | 8.1 – 8.2 | Sprint 4 (Weeks 9–10) |

**Total Issues: 24**

---

## Suggested Sprint Plan

| Sprint | Duration | Issues |
|--------|----------|--------|
| Sprint 0 (Setup) | Week 1 | 1.1, 1.2, 1.3, 1.4, 1.5 |
| Sprint 1 (Auth Backend + Categories) | Weeks 2–3 | 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 3.1 |
| Sprint 2 (Auth Frontend + Expense Backend) | Weeks 4–5 | 2.7, 2.8, 2.9, 4.1, 3.2 |
| Sprint 3 (Expense Frontend + Budget) | Weeks 6–7 | 4.2, 4.3, 4.4, 5.1, 5.2 |
| Sprint 4 (Dashboard + Export + Settings) | Weeks 8–10 | 6.1, 6.2, 7.1, 7.2, 8.1, 8.2 |

---

## Issue Label Definitions

| Label | Meaning |
|-------|---------|
| `setup` | Infrastructure, tooling, config |
| `feature` | New user-facing functionality |
| `backend` | Server-side / API work |
| `frontend` | Client-side / UI work |
| `database` | Schema, migrations, seeds |
| `auth` | Authentication / authorization |
| `devops` | CI/CD, Docker, deployment |
| `chore` | Non-feature maintenance |
