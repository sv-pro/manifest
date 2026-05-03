# Deployment

Manifest ships exclusively as a Docker image (`manifestdotbuild/manifest`). There are no publishable npm packages.

## Docker Image

- Registry: `manifestdotbuild/manifest`
- Tags: `{version}`, `{major}.{minor}`, `{major}`, `sha-<short>`
- Architecture: multi-arch (linux/amd64 + linux/arm64)
- Signing: cosign-signed

Current version: `6.0.1` (canonical source: `packages/manifest/package.json`)

## Environment Modes

| `MANIFEST_MODE` | Description |
|----------------|-------------|
| `cloud` (default) | Standard multi-tenant mode. Requires HTTPS public endpoints for custom providers. |
| `selfhosted` / `local` | Enables loopback auth shortcuts, allows `http://` and private IP custom providers. Auto-set when `/.dockerenv` exists. |

## Required Environment Variables

```env
BETTER_AUTH_SECRET=<hex-64-chars>           # Required — session signing key
DATABASE_URL=postgresql://user:pass@host/db # Required in production
PORT=3001
BIND_ADDRESS=0.0.0.0                        # Use 0.0.0.0 for Docker/Railway
```

Full list: see [development.md](development.md#environment-variables).

## Railway

`railway.toml` is present for one-click Railway deployment. Set `BIND_ADDRESS=0.0.0.0` and provide `DATABASE_URL` pointing to a Railway PostgreSQL instance.

## CI/CD

### Triggers

| Event | Action |
|-------|--------|
| PR opened/updated | `ci.yml`: tests + lint + typecheck + coverage; `docker.yml`: build validation (no push); `changeset-check`: soft warn if no changeset |
| Merge to `main` | `release.yml`: `changesets/action` opens/updates `chore: version packages` PR |
| Merge of `chore: version packages` | `release.yml` detects version bump → calls `docker.yml` → pushes to Docker Hub |
| Manual `workflow_dispatch` on Docker | Push image with specified or current version |

### Version Management

Canonical version lives in `packages/manifest/package.json`. To release:

```bash
npx changeset         # select "manifest", choose patch/minor/major, write summary
# commit the .changeset/*.md file
# → on merge to main, changesets/action opens "chore: version packages" PR
# → merging that PR triggers Docker publish
```

`.changeset/config.json` ignores `manifest-backend`, `manifest-frontend`, `manifest-shared` — always target `manifest`.

## Database Setup (Production)

Migrations run automatically on startup (`migrationsRun: true`). No manual migration step needed. The PostgreSQL user needs `CREATE TABLE` privileges.

## Docker Compose (bundled)

`docker/docker-compose.yml` provides a ready-to-run stack with the Manifest container + PostgreSQL + optional Ollama. `OLLAMA_HOST` defaults to `http://host.docker.internal:11434` inside Docker.

## See Also

- [development.md](development.md) — local dev setup
- [architecture.md](architecture.md) — single-service topology
- [telemetry.md](telemetry.md) — self-hosted telemetry behavior
