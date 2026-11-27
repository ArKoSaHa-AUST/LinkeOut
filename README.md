# LinkedOut

**Your career. Your chaos.**

LinkedOut is an anonymous-first professional community and workplace transparency platform. It's the place to share the professional reality behind the polished profile: honestly, safely, and with a sense of humor.

> LinkedIn shows the promotion. LinkedOut tells the story behind the promotion.

**Project status:** documentation phase (`v0.1.0`). Application code has not been written yet. Implementation follows [docs/TASKS.md](docs/TASKS.md) starting 30 November 2026, with the public launch targeted for 18 March 2027 ([docs/ROADMAP.md](docs/ROADMAP.md)).

---

## The problem

- Professional networks reward highlight reels. People facing rejection, layoffs, burnout or a bad first job see only success stories and feel alone.
- Being candid under a real-name professional identity carries real career risk, so people stay silent or vent in unmoderated spaces where identities leak and harassment thrives.
- Candidates can't easily find structured, contextual information about how companies hire and treat people, such as ghosting, unpaid test assignments or offers changed at the last minute.
- Hard-won career lessons stay in private chats instead of helping the next person.

## The solution

A pseudonymous community where members can:
- share setbacks, ask for advice and turn mistakes into structured lessons;
- confess workplace stories **anonymously to other members**, with pre-publication privacy checks and a preview;
- read and write **structured, clearly labelled** workplace reviews and recruitment reports;
- support each other with reactions like *Same Here* and *Survived*, instead of competing over who suffered most.

All of this is protected by a strict separation between private accounts and public identities, human moderation with appeals, and an honest promise: **we never claim absolute anonymity**.

## Key features

| Feature | Release |
|---|---|
| Email sign-up with a public pseudonym, separate from the private account | MVP |
| Onboarding that explains pseudonymity, anonymity and data use in plain language | MVP |
| **Anti-Profile**: headline, lessons, skills still under development, career survival timeline, "Open to Escape" status, and no suffering scores | MVP |
| **The Mess**: the main feed of stories and advice requests, with categories, tags, content warnings and filters | MVP |
| Threaded comments and six reactions: I'm Dead, Pain, Same Here, Melting, Corporate Circus, Survived | MVP |
| **Office Confessions**: anonymous-to-members stories with safety check, preview and corrections | MVP |
| **Learned the Hard Way**: structured lessons from mistakes | MVP |
| **Workplace Reality Pages**: company pages with unverified firsthand reviews and minimum-sample theme summaries | MVP |
| **Job Nightmare Board**: recruitment experience reports (ghosting, unpaid tests, withdrawn offers, and good experiences too) | MVP |
| Search, saved posts, block and mute, privacy settings, account deletion | MVP |
| Reporting, moderation queue, appeals, audit logs, four-eyes Identity Vault, admin dashboard | MVP |
| Communities ("Trauma Network"), notifications, polls, company representative responses, image posts, topic following, data export, Google sign-in | v1.1 |
| Direct messaging, Career Damage Report, meme generator, premium, mobile apps and more | Future |

Full catalogue: [docs/FEATURES.md](docs/FEATURES.md).

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16 (App Router), React 19, TypeScript 5.9, Tailwind CSS 4, shadcn/ui + Radix, TanStack Query, React Hook Form, Zod |
| Backend | NestJS 11 on Node.js 24 LTS (modular monolith: API + worker), Prisma 6 |
| Data | PostgreSQL 17 (with an isolated `vault` schema), Redis-compatible store (sessions, rate limits, cache, BullMQ queues), Cloudflare R2 |
| Auth | Server-side sessions with httpOnly cookies, Argon2id, CSRF double-submit tokens |
| Infra | Docker, Vercel (web), Render Frankfurt (API, worker, DB, Key Value), Cloudflare (DNS, WAF), Resend (email) |
| Quality | Jest, Vitest, Testcontainers, Playwright + axe, k6, CodeQL, Trivy, gitleaks |
| Observability | OpenTelemetry → Grafana Cloud, Sentry, Better Stack uptime |

Details and rationale: [docs/TECH_STACK.md](docs/TECH_STACK.md).

## Getting started

These steps work once the Phase 2 foundation tasks (T-018 – T-068) are complete. Full guide: [docs/ENV_SETUP.md](docs/ENV_SETUP.md).

### Prerequisites
Node.js 24 LTS, pnpm 10 (via corepack), Docker with Compose v2, Git, OpenSSL.

### Setup

```bash
git clone git@github.com:linkedout-app/linkedout.git
cd linkedout
corepack enable && pnpm install
cp .env.example .env            # then generate CSRF_SECRET and VAULT_ENCRYPTION_KEY (ENV_SETUP §3)
pnpm check-env
pnpm infra:up                   # Postgres, Redis, MinIO, Mailpit
pnpm db:migrate && pnpm db:seed
```

### Run

```bash
pnpm dev                        # web :3000, API :4000, worker
```

| URL | Purpose |
|---|---|
| http://localhost:3000 | Web app |
| http://localhost:4000/api/v1/docs | API docs (Swagger) |
| http://localhost:8025 | Mailpit (verification and reset emails) |

Dev login: `priya@linkedout.local` / `Password-For-Dev-Only-1` (more accounts in ENV_SETUP §3).

### Common commands

| Command | Purpose |
|---|---|
| `pnpm test` / `pnpm test:int` / `pnpm test:e2e` | Unit / integration / end-to-end tests |
| `pnpm lint` / `pnpm typecheck` / `pnpm format` | Code quality |
| `pnpm build` | Production build |
| `pnpm db:reset` | Recreate and reseed the local database |

## Folder overview

```text
linkedout/
├── apps/
│   ├── web/          # Next.js frontend
│   └── api/          # NestJS API + background worker, Prisma schema and migrations
├── packages/
│   ├── shared/       # Zod schemas, enums, constants shared by web and API
│   ├── ui-tokens/    # Design tokens
│   ├── email-templates/
│   ├── eslint-config/
│   └── tsconfig/
├── infra/            # Docker, docker-compose, Render blueprint, Grafana dashboards
├── load-tests/       # k6 scripts
├── scripts/          # Setup and maintenance scripts
├── docs/             # All product and engineering documentation
├── CLAUDE.md         # Conventions for AI-assisted (and human) development
└── README.md
```

Full tree with explanations: [docs/FOLDER_STRUCTURE.md](docs/FOLDER_STRUCTURE.md).

## Documentation

| Document | What it covers |
|---|---|
| [Master plan](docs/LinkedOut_Detailed_Product_and_Engineering_Plan.md) | Original product and engineering plan (source of truth) |
| [PRD](docs/PRD.md) | Goals, personas, scope, functional and non-functional requirements, KPIs, risks |
| [User stories](docs/USER_STORIES.md) | Stories by role with Given/When/Then acceptance criteria |
| [Features](docs/FEATURES.md) | Feature catalogue with priorities, roles, stories, requirements, dependencies |
| [Workflows](docs/WORKFLOW.md) | Step-by-step flows with diagrams and backend steps |
| [Architecture](docs/ARCHITECTURE.md) | System design, modules, data and auth flows, deployment, scalability |
| [Tech stack](docs/TECH_STACK.md) | Every technology, version and rationale |
| [Database schema](docs/DATABASE_SCHEMA.md) | Tables, constraints, indexes, ER diagrams, sample data |
| [API spec](docs/API_SPEC.md) | Every endpoint with requests, responses and errors |
| [UI/UX](docs/UI_UX.md) | Screens, states, design system, accessibility |
| [Folder structure](docs/FOLDER_STRUCTURE.md) | Repository layout |
| [Security](docs/SECURITY.md) | Auth, permissions, privacy, retention, OWASP coverage |
| [Testing](docs/TESTING.md) | Strategy, tools, coverage, test cases |
| [Environment setup](docs/ENV_SETUP.md) | Prerequisites, env vars, local setup, troubleshooting |
| [Deployment](docs/DEPLOYMENT.md) | Hosting, CI/CD, monitoring, backups, rollback, runbooks |
| [Roadmap](docs/ROADMAP.md) | Phases, milestones, launch criteria |
| [Tasks](docs/TASKS.md) | Ordered implementation checklist |
| [Contributing](docs/CONTRIBUTING.md) | Branching, commits, PRs, review checklist, code style |
| [Assumptions](docs/ASSUMPTIONS.md) | Decisions made where the plan was silent, and open questions |
| [Changelog](docs/CHANGELOG.md) | Release history |

## Principles

1. Honesty over personal branding.
2. Support over ridicule.
3. Experience over rumor.
4. Privacy by design.
5. Useful outcomes.
6. Fairness.

*Build the community first. Build the humor around it. Build trust into everything.*

## Contributing and security

Read [docs/CONTRIBUTING.md](docs/CONTRIBUTING.md). Report vulnerabilities privately to `security@linkedout.app`, never in public issues.
