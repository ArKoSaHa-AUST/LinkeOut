# CLAUDE.md — LinkedOut

Context for AI coding assistants (and humans) working in this repository.

## ⚠️ Before you write any code

1. **Always read `/docs` before coding.** At minimum, read the docs relevant to your task: [PRD](docs/PRD.md), [FEATURES](docs/FEATURES.md), [API_SPEC](docs/API_SPEC.md), [DATABASE_SCHEMA](docs/DATABASE_SCHEMA.md), [SECURITY](docs/SECURITY.md), [UI_UX](docs/UI_UX.md), [WORKFLOW](docs/WORKFLOW.md). The master plan [docs/LinkedOut_Detailed_Product_and_Engineering_Plan.md](docs/LinkedOut_Detailed_Product_and_Engineering_Plan.md) is the source of truth; never modify it.
2. **Follow [docs/TASKS.md](docs/TASKS.md) in order.** Take the first unticked task whose earlier tasks in that section are done. Don't jump ahead or bundle unrelated tasks.
3. **After each task:** tick it in `docs/TASKS.md` (`- [x]`) and add an entry under *Unreleased* in `docs/CHANGELOG.md` with the task ID.
4. If a doc and the code disagree, or a task needs a decision the docs don't cover, stop and ask. When a decision is made, record it in [docs/ASSUMPTIONS.md](docs/ASSUMPTIONS.md).

## Project summary

LinkedOut ("Your career. Your chaos.") is an anonymous-first professional community where members share career setbacks, workplace experiences and lessons. Core concepts:
- **Pseudonym vs anonymous.** Every member has a public username, separate from the private account. Confessions and job reports can be *anonymous to members*: each gets a per-content label ("Anonymous Stapler #4821"), and the label ↔ account mapping lives **only** in the `vault` PostgreSQL schema. Never claim absolute anonymity.
- **Features:** The Mess (feed), Anti-Profile, Office Confessions, Learned the Hard Way, Workplace Reality Pages (reviews), Job Nightmare Board, search, reactions (I'm Dead, Pain, Same Here, Melting, Corporate Circus, Survived), reporting/moderation/appeals, admin. v1.1 adds communities, notifications, polls, company responses, images, following, export, Google sign-in.
- **Roles:** Visitor, Member, Community moderator, Platform moderator (+ T&S officer permission `identity_vault.reveal`), Company representative, Administrator.

## Stack and versions

| Area | Choice |
|---|---|
| Language/runtime | TypeScript 5.9, Node.js 24 LTS, pnpm 10, Turborepo 2 |
| Web | Next.js 16 App Router, React 19.2, Tailwind CSS 4.1, shadcn/ui + Radix, TanStack Query 5, React Hook Form 7, Zod 4 |
| API | NestJS 11 (Express), Prisma 6, BullMQ 5, ioredis 5, argon2, pino, OpenTelemetry, Sentry |
| Data | PostgreSQL 17 (`public` + `vault` schemas), Redis 7.4/Valkey 8, Cloudflare R2 (MinIO locally) |
| Tests | Jest 30 + Supertest + Testcontainers (API), Vitest 3 + RTL (web), Playwright + axe (E2E), k6 (load) |
| Hosting | Vercel (web), Render Frankfurt (API, worker, Postgres, Key Value), Cloudflare, Resend |

## Folder structure (short)

```text
apps/web/            Next.js app (src/app routes, components, lib/api client, hooks, content/en.json)
apps/api/            NestJS: src/main.ts (HTTP), src/worker.ts (jobs), src/modules/<module>/, src/common/, prisma/
packages/shared/     Zod schemas, enums, constants: the single source of truth for API contracts
packages/ui-tokens/  Design tokens → CSS variables
packages/email-templates/  React Email templates
infra/               docker-compose, Dockerfiles, render.yaml, Grafana
docs/                All documentation (read first)
```

Full tree: [docs/FOLDER_STRUCTURE.md](docs/FOLDER_STRUCTURE.md).

API modules: `identity`, `profile`, `content`, `feed`, `confession`, `engagement`, `company`, `jobreport`, `search`, `moderation`, `safety`, `vault`, `audit`, `admin`, `community`, `notification`, `media`, `analytics`, `maintenance`, `health`.

## Commands

| Purpose | Command |
|---|---|
| Install | `pnpm install` |
| Local infra | `pnpm infra:up` / `pnpm infra:down` |
| DB | `pnpm db:migrate`, `pnpm db:migrate:deploy`, `pnpm db:seed`, `pnpm db:reset`, `pnpm db:generate` |
| Dev (all) | `pnpm dev` (web :3000, api :4000, worker) |
| Dev (one) | `pnpm dev:web`, `pnpm dev:api`, `pnpm dev:worker` |
| Unit tests | `pnpm test` |
| Integration tests | `pnpm test:int` (needs Docker) |
| Security tests | `pnpm test:sec` |
| E2E | `pnpm test:e2e` |
| Coverage | `pnpm test:cov` |
| Lint / format / types | `pnpm lint`, `pnpm lint:fix`, `pnpm format`, `pnpm typecheck` |
| Build | `pnpm build` |
| Env check | `pnpm check-env` |

Before declaring a task done, run `pnpm lint && pnpm typecheck && pnpm test` plus `pnpm test:int` for API changes and `pnpm test:e2e` for changed journeys.

## Coding conventions

- **Modular monolith.** A module's public surface is its `index.ts`. Never import another module's internals or query its tables; call its service facade. Only `*.repository.ts` files touch Prisma.
- **Controllers are thin:** routing, guards, validation. Business rules and authorization live in services.
- **Validation:** request schemas come from `packages/shared/src/schemas`, and the web uses the same schemas in forms. Don't duplicate them.
- **Errors:** throw `DomainError(code, message, details?)` using codes from API_SPEC §1.5. The global filter formats them.
- **Responses:** always map through serializers in `src/common/serializers`. Never return Prisma entities directly.
- **Pagination:** cursor (keyset) only, default 20, max 50.
- **Transactions** for multi-row writes and counter updates. The vault write happens **before** the public insert, with compensation (ARCHITECTURE §4.3).
- **Background work** (email, notifications, scans, deletion) goes through BullMQ queues, never inline in requests.
- **Frontend:** Server Components by default; `"use client"` only for interactivity; data via `lib/api`; Tailwind classes with tokens only (no raw hex); every screen has loading, empty and error states (UI_UX Part B).
- **Accessibility:** WCAG 2.2 AA. Use semantic elements, labels, visible focus, keyboard support and `aria-live` for async status.
- **Copy:** witty but never cruel; no manipulative notifications; no "untraceable" claims (UI_UX C.7). Strings go in `content/en.json`.
- **Migrations:** expand/contract only, backward-compatible with the previous release. Hand-written SQL for grants, triggers and generated columns.
- **Tests:** every endpoint gets integration tests for success, validation, authorization and rate limits. New public response fields must pass the contract test (no private fields).

## Naming rules

| Thing | Convention | Example |
|---|---|---|
| Files (API) | kebab-case + role suffix | `moderation-items.service.ts` |
| React components | PascalCase file and export | `ReactionBar.tsx` |
| Hooks | `useX` camelCase | `useSafetyCheck.ts` |
| Variables/functions | camelCase | `createAlias` |
| Classes/types/interfaces | PascalCase, no `I` prefix | `PostSummary` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_TAGS_PER_POST` |
| Enums values | snake_case strings matching the DB | `open_to_escape`, `same_here` |
| DB tables/columns | snake_case, plural tables | `workplace_reviews.relationship_type` |
| Prisma models | PascalCase singular + `@@map` | `WorkplaceReview` |
| API paths | kebab-case plural nouns under `/api/v1` | `/job-reports/:jobReportId` |
| JSON fields | camelCase | `publishedAt` |
| Queues | kebab-case | `account-deletion` |
| Audit actions | `module.past_tense_event` | `moderation.action_applied` |
| Feature flags | `area.flag` | `confessions.enabled` |
| Branches | `type/T-ID-slug` | `feat/T-045-register-endpoint` |
| Commits | Conventional Commits | `feat(identity): add login lockout` |

## Do's

- ✅ Read the relevant docs and the task's linked FR/US IDs before coding.
- ✅ Deny by default: every route needs a guard or an explicit `@Public()`.
- ✅ Derive identity from the session only. Re-check ownership in services.
- ✅ Return `404` for resources the caller may not see.
- ✅ Run free text through `SafetyCheckService` where the docs require it.
- ✅ Use `IdentityVaultService` for anything involving anonymous authorship.
- ✅ Add rate limits to new write endpoints (API_SPEC §1.7).
- ✅ Write audit entries for role changes, moderation, vault access and admin actions.
- ✅ Keep docs in sync. Update API_SPEC, DATABASE_SCHEMA, ENV_SETUP and the rest when behavior changes.
- ✅ Tick TASKS.md and update CHANGELOG.md after each task.

## Don'ts

- ❌ Don't modify `docs/LinkedOut_Detailed_Product_and_Engineering_Plan.md`.
- ❌ Don't expose email, internal user IDs, IPs, alias IDs or session data in public responses.
- ❌ Don't log user IDs together with anonymous content IDs, or log request bodies, tokens, cookies or emails.
- ❌ Don't query the `vault` schema with the default Prisma client, and don't add vault grants to `lo_app`.
- ❌ Don't add public counts of reactions received, failures or rejections to profiles, and don't build leaderboards.
- ❌ Don't enable `confessions.enabled` in production before Phases 5 and 6 are complete.
- ❌ Don't use `$queryRawUnsafe`, `dangerouslySetInnerHTML` or raw HTML rendering of user content.
- ❌ Don't add third-party trackers or send personal data to external services that aren't listed in ARCHITECTURE §3.4.
- ❌ Don't introduce new frameworks, a second backend stack, microservices, WebSockets or ML without an ASSUMPTIONS.md entry approved by the owner.
- ❌ Don't commit secrets or `.env` files. Don't skip hooks (`--no-verify`).
- ❌ Don't write placeholder code or `TODO`s without a linked task ID.
