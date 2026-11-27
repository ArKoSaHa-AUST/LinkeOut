# LinkedOut — Folder Structure

LinkedOut is a **pnpm + Turborepo monorepo**. This is the target structure. Folders appear as the tasks in [TASKS.md](TASKS.md) create them; no folder is created empty.

## 1. Repository root

```text
linkedout/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                  # Lint, typecheck, unit, integration, build, E2E on PRs
│   │   ├── deploy-staging.yml      # On merge to main: migrate + deploy staging
│   │   ├── deploy-production.yml   # On tag v*: manual approval, migrate + deploy prod
│   │   ├── security.yml            # CodeQL, Trivy, gitleaks, ZAP baseline (nightly)
│   │   └── load-test.yml           # Manual k6 runs against staging
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml
│   │   └── feature_request.yml
│   ├── pull_request_template.md
│   ├── CODEOWNERS
│   └── renovate.json
├── .husky/
│   ├── pre-commit                  # lint-staged + gitleaks protect --staged
│   └── commit-msg                  # commitlint
├── apps/
│   ├── web/                        # Next.js frontend (see §2)
│   └── api/                        # NestJS API + worker (see §3)
├── packages/
│   ├── shared/                     # Zod schemas, enums, types, constants (see §4)
│   ├── ui-tokens/                  # Design tokens (colors, type, spacing) as TS + CSS
│   ├── email-templates/            # React Email templates
│   ├── eslint-config/              # Shared ESLint flat configs (base, react, nest, boundaries)
│   └── tsconfig/                   # Shared tsconfig bases
├── infra/
│   ├── docker/
│   │   ├── api.Dockerfile          # Multi-stage build for api + worker image
│   │   └── clamav.Dockerfile       # ClamAV daemon image (Should)
│   ├── docker-compose.yml          # Local Postgres, Redis, MinIO, Mailpit, ClamAV
│   ├── postgres/
│   │   └── init/01-roles.sql       # Creates lo_migrator, lo_app, lo_vault, lo_readonly locally
│   ├── render.yaml                 # Render blueprint (api, worker, db, key value)
│   └── grafana/
│       ├── dashboards/             # JSON dashboards (API, queues, moderation SLA)
│       └── alerts.yaml             # Alert rules
├── load-tests/
│   ├── feed.k6.js
│   ├── post-create.k6.js
│   ├── comment-burst.k6.js
│   ├── search.k6.js
│   ├── notifications.k6.js
│   └── media-upload.k6.js
├── scripts/
│   ├── setup.sh                    # One-shot local setup
│   ├── check-env.ts                # Validates .env against schema
│   └── restore-test.sh             # Backup restore verification (staging)
├── docs/                           # All documentation (this folder)
│   └── runbooks/                   # Operational runbooks (DEPLOYMENT §11)
├── .editorconfig
├── .env.example
├── .gitignore
├── .nvmrc                          # 24
├── .prettierrc
├── commitlint.config.cjs
├── package.json                    # Root scripts (dev, build, test, lint, db:*)
├── pnpm-workspace.yaml
├── turbo.json
├── CLAUDE.md
├── CONTRIBUTING.md -> docs/CONTRIBUTING.md   # Symlink so GitHub shows it
└── README.md
```

## 2. Frontend — `apps/web`

> **Frontend-only phase:** the four-person frontend build uses the feature-folder layout and ownership map in [team/TEAM_CONTRACT.md §3](team/TEAM_CONTRACT.md#3-folder-structure-and-ownership) (ASSUMPTIONS A-70). The tree below is the long-term reference. Components listed here under `src/components/<domain>/` live in `src/features/<feature>/components/` during that phase.

```text
apps/web/
├── public/
│   ├── favicon.svg
│   ├── og-default.png              # Social preview image
│   └── robots.txt
├── src/
│   ├── app/                        # Next.js App Router
│   │   ├── layout.tsx              # Root layout: fonts, theme, <AppShell>, GET /me
│   │   ├── globals.css             # Tailwind + token CSS variables
│   │   ├── not-found.tsx           # S-38 404
│   │   ├── error.tsx               # S-38 error boundary
│   │   ├── (marketing)/
│   │   │   ├── page.tsx            # S-01 Landing (/)
│   │   │   └── legal/
│   │   │       ├── privacy/page.tsx
│   │   │       ├── terms/page.tsx
│   │   │       └── guidelines/page.tsx
│   │   ├── (auth)/
│   │   │   ├── signup/page.tsx             # S-02
│   │   │   ├── signup/complete/page.tsx    # S-02b OAuth username step (Should)
│   │   │   ├── login/page.tsx              # S-03
│   │   │   ├── verify-email/page.tsx       # S-04
│   │   │   ├── forgot-password/page.tsx    # S-05
│   │   │   └── reset-password/page.tsx     # S-05
│   │   ├── onboarding/page.tsx             # S-06
│   │   ├── (app)/                          # Authenticated-or-public app shell with nav
│   │   │   ├── mess/page.tsx               # S-07 The Mess feed
│   │   │   ├── post/new/page.tsx           # S-08 Create post (story/advice/lesson/poll)
│   │   │   ├── posts/[postId]/page.tsx     # S-11 Post detail & comments
│   │   │   ├── confessions/page.tsx        # S-10 Confessions list
│   │   │   ├── confessions/new/page.tsx    # S-09 Confession submission
│   │   │   ├── lessons/page.tsx            # S-14 Learned the Hard Way
│   │   │   ├── companies/page.tsx          # S-15 Company directory
│   │   │   ├── companies/new/page.tsx      # S-18 Submit company
│   │   │   ├── companies/[slug]/page.tsx   # S-16 Workplace Reality Page
│   │   │   ├── companies/[slug]/review/page.tsx      # S-17 Write review
│   │   │   ├── companies/[slug]/represent/page.tsx   # S-32 Representative request (Should)
│   │   │   ├── job-nightmares/page.tsx     # S-19 Board
│   │   │   ├── job-nightmares/new/page.tsx # S-20 Report form
│   │   │   ├── job-nightmares/[jobReportId]/page.tsx
│   │   │   ├── search/page.tsx             # S-21
│   │   │   ├── communities/page.tsx        # S-33 (Should)
│   │   │   ├── communities/new/page.tsx    # S-35 (Should)
│   │   │   ├── communities/[slug]/page.tsx # S-34 (Should)
│   │   │   ├── notifications/page.tsx      # S-36 (Should)
│   │   │   ├── u/[username]/page.tsx       # S-12 Public Anti-Profile
│   │   │   ├── me/page.tsx                 # S-13 My Damage (private dashboard)
│   │   │   ├── reports/page.tsx            # S-22 Report & appeal center
│   │   │   └── settings/
│   │   │       ├── layout.tsx              # Settings side nav
│   │   │       ├── profile/page.tsx        # S-25
│   │   │       ├── privacy/page.tsx        # S-26
│   │   │       ├── sessions/page.tsx       # S-27
│   │   │       ├── blocked/page.tsx        # S-28
│   │   │       ├── account/page.tsx        # S-29 Account & data (deletion, export)
│   │   │       └── notifications/page.tsx  # S-37 (Should)
│   │   ├── mod/                            # Moderator area (role-gated)
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx                    # S-23 Queue
│   │   │   ├── items/[itemId]/page.tsx     # S-24 Item detail
│   │   │   └── appeals/page.tsx            # S-23 Appeals tab
│   │   └── admin/                          # Admin area (role-gated)
│   │       ├── layout.tsx
│   │       ├── page.tsx                    # S-30 Metrics
│   │       ├── users/page.tsx              # S-31
│   │       ├── categories/page.tsx         # S-31
│   │       ├── companies/page.tsx          # S-31
│   │       ├── representatives/page.tsx    # S-31 (Should)
│   │       ├── escalations/page.tsx        # S-31
│   │       ├── feature-flags/page.tsx      # S-31
│   │       └── audit/page.tsx              # S-31
│   ├── components/
│   │   ├── ui/                     # shadcn/ui primitives (button, dialog, tabs, toast, …)
│   │   ├── layout/                 # AppShell, TopNav, SideNav, MobileTabBar, Footer
│   │   ├── feed/                   # FeedList, FeedFilters, PostCard, ContentWarningGate
│   │   ├── post/                   # Composer, MarkdownEditor, LessonFields, PollFields, PostBody, Corrections
│   │   ├── confession/             # ConfessionWizard, IdentityModePicker, AnonymousBadge
│   │   ├── engagement/             # ReactionBar, ReactionPicker, CommentThread, CommentForm, SaveButton
│   │   ├── safety/                 # SafetyCheckPanel, FindingHighlight, ReportDialog, BlockMenu
│   │   ├── company/                # CompanyHeader, ThemeSummary, ReviewCard, ReviewForm, JobReportCard
│   │   ├── profile/                # ProfileHeader, LessonsList, SkillsList, Timeline
│   │   ├── community/              # CommunityCard, RulesPanel, PinnedResources (Should)
│   │   ├── moderation/             # QueueTable, ItemContext, ActionForm, AppealCard, RevealPanel
│   │   └── common/                 # EmptyState, ErrorState, Skeletons, Pagination, Avatar, RelativeTime
│   ├── lib/
│   │   ├── api/
│   │   │   ├── client.ts           # fetch wrapper: credentials, CSRF header, error parsing
│   │   │   ├── server.ts           # Server-side fetch forwarding cookies
│   │   │   └── endpoints/          # Typed functions per API group (auth.ts, posts.ts, …)
│   │   ├── auth/session.ts         # getMe() helpers, role checks for UI
│   │   ├── markdown.tsx            # react-markdown + sanitize config
│   │   ├── query-client.ts         # TanStack Query setup
│   │   └── utils.ts                # cn(), formatters
│   ├── hooks/                      # useFeed, useReaction, useSafetyCheck, useUnreadCount, …
│   ├── content/                    # UI copy (en.json) — externalized strings (NFR-I1)
│   ├── middleware.ts               # Redirects unauthenticated users from member routes (UX only)
│   └── styles/tokens.css           # Generated from packages/ui-tokens
├── tests/
│   ├── unit/                       # Vitest + RTL component tests (*.test.tsx)
│   └── e2e/                        # Playwright specs (*.spec.ts), fixtures, axe helpers
├── next.config.ts                  # Security headers, CSP, images config
├── playwright.config.ts
├── vitest.config.ts
├── postcss.config.mjs
├── tsconfig.json
└── package.json
```

## 3. Backend — `apps/api`

```text
apps/api/
├── prisma/
│   ├── schema.prisma               # Models for public + vault schemas (multiSchema)
│   ├── migrations/                 # Timestamped SQL migrations (incl. roles grants, triggers)
│   └── seed/
│       ├── index.ts                # Entry: reference data (all envs) + dev data (dev/staging)
│       ├── reference.ts            # Categories, industries, reserved usernames, feature flags
│       └── dev-fixtures.ts         # Users, companies, posts, reviews, reports
├── src/
│   ├── main.ts                     # HTTP bootstrap: helmet, CORS, pino, OTel, versioning, Swagger
│   ├── worker.ts                   # Worker bootstrap: registers BullMQ processors and repeatable jobs
│   ├── app.module.ts               # Root module importing all feature modules
│   ├── worker.module.ts            # Worker root module (processors only)
│   ├── config/
│   │   ├── env.schema.ts           # Zod schema for environment variables
│   │   └── config.module.ts
│   ├── common/
│   │   ├── prisma/                 # PrismaService (lo_app), VaultPrismaService (lo_vault)
│   │   ├── redis/                  # Redis client provider
│   │   ├── queue/                  # Queue names, producers, BullMQ module setup
│   │   ├── guards/                 # SessionGuard, VerifiedGuard, RolesGuard, CommunityRoleGuard,
│   │   │                           # RepresentativeGuard, VaultPermissionGuard, CsrfGuard
│   │   ├── decorators/             # @CurrentUser, @Roles, @Public, @RequireVerified, @FeatureFlag
│   │   ├── filters/                # AllExceptionsFilter → standard error format
│   │   ├── interceptors/           # RequestId, Idempotency, Serialization
│   │   ├── pagination/             # Cursor encode/decode helpers
│   │   ├── serializers/            # Public DTO mappers (AuthorRef, PostSummary, …)
│   │   ├── throttling/             # Throttler config + Redis storage + named limits
│   │   ├── logging/                # pino config + redaction paths
│   │   └── telemetry/              # OpenTelemetry SDK init
│   ├── modules/
│   │   ├── identity/               # F-01, F-02, F-04, F-08, F-09
│   │   │   ├── identity.module.ts
│   │   │   ├── index.ts            # Public facade exports only
│   │   │   ├── auth.controller.ts
│   │   │   ├── me.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── session.service.ts
│   │   │   ├── password.service.ts         # argon2 + HIBP check
│   │   │   ├── token.service.ts
│   │   │   ├── oauth.service.ts            # (Should)
│   │   │   ├── deletion.service.ts
│   │   │   ├── export.service.ts           # (Should)
│   │   │   ├── processors/                 # account-deletion, data-export processors
│   │   │   ├── dto/
│   │   │   └── __tests__/
│   │   ├── profile/                # F-03, F-05, F-06
│   │   ├── content/                # F-10, F-16, F-28 (posts, lessons, polls, taxonomy)
│   │   ├── feed/                   # F-11, F-30
│   │   ├── confession/             # F-15
│   │   ├── engagement/             # F-12, F-13, F-14
│   │   ├── company/                # F-17, F-25
│   │   ├── jobreport/              # F-18
│   │   ├── search/                 # F-19
│   │   ├── safety/                 # F-07 blocks/mutes + SafetyCheckService and rules/
│   │   │   └── rules/              # pii.rule.ts, secrets.rule.ts, named-individual.rule.ts,
│   │   │                           # threats.rule.ts, interview-dump.rule.ts, velocity.rule.ts
│   │   ├── moderation/             # F-20, F-21, F-22 (reports, items, actions, appeals, campaigns)
│   │   ├── vault/                  # F-23 IdentityVaultService, reveal workflow, alias generator
│   │   ├── audit/                  # F-23 AuditService (append-only)
│   │   ├── admin/                  # F-24 controllers orchestrating other facades, feature flags
│   │   ├── community/              # F-26 (Should)
│   │   ├── notification/           # F-27 (Should) + email processor
│   │   ├── media/                  # F-29 (Should) + media-scan processor
│   │   ├── analytics/              # Admin metrics aggregates
│   │   ├── maintenance/            # Repeatable jobs: purge, retention, reconciliation
│   │   └── health/                 # /health/live, /health/ready
│   └── types/                      # Express request augmentation (req.user, req.session)
├── test/
│   ├── integration/                # Supertest + Testcontainers specs (*.int-spec.ts)
│   ├── security/                   # IDOR, identity-leak, CSRF, rate-limit specs (*.sec-spec.ts)
│   ├── contract/                   # Public DTO "no private fields" contract tests
│   ├── fixtures/                   # Factories for users, posts, companies
│   └── setup/                      # Global Testcontainers setup/teardown
├── jest.config.ts
├── nest-cli.json
├── tsconfig.json
├── tsconfig.build.json
└── package.json
```

**Module internal convention** (every folder under `src/modules/<name>/`):

| File / folder | Purpose |
|---|---|
| `<name>.module.ts` | Nest module definition; imports only other modules' exported facades. |
| `index.ts` | The only import surface for other modules (`export { PostsService } from './posts.service'`). |
| `*.controller.ts` | HTTP layer: routing, guards, DTO validation, calls services. No business logic. |
| `*.service.ts` | Business logic and authorization rules. |
| `*.repository.ts` | Prisma queries for this module's tables (only file that touches Prisma). |
| `dto/` | Zod-based DTOs re-exported from `packages/shared` plus response mappers. |
| `processors/` | BullMQ processors (loaded by `worker.module.ts` only). |
| `events.ts` | Domain event names and payload types emitted by the module. |
| `__tests__/` | Unit tests for services/rules (`*.spec.ts`). |

## 4. Shared packages

```text
packages/shared/
├── src/
│   ├── enums.ts                    # All enums mirroring DATABASE_SCHEMA §3 (single source)
│   ├── constants.ts                # Limits (title 150, body 20000, tags 5, …), reaction metadata
│   ├── schemas/                    # Zod request/response schemas per API group
│   │   ├── auth.ts
│   │   ├── profile.ts
│   │   ├── posts.ts
│   │   ├── comments.ts
│   │   ├── confessions.ts
│   │   ├── companies.ts
│   │   ├── job-reports.ts
│   │   ├── reports.ts
│   │   ├── moderation.ts
│   │   ├── communities.ts
│   │   ├── notifications.ts
│   │   ├── admin.ts
│   │   └── common.ts               # Pagination, error envelope, AuthorRef
│   ├── copy/                       # Shared user-facing strings used by API emails and web
│   └── index.ts
├── package.json
└── tsconfig.json

packages/ui-tokens/
├── src/tokens.ts                   # Colors, typography, spacing, radii, shadows, breakpoints
├── build.ts                        # Emits tokens.css (CSS variables) for apps/web
└── package.json

packages/email-templates/
├── src/
│   ├── VerifyEmail.tsx
│   ├── PasswordReset.tsx
│   ├── LoginLocked.tsx
│   ├── RegistrationAttempt.tsx
│   ├── ModerationDecision.tsx
│   ├── ReportOutcome.tsx
│   ├── AppealOutcome.tsx
│   ├── DeletionScheduled.tsx
│   ├── DraftExpiryReminder.tsx
│   ├── ExportReady.tsx             # (Should)
│   ├── RepresentativeVerify.tsx    # (Should)
│   └── NotificationDigest.tsx      # (Should)
└── package.json
```

## 5. Documentation — `docs/`

```text
docs/
├── LinkedOut_Detailed_Product_and_Engineering_Plan.md   # Master plan (source of truth, read-only)
├── PRD.md
├── USER_STORIES.md
├── FEATURES.md
├── WORKFLOW.md
├── ARCHITECTURE.md
├── TECH_STACK.md
├── DATABASE_SCHEMA.md
├── API_SPEC.md
├── UI_UX.md
├── FOLDER_STRUCTURE.md
├── SECURITY.md
├── TESTING.md
├── ENV_SETUP.md
├── DEPLOYMENT.md
├── ROADMAP.md
├── TASKS.md
├── CONTRIBUTING.md
├── ASSUMPTIONS.md
├── CHANGELOG.md
└── runbooks/                        # Created in TASKS Phase 7 (one file per runbook in DEPLOYMENT §11)
```

## 6. Naming conventions for files

| Kind | Convention | Example |
|---|---|---|
| React components | PascalCase `.tsx` | `ReactionBar.tsx` |
| Hooks | camelCase starting with `use` | `useSafetyCheck.ts` |
| Next.js routes | Next.js conventions (`page.tsx`, `layout.tsx`), kebab-case folders | `job-nightmares/new/page.tsx` |
| Nest files | kebab-case with role suffix | `moderation-items.service.ts` |
| Tests | `*.spec.ts` (unit), `*.int-spec.ts` (integration), `*.sec-spec.ts` (security), `*.spec.ts` under `tests/e2e` (Playwright), `*.test.tsx` (web unit) | `vault.service.spec.ts` |
| Prisma migrations | Prisma default timestamp + snake_case name | `20261214093000_create_identity_tables` |
