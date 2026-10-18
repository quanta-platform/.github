# .github

Organization profile and shared CI/CD workflows for Quanta Platform.

## Environment URLs

### Production (deploys from `main`)

| Service    | URL                                  |
|------------|--------------------------------------|
| Marketing  | https://pandamaatru.com              |
| Storefront | https://{slug}.pandamaatru.com       |
| API        | https://api.pandamaatru.com          |
| Admin      | https://admin.pandamaatru.com        |

### QA (deploys from `develop`)

| Service    | URL                                  |
|------------|--------------------------------------|
| Marketing  | https://qa.pandamaatru.com           |
| Storefront | https://{slug}.pandamaatru.com       |
| API        | https://qa-api.pandamaatru.com       |
| Admin      | https://qa-admin.pandamaatru.com     |

### Health Checks

- API / Worker: `GET /health`
- Storefront / Admin: `GET /api/health`

## Shared Workflows

Located in `.github/workflows/`:

| Workflow            | File                | Description                                      |
|---------------------|---------------------|--------------------------------------------------|
| CI — NestJS         | `ci-nestjs.yml`     | Lint, test, build for NestJS repos (api, worker) |
| CI — Next.js        | `ci-nextjs.yml`     | Lint, unit tests (Vitest), build for Next.js repos (storefront, admin)|
| CD — Cloud Run      | `cd-cloud-run.yml`  | Build Docker image, push to Artifact Registry, deploy to Cloud Run |

### Usage

Each service repo calls these via `workflow_call`:

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [develop, main]
jobs:
  ci:
    uses: quanta-platform/.github/.github/workflows/ci-nestjs.yml@main
  deploy:
    needs: ci
    uses: quanta-platform/.github/.github/workflows/cd-cloud-run.yml@main
    with:
      service-name: quanta-api
    secrets: inherit
```

### Inputs worth knowing

`ci-nextjs.yml`
- `run-unit-tests` (default `true`) — runs `pnpm test` (`vitest run`) before the build. Set `false` only as a temporary escape hatch.
- `test-timezone` (default `UTC`) — `TZ` for the tests. Suites are kept timezone-independent; this just pins CI.

`ci-nestjs.yml` already runs `pnpm test` (Jest), plus `pnpm test:integration` when `run-integration-tests: true`.

`cd-cloud-run.yml`
- `run-migrations` (default `false`; the api sets `true`) — before **Deploy to Cloud Run** in both
  `deploy-qa` and `deploy-prod`, creates/updates the Cloud Run Job `<service-name>-migrate` on the
  just-pushed image and runs it with `gcloud run jobs execute --wait`. A failed migration fails the
  job, the deploy step never runs, and the current revision keeps serving.
  - Runs as `<service-name>-sa` (the service's own identity) with `DATABASE_URL` from
    `migrate-database-secret` (default `NEON_DATABASE_URL_POOLED`, the same secret the api
    service reads). The api runner rewrites Neon's `-pooler` host to the direct endpoint.
  - 1 task, 0 retries, 30 min task timeout; the runner takes a `pg_advisory_lock`, so overlapping
    executions queue rather than race.
  - Because the old revision serves against the new schema until the deploy finishes, migrations
    must be backward compatible (expand → deploy → contract).
  - Needs nothing beyond what the deployer SA already has: `roles/run.admin` and
    `roles/iam.serviceAccountUser` (to run the job as `<service>-sa`).
- `migrate-args` (default `dist/database/migrate.js`) — `node` args for the job.

### Branch → Environment Mapping

| Branch    | Environment | Action        |
|-----------|-------------|---------------|
| `develop` | QA          | Build + Deploy|
| `main`    | Production  | Build + Deploy|

### Required Repo Secrets

Each service repo must have these secrets set (repo-level, not org-level):

| Secret                | Description                      |
|-----------------------|----------------------------------|
| `GCP_QA_PROJECT_ID`   | `quanta-qa`                     |
| `GCP_PROD_PROJECT_ID` | `quanta-prod-platform`          |
| `GCP_QA_SA_KEY`       | Deployer SA key JSON for QA     |
| `GCP_PROD_SA_KEY`     | Deployer SA key JSON for Prod   |
| `GCP_REGION`          | `asia-southeast1`               |
