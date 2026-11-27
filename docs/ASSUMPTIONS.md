# LinkedOut — Assumptions and Decisions

This file records every decision made where the master plan ([LinkedOut_Detailed_Product_and_Engineering_Plan.md](LinkedOut_Detailed_Product_and_Engineering_Plan.md)) was silent, offered alternatives, or contradicted itself. The product owner confirmed on 27 Nov 2026 that the recommended defaults should be used ("use your recommendations").

**Status values:** **Decided** (adopted; change only with a new entry) · **Open** (needs owner confirmation; the default is in effect until then).

To change a decision, add a new row that references the old ID and update the affected docs in the same PR.

---

## 1. Contradictions in the plan and how they were resolved

| ID | Plan conflict | Resolution | Affected docs |
|---|---|---|---|
| C-1 | §10 backend "Java/Spring Boot **or** Node.js/NestJS", while §10 also names BullMQ (Node-only) | NestJS (A-23) | TECH_STACK, ARCHITECTURE |
| C-2 | §10 auth "session-based **or** token auth" | Server-side sessions in Redis (A-24) | SECURITY, ARCHITECTURE, API_SPEC |
| C-3 | §5 lists reactions Relatable, Pain, Same Here, Survived; §6.4 lists I'm Dead, Pain, Same Here, Melting, Corporate Circus, Survived | The six from §6.4. §5 is described as metaphors (A-04) | FEATURES, DATABASE_SCHEMA, UI_UX |
| C-4 | §11 "one active reaction **of a given design** per user and post" is ambiguous | One reaction per member per post (A-04) | DATABASE_SCHEMA |
| C-5 | §17 puts Saved posts and Job interview reports in "Should", but the §18 roadmap builds saves (weeks 7–8) and Job Nightmare reports (Phase 4) before launch | Follow the roadmap: both MVP (A-01) | PRD, FEATURES, TASKS |
| C-6 | §26 build order puts Search 9th (after companies), while §18 puts search in weeks 7–8 | Search for posts in weeks 7–8, extended to companies/job reports in weeks 9–10 (A-02) | TASKS, ROADMAP |
| C-7 | §26 puts Reporting & moderation (6th) before Company pages (7th), while §18 schedules moderation in weeks 11–12 after companies | Reporting + queue built alongside confessions (weeks 9–10); actions, appeals, reveal in weeks 11–12; confessions kept behind a feature flag until all done (A-03) | TASKS, ROADMAP |
| C-8 | §11 `profiles.visibility` duplicates `privacy_settings.profile_visibility` | Single source: `privacy_settings.profile_visibility` (A-36) | DATABASE_SCHEMA |
| C-9 | §11 `anonymous_identity_aliases` has one alias per user, which would link all of a member's anonymous posts | Per-content aliases in an isolated vault schema (A-05, A-35) | DATABASE_SCHEMA, SECURITY |
| C-10 | §12 has `/confessions` endpoints but §11 has no confessions table | Confessions are `posts` with `post_type = confession` + `confession_details` (A-08) | DATABASE_SCHEMA, API_SPEC |
| C-11 | §12 `POST /moderation/:targetId/action` cannot identify the target type | `POST /moderation/items/:itemId/actions` (A-46) | API_SPEC |
| C-12 | §4 secondary text `#737373` on `#171717` is 3.8:1, failing WCAG AA for body text, which conflicts with §4 "high contrast" | Added `text-secondary #A3A3A3` (7.1:1); `#737373` used for borders and large/decorative text (A-52) | UI_UX |
| C-13 | §12 `company_responses.review_id` only, while §6.6/§6.7 show company responses to interview reports too | Responses target reviews or job reports (A-41) | DATABASE_SCHEMA, API_SPEC |
| C-14 | §9 nav lists "Trauma Network" and "Notifications" as MVP navigation, but §17 makes Communities and Notifications "Should" | Nav items hidden until their feature ships (A-53) | UI_UX |

---

## 2. Product and scope

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-01 | **MVP scope** = plan §17 "Must have" + Saved posts + Job Nightmare Board + Learned the Hard Way + Appeals + Audit/Identity Vault + Account deletion request. **Should (v1.1)** = Communities, Notifications, Polls, Company responses/representatives, Image posts, Topic following, Data export, Google OAuth. **Future** = everything in §17 "Later" + Hall of Shame, meme generator, events, learning paths, push. | Matches the §18 roadmap deliverables and §6.1 acceptance criteria (deletion). Appeals and audit are needed before anonymous posting (§26). | Decided |
| A-02 | Search ships in weeks 7–8 for posts/lessons/members and is extended in weeks 9–10 to confessions, job reports and companies. | Search only depends on content that exists by then. | Decided |
| A-03 | Confessions are built in weeks 9–10 behind `confessions.enabled` (off in production) and enabled only at the launch gate after moderation, appeals, reveal and audit ship. | Honors §26 "Do not launch anonymous posting before…" without blocking development. | Decided |
| A-04 | Reactions: I'm Dead, Pain, Same Here, Melting, Corporate Circus, Survived. **One reaction per member per post**, changeable. Counts appear only on posts. | §6.4 is the detailed spec. One reaction keeps counts meaningful and avoids "suffering scores" (§6.4). | Decided |
| A-05 | Anonymous labels are **generated per piece of content** ("Anonymous Stapler #4821": 60 office objects × 4-digit number). The OP keeps the label inside their own thread. | Prevents linking a member's anonymous posts together (stronger privacy than one alias per user). The humor fits the brand. | Decided |
| A-06 | Anonymous mode is available for **confessions and job reports**. Workplace reviews are **always** anonymous to members and companies. Stories, advice, lessons and comments are pseudonymous only (except OP replies in their own anonymous thread). | Anonymity where the risk of workplace retaliation is highest. Pseudonymity elsewhere keeps accountability and community. | Decided |
| A-07 | Default visibility: stories/advice/lessons `public`; confessions `members` (author may choose public); job reports `public`; reviews public. | Visitors can learn from content (Journey A step 2) while sensitive confessions get a smaller audience. | Decided |
| A-08 | Confessions are stored as `posts` (`post_type = confession`) + `confession_details`; lessons as `posts` + `lesson_details`. | One feed, one comment and reaction system, one moderation target type. | Decided |
| A-09 | Plan post formats map as follows: text story → `story`; question seeking advice → `advice`; lesson learned → `lesson`; poll → `poll` (Should); image or meme → `meme` + media (Should); optional long-form story → `story` with > 3,000 chars rendered as long-form; workplace experience report → workplace review (F-17); interview experience report → job report (F-18). | Avoids duplicate concepts while covering every plan format. | Decided |
| A-10 | Comments and reactions apply to posts only in MVP; reviews and job reports get reports, corrections (via moderation) and company responses, not comment threads. | Keeps company pages structured and reduces pile-ons (§6.6, §6.8 "not coordinated attacks"). | Decided |
| A-11 | Categories: the 12 from §6.3, seeded and admin-managed. Tags: free-form, normalized, max 5 per post. | §17 "Categories and tags". | Decided |
| A-12 | UI language is English only at launch. Posts carry a language tag (default `en`) for the §6.3 language filter. | Launch audience; i18n-ready strings. | Decided |
| A-13 | Minimum age **16**. Users confirm their age; date of birth is not stored. | GDPR digital-consent age varies from 13 to 16; 16 is the safe upper bound. Minimizes data collection. | Decided |
| A-14 | Evidence status values: `unverified` (default) and `verified` (staff only, Future process). There is no evidence upload in the MVP. All reviews and job reports show "Unverified firsthand account". | §6.6 "label as unverified unless independently verified". Uploading evidence creates confidentiality risk. | Decided |
| A-15 | Company theme summaries need **≥ 5 approved reviews in the last 24 months** per topic; summaries pause during campaign review. | §6.6 "avoid ratings based on tiny or manipulated samples". | Decided |
| A-16 | Review topics: work-life balance, management, communication, compensation & payment, career growth (1–5 each, optional per topic). | Covers the §6.6 page themes (work-life, management/communication, salary/payment) and Journey C filters. | Decided |
| A-17 | Account-age gates: reviews ≥ 7 days, community creation ≥ 14 days, ≤ 3 links per item for accounts < 7 days, 1 confession per day for accounts < 24 h. | Abuse prevention for fake reviews and coordinated attacks (§23). | Decided |
| A-18 | Career status values include `open_to_escape` ("Open to Escape" from §5). | Brand concept mapped to a concrete field. | Decided |
| A-19 | "My Damage" (§9 nav) is the private member dashboard in MVP (my posts, drafts, saved, decisions). The Career Damage Report metrics (§6.13) are Future (F-32). | §17 puts career analytics in "Later". The nav item still needs a home. | Decided |
| A-20 | Profile "career lessons" are short text items (≤ 280 chars, max 10). The "experience timeline" uses role categories and industries, with no employer names required. | Reduces identifiability (§6.2, §14). | Decided |
| A-21 | Community examples from §6.8 are seeded when communities launch (v1.1). | Gives the feature an immediate starting point. | Decided |
| A-22 | Beta: invite-only, 50 → 150 participants from the §24 audiences, 2 weeks. | Moderation capacity of a small team. | Decided |

## 3. Architecture

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-23 | **Backend: NestJS (Node.js 24, TypeScript)**, not Spring Boot. | One language across web, API, worker and shared schemas for a 2–3 person team. BullMQ (named in the plan) is Node-only. The plan says to avoid two stacks. | Decided |
| A-24 | **Authentication: server-side sessions** (random token in an httpOnly cookie; Redis + PostgreSQL), not JWT. | Immediate revocation (session management, suspension), simpler security model, no token storage in JS. The plan lists both options. | Decided |
| A-25 | pnpm + Turborepo monorepo with `apps/web`, `apps/api` (API + worker entry points) and `packages/*`. | Shared Zod schemas and types; one CI. | Decided |
| A-26 | Prisma 6 ORM with SQL migrations; hand-written SQL for grants, triggers and generated columns. | Type safety and approachable migrations for a small team. | Decided |
| A-27 | Hosting: **Vercel** (web), **Render, Frankfurt region** (API, worker, PostgreSQL with PITR, Key Value), **Cloudflare** (DNS, WAF, R2 storage), **Resend** (email), **Sentry**, **Grafana Cloud**. | Managed services minimize DevOps load. The EU region supports the GDPR baseline. Fits a ≤ USD 150/month budget. | Decided |
| A-28 | No AI/ML in MVP. Safety checks are deterministic rules (regex, libphonenumber, Luhn, entropy, term lists, velocity). | §17 puts AI-assisted categorization in "Later" and ML ranking out of scope. Rules are explainable. | Decided |
| A-29 | Identity Vault = a separate **PostgreSQL schema with a separate DB role** in the same database instance, not a separate database. | Strong-enough isolation for MVP cost; can move to its own instance later (ARCHITECTURE §9). | Decided |
| A-30 | Feature flags are stored in the `feature_flags` table and managed in the admin UI, with no third-party flag service. | Simple, auditable, no extra vendor receiving data. | Decided |
| A-31 | PostgreSQL full-text search uses the `simple` configuration + `unaccent`, ranked with a recency decay. OpenSearch only when NFR-P4 fails. | Multi-language content; the plan says "PostgreSQL initially; OpenSearch later". | Decided |
| A-32 | Notifications use 60-second polling of the unread count; no WebSockets or SSE in MVP or v1.1. | §17 excludes real-time chat; polling is enough for notifications. | Decided |
| A-33 | The background job queue (BullMQ) runs in a separate worker process from the same Docker image. | Independent scaling and failure isolation. | Decided |
| A-34 | UUID v7 primary keys (application-generated). | Non-sequential to the public (§11 "avoid exposing sequential IDs") while index-friendly. | Decided |

## 4. Database

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-35 | `vault.anonymous_identity_aliases` maps each anonymous post, job report and review to its account. Public tables store `alias_id` and the label, with `author_user_id` NULL. | §11 asks for tightly restricted access and a separate mapping. Removing the author ID from public rows means a public-schema leak reveals nothing. | Decided |
| A-36 | Profile visibility lives only in `privacy_settings`; `profiles` has no visibility column. | Resolves C-8. | Decided |
| A-37 | `privacy_settings.email_notification_settings` (plan) moved to `notification_preferences` (per type, per channel, quiet hours). | §6.11 needs richer controls than one field. | Decided |
| A-38 | Added tables not in §11: sessions, auth_tokens, oauth_accounts, username_history, reserved_usernames, profile_lessons, profile_skills, profile_timeline_entries, industries, categories, tags, post_tags, confession_details, lesson_details, post_revisions, post_corrections, saved_posts, polls, poll_options, poll_votes, post_media, review_topic_ratings, communities' rules/resources/join requests, topic_follows, blocks, mutes, moderation_items, identity_reveal_requests, staff_permissions, account_deletion_requests, data_export_requests, notification_preferences, feature_flags, beta_invites. | Needed to implement the features §6 describes. §11 calls itself "a logical starting model". | Decided |
| A-39 | `workplace_reviews.category` and `rating` (plan) became the `review_topic_ratings` child table. | Structured per-topic feedback (§6.6). | Decided |
| A-40 | Reviews and anonymous job reports don't store `author_user_id`; the one-review-per-company rule uses the vault `scope_key`. | Consistent anonymity design (A-35). | Decided |
| A-41 | `company_responses` targets `workplace_review` or `job_report` (plan only had `review_id`). | Resolves C-13. | Decided |
| A-42 | Appeals on anonymous content store `appellant_alias_id` instead of the user ID. | Prevents appeals from linking a user to anonymous content. | Decided |
| A-43 | Notifications about a member's own anonymous content store an encrypted reference. | Prevents notifications from becoming an identity-mapping table. | Decided |

## 5. API

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-44 | Base URL `https://api.linkedout.app/api/v1`; URI versioning; camelCase JSON; standard error envelope; cursor pagination (default 20, max 50). | §12 asks for versioned REST with consistent errors and pagination. | Decided |
| A-45 | Reactions: `POST /posts/:id/reactions` upserts the member's single reaction; `DELETE` removes it. | Matches the plan's paths with the A-04 rule. | Decided |
| A-46 | `POST /moderation/:targetId/action` → `POST /moderation/items/:itemId/actions`; `POST /moderation/:actionId/appeal` → `POST /moderation/actions/:actionId/appeal`. | Resolves C-11 and keeps REST resource naming consistent. | Decided |
| A-47 | `GET /companies/:companyIdOrSlug` accepts a UUID or a slug. | SEO-friendly web URLs (`/companies/acme-widgets`) without a second endpoint. | Decided |
| A-48 | Added endpoints not in §12: CSRF, username availability, resend verification, OAuth, `/me`, sessions, onboarding, username change, profile lists, blocks, mutes, saves, follows, exports, notification preferences, categories, industries, tags, safety check, corrections, pinned comment, comment edit/delete/replies, lessons, reviews edit/delete, job report detail/edit/delete, representation requests, company responses edit, search, my reports, my moderation actions, moderation items/claim/appeals, identity reveals, communities extras, notifications, media, admin, health. | §12 lists "example endpoint groups"; the features need these. | Decided |
| A-49 | Lessons are created via `POST /posts` (`postType: lesson`); confessions keep dedicated `/confessions` endpoints. | Confessions have distinct privacy handling (vault, flags, preview confirmation) that justifies their own endpoints (plan §12). | Decided |
| A-50 | Free-text endpoints require `acknowledgedFindings` for medium-risk safety findings (`422 SAFETY_CONFIRMATION_REQUIRED`). | Enforces "preview and confirmation" (§6.5) server-side, not just in the UI. | Decided |

## 6. Design

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-51 | Plan colors kept: `#171717`, `#F87171`, `#F5F5F5`, `#737373`. Added surfaces, semantic colors, focus ring `#FBBF24`. | Needed for a complete, accessible design system. | Decided |
| A-52 | Added `text-secondary #A3A3A3`. Text on red buttons is dark (`#171717`, 6.5:1) because white on `#F87171` is 2.8:1. Destructive buttons use `#DC2626` with white text (4.8:1). | WCAG 2.2 AA (C-12). | Decided |
| A-53 | Plan nav labels kept (The Mess, Trauma Network, Job Nightmares, Confessions, Company Reality, Learned the Hard Way, Notifications, My Damage). Items for features not yet live are hidden by feature flag. | Resolves C-14 without renaming the brand concepts. | Decided |
| A-54 | Typography: Bricolage Grotesque (display headings) + Inter (UI/body) + JetBrains Mono, self-hosted. | §4 "clear sans-serif; bold display headings". Self-hosting avoids third-party requests. | Decided |
| A-55 | Dark theme only in MVP; tokens are semantic so a light theme can follow. | §4 "dark-first". | Decided |
| A-56 | Reaction icons from lucide (skull, heart-crack, hand, flame, tent, shield-check) with text labels. | Accessible, consistent, not emoji-dependent. | Decided |

## 7. Privacy and legal

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-57 | **GDPR is the privacy baseline for all users** regardless of location; data hosted in the EU (Frankfurt). | Strictest common framework; covers the launch audience; the plan requires a documented retention schedule. | Decided |
| A-58 | Retention: soft-deleted content 30 days; drafts 180 days inactive; session IPs 90 days; notifications 180 days; moderation records, audit logs and username history 2 years; backups up to 12 months (see SECURITY §9.2). | Balances safety/legal needs against minimization (§6.1). | Decided |
| A-59 | Account deletion: 30-day grace period, then purge. Moderation and audit records are retained with a tombstone ID. | §6.1 acceptance criteria. | Decided |
| A-60 | Identity reveal: four-eyes approval, justification categories (`credible_threat`, `legal_order`, `repeat_abuse_ban_evasion`), 24 h access window, email never returned by the API. | §7 "only authorized privacy or safety personnel… with audit logging". | Decided |
| A-61 | Legal requests handled per SECURITY §10, with a semi-annual transparency report. | §14 "explain lawful disclosure obligations", §22 legal request runbook. | Decided |
| A-62 | Name and trademark risk: "LinkedOut" may conflict with "LinkedIn". A trademark search is task T-001 before launch. | Risk R-11. | **Open** |

## 8. Team and timeline

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-63 | Team of **2–3 people** (student or early-stage team). Roles from §21 are combined: product owner + T&S lead, full-stack engineer (frontend focus) + UI/UX, full-stack engineer (backend focus) + DevOps/QA. Moderation and privacy ownership are explicitly assigned to named people (T-005). | §21: one person can cover multiple roles, but moderation and privacy must be explicitly assigned. | Decided |
| A-64 | Start Monday **30 Nov 2026**; 16 weeks; public launch target **Thu 18 Mar 2027**; v1.1 on 14 May 2027. | §18's 12–16 weeks; the upper bound because of moderation/security work and the December holidays. | Decided |
| A-65 | Effort: 576 task-hours for the MVP (+25% overhead ≈ 720 h ≈ 45 team-hours/week). | Derived from TASKS.md. | Decided |
| A-66 | Production infrastructure budget ≤ USD 150/month at MVP scale. | Student/early-stage constraint; see DEPLOYMENT §1. | Decided |
| A-67 | Domain `linkedout.app` with subdomains `api.`, `files.`, `staging.`, `status.`. | A concrete name is needed for cookies, CORS and DNS docs. | **Open** (availability and purchase to confirm) |
| A-68 | GitHub organization `linkedout-app`, repository `linkedout`. | Needed for CI, container registry and links. | **Open** |
| A-69 | Moderator coverage for the first 14 days after launch: 08:00–24:00 IST/CET with on-call for P1 safety. SLAs: high priority ≤ 4 h, normal ≤ 24 h. | §15 workflow + §23 moderation overload risk. | Decided |

## 8b. Frontend-only phase

| ID | Decision | Reasoning | Status |
|---|---|---|---|
| A-70 | During the frontend-only phase, `apps/web` uses a **feature-folder layout** (`src/features/<feature>/{api,mocks,hooks,components,pages}`) with per-feature `copy.ts`, mock services behind a one-line `api/index.ts` switch, and shared types in `src/shared/types` (same names as the future `packages/shared`). The work is split across Arko, Shoaib, Tuli and Abnt per [team/TEAM_CONTRACT.md](team/TEAM_CONTRACT.md). | Lets four people work fully in parallel with no shared files and no backend. Swapping to the real API changes one file per feature. Supersedes the `src/components/<domain>` and single `en.json` layout in FOLDER_STRUCTURE §2 for this phase. | Decided |
| A-71 | Feature flags default to **on** in the mock environment (`confessions.enabled`, `communities.enabled`) so every screen can be built and demoed. Production launch gating (A-03) is unaffected. | Mock-only convenience. | Decided |

## 9. Open questions (owner decisions still needed)

| # | Question | Default in effect | Needed by |
|---|---|---|---|
| Q-1 | Is "LinkedOut" cleared for trademark use, or is a rename needed? (A-62) | Proceed with "LinkedOut" | Before beta (T-001, week 1) |
| Q-2 | Confirm or replace the domain `linkedout.app` (A-67) | `linkedout.app` | Week 1 (T-016) |
| Q-3 | Confirm the GitHub organization name (A-68) | `linkedout-app` | Week 1 (T-015) |
| Q-4 | Who are the named product owner, T&S lead and second identity-reveal approver? | Assigned at T-005 | Week 1 |
| Q-5 | Is legal counsel available (university legal clinic or retained lawyer) for defamation and lawful-request handling? | Assume a university legal clinic | Before beta |
| Q-6 | Should the launch geography be restricted (e.g. India + EU first) for moderation language coverage? | Global, English-only moderation | Before launch |
| Q-7 | Confirm team size and weekly hours (A-63/A-65); with fewer hours, the launch date moves rather than the scope. | 2–3 people, ~45 team-hours/week | Week 1 |
