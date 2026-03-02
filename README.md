# .github

Organization profile and shared CI/CD configs.

## Environment URLs

### Production (deploys from `main`)

| Service    | URL                              |
|------------|----------------------------------|
| Storefront | https://pandamaatru.com          |
| API        | https://api.pandamaatru.com      |
| Admin      | https://admin.pandamaatru.com    |

### QA (deploys from `develop`)

| Service    | URL                              |
|------------|----------------------------------|
| Storefront | https://qa.pandamaatru.com       |
| API        | https://qa-api.pandamaatru.com   |
| Admin      | https://qa-admin.pandamaatru.com |

### Health Checks

- API / Worker: `GET /health`
- Storefront / Admin: `GET /api/health`
