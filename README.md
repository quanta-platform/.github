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
| CI — Next.js        | `ci-nextjs.yml`     | Lint, build for Next.js repos (storefront, admin)|
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
