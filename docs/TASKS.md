# LinkedOut — Task Checklist

Tasks are in **build order** (plan §26) and grouped by roadmap phase ([ROADMAP.md](ROADMAP.md)) and feature. Each task is sized at 1–4 hours and links requirements (FR/NFR), stories (US), tests (TC/SEC) or risks (R).

**Rules**
1. Work top to bottom inside a phase. A task may start only when the tasks it depends on (earlier in the same section) are done.
2. A task is done when it meets the Definition of Done ([TESTING.md §8](TESTING.md#8-definition-of-done-for-tests)): code reviewed and merged, tests green, docs updated.
3. After each task, tick it here (`- [x]`) and add a line to [CHANGELOG.md](CHANGELOG.md) under *Unreleased*.
4. Anonymous posting (`confessions.enabled`) stays **off in production** until every task in Phases 5 and 6 is done (plan §26).

Format: `- [ ] **T-ID** · estimate · task · links`

### Effort summary

| Phase | Weeks | Tasks | Estimated hours |
|---|---|---|---|
| 1 Discovery & design | 1–2 | 17 | 49 |
| 2 Foundation | 3–4 | 51 | 134 |
| 3 Profiles & feed | 5–6 | 34 | 93 |
| 4 Social interaction | 7–8 | 19 | 51 |
| 5 Confessions, companies & reports | 9–10 | 31 | 88 |
| 6 Trust & moderation | 11–12 | 30 | 84 |
| 7 Beta & refinement | 13–14 | 18 | 48 |
| 8 Public launch preparation | 15–16 | 11 | 29 |
| **MVP total** | **1–16** | **211** | **576** |
| 9 v1.1 Should-have | 17–24 | 41 | 102 |
| 10 Future backlog spikes | — | 13 | 39 |

The MVP needs 576 task-hours, plus about 25% for reviews, meetings and rework, so roughly 720 hours. That is about 45 hours per week across the team: for example, 3 people at ~15 h/week (student project) or 2 people at ~23 h/week.

---

## Phase 1 — Discovery & design (Weeks 1–2: 30 Nov – 13 Dec 2026)

### 1.1 Product, policy and legal
- [ ] **T-001** · 2h · Confirm name and tagline; run a trademark/conflict search for "LinkedOut"; record the outcome in ASSUMPTIONS.md · R-11
- [ ] **T-002** · 3h · Finalize Community Guidelines text with CG policy codes from SECURITY §11 · FR-11, FR-47
- [ ] **T-003** · 3h · Draft Privacy Policy (data inventory, lawful bases, retention, lawful disclosure, rights) · FR-11, FR-18, NFR-S6
- [ ] **T-004** · 2h · Draft Terms of Service (age ≥ 16, peer-advice disclaimer, content licence, termination) · FR-1
- [ ] **T-005** · 2h · Define moderation SLAs, moderator rota and escalation contacts; assign the T&S lead role explicitly · FR-46, FR-47, R-9
- [ ] **T-006** · 2h · Write the legal request procedure and confirm a counsel contact · FR-51, R-8

### 1.2 UX and visual design
- [ ] **T-007** · 3h · Build design tokens (colors, type, spacing, radii) in Figma per UI_UX Part C, with contrast checks · NFR-A3
- [ ] **T-008** · 4h · Wireframes: landing, sign up, log in, verify, reset, onboarding (S-01–S-06) · US-005, US-013
- [ ] **T-009** · 4h · Wireframes: The Mess, PostCard, composer, post detail (S-07, S-08, S-11, S-14) · US-001, US-022, US-028, US-037
- [ ] **T-010** · 4h · Wireframes: confession wizard and SafetyCheckPanel (S-09, S-10) · US-033, US-034
- [ ] **T-011** · 4h · Wireframes: Anti-Profile, My Damage, settings (S-12, S-13, S-25–S-29) · US-014, US-016, US-017
- [ ] **T-012** · 4h · Wireframes: company directory, Workplace Reality Page, review form, Job Nightmare board and form (S-15–S-20) · US-003, US-039, US-042
- [ ] **T-013** · 3h · Wireframes: report & appeal center, moderator dashboard, item detail, admin (S-22–S-24, S-30, S-31) · US-045, US-060, US-070
- [ ] **T-014** · 3h · Usability test of wireframes with 5 target users (privacy explanation comprehension); record findings · US-013, TC-116

### 1.3 Accounts and repository
- [ ] **T-015** · 2h · Create GitHub repository, branch protection, CODEOWNERS, issue/PR templates · NFR-M1
- [ ] **T-016** · 2h · Register domain, set up Cloudflare zone and mailboxes (support, privacy, legal, security) · DEPLOYMENT §6
- [ ] **T-017** · 2h · Create Render, Vercel, Cloudflare R2, Resend, Sentry and Grafana Cloud accounts with team access and 2FA · DEPLOYMENT §1

---

## Phase 2 — Foundation (Weeks 3–4: 14 – 27 Dec 2026)

### 2.1 Monorepo and tooling
- [ ] **T-018** · 3h · Initialize pnpm workspace + Turborepo, `packages/tsconfig`, `packages/eslint-config` (incl. boundaries, jsx-a11y), Prettier, EditorConfig, `.nvmrc` · NFR-M1
- [ ] **T-019** · 2h · Husky + lint-staged + commitlint + gitleaks pre-commit hook · NFR-S7
- [ ] **T-020** · 3h · `packages/shared`: enums (DATABASE_SCHEMA §3), constants (limits), common Zod schemas (pagination, error envelope, AuthorRef) · FR-10, FR-24
- [ ] **T-021** · 2h · `packages/ui-tokens` with CSS-variable build and a unit test asserting token contrast ratios · NFR-A3

### 2.2 Local infrastructure and CI
- [ ] **T-022** · 3h · `infra/docker-compose.yml` (Postgres 17, Redis 7.4 noeviction, MinIO + bucket init, Mailpit, ClamAV profile) and `infra/postgres/init/01-roles.sql` · NFR-S5
- [ ] **T-023** · 2h · `.env.example`, API `env.schema.ts` (Zod), `scripts/check-env.ts`, root scripts per ENV_SETUP §5 · NFR-M4
- [ ] **T-024** · 4h · `ci.yml`: lint, typecheck, unit, build with pnpm and Turborepo caching · NFR-M4
- [ ] **T-025** · 3h · `security.yml`: CodeQL, gitleaks, `pnpm audit`, Renovate config · NFR-S7

### 2.3 API skeleton
- [ ] **T-026** · 3h · NestJS bootstrap: config module, helmet, CORS allow-list, `/api/v1` prefix, 256 KB body limit, Swagger toggle · NFR-S3
- [ ] **T-027** · 3h · pino logging with redaction paths, request-ID middleware, OpenTelemetry init (env-gated) · NFR-M3, NFR-S4
- [ ] **T-028** · 3h · Global exception filter → standard error format and `ErrorCode` enum · NFR-M3
- [ ] **T-029** · 2h · Health module: `/health/live`, `/health/ready` (DB, Redis) · NFR-AV1
- [ ] **T-030** · 3h · Prisma setup: multi-schema, `PrismaService` (lo_app) and `VaultPrismaService` (lo_vault), migrate/generate scripts · NFR-S5, NFR-M4
- [ ] **T-031** · 2h · Redis provider, BullMQ module, `worker.ts` bootstrap with a heartbeat job · NFR-SC2
- [ ] **T-032** · 3h · Throttler with Redis storage and named limits from API_SPEC §1.7; fail-closed on Redis errors · FR-6, NFR-AV4
- [ ] **T-033** · 2h · Cursor pagination helper (encode/decode, keyset query builder) with unit tests · FR-24
- [ ] **T-034** · 3h · Integration test harness: Testcontainers global setup, migrations, roles, base factories · NFR-M2

### 2.4 Web skeleton
- [ ] **T-035** · 3h · Next.js app: App Router, Tailwind 4 + tokens, self-hosted fonts, security headers and CSP with nonces · NFR-P3, NFR-S3
- [ ] **T-036** · 3h · Themed shadcn/ui base components (button, input, textarea, select, checkbox, switch, dialog, dropdown, tabs, toast, tooltip) · NFR-A1
- [ ] **T-037** · 3h · API client (credentials, CSRF header, typed errors), TanStack Query provider, server-side fetch forwarding cookies · FR-3
- [ ] **T-038** · 3h · AppShell: side nav, top bar, mobile tab bar, footer, skip link, route-change focus management · NFR-A2
- [ ] **T-039** · 2h · System pages S-38 (404, error boundary, maintenance, suspended) · NFR-AV4

### 2.5 Identity and authentication — F-01, F-03, F-23 (build order #1)
- [ ] **T-040** · 3h · Migration: users, sessions, auth_tokens, profiles, privacy_settings, reserved_usernames, username_history; seed reserved usernames · FR-1, FR-8
- [ ] **T-041** · 2h · Audit module: `AuditService` (append-only) + migration with the immutability trigger · FR-50
- [ ] **T-042** · 3h · `PasswordService`: Argon2id hash/verify/rehash and HIBP range check (fail-open + log) with tests · FR-1, NFR-S1
- [ ] **T-043** · 3h · `SessionService`: create/lookup/revoke, Redis mirror, sliding idle + absolute expiry, `user_sessions` index · FR-3, FR-5
- [ ] **T-044** · 3h · CSRF guard (double-submit + HMAC), `GET /auth/csrf`, `SessionGuard`, `VerifiedGuard`, `@Public`, CI test failing on unguarded routes · FR-3, SEC-08
- [ ] **T-045** · 3h · Email infrastructure: `email` queue, SMTP/Resend transports, `packages/email-templates` with VerifyEmail and RegistrationAttempt · FR-2
- [ ] **T-046** · 2h · `GET /auth/username-availability` and shared username rules · FR-8, US-011
- [ ] **T-047** · 4h · `POST /auth/register`: enumeration-safe responses, invite-code support, profile + privacy defaults in one transaction · FR-1, US-005
- [ ] **T-048** · 3h · `TokenService`, `POST /auth/verify-email`, `POST /auth/resend-verification` · FR-2, US-006
- [ ] **T-049** · 3h · `POST /auth/login` with lockout, LoginLocked email and audit entries · FR-3, FR-6, US-007
- [ ] **T-050** · 1h · `POST /auth/logout` · FR-3, US-007
- [ ] **T-051** · 3h · `POST /auth/forgot-password`, `POST /auth/reset-password`, PasswordReset email, revoke all sessions · FR-4, US-008
- [ ] **T-052** · 2h · `GET /me/sessions`, `DELETE /me/sessions/:id`, `DELETE /me/sessions` with GeoLite2 and device labels · FR-5, US-009
- [ ] **T-053** · 2h · `GET /me` with the private self serializer · FR-10
- [ ] **T-054** · 3h · Web S-02 Sign up with live username check and password meter · US-005, US-011
- [ ] **T-055** · 2h · Web S-03 Log in (locked, suspended, deletion-pending states) · US-007
- [ ] **T-056** · 2h · Web S-04 Verify email and S-05 Forgot/Reset password · US-006, US-008
- [ ] **T-057** · 2h · Web settings layout + S-27 Sessions · US-009
- [ ] **T-058** · 3h · Integration tests TC-001–TC-012 · FR-1–FR-6
- [ ] **T-059** · 2h · Security tests SEC-10, SEC-18, SEC-19 · FR-4, FR-6

### 2.6 Onboarding and privacy model — F-04, F-06, F-03
- [ ] **T-060** · 2h · `POST /me/onboarding/complete` and `ONBOARDING_REQUIRED` enforcement in `VerifiedGuard` · FR-11, US-013
- [ ] **T-061** · 4h · Web S-06 onboarding (4 steps + interests) · FR-11, US-013
- [ ] **T-062** · 2h · `GET/PATCH /me/privacy` · FR-15, US-017
- [ ] **T-063** · 2h · Web S-26 Privacy settings · FR-15, US-017
- [ ] **T-064** · 3h · Contract test framework: private-field deny-list walker over public schemas and recorded responses · FR-10, SEC-04

### 2.7 Staging environment
- [ ] **T-065** · 3h · `infra/docker/api.Dockerfile` (multi-stage, distroless, non-root) + Trivy scan in CI · NFR-S7
- [ ] **T-066** · 3h · `infra/render.yaml` for staging (api, worker, Postgres, Key Value) + `deploy-staging.yml` with the migration step · NFR-M4
- [ ] **T-067** · 2h · Vercel project: staging alias + PR previews pointing to the staging API · NFR-M4
- [ ] **T-068** · 2h · Post-deploy smoke tests (DEPLOYMENT §4.3) · NFR-AV1

---

## Phase 3 — Profiles & feed (Weeks 5–6: 28 Dec 2026 – 10 Jan 2027)

### 3.1 Anti-Profile — F-05, F-03 (build order #2)
- [ ] **T-069** · 2h · Migration: industries, profile_lessons, profile_skills, profile_timeline_entries; seed industries · FR-12, FR-13
- [ ] **T-070** · 3h · `PATCH /me/profile` with validation (country codes, enums) · FR-12, US-014
- [ ] **T-071** · 3h · `PUT /me/profile/lessons|skills|timeline` (replace semantics, limits) · FR-13, US-015
- [ ] **T-072** · 2h · `PATCH /me/username` with the 30-day change and 90-day reservation rules · FR-9, US-012
- [ ] **T-073** · 3h · `GET /profiles/:username` with the visibility matrix (posts/comments sub-routes return pseudonymous content only) · FR-14, FR-10, US-016
- [ ] **T-074** · 3h · Web S-12 Public Anti-Profile (no counts or rankings) · FR-14, US-016
- [ ] **T-075** · 4h · Web S-25 Settings → Profile (lessons, skills, timeline editors, username change) · FR-9, FR-12, FR-13, US-012, US-014, US-015
- [ ] **T-076** · 2h · Integration tests TC-013–TC-022 · FR-8–FR-15

### 3.2 Safety check core — F-21 (required before any publishing)
- [ ] **T-077** · 4h · `SafetyCheckService` framework + rules `secrets.credential`, `pii.payment_card`, `pii.email_address`, `pii.phone_number` · FR-45
- [ ] **T-078** · 3h · Rules `pii.government_id`, `pii.street_address`, `pii.social_profile`, `confidential_markers` · FR-45
- [ ] **T-079** · 3h · Rules `named_individual`, `threat_terms`, `harassment_terms` (versioned term lists) · FR-45, R-1, R-2
- [ ] **T-080** · 2h · Rules `velocity.new_account`, `duplicate_content`, new-account link limits · FR-45, R-7
- [ ] **T-081** · 2h · `POST /safety/check` + `acknowledgedFindings` enforcement helper (422 codes) · FR-45, US-034
- [ ] **T-082** · 3h · Safety rule test corpus (≥ 150 positive and negative samples) · FR-45
- [ ] **T-083** · 3h · Web SafetyCheckPanel with editor range highlighting and "Jump to text" · US-034
- [ ] **T-084** · 1h · Apply the profile safety check to the headline and bio in `PATCH /me/profile` · FR-12, US-014

### 3.3 Posts, categories, lessons — F-10, F-16 (build order #3)
- [ ] **T-085** · 3h · Migration: categories, tags, post_tags, posts (CHECKs, generated columns, indexes), post_revisions, post_corrections, lesson_details, moderation_items; seed 12 categories · FR-20, FR-22
- [ ] **T-086** · 2h · `GET /categories`, `GET /industries`, `GET /tags?q=` · FR-22
- [ ] **T-087** · 4h · `POST /posts` for story/advice: tags upsert, safety check, held → moderation item · FR-20, FR-45, US-022, US-023
- [ ] **T-088** · 3h · `GET /posts/:id` with visibility rules, author-only held view, PostDetail + AuthorRef serializers · FR-20, FR-10, US-002
- [ ] **T-089** · 3h · `PATCH`/`DELETE /posts/:id` with revisions and soft delete · FR-21, US-025
- [ ] **T-090** · 2h · `POST /posts/:id/corrections` (max 5, immutable for authors) · FR-34, US-036
- [ ] **T-091** · 2h · Lesson post type in create/update + `GET /lessons` · FR-35, US-037, US-038
- [ ] **T-092** · 1h · Content warning validation and serialization · FR-23, US-024

### 3.4 The Mess feed — F-11
- [ ] **T-093** · 4h · `GET /feed` Latest: keyset pagination, filters (category, type, language, excludeCategory), visibility and moderation predicates · FR-24, FR-25, US-001, US-026
- [ ] **T-094** · 2h · Visitor first-page Redis cache (30 s) and invalidation on post events · NFR-P2
- [ ] **T-095** · 3h · Web PostCard, ContentWarningGate and sanitized Markdown renderer · US-001, US-024, SEC-06
- [ ] **T-096** · 4h · Web S-08 composer (story/advice/lesson modes, editor + preview, tags, warnings, visibility, language) · US-022, US-023, US-024, US-037
- [ ] **T-097** · 4h · Web S-07 The Mess with URL filters, infinite scroll, empty/loading/error states · US-001, US-026, US-027
- [ ] **T-098** · 3h · Web S-11 post detail (body, lesson layout, corrections, author menu) · US-002, US-025, US-036
- [ ] **T-099** · 2h · Web S-14 Learned the Hard Way listing · US-038
- [ ] **T-100** · 2h · Web S-01 landing page with live public preview · US-001
- [ ] **T-101** · 3h · Integration tests TC-029–TC-040, TC-057, TC-058 · FR-20–FR-25, FR-35
- [ ] **T-102** · 3h · E2E Journey A part 1 (sign up → verify → onboard → post → see in feed) with axe checks · US-005, US-013, US-022, TC-112

---

## Phase 4 — Social interaction (Weeks 7–8: 11 – 24 Jan 2027)

### 4.1 Comments, reactions, saves — F-12, F-13, F-14 (build order #4)
- [ ] **T-103** · 2h · Migration: comments, reactions, saved_posts · FR-26, FR-29, FR-30
- [ ] **T-104** · 4h · Comments: list with nested replies, replies endpoint, create with depth cap, safety check and counters · FR-26, US-028
- [ ] **T-105** · 2h · `PATCH`/`DELETE /comments/:id` with "[deleted]" semantics · FR-27, US-029
- [ ] **T-106** · 2h · `PUT`/`DELETE /posts/:id/pinned-comment` · FR-28, US-030
- [ ] **T-107** · 3h · Reactions upsert/delete with row locking and `reaction_counts` updates · FR-29, US-031
- [ ] **T-108** · 2h · `PUT`/`DELETE /posts/:id/save` and `GET /me/saved` · FR-30, US-032
- [ ] **T-109** · 2h · Batched viewer fields (reaction, saved, isAuthor) for feed and detail · FR-24, FR-29
- [ ] **T-110** · 4h · Web CommentThread, CommentForm, edit/delete, pin clarification · US-028, US-029, US-030
- [ ] **T-111** · 3h · Web ReactionBar/Picker (keyboard accessible, optimistic updates) · US-031, TC-047
- [ ] **T-112** · 3h · Web SaveButton and S-13 My Damage (My posts, Saved tabs) · US-032
- [ ] **T-113** · 3h · Integration tests TC-041–TC-048 incl. concurrency · FR-26–FR-30

### 4.2 Block & mute — F-07
- [ ] **T-114** · 2h · Migration: blocks, mutes · FR-16, FR-17
- [ ] **T-115** · 3h · Block/mute endpoints (members, categories, tags, alias blocks via content ID) · FR-16, FR-17, US-018, US-019
- [ ] **T-116** · 3h · Apply block/mute predicates in feed, comments, profiles, search; interaction checks returning `BLOCKED` · FR-16, FR-17, US-018
- [ ] **T-117** · 2h · Web overflow menus (block, mute, hide category) and S-28 Blocked & muted · US-018, US-019, US-027
- [ ] **T-118** · 2h · Tests TC-023–TC-025 and SEC-20 · FR-16, FR-17

### 4.3 Search — F-19
- [ ] **T-119** · 4h · Search module: posts and lessons full-text with exclusions and highlights; member trigram search · FR-43, US-004, US-044
- [ ] **T-120** · 3h · Web S-21 search results (tabs, grouped results, highlight rendering) · US-004, US-044
- [ ] **T-121** · 2h · Tests TC-070–TC-072, SEC-06 (highlights), SEC-07 (fuzzing) · FR-43

---

## Phase 5 — Confessions, companies & reports (Weeks 9–10: 25 Jan – 7 Feb 2027)

### 5.1 Identity Vault and Office Confessions — F-23, F-15 (build order #5)
- [ ] **T-122** · 3h · Migration: `vault` schema, anonymous_identity_aliases, grants for lo_vault, staff_permissions, identity_reveal_requests, feature_flags; seed flags · FR-32, FR-51, NFR-S5
- [ ] **T-123** · 4h · `IdentityVaultService` (createAlias with label generator, isOwner, listOwnTargets, hasScope, deleteForUser) with unit tests · FR-32
- [ ] **T-124** · 2h · Feature flag service, `@FeatureFlag` guard, flags in `GET /me` · FR-31
- [ ] **T-125** · 4h · Migration confession_details + `POST /confessions` (anonymous/pseudonym, drafts, hold rules, vault-first ordering with compensation) · FR-31, FR-32, FR-33, US-033, US-035
- [ ] **T-126** · 3h · `GET /confessions`, `GET/PATCH/DELETE /confessions/:id` (publish draft, identity lock) · FR-31, FR-33
- [ ] **T-127** · 2h · Vault reconciliation job and draft reminder/expiry maintenance jobs + DraftExpiryReminder email · FR-33, US-035
- [ ] **T-128** · 2h · Vault ownership for OP anonymous comments, corrections and pinned clarifications · FR-32, FR-34, TC-056
- [ ] **T-129** · 2h · `GET /me/posts` including the member's anonymous items · FR-32, US-033
- [ ] **T-130** · 4h · Web S-09 confession wizard (write, safety check & preview, confirm, draft) · US-033, US-034, US-035
- [ ] **T-131** · 2h · Web S-10 confessions list, anonymous and OP badges, Drafts tab in S-13 · US-033, US-035
- [ ] **T-132** · 3h · Security tests SEC-05 (identity leakage across all endpoints), SEC-11, SEC-12 (log capture) · FR-10, FR-32
- [ ] **T-133** · 2h · Integration tests TC-049–TC-056 · FR-31–FR-34

### 5.2 Reporting and moderation queue core — F-20, F-21 (build order #6)
- [ ] **T-134** · 2h · Migration: reports, moderation_actions, appeals · FR-44, FR-47, FR-49
- [ ] **T-135** · 3h · `POST /reports` with item aggregation and priority rules; `GET /me/reports` · FR-44, US-045, US-046
- [ ] **T-136** · 3h · `GET /moderation/queue`, `GET /moderation/items/:id` (full context), claim/release · FR-46, US-060
- [ ] **T-137** · 2h · Web ReportDialog on posts, comments, profiles, reviews, job reports · US-045
- [ ] **T-138** · 3h · Web S-23 moderator queue (filters, keyboard navigation, header stats) · US-060
- [ ] **T-139** · 2h · Tests TC-073–TC-078 · FR-44, FR-46

### 5.3 Workplace Reality Pages — F-17 (build order #7)
- [ ] **T-140** · 3h · Migration: companies, workplace_reviews, review_topic_ratings, job_reports with trigram and full-text indexes · FR-36, FR-38, FR-41
- [ ] **T-141** · 3h · Company directory, `POST /companies` with normalization and duplicate detection, `GET /companies/:idOrSlug` (merged, pending) · FR-36, FR-37, US-041
- [ ] **T-142** · 4h · `POST /companies/:id/reviews` (age gate, vault scope, safety check, holds) + `PATCH`/`DELETE /reviews/:id` · FR-38, US-039
- [ ] **T-143** · 3h · `GET /companies/:id/reviews` with filters, consent handling and labels · FR-39, FR-40, US-040
- [ ] **T-144** · 3h · Theme summary maintenance job (n ≥ 5, 24 months, dirty flag) and themes in the company response · FR-39, US-003
- [ ] **T-145** · 3h · Web S-15 directory and S-18 submit company · US-041
- [ ] **T-146** · 4h · Web S-16 Workplace Reality Page (ThemeSummary, tabs, filters, ReviewCards, notices) · US-003, US-040
- [ ] **T-147** · 3h · Web S-17 review form · US-039
- [ ] **T-148** · 3h · Tests TC-059–TC-066 · FR-36–FR-40

### 5.4 Job Nightmare Board — F-18 (build order #8)
- [ ] **T-149** · 4h · Job report endpoints (create incl. newCompany, list filters, get, patch, delete; anonymous via vault) + `interview_dump` rule · FR-41, FR-42, US-042, US-043
- [ ] **T-150** · 3h · Web S-19 board + detail and S-20 report form · US-042, US-043
- [ ] **T-151** · 2h · Extend search to confessions, job reports and companies · FR-43, US-044
- [ ] **T-152** · 2h · Tests TC-067–TC-069 · FR-41, FR-42

---

## Phase 6 — Trust & moderation (Weeks 11–12: 8 – 21 Feb 2027)

### 6.1 Moderation actions — F-21
- [ ] **T-153** · 4h · `POST /moderation/items/:id/actions`: effects for all 8 action types, report resolution, audit · FR-47, US-061, US-062, US-065
- [ ] **T-154** · 3h · `resolveForEnforcement` for anonymous targets, posting-restriction enforcement, suspension session revocation · FR-47, TC-081
- [ ] **T-155** · 3h · ModerationDecision and ReportOutcome email templates + enqueueing · FR-47, US-046
- [ ] **T-156** · 4h · Web S-24 item detail (content with highlights, reports, history, ActionForm, claim state) · US-061, US-062, US-065
- [ ] **T-157** · 2h · `GET /me/moderation-actions` · FR-49, US-047
- [ ] **T-158** · 2h · Tests TC-079–TC-082, TC-084 · FR-47

### 6.2 Appeals — F-22
- [ ] **T-159** · 3h · Appeal create, list, decide (different-moderator rule), reversal and modification effects · FR-49, US-047, US-063
- [ ] **T-160** · 3h · Web S-22 Report & appeal center and `/mod/appeals` tab · US-046, US-047, US-063
- [ ] **T-161** · 2h · AppealOutcome email + tests TC-085–TC-088 · FR-49

### 6.3 Campaign detection and post-publish scans — F-21
- [ ] **T-162** · 3h · `safety-scan` processor: post-publish re-scan, 3 campaign rules, `summaries_paused` · FR-48, US-064
- [ ] **T-163** · 2h · Tests TC-083 and campaign resolution · FR-48

### 6.4 Identity reveal (four-eyes) — F-23
- [ ] **T-164** · 3h · Reveal request/approve/reject/view endpoints, 24 h window, audit, denied-attempt alert hook · FR-51, US-066
- [ ] **T-165** · 2h · Web RevealPanel in S-24 · US-066
- [ ] **T-166** · 2h · Tests TC-089, TC-090 and SEC-13 · FR-51

### 6.5 Admin dashboard (basic) — F-24
- [ ] **T-167** · 3h · Admin users search, role change (last-admin rule), permission grant/revoke with session rewrite · FR-52, US-070
- [ ] **T-168** · 2h · Admin categories CRUD and ordering · FR-53, US-071
- [ ] **T-169** · 3h · Admin companies approve/edit/reject/merge incl. vault re-scoping · FR-54, US-072
- [ ] **T-170** · 3h · Admin metrics aggregates, escalations list, feature flags with launch gate, beta invites · FR-55, US-074
- [ ] **T-171** · 2h · `GET /admin/audit-logs` with filters · FR-50, US-075
- [ ] **T-172** · 4h · Web S-30 dashboard and S-31 users, categories, companies · US-070, US-071, US-072, US-074
- [ ] **T-173** · 3h · Web S-31 escalations, feature flags (launch-gate modal), audit log viewer, beta invites · US-075, FR-55
- [ ] **T-174** · 2h · Tests TC-091–TC-097 · FR-50–FR-55

### 6.6 Account deletion and retention — F-08
- [ ] **T-175** · 3h · `DELETE /me/account`, `POST /me/account/cancel-deletion`, deletion requests, DeletionScheduled email · FR-18, US-020
- [ ] **T-176** · 4h · `account-deletion` processor (purge order, vault deleteForUser, legal hold, tombstone, final email) · FR-18
- [ ] **T-177** · 2h · Web S-29 Account & data · US-020
- [ ] **T-178** · 3h · Maintenance jobs: soft-delete purge, session IP nulling, token cleanup, notification/audit retention · NFR-S6
- [ ] **T-179** · 2h · Tests TC-026–TC-028 · FR-18

### 6.7 Privacy and abuse testing
- [ ] **T-180** · 4h · Complete the security suite SEC-01–SEC-03, SEC-09, SEC-13–SEC-16 · NFR-S3
- [ ] **T-181** · 3h · Scripted abuse session on staging (spam, brigading, doxxing attempts); file and fix findings · R-2, R-3, R-4, R-7
- [ ] **T-182** · 3h · E2E Journeys A (complete), B and C with axe; keyboard-only Journey A · US-033, US-039, TC-112, TC-113

---

## Phase 7 — Beta & refinement (Weeks 13–14: 22 Feb – 7 Mar 2027)

### 7.1 Production readiness for beta
- [ ] **T-183** · 2h · Provision production (Render prod services with PITR, Vercel prod, R2 buckets, Resend domain DNS) · DEPLOYMENT §1
- [ ] **T-184** · 2h · `deploy-production.yml` with approvals, pre-migration snapshot and smoke tests · NFR-M4
- [ ] **T-185** · 3h · Grafana dashboards and alert rules (DEPLOYMENT §7) with test alerts · NFR-M3, NFR-AV1
- [ ] **T-186** · 3h · Encrypted daily logical backup job + `scripts/restore-test.sh`; first restore test logged · NFR-AV2, NFR-AV3
- [ ] **T-187** · 2h · Nightly all-browser E2E and ZAP baseline against staging · NFR-S3
- [ ] **T-188** · 2h · Publish policy pages (privacy, terms, guidelines) and crisis resources page · FR-11

### 7.2 Closed beta operations
- [ ] **T-189** · 2h · Enable `beta.invite_only` in production, generate invite codes, write the invite email · R-9
- [ ] **T-190** · 2h · In-app feedback link and beta surveys (trust, helpfulness, negativity) · G-1, G-2
- [ ] **T-191** · 2h · Moderator training session and beta rota · FR-46, R-9
- [ ] **T-192** · 2h · Deploy `v0.9.0-beta` to production and invite the first 50 testers · G-5
- [ ] **T-193** · 3h · Triage beta feedback week 1 into issues with severity · G-2
- [ ] **T-194** · 4h · Fix beta bugs, batch 1 · G-5
- [ ] **T-195** · 4h · Fix beta bugs, batch 2 · G-5
- [ ] **T-196** · 2h · Negativity-balance review: adjust copy, empty states and the Workplace Wins aside · R-5

### 7.3 Performance and accessibility
- [ ] **T-197** · 4h · k6 load tests (feed, post create, comment burst, search, soak) on staging; record results · NFR-P1, NFR-P4, NFR-SC3
- [ ] **T-198** · 3h · Fix performance findings (indexes, query plans, cache keys) · NFR-P1
- [ ] **T-199** · 3h · Lighthouse CI budgets in CI and fixes · NFR-P3, NFR-P5
- [ ] **T-200** · 3h · Manual screen-reader audit (NVDA + VoiceOver) of Journeys A–C and fixes · NFR-A1, TC-114

---

## Phase 8 — Public launch preparation (Weeks 15–16: 8 – 21 Mar 2027)

- [ ] **T-201** · 4h · External security review/pentest: scope, access and walkthrough · NFR-S3
- [ ] **T-202** · 4h · Fix pentest findings (critical/high must be zero) · NFR-S7
- [ ] **T-203** · 3h · Runbooks: service-outage, database-recovery, restore-test, secret-rotation · DEPLOYMENT §11
- [ ] **T-204** · 3h · Runbooks: data-breach, account-takeover, anonymous-identity-exposure · DEPLOYMENT §11, R-6
- [ ] **T-205** · 2h · Runbooks: malicious-content, moderation-backlog, legal-privacy-request · DEPLOYMENT §11, R-8, R-9
- [ ] **T-206** · 2h · `security.txt`, status page, support mailbox auto-replies · SECURITY §15
- [ ] **T-207** · 2h · Verify the launch criteria in ROADMAP and sign off with the T&S lead · ROADMAP
- [ ] **T-208** · 2h · Launch gate: enable `confessions.enabled` and disable `beta.invite_only` in production (audit-logged) · FR-31, US-033
- [ ] **T-209** · 2h · Launch communications (landing copy, launch message from plan §24, community outreach list) · G-3
- [ ] **T-210** · 1h · CHANGELOG v1.0.0 and tag the release · NFR-M4
- [ ] **T-211** · 4h · Launch-week watch: dashboards, moderation coverage, hotfix buffer · NFR-AV1, R-9

---

## Phase 9 — v1.1 Should-have release (Weeks 17–24: 22 Mar – 16 May 2027)

### 9.1 Notifications (F-27, build order #11)
- [ ] **T-212** · 3h · Migration notifications, notification_preferences; NotificationService + `notifications` processor · FR-64
- [ ] **T-213** · 3h · Reaction batching, sensitive-reference encryption, preferences and quiet-hours scheduling · FR-64, FR-65, FR-66
- [ ] **T-214** · 2h · Notification endpoints (list, unread count, read, read-all, preferences) · FR-64, FR-65, US-052, US-053
- [ ] **T-215** · 3h · Web bell with 60 s polling and S-36 notification center · US-052
- [ ] **T-216** · 2h · Web S-37 notification settings · US-053
- [ ] **T-217** · 2h · Tests TC-106–TC-108 · FR-64–FR-66

### 9.2 Communities (F-26, build order #10)
- [ ] **T-218** · 3h · Migration: communities, community_members, rules, pinned resources, join requests; seed example communities · FR-59
- [ ] **T-219** · 4h · Create, discover, get, update, join and leave endpoints with limits and age gate · FR-59, FR-60, US-048, US-049
- [ ] **T-220** · 3h · Private communities and join-request decisions · FR-60, FR-63, US-051
- [ ] **T-221** · 3h · Community posting, community feed, visibility in feed and search · FR-61, US-050
- [ ] **T-222** · 3h · Scoped community moderation queue and actions (incl. ban), mirroring to the platform queue · FR-62, US-058
- [ ] **T-223** · 2h · Rules, pinned resources, moderators, announcements, invitations endpoints · FR-62, US-059
- [ ] **T-224** · 4h · Web S-33 directory, S-34 community page, S-35 create community · US-048, US-049, US-050, US-051
- [ ] **T-225** · 3h · Web community moderation tab · US-058, US-059
- [ ] **T-226** · 3h · Tests TC-103–TC-105 · FR-59–FR-63

### 9.3 Polls (F-28)
- [ ] **T-227** · 2h · Migration polls, poll_options, poll_votes · FR-67
- [ ] **T-228** · 3h · Poll creation in `POST /posts`, voting endpoint, result visibility rules · FR-67, US-054, US-055
- [ ] **T-229** · 3h · Web poll composer mode and PollCard · US-054, US-055
- [ ] **T-230** · 1h · Tests TC-109 · FR-67

### 9.4 Company representatives & responses (F-25)
- [ ] **T-231** · 3h · Migration company_memberships, company_responses; representation request + work-email verification · FR-56, US-067
- [ ] **T-232** · 2h · Admin decision endpoint and membership expiry job · FR-57, US-073
- [ ] **T-233** · 3h · Company responses create/edit/delete with safety checks and badge serialization · FR-58, US-068
- [ ] **T-234** · 3h · Web S-32 request, response area in S-16, admin Representatives page · US-067, US-068, US-073
- [ ] **T-235** · 2h · Tests TC-100–TC-102 · FR-56–FR-58, US-069

### 9.5 Media uploads (F-29)
- [ ] **T-236** · 3h · Migration media, post_media; presign and complete endpoints · FR-68
- [ ] **T-237** · 4h · `media-scan` processor (magic bytes, sharp re-encode + EXIF strip, ClamAV) and ClamAV service on Render · FR-68
- [ ] **T-238** · 3h · Web image picker with alt text, post images, avatar upload · FR-68, FR-12, US-056
- [ ] **T-239** · 2h · Tests TC-110 and SEC-17 · FR-68

### 9.6 Topic following (F-30)
- [ ] **T-240** · 2h · Migration topic_follows + follow endpoints · FR-69
- [ ] **T-241** · 2h · Following feed mode · FR-70, US-057
- [ ] **T-242** · 2h · Web Following tab and follow buttons on categories/tags · US-057
- [ ] **T-243** · 1h · Tests TC-111 · FR-69, FR-70

### 9.7 Data export (F-09)
- [ ] **T-244** · 3h · `data-export` processor (all data incl. own anonymous content) · FR-19
- [ ] **T-245** · 2h · Export endpoints, ExportReady email, bucket lifecycle rule · FR-19, US-021
- [ ] **T-246** · 1h · Web export section in S-29 · US-021
- [ ] **T-247** · 1h · Tests TC-099 · FR-19

### 9.8 Google sign-in (F-02)
- [ ] **T-248** · 3h · OAuth service (PKCE, state, nonce, ID token validation) and callback · FR-7
- [ ] **T-249** · 2h · Complete-signup and account-linking flows · FR-7, US-010
- [ ] **T-250** · 2h · Web Google buttons and S-02b · US-010
- [ ] **T-251** · 2h · Tests TC-098 · FR-7

### 9.9 Release v1.1
- [ ] **T-252** · 2h · Enable `communities.enabled`, CHANGELOG v1.1.0, tag and deploy · NFR-M4

---

## Phase 10 — Future backlog (unscheduled; each starts with a design spike)

- [ ] **T-253** · 4h · Spike: Direct messaging (F-31) design: message requests, permissions, safety notices, anonymous-author protection · FR-71, US-076
- [ ] **T-254** · 3h · Spike: Career Damage Report (F-32) private metrics and data model · FR-72, US-077
- [ ] **T-255** · 3h · Spike: Corporate Meme Generator (F-33) templates, rendering, moderation · FR-73, US-078
- [ ] **T-256** · 3h · Spike: Hall of Shame (F-34) editorial curation process · FR-74, US-079
- [ ] **T-257** · 4h · Spike: Relevant feed ranking (F-35) signals and offline evaluation · FR-75, US-080
- [ ] **T-258** · 3h · Spike: AI-assisted categorization (F-36) provider, privacy review, suggestion UX · FR-76, US-084
- [ ] **T-259** · 3h · Spike: Premium membership (F-37) packaging and billing provider · FR-77, US-082
- [ ] **T-260** · 3h · Spike: Employer analytics (F-38) k-anonymity aggregation · FR-78, US-083
- [ ] **T-261** · 3h · Spike: TOTP MFA (F-39), mandatory for staff · FR-79, US-081
- [ ] **T-262** · 4h · Spike: Mobile apps (F-40) approach (React Native/Expo vs PWA) · FR-80, US-085
- [ ] **T-263** · 2h · Spike: Learning paths (F-41) content model · FR-81, US-086
- [ ] **T-264** · 2h · Spike: Push notifications (F-42) web push + mobile · FR-82, US-087
- [ ] **T-265** · 2h · Spike: Survivor events (F-43) RSVP privacy model · FR-83, US-088
