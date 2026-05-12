# Expense Manager — CLAUDE.md

## Project Overview

A web application for tracking, categorizing, and analyzing personal and team expenses. Users can log expenses, set budgets, view analytics, and collaborate on shared expenses.

**PRD:** See `PRD.md` for full product requirements.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React.js + TypeScript |
| Mobile (Phase 4) | React Native |
| Backend API | Node.js + Express + TypeScript |
| Database | PostgreSQL (primary), Redis (sessions/cache) |
| Auth | JWT + OAuth2 (Google) |
| File Storage | AWS S3 / GCS (Phase 2) |
| Notifications | Firebase Cloud Messaging + SendGrid (Phase 2+) |
| Deployment | Docker + Kubernetes |

## Repository Structure (target)

```
expence-manager/
├── frontend/          # React + TypeScript web app
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── store/
│   │   ├── api/
│   │   └── utils/
│   ├── public/
│   └── package.json
├── backend/           # Node.js + Express API
│   ├── src/
│   │   ├── routes/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── models/
│   │   ├── middleware/
│   │   └── utils/
│   ├── migrations/
│   ├── seeds/
│   └── package.json
├── docker-compose.yml
├── PRD.md
└── CLAUDE.md
```

## Development Guidelines

### General
- Use TypeScript strictly — no `any` types unless unavoidable
- Follow REST conventions for all API routes
- Write tests for all service-layer logic
- Soft-delete expenses (never hard-delete user financial data)

### Backend
- All routes require authentication unless explicitly public
- Use middleware for auth validation (JWT verification)
- Database migrations via a migration tool (e.g., `node-postgres-migrate` or `knex`)
- Seed default categories on first run

### Frontend
- Mobile-first responsive design
- Use React Query or SWR for server state
- Form validation with `react-hook-form` + `zod`
- Global state (auth, user preferences) via React Context or Zustand

### Naming Conventions
- Files: `kebab-case` for all files
- Components: `PascalCase`
- Functions/variables: `camelCase`
- DB columns: `snake_case`
- API routes: `/api/v1/<resource>`

### Git Workflow
- Branch naming: `feature/<issue-number>-<short-description>`
- Commit messages follow Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `test:`
- All PRs must reference the issue number

## Phase Overview

| Phase | Scope | Timeline |
|-------|-------|----------|
| **Phase 1** | MVP — Auth, Expenses, Categories, Budget, Dashboard, CSV export | Months 1–3 |
| Phase 2 | Receipts, Custom categories, Analytics charts, Multi-currency, Notifications, PDF export | Months 4–6 |
| Phase 3 | Groups, Shared expenses, Approval workflow, Settlement tracker | Months 7–9 |
| Phase 4 | Recurring expenses, Bulk import, AI category suggestions, Mobile app, GDPR | Months 10–12 |

## Phase 1 Issues

See `PHASE1_TASKS.md` for the full breakdown of all GitHub issues for Phase 1.
