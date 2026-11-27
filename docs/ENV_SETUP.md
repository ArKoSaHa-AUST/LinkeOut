# LinkedOut — Environment Setup

This guide gets a new developer from a fresh machine to a running local LinkedOut (web, API and worker) with seeded data in about 20 minutes.

Related: [README.md](../README.md) · [TECH_STACK.md](TECH_STACK.md) · [DEPLOYMENT.md](DEPLOYMENT.md) · [FOLDER_STRUCTURE.md](FOLDER_STRUCTURE.md)

## 1. Prerequisites

| Tool | Version | Check | Install notes |
|---|---|---|---|
| Git | ≥ 2.40 | `git --version` | — |
| Node.js | 24 LTS (see `.nvmrc`) | `node -v` | Use `fnm` or `nvm`: `fnm use` / `nvm use` |
| pnpm | 10.x | `pnpm -v` | `corepack enable && corepack prepare pnpm@10 --activate` |
| Docker Engine + Compose v2 | Docker ≥ 27 | `docker compose version` | Docker Desktop (macOS/Windows) or Docker Engine (Linux); ≥ 4 GB RAM allotted (6 GB with ClamAV) |
| OpenSSL | any | `openssl version` | For generating secrets |
| Playwright browsers | bundled | `pnpm --filter web exec playwright install --with-deps` | Needed for E2E only |
| k6 | 1.x | `k6 version` | Needed for load tests only |

Free ports: `3000` (web), `4000` (API), `5432` (PostgreSQL), `6379` (Redis), `9000`/`9001` (MinIO), `1025`/`8025` (Mailpit), `3310` (ClamAV, optional).

## 2. `.env.example`

A single root `.env` is read by the API, worker, scripts and (for `NEXT_PUBLIC_*` and server-side variables) the web app. `apps/api/src/config/env.schema.ts` validates it at startup, and `pnpm check-env` validates it on demand.

```dotenv
# ─── General ──────────────────────────────────────────────
NODE_ENV=development
APP_ENV=local
LOG_LEVEL=debug

# ─── URLs & cookies ───────────────────────────────────────
WEB_URL=http://localhost:3000
API_URL=http://localhost:4000
API_PORT=4000
NEXT_PUBLIC_API_URL=http://localhost:4000/api/v1
COOKIE_DOMAIN=localhost
COOKIE_SECURE=false
CORS_ORIGINS=http://localhost:3000
TRUST_PROXY=loopback

# ─── Database (PostgreSQL 17) ─────────────────────────────
DATABASE_URL=postgresql://lo_app:lo_app_dev_pw@localhost:5432/linkedout?schema=public
VAULT_DATABASE_URL=postgresql://lo_vault:lo_vault_dev_pw@localhost:5432/linkedout?schema=vault
MIGRATION_DATABASE_URL=postgresql://lo_migrator:lo_migrator_dev_pw@localhost:5432/linkedout
READONLY_DATABASE_URL=postgresql://lo_readonly:lo_readonly_dev_pw@localhost:5432/linkedout?schema=public

# ─── Redis / Key Value ────────────────────────────────────
REDIS_URL=redis://localhost:6379/0

# ─── Security secrets (generate your own; see §3) ─────────
CSRF_SECRET=replace-with-64-hex-chars
VAULT_ENCRYPTION_KEY=replace-with-base64-32-bytes
VAULT_ENCRYPTION_KEY_ID=k1
ARGON2_MEMORY_KIB=19456
ARGON2_TIME_COST=2
HIBP_ENABLED=true
RATE_LIMIT_ENABLED=true

# ─── Email ────────────────────────────────────────────────
EMAIL_PROVIDER=smtp
SMTP_HOST=localhost
SMTP_PORT=1025
RESEND_API_KEY=
EMAIL_FROM="LinkedOut <no-reply@linkedout.local>"
EMAIL_REPLY_TO=support@linkedout.local

# ─── Object storage (MinIO locally, Cloudflare R2 in cloud) ─
S3_ENDPOINT=http://localhost:9000
S3_REGION=auto
S3_ACCESS_KEY_ID=linkedout
S3_SECRET_ACCESS_KEY=linkedout-dev-secret
S3_FORCE_PATH_STYLE=true
S3_BUCKET_MEDIA=linkedout-media
S3_BUCKET_EXPORTS=linkedout-exports
MEDIA_PUBLIC_BASE_URL=http://localhost:9000/linkedout-media

# ─── Malware scanning (Should, F-29) ──────────────────────
CLAMAV_ENABLED=false
CLAMAV_HOST=localhost
CLAMAV_PORT=3310

# ─── Geo lookup for session list ──────────────────────────
GEOIP_DB_PATH=./.data/GeoLite2-City.mmdb
MAXMIND_LICENSE_KEY=

# ─── OAuth (Should, F-02) ─────────────────────────────────
GOOGLE_OAUTH_ENABLED=false
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=http://localhost:4000/api/v1/auth/oauth/google/callback

# ─── Observability ────────────────────────────────────────
SENTRY_DSN=
NEXT_PUBLIC_SENTRY_DSN=
SENTRY_ENVIRONMENT=local
OTEL_ENABLED=false
OTEL_SERVICE_NAME=linkedout-api
OTEL_EXPORTER_OTLP_ENDPOINT=
OTEL_EXPORTER_OTLP_HEADERS=

# ─── Developer conveniences ───────────────────────────────
SWAGGER_ENABLED=true
SEED_DEV_DATA=true
```

### 2.1 Variable reference

| Variable | Required | Example (local) | Description |
|---|---|---|---|
| `NODE_ENV` | Yes | `development` | Node mode: `development`, `test`, `production`. |
| `APP_ENV` | Yes | `local` | Deployment environment: `local`, `test`, `staging`, `production`. Drives safe defaults (e.g. Swagger off in production). |
| `LOG_LEVEL` | Yes | `debug` | pino level: `fatal`…`trace`. Production uses `info`. |
| `WEB_URL` | Yes | `http://localhost:3000` | Public web origin; used in email links. |
| `API_URL` | Yes | `http://localhost:4000` | Public API origin. |
| `API_PORT` | Yes | `4000` | Port the API listens on (Render injects `PORT`; the app prefers `PORT` if set). |
| `NEXT_PUBLIC_API_URL` | Yes | `http://localhost:4000/api/v1` | API base URL used by the browser. |
| `COOKIE_DOMAIN` | Yes | `localhost` | Cookie `Domain`. `.linkedout.app` in production, `.staging.linkedout.app` in staging. |
| `COOKIE_SECURE` | Yes | `false` | `true` everywhere except plain-HTTP local development. |
| `CORS_ORIGINS` | Yes | `http://localhost:3000` | Comma-separated allowed origins (credentials enabled). |
| `TRUST_PROXY` | Yes | `loopback` | Express trust-proxy setting so rate limits use the real client IP. Behind Cloudflare+Render use `2`. |
| `DATABASE_URL` | Yes | see above | Connection for role `lo_app` (no vault access). |
| `VAULT_DATABASE_URL` | Yes | see above | Connection for role `lo_vault`, used only by `IdentityVaultService`. Never set on the web app. |
| `MIGRATION_DATABASE_URL` | Yes (CI/migrations) | see above | Owner role for `prisma migrate`. Not set on running services in production. |
| `READONLY_DATABASE_URL` | No | see above | Admin metrics and restore verification. Falls back to `DATABASE_URL` locally. |
| `REDIS_URL` | Yes | `redis://localhost:6379/0` | Sessions, rate limits, cache, BullMQ. Use `rediss://` in cloud. |
| `CSRF_SECRET` | Yes | 64 hex chars | HMAC key binding CSRF tokens to sessions. |
| `VAULT_ENCRYPTION_KEY` | Yes | base64 of 32 random bytes | AES-256-GCM key for sensitive notification references. |
| `VAULT_ENCRYPTION_KEY_ID` | Yes | `k1` | Key identifier stored with ciphertext to support rotation. |
| `ARGON2_MEMORY_KIB` | Yes | `19456` | Argon2id memory cost (19 MiB). Tests may lower it to `4096` for speed. |
| `ARGON2_TIME_COST` | Yes | `2` | Argon2id iterations. |
| `HIBP_ENABLED` | Yes | `true` | Breached-password check. Set `false` offline (tests stub it). |
| `RATE_LIMIT_ENABLED` | Yes | `true` | Must be `true` in staging/production; tests toggle it per suite. |
| `EMAIL_PROVIDER` | Yes | `smtp` | `smtp` (Mailpit locally) or `resend`. |
| `SMTP_HOST` / `SMTP_PORT` | If smtp | `localhost` / `1025` | Mailpit SMTP. |
| `RESEND_API_KEY` | If resend | `re_…` | Resend API key (staging/production). |
| `EMAIL_FROM` | Yes | `LinkedOut <no-reply@linkedout.local>` | From header. Production: `LinkedOut <no-reply@linkedout.app>`. |
| `EMAIL_REPLY_TO` | Yes | `support@linkedout.local` | Reply-To header. |
| `S3_ENDPOINT` | Yes | `http://localhost:9000` | MinIO locally; `https://8c1f4e2a9b7d.r2.cloudflarestorage.com (your R2 account ID)` in cloud. |
| `S3_REGION` | Yes | `auto` | R2 uses `auto`. |
| `S3_ACCESS_KEY_ID` / `S3_SECRET_ACCESS_KEY` | Yes | MinIO credentials | Per-environment, least-privilege bucket tokens. |
| `S3_FORCE_PATH_STYLE` | Yes | `true` | Required for MinIO. `false` for R2. |
| `S3_BUCKET_MEDIA` / `S3_BUCKET_EXPORTS` | Yes | `linkedout-media` / `linkedout-exports` | Bucket names. |
| `MEDIA_PUBLIC_BASE_URL` | Yes | MinIO URL | Public base URL for clean media (`https://files.linkedout.app` in production). |
| `CLAMAV_ENABLED` | Yes | `false` | Enable once F-29 ships; requires the `clamav` compose profile locally. |
| `CLAMAV_HOST` / `CLAMAV_PORT` | If enabled | `localhost` / `3310` | clamd TCP endpoint. |
| `GEOIP_DB_PATH` | No | `./.data/GeoLite2-City.mmdb` | Offline GeoLite2 DB. If missing, sessions show "Unknown location". |
| `MAXMIND_LICENSE_KEY` | No | — | Lets `pnpm geoip:download` fetch the DB. |
| `GOOGLE_OAUTH_ENABLED` | Yes | `false` | Feature switch for F-02. |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` / `GOOGLE_REDIRECT_URI` | If enabled | — | Google Cloud OAuth client (Web). |
| `SENTRY_DSN` / `NEXT_PUBLIC_SENTRY_DSN` | No (prod: yes) | — | Error tracking for API/worker and web. Empty disables Sentry. |
| `SENTRY_ENVIRONMENT` | No | `local` | Sentry environment tag. |
| `OTEL_ENABLED` | Yes | `false` | Enable the OpenTelemetry SDK. |
| `OTEL_SERVICE_NAME` | If enabled | `linkedout-api` | `linkedout-worker` for the worker process. |
| `OTEL_EXPORTER_OTLP_ENDPOINT` / `OTEL_EXPORTER_OTLP_HEADERS` | If enabled | Grafana Cloud OTLP URL / `Authorization=Basic …` | Telemetry export. |
| `SWAGGER_ENABLED` | Yes | `true` | Serves `/api/v1/docs`. Forced `false` when `APP_ENV=production`. |
| `SEED_DEV_DATA` | Yes | `true` | Whether `pnpm db:seed` inserts dev fixtures (always `false` in production). |

## 3. Local setup, step by step

```bash
# 1. Clone and enter the repo
git clone git@github.com:linkedout-app/linkedout.git
cd linkedout

# 2. Use the pinned Node version and enable pnpm
fnm use            # or: nvm use
corepack enable

# 3. Install dependencies (also installs Husky git hooks)
pnpm install

# 4. Create your env file and generate secrets
cp .env.example .env
sed -i.bak "s/^CSRF_SECRET=.*/CSRF_SECRET=$(openssl rand -hex 32)/" .env
sed -i.bak "s#^VAULT_ENCRYPTION_KEY=.*#VAULT_ENCRYPTION_KEY=$(openssl rand -base64 32)#" .env
rm .env.bak
pnpm check-env

# 5. Start local infrastructure (Postgres, Redis, MinIO, Mailpit)
pnpm infra:up                 # docker compose -f infra/docker-compose.yml up -d
# optional, for media scanning work:
pnpm infra:up --profile media # adds ClamAV (first start downloads signatures, ~2–5 min)

# 6. Create schema, roles grants and seed data
pnpm db:migrate               # prisma migrate dev (uses MIGRATION_DATABASE_URL)
pnpm db:seed                  # reference data + dev fixtures

# 7. (Optional) download GeoLite2 for session locations
pnpm geoip:download

# 8. Run everything (web :3000, api :4000, worker)
pnpm dev
```

Then open:

| URL | What |
|---|---|
| http://localhost:3000 | Web app |
| http://localhost:4000/api/v1/docs | Swagger UI |
| http://localhost:4000/api/v1/health/ready | Readiness check |
| http://localhost:8025 | Mailpit (verification/reset emails) |
| http://localhost:9001 | MinIO console (`linkedout` / `linkedout-dev-secret`) |
| http://localhost:4000/admin/queues | Bull Board (local/staging only, admin session required) |

**Dev accounts** (password `Password-For-Dev-Only-1`): `admin@linkedout.local`, `mod@linkedout.local`, `mod2@linkedout.local`, `priya@linkedout.local`, `marcus@linkedout.local`, `aisha@linkedout.local`, `rep@acme-widgets.test`.

## 4. Database setup details

### 4.1 Containers and roles

`infra/docker-compose.yml` starts `postgres:17` with database `linkedout`. On first start it runs `infra/postgres/init/01-roles.sql`, which:
1. Creates roles `lo_migrator` (owner), `lo_app`, `lo_vault` and `lo_readonly` with the local passwords in `.env.example`.
2. Creates the schemas `public` (owned by `lo_migrator`) and `vault` (owned by `lo_migrator`).
3. Enables extensions `citext`, `pg_trgm` and `unaccent`.

Grants are applied by migrations (not the init script) so that staging and production get identical privileges: `lo_app` on `public` only, `lo_vault` on `vault` plus `SELECT(id, account_status)` on `public.users`, and the append-only trigger on `audit_logs`.

### 4.2 Migrations

| Command | Use |
|---|---|
| `pnpm db:migrate` | Create/apply migrations in development (`prisma migrate dev`). Name them descriptively: `pnpm db:migrate --name add_job_reports`. |
| `pnpm db:migrate:deploy` | Apply pending migrations without generating new ones (CI, staging, production). |
| `pnpm db:reset` | Drop, re-create, migrate and seed. **Destroys local data.** |
| `pnpm db:studio` | Prisma Studio (connects as `lo_app`, so the vault is not visible, by design). |
| `pnpm db:generate` | Regenerate both Prisma clients after schema changes. |

Hand-written SQL (grants, triggers, generated `tsvector` columns, partial indexes) goes in the migration's `migration.sql`. Prisma does not express all of these, so they must stay in migrations.

### 4.3 Seed data

`pnpm db:seed` runs `apps/api/prisma/seed/index.ts`:

| Step | Environments | Contents |
|---|---|---|
| Reference data | All | 12 categories, 20 industries, reserved usernames, feature flags (`confessions.enabled=false` in production, `true` locally) |
| Dev fixtures | `SEED_DEV_DATA=true` (local, staging) | Users listed above, 3 companies, 40 posts (6 lessons, 5 confessions, 3 anonymous), 120 comments, 300 reactions, 12 reviews (7 for Acme Widgets), 10 job reports, 6 reports → 4 moderation items, 1 appeal |

The seed is idempotent (upserts by slug/email), so running it twice is safe.

## 5. Everyday commands

| Command | Description |
|---|---|
| `pnpm dev` | Web + API + worker with hot reload (Turborepo) |
| `pnpm dev:web` / `pnpm dev:api` / `pnpm dev:worker` | Run one app |
| `pnpm build` | Production build of all apps and packages |
| `pnpm lint` / `pnpm lint:fix` | ESLint (incl. module boundaries, a11y) |
| `pnpm format` | Prettier write |
| `pnpm typecheck` | `tsc --noEmit` across the workspace |
| `pnpm test` | Unit tests (Jest for API, Vitest for web/shared) |
| `pnpm test:int` | API integration tests (Testcontainers; needs Docker) |
| `pnpm test:sec` | Security test suite |
| `pnpm test:e2e` | Playwright E2E (starts the stack via `pnpm e2e:stack` if not running) |
| `pnpm test:cov` | Unit + integration with coverage thresholds |
| `pnpm test:load -- feed` | Run a k6 script against `LOAD_TEST_BASE_URL` |
| `pnpm infra:up` / `pnpm infra:down` / `pnpm infra:logs` | Manage local containers |
| `pnpm check-env` | Validate `.env` |

## 6. IDE setup

- VS Code recommended extensions (in `.vscode/extensions.json`): ESLint, Prettier, Tailwind CSS IntelliSense, Prisma, Playwright Test, EditorConfig, Docker.
- Enable "format on save" with Prettier, and ESLint "fix on save".
- TypeScript: use the workspace version (`typescript.tsdk = node_modules/typescript/lib`).

## 7. Common errors and fixes

| Symptom | Cause | Fix |
|---|---|---|
| `Error: Invalid environment variables: CSRF_SECRET …` at API start | `.env` missing or still has placeholder values | Re-run step 4 of §3; `pnpm check-env` lists each invalid key. |
| `P1001: Can't reach database server at localhost:5432` | Postgres container not running or port in use | `pnpm infra:up`; check `docker ps`; stop any local Postgres (`sudo systemctl stop postgresql`) or change the port mapping. |
| `permission denied for schema vault` in API logs | Code path uses `PrismaService` instead of `VaultPrismaService` | Correct by design: vault access must go through `IdentityVaultService`. Fix the calling code. |
| `role "lo_app" does not exist` | Postgres volume created before the init script existed | `pnpm infra:down -v` (deletes local data) then `pnpm infra:up`, `pnpm db:migrate`, `pnpm db:seed`. |
| `ERR_PNPM_UNSUPPORTED_ENGINE` | Wrong Node version | `fnm use` / `nvm use` (Node 24). |
| `ECONNREFUSED 127.0.0.1:6379` / API returns 503 | Redis not running | `pnpm infra:up`; this 503 is the intended fail-closed behavior. |
| Login succeeds but every next request is 401 | Cookie not stored: `COOKIE_DOMAIN`/`COOKIE_SECURE` mismatch | Locally use `COOKIE_DOMAIN=localhost`, `COOKIE_SECURE=false`, and open `http://localhost:3000` (not `127.0.0.1`). |
| `403 CSRF_INVALID` on every POST | Missing `lo_csrf` cookie or header | Make sure the web client calls `GET /auth/csrf` first; check that `CORS_ORIGINS` matches the web origin exactly (no trailing slash). |
| No verification email arrives | Email goes to Mailpit, not your inbox | Open http://localhost:8025. In staging, check the Resend dashboard and the worker logs. |
| Worker not processing jobs | Worker not started or different `REDIS_URL` | `pnpm dev:worker`; confirm the same `REDIS_URL`; check Bull Board. |
| `OOM command not allowed` from Redis | `maxmemory` with an evicting policy | The compose file sets `noeviction`; for cloud, set the Key Value maxmemory policy to `noeviction`. |
| Testcontainers: `Could not find a working container runtime strategy` | Docker not accessible to your user | Add the user to the `docker` group (Linux) or start Docker Desktop; on rootless Docker set `DOCKER_HOST`. |
| Playwright: `browserType.launch: Executable doesn't exist` | Browsers not installed | `pnpm --filter web exec playwright install --with-deps`. |
| `pnpm db:migrate` wants to reset the database | Drift between migrations and the local DB (manual changes) | Run `pnpm db:reset` locally. Never edit applied migrations; create a new one. |
| `register` returns 400 `breached_password` for test passwords | HIBP check working as intended | Use a long passphrase, or set `HIBP_ENABLED=false` locally only. |
| ClamAV container unhealthy for minutes | Signature download on first start | Wait for `freshclam` to finish (`pnpm infra:logs clamav`). |
| Sessions show "Unknown location" | GeoLite2 DB missing | `pnpm geoip:download` with `MAXMIND_LICENSE_KEY`, or ignore locally. |
| Port 3000/4000 already in use | Another process is listening | `lsof -i :3000` and stop it, or set `PORT`/`API_PORT`. |
