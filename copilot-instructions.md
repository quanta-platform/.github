# Quanta Platform — GitHub Copilot Instructions

## Project
Quanta is a multi-tenant e-commerce platform for MSMEs (NestJS + Next.js + Neon Postgres).
Full architecture: see `quanta-agent-guardrails.md` in the `agent-configs` repo (`github.com/quanta-platform/agent-configs`).

## Naming
storefront=Storefront, admin=Admin, api=API, tenant=Tenant, catalog=Catalog,
pricing=Pricing, orders=Orders, payments=Payments, inventory=Inventory, supply-chain=Supply Chain,
messaging=Messaging, ui-kit=UI Kit, api-client=API Client

## Tech Stack
- Backend: NestJS (TypeScript) — modular monolith (`apps/api/`)
- Frontend: Next.js (TypeScript) + Tailwind CSS — storefront + admin
- Database: Neon Postgres (system of record) — Firestore is projection only
- Auth: Firebase Auth — server-side ID token verification
- Package manager: pnpm only (never npm/yarn)
- Deploy: Cloud Run | CI/CD: GitHub Actions

## Architecture Rules
- Module pattern: Controller -> Service -> Repository
- No cross-module direct imports — use service layer or events
- All API inputs use DTOs with `class-validator` decorators
- Every DB query MUST filter by `tenant_id` (UUID)
- `tenant_id` derived from auth middleware (`req.tenantId`), never from client input
- TypeScript strict mode — no `any`, use `unknown` and narrow
- One class per file, max ~300 lines

## Mandatory Testing
- Unit tests: `*.spec.ts` (co-located) — >= 95% coverage for changed files
- Unit test every service method
- **E2E tests are mandatory for every new/changed API endpoint** — write in `quanta-platform/e2e-tests` using Vitest + fetch
- E2E test files: `e2e-tests/tests/api/<module>/<feature>.test.ts`
- Use helpers from `tests/api/helpers.ts`: `apiGet`, `apiPost`, `apiPut`, `apiPatch`, `apiDelete`, `getAuthToken`, `authHeaders`, `uniqueSuffix`
- Always test tenant isolation and auth enforcement

## Security
- No `eval()`, no dynamic SQL, no `dangerouslySetInnerHTML` without sanitization
- No secrets/credentials in code — use env vars and Secret Manager
- Firebase ID tokens verified server-side on every protected request
- RBAC via NestJS guards: owner > admin > staff > viewer

## Multi-Tenancy
- Every business table has `tenant_id` column
- Tenant resolved from Firebase Auth token -> membership table
- All repository methods require and enforce tenantId parameter
- Test tenant isolation: tenant A must never access tenant B's data

## Workflow
- All work branches from `develop`, PRs target `develop`
- Branches: `feat/<module>/<desc>`, `fix/<module>/<desc>`, `chore/<desc>`, `infra/<desc>`
- Conventional commits: `feat(catalog): add product variants`
- Deploy via GitHub Actions only — no manual deploys

## Traceability
- SRS: `docs/SRS.md` | Matrix: `docs/traceability-matrix.md` | Tracking: GitHub Project #1
- Tag GitHub issues with SRS IDs: `**SRS Requirements:** CM-001, CM-002`
- Update traceability matrix after each task

## Don't
- Commit .env files or secrets
- Force push to main/develop
- Skip git hooks
- Write business data to Firestore
- Import directly across module boundaries
- Use npm or yarn (use pnpm)
- Skip E2E tests for new/changed endpoints
