# LinkedOut — Technology Stack

Versions are the **release lines to target at project start (30 November 2026)**. Exact patch versions are pinned in `pnpm-lock.yaml`, Dockerfiles and `.nvmrc`. Renovate proposes upgrades weekly; major upgrades need an ADR entry in [ASSUMPTIONS.md](ASSUMPTIONS.md).

The plan left two choices open, and both are now decided (see [ASSUMPTIONS.md](ASSUMPTIONS.md#3-architecture)):
- **Backend:** NestJS (Node.js/TypeScript), not Spring Boot.
- **Authentication:** server-side sessions in Redis with httpOnly cookies, not JWT.

## 1. Language, runtime and repository

| Technology | Version | Purpose | Why chosen over alternatives |
|---|---|---|---|
| TypeScript | 5.9 | Single language for web, API, worker and shared schemas | One language lets a 2–3 person team share types end-to-end; Java/Kotlin (Spring Boot) would mean two stacks and duplicated DTOs (plan §10: avoid two stacks). |
| Node.js | 24 LTS | Runtime for API, worker, tooling | Active LTS through April 2028; required by BullMQ (Node-only); native `fetch`, test runner, stable ESM. |
| pnpm | 10 | Package manager with workspaces | Faster and more disk-efficient than npm/yarn; strict dependency isolation catches phantom deps. |
| Turborepo | 2.x | Monorepo task runner and caching | Simple config, remote cache on Vercel for free; Nx is more powerful but heavier to learn. |

## 2. Frontend

| Technology | Version | Purpose | Why chosen over alternatives |
|---|---|---|---|
| Next.js (App Router) | 16.x | Web framework, SSR/RSC, routing | Named in the plan; server components give SEO and fast LCP for public stories and company pages; Remix/React Router 7 viable but smaller ecosystem for our hosting choice. |
| React | 19.2 | UI library | Required by Next.js 16; Server Components and Actions. |
| Tailwind CSS | 4.1 | Styling with design tokens | Named in the plan; CSS-first config maps directly to tokens in UI_UX.md; no runtime cost vs CSS-in-JS. |
| shadcn/ui (CLI) + Radix UI primitives | shadcn CLI 3.x, Radix 1.x | Accessible component foundation (dialog, dropdown, tabs, toast, tooltip, popover) | Copy-in components we own and theme; Radix handles focus management and ARIA correctly. MUI/Chakra impose their own visual language and runtime styling. |
| TanStack Query | 5.x | Client-side server-state, infinite scroll, optimistic reactions | Mature caching/invalidation; SWR has fewer mutation features. |
| React Hook Form | 7.x | Forms | Uncontrolled inputs → fast; first-class Zod resolver. |
| Zod | 4.x | Schemas shared between web and API | Single source of truth for validation and TS types; Yup lacks the same TS inference quality. |
| react-markdown + rehype-sanitize | 10.x / 6.x | Safe Markdown rendering | Allow-list sanitization; no `dangerouslySetInnerHTML`. |
| lucide-react | 0.5xx | Icons | Consistent, tree-shakeable, MIT. |
| next/font (self-hosted Inter, Bricolage Grotesque) | bundled | Typography | No third-party font requests (privacy) and no layout shift. |
| date-fns | 4.x | Relative times ("3 hours ago"), time zones | Tree-shakeable; Moment is deprecated. |

## 3. Backend

| Technology | Version | Purpose | Why chosen over alternatives |
|---|---|---|---|
| NestJS | 11.x | API and worker framework | One of the plan's two options; modules, DI, guards and interceptors map cleanly to the modular monolith; same language as frontend. Spring Boot is equally capable but splits the stack. |
| Express (via `@nestjs/platform-express`) | 5.x | HTTP adapter | Default, widest middleware compatibility; Fastify adapter possible later for throughput. |
| Prisma ORM | 6.x | Data access and migrations | Type-safe client, readable schema, `prisma migrate` with SQL migration files, multi-schema support for `vault`. Drizzle is lighter but younger migration tooling; TypeORM has weaker typing. |
| PostgreSQL | 17 | Primary database and full-text search | Named in the plan; relational integrity, `tsvector`, `pg_trgm`, row-level privileges for the vault schema. |
| Redis-compatible store (Valkey 8 on Render Key Value; Redis 7.4 locally) | 8 / 7.4 | Sessions, rate limiting, cache, queues | Named in the plan; BullMQ requirement; Valkey is the open-source Redis fork offered by Render. |
| BullMQ | 5.x | Background jobs, retries, scheduled jobs | Named in the plan; repeatable jobs replace cron; dashboards via Bull Board (admin-only, staging). |
| ioredis | 5.x | Redis client | Required by BullMQ; cluster-ready. |
| `@nestjs/throttler` + Redis storage | 6.x | Rate limiting | Native Nest integration; distributed counters across API instances. |
| argon2 (node-argon2) | 0.4x | Password hashing (Argon2id) | OWASP-recommended algorithm; bcrypt has 72-byte limit and weaker memory hardness. |
| helmet | 8.x | Security headers for API | Standard, minimal config. |
| zod + `nestjs-zod` | 4.x / 4.x | Request validation with shared schemas | Same schemas as frontend; class-validator would duplicate definitions. |
| `@nestjs/swagger` | 11.x | OpenAPI generation | Auto docs from controllers and Zod DTOs. |
| `@nestjs/event-emitter` | 3.x | In-process domain events | Decouples modules without a broker in MVP. |
| libphonenumber-js | 1.12 | Phone-number detection in safety checks | Accurate international parsing vs fragile regex. |
| `@maxmind/geoip2-node` + GeoLite2-City DB | 6.x | City-level session location (offline) | No IP sent to third parties. |
| React Email + Resend SDK | 4.x / 6.x | Email templates and delivery | Templates written in React with shared design tokens; Resend has simple API, EU region, good deliverability; SES is cheaper at scale but heavier setup. |
| sharp | 0.34 | Image re-encode, resize, EXIF strip (Should) | Fastest Node image library (libvips). |
| ClamAV (clamd container) | 1.4 | Malware scanning (Should) | Open source, self-hosted so files never leave our network. |
| pino + nestjs-pino | 9.x / 4.x | Structured JSON logs with redaction | Fast, redaction built-in. |
| OpenTelemetry JS SDK (`@opentelemetry/sdk-node`, auto-instrumentations) | 2.x line | Traces and metrics (OTLP) | Named in the plan; vendor-neutral. |
| Sentry SDK (`@sentry/nestjs`, `@sentry/nextjs`) | 10.x | Error tracking | Source maps, release tracking; PII scrubbing via `beforeSend`. |

## 4. Infrastructure and hosting

| Technology | Version / plan | Purpose | Why chosen over alternatives |
|---|---|---|---|
| Docker Engine + Docker Compose | 27+ / v2 | Local infra and API/worker images | Named in the plan (repeatable environments). |
| Vercel | Pro plan in production | Hosting for Next.js web | First-class Next.js support, PR previews, edge network. Self-hosting Next.js on Render is possible but loses ISR/edge features. |
| Render | Web Service (Docker) ×2, Background Worker, PostgreSQL (Pro with PITR), Key Value | API, worker, DB, Redis | Managed, Docker-based, Frankfurt region, PITR, private networking, low ops burden for a small team. AWS (ECS/RDS) would require far more DevOps work; Fly.io lacks managed Postgres with the same maturity. |
| Cloudflare | Free/Pro | DNS, TLS, WAF, CDN, R2 object storage | R2 is S3-compatible with zero egress fees; WAF and bot protection in front of the API. |
| MinIO | latest release | Local S3-compatible storage | Mirrors R2 API locally. |
| Mailpit | 1.2x | Local email capture | Web UI to read verification/reset emails in development. |
| GitHub + GitHub Actions | — | Source control and CI/CD | Named in the plan. |
| GitHub CodeQL, Dependabot alerts, Renovate | — | SAST, dependency scanning, updates | Free for public/private repos with Advanced Security on public; Renovate gives grouped PRs. |
| Trivy | 0.6x | Container image vulnerability scanning | Fast, free, SARIF output to GitHub. |
| gitleaks | 8.x | Secret scanning in CI and pre-commit | Blocks committed secrets. |
| Grafana Cloud (free/pro tier) | — | Logs (Loki), metrics (Prometheus/Mimir), traces (Tempo), dashboards, alerting | Accepts OTLP directly; one place for all telemetry; Datadog is costlier. |
| Better Stack Uptime (or Grafana Synthetic Monitoring) | — | External uptime checks and status page | Measures NFR-AV1 independently of the hosting provider. |

## 5. Testing and quality

| Technology | Version | Purpose | Why chosen over alternatives |
|---|---|---|---|
| Jest + ts-jest/SWC | 30.x | API unit and integration tests | NestJS default testing setup and docs. |
| Supertest | 7.x | HTTP integration tests against Nest app | Standard with Nest. |
| Testcontainers (Node) | 11.x | Real PostgreSQL and Redis in integration tests | Tests run against real engines (CHECK constraints, tsvector, roles) instead of mocks. |
| Vitest | 3.x | Web and shared package unit tests | Fast, Vite-native, Jest-compatible API. |
| React Testing Library | 16.x | Component tests | Tests behavior via accessible queries. |
| MSW (Mock Service Worker) | 2.x | API mocking in frontend tests | Same mocks in tests and Storybook-free local dev. |
| Playwright | 1.5x | End-to-end tests (Chromium, Firefox, WebKit, mobile viewport) | Reliable auto-waiting, trace viewer, parallelism. Cypress lacks WebKit. |
| @axe-core/playwright | 4.10 | Automated accessibility checks | WCAG 2.2 rule set in E2E runs. |
| k6 | 1.x | Load tests | Scriptable in JS, thresholds map to NFRs, CI-friendly. |
| OWASP ZAP (baseline scan, Docker) | 2.16 | DAST against staging | Free, GitHub Action available. |
| Lighthouse CI | 0.15 | Performance budgets | Enforces NFR-P3/P5 on PRs. |

## 6. Developer tooling

| Technology | Version | Purpose |
|---|---|---|
| ESLint (flat config) + typescript-eslint | 9.x / 8.x | Linting |
| eslint-plugin-boundaries | 5.x | Enforces module boundaries (NFR-M1) |
| eslint-plugin-jsx-a11y | 6.x | Accessibility linting |
| Prettier (+ prettier-plugin-tailwindcss) | 3.x | Formatting |
| Husky + lint-staged | 9.x / 16.x | Pre-commit lint/format, gitleaks |
| commitlint (`@commitlint/config-conventional`) | 19.x | Conventional Commits enforcement |
| tsx | 4.x | Running TS scripts (seed, maintenance) |
| Bull Board | 6.x | Queue dashboard (staging, admin-only) |

## 7. Explicitly not used in MVP

| Technology | Reason |
|---|---|
| Microservices / Kubernetes | Out of scope (plan §17); modular monolith instead. |
| OpenSearch / Elasticsearch | PostgreSQL search first (plan §10); trigger in ARCHITECTURE §9. |
| GraphQL | REST is the plan's choice (§10) and simpler to rate-limit and cache. |
| WebSockets / real-time chat | Out of scope (plan §17); 60 s polling for notification count. |
| Third-party analytics trackers (GA, Mixpanel) | Privacy-by-design; product metrics derived server-side from the database and logs. |
| ML models | Out of scope; rules-based safety checks. |
| JWT access tokens | Replaced by server sessions (revocation, simplicity). |
