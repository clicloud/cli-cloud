# CLI Cloud Backend

Backend documentation scaffold for the CLI Cloud container deployment service inside the `CLI-CLOUD-APP` monorepo.

## Status

- `apps/backend/` is currently a documentation scaffold in this repository snapshot.
- The backend itself is treated as functionally complete at the product level, but the runtime source code is not present in this folder yet.
- This README documents the intended service boundary, deployment contract, and the repo links that already support backend work.

## What Lives Here Today

Current files in `apps/backend/`:

- `README.md` — backend overview and working notes
- `AGENTS.md` — repo instruction pointer
- `CLAUDE.md` — repo instruction pointer
- `.env.example` — environment variable scaffold for the backend service

Related monorepo assets already in place:

- [`../../.docs/container-catalog-spec.md`](../../.docs/container-catalog-spec.md) — canonical catalog entry spec
- [`../../.docs/container-deploy-test-scaffolding.md`](../../.docs/container-deploy-test-scaffolding.md) — backend test-plan scaffold
- `../../packages/container-templates/` — local starter templates already checked into this repo

## Architecture Overview

The backend owns the container deployment control plane for cli.cloud.

Primary responsibilities:

- authenticate the caller
- validate deployment input
- resolve a catalog template or custom image
- start and track the deployment lifecycle
- expose container status and management endpoints
- return enough metadata for the frontend to show progress and project URLs

High-level flow:

1. The frontend sends a deploy request with a template ID or custom image plus project settings.
2. The backend validates auth, naming rules, and environment variables.
3. The backend resolves the selected template from the catalog and prepares runtime configuration.
4. The deployment pipeline builds or pulls the container image, starts the workload, and stores state.
5. The backend exposes status, logs, and lifecycle actions back to the client.

## Catalog Coverage

The current monorepo already contains these local starter templates under `packages/container-templates/`:

| UI Label | Template ID | Runtime | Default Port | Local Template Path |
|---|---|---|---|---|
| Web App | `web-app` | Node.js 20 | 3000 | `../../packages/container-templates/nextjs-starter` |
| AI Backend | `ai-backend` | Python 3.11 | 8000 | `../../packages/container-templates/fastapi-starter` |
| Chat Bot | `chat-bot` | Python 3.11 | none | `../../packages/container-templates/telegram-bot-starter` |
| Simple Site | `simple-site` | Static | 80 | `../../packages/container-templates/static-starter` |
| API Server | `api-server` | Node.js 20 | 3000 | `../../packages/container-templates/express-starter` |
| OpenClaw | `openclaw` | Custom image | 8080 | external image / registry-backed |

For the full catalog contract, labels, and runtime defaults, use [`../../.docs/container-catalog-spec.md`](../../.docs/container-catalog-spec.md).

## API Surface

The working API surface described by the current monorepo docs is:

| Method | Route | Purpose |
|---|---|---|
| `POST` | `/deploy` | Create a deployment from a catalog template or custom image |
| `GET` | `/deploy/:id` | Poll deployment status and lifecycle state |
| `DELETE` | `/deploy/:id` | Stop or terminate a deployment |
| `GET` | `/containers` | List the authenticated user's running containers |
| `GET` | `/containers/:id` | Inspect a specific container |
| `DELETE` | `/containers/:id` | Remove a specific container |
| `GET` | `/catalog` | Return available deployment templates |
| `POST` | `/auth/login` | Exchange CLI credentials for an authenticated session or token |

Expected deploy payload shape from the current planning docs:

```json
{
  "templateId": "web-app",
  "projectName": "my-app",
  "envVars": {
    "NODE_ENV": "production"
  }
}
```

Expected success response shape:

```json
{
  "projectId": "proj_123",
  "status": "pending",
  "url": "https://my-app.cli.cloud"
}
```

These route names and payloads are grounded in the repo docs today. If the implementation ships with different route names or response contracts, update this README and the frontend integration together.

## Environment Variables

An example file now exists at [`./.env.example`](./.env.example).

| Variable | Required | Description |
|---|---|---|
| `PORT` | No | HTTP listen port for the backend service |
| `DATABASE_URL` | Yes | Persistent store for deployments, users, and audit records |
| `JWT_SECRET` | Yes | Signing secret for tokens or session validation |
| `CONTAINER_RUNTIME` | Yes | Runtime driver name such as `docker` or `k8s` |
| `REGISTRY_HOST` | No | Container registry host used for pushes and pulls |
| `REGISTRY_USERNAME` | No | Registry username when auth is required |
| `REGISTRY_PASSWORD` | No | Registry password or access token |
| `DEFAULT_REGION` | No | Default deployment region if the platform supports more than one |
| `LOG_LEVEL` | No | Application log level, for example `info` or `debug` |

If more secrets or provider-specific variables are introduced, add them to `.env.example` and this table in the same change.

## Local Setup Notes

There is no installable backend package manifest in `apps/backend/` yet in this snapshot, so do not assume `npm ci`, `composer install`, or `docker compose up` will work from this folder today.

What is ready right now:

1. Review the service contract in this README.
2. Review the catalog spec in [`../../.docs/container-catalog-spec.md`](../../.docs/container-catalog-spec.md).
3. Review the test scaffold in [`../../.docs/container-deploy-test-scaffolding.md`](../../.docs/container-deploy-test-scaffolding.md).
4. Review the starter templates under `../../packages/container-templates/`.
5. Copy `./.env.example` to `.env` when the backend runtime files land.

Expected backend app scaffold once source code is added:

- application entrypoint
- package or dependency manifest
- Dockerfile and/or compose file
- test directory
- deployment runtime adapter
- catalog registry loader

## Container and Deployment Notes

- Template-backed deploys should resolve against the local template catalog in this monorepo unless and until the project moves that registry elsewhere.
- Custom-image deploys such as OpenClaw should be treated separately from source-template deploys because build, validation, and port assumptions differ.
- Deployment status should remain pollable from the client after `POST /deploy`, which matches the existing deploy dialog spec.
- Secret environment values should be masked in read APIs and logs.
- Build and runtime failures should return explicit, operator-usable errors rather than generic frontend-only messaging.
- Any implementation should preserve an audit trail for who deployed what, when, and from which template or image.

## Documentation Maintenance Rules

- Keep this file aligned with the real repo state. Do not document commands, manifests, or directories that are not present.
- When the backend runtime code lands, update this README in the same pull request so setup and deployment steps stay accurate.
- If API routes or payloads change, update this file and any frontend docs that consume those routes together.
