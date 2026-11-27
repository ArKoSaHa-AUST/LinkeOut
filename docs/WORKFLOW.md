# LinkedOut — Workflows

Step-by-step flows for every major action, with Mermaid flowcharts and the backend steps behind each. Endpoint details are in [API_SPEC.md](API_SPEC.md), and screen details in [UI_UX.md](UI_UX.md).

| # | Workflow | Features | Plan journey |
|---|---|---|---|
| W-01 | Sign up & verify email | F-01, F-03 | A (steps 3–4) |
| W-02 | Log in, lockout, log out | F-01 | B (step 1) |
| W-03 | Password reset | F-01 | — |
| W-04 | Onboarding | F-04 | A |
| W-05 | Edit Anti-Profile, privacy settings & change username | F-03, F-05, F-06 | — |
| W-06 | Create a post (story, advice, lesson) | F-10, F-16, F-21 | A (steps 5–9) |
| W-07 | Browse The Mess | F-11, F-07 | A (step 2) |
| W-08 | Comment, reply, pin clarification | F-12 | A (step 10) |
| W-09 | React | F-13 | — |
| W-10 | Save | F-14 | A (step 11) |
| W-11 | Office Confession | F-15, F-21, F-23 | B |
| W-12 | Research a company & write a review | F-17 | C |
| W-13 | Submit a company | F-17, F-24 | — |
| W-14 | Job Nightmare report | F-18 | — |
| W-15 | Search | F-19 | C (step 1) |
| W-16 | Block & mute | F-07 | — |
| W-17 | Report content | F-20 | — |
| W-18 | Moderation review & action | F-21 | — |
| W-19 | Appeal | F-22 | — |
| W-20 | Review campaign detection | F-21 | — |
| W-21 | Identity reveal (four-eyes) | F-23 | — |
| W-22 | Account deletion | F-08 | — |
| W-23 | Admin: company approval, merge, launch gate | F-24 | — |
| W-24 | Company representative verification & response (Should) | F-25 | D |
| W-25 | Communities (Should) | F-26 | — |
| W-26 | Notifications (Should) | F-27 | — |
| W-27 | Polls (Should) | F-28 | — |
| W-28 | Media upload (Should) | F-29 | — |
| W-29 | Data export (Should) | F-09 | — |
| W-30 | Google sign-in (Should) | F-02 | — |
| W-31 | Following feed (Should) | F-30 | — |
| W-32 | Error and edge cases | all | — |
| W-33 | Future feature flows (outline) | F-31–F-43 | — |

---

## W-01 — Sign up & verify email

```mermaid
flowchart TD
  A[Visitor opens /signup] --> B[GET /auth/csrf]
  B --> C[Types username → GET /auth/username-availability]
  C --> D[Submits form → POST /auth/register]
  D --> E{Valid? rate limit ok? invite ok?}
  E -->|No| F[Inline errors / 429]
  E -->|Yes| G{Email already registered?}
  G -->|Yes| H[Queue 'someone tried to register' email]
  G -->|No| I[Create user + profile + privacy_settings<br/>status pending_verification]
  I --> J[Create email_verification token, queue email]
  H --> K[202 'Check your inbox']
  J --> K
  K --> L[User clicks link /verify-email?token=]
  L --> M[POST /auth/verify-email]
  M --> N{Token valid & unused & < 24h?}
  N -->|No| O[Expired panel + Resend]
  N -->|Yes| P[Set email_verified_at, status active, create session]
  P --> Q[Redirect /onboarding]
```

**Backend steps**
1. `AuthController.register` → Zod validation (email, password 12–128, username regex, `ageConfirmed`, `acceptTerms`).
2. Throttler `register` (5/h/IP). If `beta.invite_only`, the invite code is validated and its `used_count` incremented atomically.
3. `PasswordService.checkBreached` (HIBP range API, 2 s timeout, fail-open + log).
4. Username check against `profiles`, `reserved_usernames` and `username_history.reserved_until > now()`.
5. In one transaction: insert `users` (Argon2id hash, `terms_version`, `age_confirmed_at`), `profiles`, and `privacy_settings` (defaults).
6. `TokenService.create(email_verification, 24h)` stores the SHA-256 and enqueues the `email` job `VerifyEmail`.
7. Already-registered email: no DB writes; the `RegistrationAttempt` email is enqueued. Responses are identical (`202`).
8. Verify: look up the token by hash and check `used_at IS NULL AND expires_at > now()`. Then set `used_at`, `users.email_verified_at` and `account_status = active`, create a session (W-02 steps 6–8), and audit `auth.email_verified`.

## W-02 — Log in, lockout, log out

```mermaid
flowchart TD
  A[POST /auth/login] --> B{Rate limit ok?}
  B -->|No| Z[429]
  B -->|Yes| C{User exists?}
  C -->|No| D[Verify dummy hash] --> E[401 INVALID_CREDENTIALS]
  C -->|Yes| F{locked_until > now?}
  F -->|Yes| G[423 ACCOUNT_LOCKED]
  F -->|No| H{Password valid?}
  H -->|No| I[failed_login_count++] --> J{count ≥ 10?}
  J -->|Yes| K[locked_until = now+15m, email notice] --> E
  J -->|No| E
  H -->|Yes| L{account_status}
  L -->|suspended| M[403 ACCOUNT_SUSPENDED + appeal link]
  L -->|deletion_pending| N[200 limited: offer cancel deletion]
  L -->|pending_verification| O[Session created, UI → /verify-email]
  L -->|active / restricted| P[Create session, reset counter, audit]
  P --> Q[Set lo_sid cookie → /mess]
```

**Backend steps**
1. Normalize the email (lowercase via citext).
2. Argon2 verify (dummy-hash verification when the user is missing keeps timing equal).
3. On failure: increment `failed_login_count`; at 10, set `locked_until` and enqueue the `LoginLocked` email with a reset link; audit `auth.login_failed` (no password data).
4. On success: reset the counter, rehash if the Argon2 parameters changed, set `last_login_at`.
5. Create the session token (32 random bytes), insert `sessions` (device label from UA, approx location from GeoLite2), `SET sess:{hash}` with TTL, and add it to `user_sessions:{userId}`.
6. Audit `auth.login_succeeded`.
7. **Logout:** `DEL sess:{hash}`, set `sessions.revoked_at`, clear the cookie, audit `auth.logout`.

## W-03 — Password reset

```mermaid
flowchart TD
  A["/forgot-password: enter email"] --> B[POST /auth/forgot-password]
  B --> C{User exists & active?}
  C -->|Yes| D[Create reset token 60m, queue email]
  C -->|No| E[No-op]
  D & E --> F[202 neutral message]
  F --> G[User opens /reset-password?token=]
  G --> H[POST /auth/reset-password]
  H --> I{Token valid?}
  I -->|No| J[TOKEN_INVALID → request new link]
  I -->|Yes| K{Password ok? not breached?}
  K -->|No| L[400 VALIDATION_ERROR]
  K -->|Yes| M[Update hash, revoke ALL sessions, mark token used]
  M --> N[New session, confirmation email, audit auth.password_reset]
```

## W-04 — Onboarding

```mermaid
flowchart LR
  A[Verified, onboarding incomplete] --> S1[1 Pseudonym vs anonymous]
  S1 --> S2[2 Data we collect]
  S2 --> S3[3 Lawful disclosure]
  S3 --> S4[4 Guidelines + acknowledge]
  S4 --> S5[5 Pick interests optional]
  S5 --> B[POST /me/onboarding/complete]
  B --> C{All 4 steps acknowledged?}
  C -->|No| S1
  C -->|Yes| D[Set onboarding_completed_at · save interest categories as suggested filters] --> E["/mess"]
```

**Backend steps:** validate that `acknowledgedSteps` contains all 4 values. Set `users.onboarding_completed_at`. Store interest categories in `privacy_settings`-adjacent user preferences (local UI filter defaults returned in `GET /me`; follows are created only once F-30 ships). Audit `identity.onboarding_completed` with the guideline version.

## W-05 — Edit Anti-Profile, privacy settings & change username

1. Member opens S-25 → `GET /me`.
2. Edits fields → `PATCH /me/profile`. The server validates lengths and enums and runs `SafetyCheckService(context=profile)` on the headline and bio.
3. If there are medium findings without acknowledgement → `422 SAFETY_CONFIRMATION_REQUIRED`. The UI shows the findings, and the member confirms or edits.
4. Lessons, skills and timeline are saved with `PUT` replace semantics in one transaction (delete + insert with positions).
5. **Privacy settings (F-06):** S-26 → `GET /me/privacy` → change profile visibility, discoverability, default confession identity, default post visibility → `PATCH /me/privacy`. The server validates the enums and applies them on the next request. Profile reads (`GET /profiles/:username`), member search and composer defaults read these values. Audit `profile.privacy_changed`.
6. Username change → `PATCH /me/username`:

```mermaid
flowchart TD
  A[PATCH /me/username] --> B{username_changed_at within 30 days?}
  B -->|Yes| C[403 change_too_soon + next date]
  B -->|No| D{Available? not reserved/held?}
  D -->|No| E[409 CONFLICT]
  D -->|Yes| F[Tx: insert username_history old name reserved_until now+90d · update profiles.username, username_changed_at]
  F --> G[Invalidate cached AuthorRefs · audit profile.username_changed] --> H[200]
```

## W-06 — Create a post (story, advice, lesson)

```mermaid
flowchart TD
  A[Share a Story / Ask for Advice / Write a Lesson] --> B{Verified & onboarded & not restricted?}
  B -->|No| C[CTA panel: verify / onboard / restriction end date]
  B -->|Yes| D[Compose in S-08]
  D --> E[Preview → POST /safety/check]
  E --> F{Findings?}
  F -->|blocking| G[Must edit text]
  F -->|medium| H[Show findings · acknowledge or edit]
  F -->|none/low| I[Show tips]
  G --> D
  H & I --> J[Publish → POST /posts with acknowledgedFindings]
  J --> K{Server re-check}
  K -->|blocking| L[422 SAFETY_CHECK_BLOCKED]
  K -->|confirmation missing| M[422 SAFETY_CONFIRMATION_REQUIRED]
  K -->|high| N[Save moderation_status=held + moderation_item automated_hold]
  K -->|ok| O[Save moderation_status=clear, published_at=now]
  N --> P[Redirect to post with 'waiting for review' notice]
  O --> Q[Redirect to post · invalidate feed cache]
```

**Backend steps**
1. Guards: Session → CSRF → Verified → Throttler (`posts`: 10/h, 30/day).
2. Zod validation by `postType`. Lessons need `lesson.whatILearned`; the server composes `body` from the sections for search and excerpts.
3. Tags normalized (lowercase, hyphenated, max 40 chars) and upserted; `usage_count` incremented.
4. `SafetyCheckService.check(context, fields, author)` runs the rules from SECURITY §8.1 plus velocity rules for the author.
5. Insert `posts` (+ `lesson_details`, + `post_tags`) in one transaction. If held, upsert a `moderation_items` row (`automated_hold = true`, findings, priority from severity).
6. Emit `post.created`. Handlers invalidate `feed:v1:*`, enqueue `safety-scan` (post-publish re-scan), and later enqueue follower digests (F-30).
7. Return PostDetail (`moderationNotice` if held).

## W-07 — Browse The Mess

```mermaid
flowchart TD
  A[GET /feed?mode=latest&filters] --> B{Session?}
  B -->|No| C{Cache hit feed:v1:latest:hash?}
  C -->|Yes| D[Return cached page]
  C -->|No| E[Query public, clear, published posts by keyset] --> F[Cache 30s] --> D
  B -->|Yes| G[Query public+members posts<br/>exclude blocked users/aliases, muted users/categories/tags,<br/>private communities not joined]
  G --> H[Batch load viewer reactions & saves] --> I[Return page + nextCursor]
```

**Backend steps:** decode the cursor; `WHERE (published_at, id) < (:ts, :id)`; `LIMIT n+1` to compute `hasMore`; map rows through the PostSummary serializer (author: profile for pseudonym, label for anonymous); apply ContentWarning metadata (the UI collapses). Hidden categories for visitors come via `excludeCategory`.

## W-08 — Comment, reply, pin clarification

```mermaid
flowchart TD
  A[Write comment] --> B[POST /posts/:id/comments]
  B --> C{Post visible? not blocked by post author?}
  C -->|No| D[404 / 403 BLOCKED]
  C -->|Yes| E{identityMode=anonymous?}
  E -->|Yes| F{vault.isOwner user, post.alias_id?}
  F -->|No| G[403 FORBIDDEN]
  F -->|Yes| H[Use post alias, isOp=true]
  E -->|No| H2[Author = user]
  H & H2 --> I[Safety check context=comment]
  I --> J[Compute depth = min parent.depth+1, 2]
  J --> K[Tx: insert comment, posts.comment_count++]
  K --> L[Emit comment.created → notifications Should]
  L --> M[201 Comment]
```

- **Edit:** `PATCH /comments/:id`. Owner check (user ID or vault for OP comments), safety re-check, `edited_at`.
- **Delete:** with replies → `deleted_at` set and the body shown as "[deleted]"; without replies → soft-deleted and hidden; `comment_count` decremented.
- **Pin clarification:** `PUT /posts/:id/pinned-comment` validates that the comment belongs to the post and was written by the post author (same user ID, or same alias for anonymous posts), then sets `posts.pinned_comment_id`.

## W-09 — React

```mermaid
flowchart LR
  A[Click reaction] --> B[Optimistic UI update]
  B --> C[POST /posts/:id/reactions type]
  C --> D{Existing reaction?}
  D -->|None| E[Insert · counts.type +1]
  D -->|Same type| F[No-op 200]
  D -->|Different| G[Update type · counts.old -1, counts.new +1]
  E & F & G --> H[Return counts] --> I{Error?}
  I -->|Yes| J[Roll back optimistic state + toast]
```

Backend: a single transaction with `SELECT … FOR UPDATE` on the reaction row and a `jsonb_set` counter update on `posts.reaction_counts`. Block check: if the post author blocked the reactor → `403 BLOCKED`. Toggle-off uses `DELETE`.

## W-10 — Save

`PUT /posts/:id/save` → `INSERT … ON CONFLICT DO NOTHING` into `saved_posts`. `DELETE` removes it. My Damage → Saved lists `GET /me/saved` (join with visibility checks; unavailable posts are flagged). Saves are never exposed publicly or counted.

## W-11 — Office Confession (Journey B)

```mermaid
flowchart TD
  A[Selects Confessions → Confess anonymously] --> B{confessions.enabled?}
  B -->|No| C[Coming soon panel]
  B -->|Yes| D[Step 1 Write: title, body, experience type, desired response, CW, optional industry/region/period]
  D --> E[Step 2 Safety check & preview: POST /safety/check context=confession]
  E --> F{Findings}
  F -->|blocking e.g. credential| G[Edit required]
  F -->|named individual / PII| H[Highlight + 'describe the role instead' · will be held]
  F -->|none| I[Preview with label sample]
  G --> D
  H --> J[Step 3 Confirm checklist]
  I --> J
  J --> K{Publish or Save draft?}
  K -->|Draft| L[POST /confessions saveAsDraft=true → publication_status=draft]
  K -->|Publish| M[POST /confessions previewConfirmed=true]
  M --> N[Server safety re-check + rate limit 5/24h]
  N --> O{identityMode}
  O -->|anonymous| P[VaultService.createAlias → label 'Anonymous Stapler #4821']
  O -->|pseudonym| Q[author_user_id = user]
  P & Q --> R{Risk medium/high?}
  R -->|Yes| S[moderation_status=held + moderation item] --> T[Notice: waiting for review]
  R -->|No| U[Published] --> V[Members discuss constructively]
  S --> W[Moderator approves → published W-18]
```

**Backend steps**
1. `@FeatureFlag('confessions.enabled')` guard.
2. Validation: required fields; `previewConfirmed = true` unless drafting; `periodQuarter` requires `periodYear`.
3. Safety check with `context=confession`. Any medium finding means hold (SECURITY §8.1).
4. Pre-generate the post UUIDv7. For anonymous mode, `VaultService.createAlias(userId, 'post', postId)` produces the label: a random pick from 60 office objects plus a random 4-digit number, unique among aliases (retry on collision).
5. Insert `posts` (`post_type=confession`, `identity_mode`, `author_user_id` NULL when anonymous, `alias_id`, `anonymous_label`) and `confession_details` in one transaction. On failure, delete the vault row (ARCHITECTURE §4.3).
6. If held, create a moderation item with priority by severity (named individual → normal, threats → high).
7. Logging: only `postId` and the outcome are logged. The user ID is never logged in this flow.
8. The response shows the AuthorRef `{kind:"anonymous", label}`. My Damage lists it via `vault.listOwnTargets(userId)`.
9. **Drafts:** `publication_status=draft`. The daily maintenance job sends the 7-day warning at 173 days of inactivity and deletes at 180 days (including the vault row).
10. **Corrections:** `POST /posts/:id/corrections` → ownership via vault → insert `post_corrections` (max 5).

## W-12 — Research a company & write a review (Journey C)

```mermaid
flowchart TD
  A[Search company → GET /companies?q=] --> B[Open /companies/slug]
  B --> C[GET /companies/:slug → themes if n≥5, paused banner if campaign]
  C --> D[Filter by topic / relationship / period → GET /companies/:id/reviews]
  D --> E[Read date, relationship, 'Unverified firsthand account' label, company response]
  E --> F[Member decides independently]
  E --> G{Wants to add own experience?}
  G -->|Yes| H{Account ≥ 7 days & verified?}
  H -->|No| I[Explain gate + date]
  H -->|Yes| J{vault.hasScope review:companyId?}
  J -->|Yes| K[Offer edit of existing review]
  J -->|No| L[S-17 form → POST /companies/:id/reviews]
  L --> M[Safety check · named individuals → held]
  M --> N[Vault alias with scope_key review:companyId]
  N --> O[Insert review + topic ratings]
  O --> P[Mark company themes dirty · campaign detector W-20]
  P --> Q[201 → review visible / held]
```

**Theme summary computation:** a maintenance job runs every 15 minutes for dirty companies. Per topic, it takes approved reviews (`moderation_status = clear`, not deleted, `published_at` within 24 months) and computes `n`, the median and the 1–5 distribution, storing them only when `n ≥ 5`. Non-consenting narratives still count.

## W-13 — Submit a company

1. Member searches and finds nothing → S-18 → `POST /companies`.
2. The server normalizes the name (lowercase, unaccent, strips legal suffixes) and extracts the registrable domain from the website.
3. Duplicate check on `normalized_name` and `website_domain` among pending/active companies → `409` with the existing company.
4. Insert `companies` (`page_status = pending`, `submitted_by_user_id`). Reviews and job reports referencing it are stored but unpublished.
5. Admin approves in S-31 (W-23). Attached clear content gets `published_at = now()`.

## W-14 — Job Nightmare report

```mermaid
flowchart TD
  A["/job-nightmares/new"] --> B[Pick company or add new inline]
  B --> C[Role category, month, stage reached, type, outcome, narrative, identity mode, visibility]
  C --> D[POST /safety/check context=job_report]
  D --> E{interview_dump / PII / named individual?}
  E -->|Yes| F[Warn: no confidential questions or recruiter details · medium → held]
  E -->|No| G[OK]
  F & G --> H[POST /job-reports]
  H --> I{newCompany?}
  I -->|Yes| J[Create pending company W-13] --> K
  I -->|No| K[Validate company active]
  K --> L{anonymous?}
  L -->|Yes| M[Vault alias target_type=job_report]
  L -->|No| N[author_user_id]
  M & N --> O[Insert job_report] --> P[Show on board + company page]
```

## W-15 — Search

1. `GET /search?q=&type=`; `q` is trimmed, 2–100 chars, and parsed with `websearch_to_tsquery('simple', unaccent(q))`.
2. Posts, lessons, confessions and job reports: `search_vector @@ query` with visibility/moderation/block/mute predicates, ranked by `ts_rank_cd` × recency decay (`1 / (1 + age_days/30)`).
3. Companies: `similarity(normalized_name, q) > 0.3` ordered by similarity, then `review_count`.
4. Members: username trigram, only `discoverable = true` and visibility ≠ hidden.
5. Highlights via `ts_headline` with `StartSel=<mark>, StopSel=</mark>` on escaped text.
6. `type=all` returns the top 5 per group; specific types are paginated with cursors (`rank|id`).

## W-16 — Block & mute

```mermaid
flowchart TD
  A[Overflow menu → Block] --> B{Target}
  B -->|member| C[POST /me/blocks username]
  B -->|anonymous label| D[POST /me/blocks postId/jobReportId]
  C --> E[Insert blocks.blocked_user_id]
  D --> F[Resolve alias_id from content, insert blocks.blocked_alias_id]
  E & F --> G[Feed/search/thread queries exclude · interactions on blocker's content rejected]
  H[Overflow → Mute author/category/tag] --> I[POST /me/mutes] --> J[Feed/search exclude only]
```

Rules: self-block → 400. Blocking an alias never queries the vault, because the alias ID is taken from the content row. Interaction checks run when the blocked party comments or reacts on the blocker's pseudonymous content; for the blocker's anonymous content, the vault resolves the owner internally.

## W-17 — Report content

```mermaid
flowchart TD
  A[Report on post/comment/profile/review/job report/response/community] --> B[ReportDialog: reason + details]
  B --> C[POST /reports]
  C --> D{Target visible? already reported by me?}
  D -->|Not visible| E[404]
  D -->|Duplicate| F[409 ALREADY_EXISTS]
  D -->|OK| G[Upsert moderation_item for target: report_count++, priority recomputed]
  G --> H[Insert report linked to item]
  H --> I{Community content & reason not threat/doxxing?}
  I -->|Yes| J[Item scope=community → community queue]
  I -->|No| K[Platform queue]
  J & K --> L[201 'Thanks — our moderators will review this']
```

## W-18 — Moderation review & action

```mermaid
flowchart TD
  A[Moderator opens /mod queue] --> B[GET /moderation/queue sorted by priority_score]
  B --> C[Open item → GET /moderation/items/:id]
  C --> D[Claim → POST /claim]
  D --> E[Review content, reports, findings, history, policy]
  E --> F{Decision}
  F -->|No violation| G[no_action → reports no_violation · held content published]
  F -->|Violation| H[Choose proportionate action + reason + policy ref + member message]
  H --> I{Action}
  I -->|request_edits| J[Content stays held/visible · author emailed with guidance]
  I -->|add_content_warning| K[Append CW]
  I -->|limit_visibility| L[moderation_status=limited: hidden from feeds/search, visible by direct link to members]
  I -->|remove_content| M[moderation_status=removed]
  I -->|restrict_posting| N[users.posting_restricted_until = now + 1/7/30d via vault if anonymous]
  I -->|suspend_account| O[account_status=suspended, revoke sessions]
  I -->|escalate| P[item status escalated → admin escalations]
  G & J & K & L & M & N & O & P --> Q[Insert moderation_action, resolve reports, audit]
  Q --> R[Email affected member with explanation + appeal link]
  Q --> S[Email reporters outcome summary]
```

**Backend steps**
1. Claim sets `assigned_moderator_id` (conditional update `WHERE assigned_moderator_id IS NULL`) → `409` if it was taken in the meantime.
2. Action validation per type (API §15).
3. Transaction: apply the effect to the target, insert `moderation_actions`, update the item (`resolved`, or `escalated`), update reports (`actioned` / `no_violation`), audit `moderation.action_applied`.
4. For anonymous targets with account-level actions: `VaultService.resolveForEnforcement(aliasId)` returns the user ID inside the service and applies the restriction. Audit `vault.enforcement_resolved` (no identity is returned to the moderator).
5. Enqueue the `ModerationDecision` email to the affected member (the address is resolved internally) and `ReportOutcome` emails to reporters.

## W-19 — Appeal

```mermaid
flowchart TD
  A[Member: S-22 Decisions about me] --> B[GET /me/moderation-actions]
  B --> C[Appeal → POST /moderation/actions/:id/appeal]
  C --> D{Affected member? within 30 days? no prior appeal?}
  D -->|No| E[403 / 400 window closed / 409]
  D -->|Yes| F[Insert appeal pending: user id or alias id]
  F --> G[Appears in /mod/appeals]
  G --> H[Different moderator decides]
  H --> I{Outcome}
  I -->|upheld| J[No change]
  I -->|reversed| K[Undo effect: restore content/status/lift restriction · action.reversed_at]
  I -->|modified| L[Undo + apply new action]
  J & K & L --> M[Email member with decision note · audit moderation.appeal_decided]
```

## W-20 — Review campaign detection

```mermaid
flowchart TD
  A[review.created] --> B[safety-scan job]
  B --> C{> 5 reviews for company in 24h?}
  B --> D{> 3 from accounts < 14 days in 7 days?}
  B --> E{Similarity > 0.8 with another review in 30 days? pg_trgm}
  C & D & E -->|any yes| F[Upsert moderation_item target_type=review_campaign target_id=company, related_target_ids += review ids, priority normal]
  F --> G[companies.summaries_paused = true]
  G --> H[Moderator reviews group]
  H --> I{Legitimate?}
  I -->|Yes| J[no_action → summaries resume]
  I -->|No| K[remove_content for manipulated reviews, restrict authors via vault · summaries recomputed]
```

## W-21 — Identity reveal (four-eyes)

```mermaid
sequenceDiagram
  autonumber
  participant O1 as T&S officer 1
  participant API
  participant V as VaultService
  participant O2 as T&S officer 2
  participant AUD as audit_logs
  O1->>API: POST /trust-safety/identity-reveals {aliasSource, category, justification}
  API->>API: VaultPermissionGuard (identity_vault.reveal)
  API->>AUD: vault.reveal_requested
  API-->>O1: 201 pending_approval
  O2->>API: POST /identity-reveals/:id/approve
  API->>API: approver ≠ requester, permission check
  API->>AUD: vault.reveal_approved
  O1->>API: GET /identity-reveals/:id (within 24h)
  API->>V: revealForApprovedRequest(id)
  V-->>API: userId, username, status
  API->>AUD: vault.reveal_viewed
  API-->>O1: account summary (no email)
```

Unauthorized attempts are audit-logged (`vault.reveal_denied`) and trigger a Grafana alert.

## W-22 — Account deletion

```mermaid
flowchart TD
  A[S-29 Delete account] --> B[Show retained-data table]
  B --> C[Password + type DELETE → DELETE /me/account]
  C --> D{Password ok?}
  D -->|No| E[401]
  D -->|Yes| F[status deletion_pending · deletion request scheduled_for now+30d · revoke sessions · hide profile & content]
  F --> G[Email with cancel instructions]
  G --> H{Logs in & cancels within 30 days?}
  H -->|Yes| I[POST /me/account/cancel-deletion → status active, content visible]
  H -->|No| J[Daily account-deletion job]
  J --> K{Legal hold / escalated items?}
  K -->|Yes| L[Retain held content hidden · delete everything else]
  K -->|No| M[Delete content, vault aliases, profile, settings, sessions, tokens · tombstone email]
  L & M --> N[Final confirmation email sent before tombstoning · audit identity.account_deleted]
```

## W-23 — Admin: company approval, merge, launch gate

1. **Approve:** S-31 Companies → `PATCH /admin/companies/:id {pageStatus: active, websiteConfirmed}` → publishes attached clear reviews and reports → audit `admin.company_approved`.
2. **Merge:** `POST /admin/companies/:id/merge {intoCompanyId}`. In one transaction: move `workplace_reviews`, `job_reports`, `company_memberships`, `company_responses`; update the vault `scope_key` for moved reviews via `VaultService.rescopeReviews` (where duplicates collide, keep the newest review and hide the older one with `limit_visibility` and a note); set the source to `merged` with `merged_into_company_id`; recompute themes.
3. **Launch gate for confessions:** `PATCH /admin/feature-flags/confessions.enabled {enabled: true, launchGateConfirmed: true}` is allowed only when F-20, F-21, F-22 and F-23 are deployed and the ROADMAP launch criteria are checked; audit `admin.feature_flag_changed`.

## W-24 — Company representative verification & response (Should, Journey D)

```mermaid
flowchart TD
  A[Rep opens /companies/slug/represent] --> B[POST representation-requests workEmail, jobTitle]
  B --> C{Company website confirmed & email domain matches?}
  C -->|No| D[400 domain_mismatch]
  C -->|Yes| E[membership pending_email · token emailed to work address · full email not stored]
  E --> F[Rep clicks link → verify-email → pending_review]
  F --> G[Admin reviews → decision verified]
  G --> H[Membership verified, expires in 12 months]
  H --> I[Rep writes response to review/job report]
  I --> J[POST /companies/:id/responses]
  J --> K[Safety check: PII/policy → held if medium+]
  K --> L[Response shown with 'Official representative' badge]
  L --> M[Review author notified Should · author identity stays private]
```

## W-25 — Communities (Should)

1. **Create:** `POST /communities` (age ≥ 14 days, ≤ 3 owned, ≥ 1 rule, coordinated-attack acknowledgement) → owner membership row.
2. **Join:** public → member immediately (`member_count++`); private → join request → a community moderator approves or declines (a declined member can retry after 30 days).
3. **Post:** the composer's community select → `visibility = community` for private communities, public communities keep public/members visibility → the post appears in the community feed (and in The Mess if the community is public).
4. **Moderate:** reports on community content create `scope = community` items → community moderators act (remove from community, add a content warning, ban from community). Threats and doxxing are mirrored to the platform queue.
5. **Leave:** delete the membership; the owner must transfer ownership first.

## W-26 — Notifications (Should)

```mermaid
flowchart TD
  A[Domain event e.g. comment.created] --> B[notifications queue job]
  B --> C{Recipient = actor? blocked? muted actor?}
  C -->|Yes| X[Drop]
  C -->|No| D{Type = reactions_on_post?}
  D -->|Yes| E[Upsert row by batch_key reactions:post:hour, actorCount++]
  D -->|No| F[Insert notification · is_sensitive if about recipient's anonymous content → encrypt reference]
  E & F --> G{Email enabled for type? or moderation_decision}
  G -->|No| H[In-app only]
  G -->|Yes| I{Quiet hours now?}
  I -->|Yes| J[Delay email job until quiet end]
  I -->|No| K[Send via email queue]
```

The web app polls `GET /notifications/unread-count` every 60 s while the tab is visible (Page Visibility API).

## W-27 — Polls (Should)

1. Create via `POST /posts {postType: poll, poll: {options 2–6, durationDays 1–14}}` → `polls.closes_at = published_at + duration`.
2. Vote: `POST /posts/:id/poll/votes` → insert `poll_votes` (PK prevents double votes) and increment `vote_count`/`total_votes` in one transaction.
3. Results are returned only after the viewer votes or the poll closes; individual votes are never exposed.

## W-28 — Media upload (Should)

```mermaid
flowchart LR
  A[Select image] --> B[Client checks type/size] --> C[POST /media/uploads purpose, mime, size, alt]
  C --> D[media row scan_status=pending · presigned PUT 10 min]
  D --> E[Browser PUT directly to R2 uploads/]
  E --> F[POST /media/:id/complete]
  F --> G[media-scan job: magic bytes → sharp re-encode, strip EXIF → ClamAV]
  G --> H{Clean?}
  H -->|Yes| I[Move to m/ prefix, scan_status=clean]
  H -->|No| J[Delete object, scan_status=infected, alert]
  I --> K[Attach to post via mediaIds]
```

## W-29 — Data export (Should)

`POST /me/exports` (password re-auth) → `data_export_requests` pending → the `data-export` job collects the profile, settings, posts (including own anonymous content via the vault), comments, reactions, saves, reports filed and moderation decisions about the member → writes JSON files → ZIP → uploads to `linkedout-exports/{exportId}.zip` → emails a 7-day presigned link → the lifecycle rule deletes it after 7 days.

## W-30 — Google sign-in (Should)

```mermaid
flowchart TD
  A[Continue with Google] --> B[GET /auth/oauth/google → state, nonce, PKCE cookie]
  B --> C[Google consent] --> D[Callback: validate state, exchange code, verify ID token]
  D --> E{oauth_accounts match?}
  E -->|Yes| F[Create session → /mess]
  E -->|No| G{users.email match?}
  G -->|Yes| H["/login?link=google: password confirm then link"]
  G -->|No| I["/signup/complete: username + policies → POST /auth/oauth/complete-signup"]
  I --> J[Create user email_verified_at=now, profile, oauth link, session → onboarding]
```

## W-31 — Following feed (Should)

`GET /feed?mode=following` → posts where `category_id IN followed categories OR EXISTS post_tags in followed tags OR community_id IN joined communities`, with the same exclusions as W-07, ordered by `published_at`. Empty → suggestions from `GET /categories`.

---

## W-33 — Future feature flows (outline, detailed during each spike)

These flows set the intended shape and the safety rules each Future feature must keep. Each is detailed in its Phase 10 spike ([TASKS.md](TASKS.md)).

| Feature | Flow outline | Backend steps | Non-negotiable rules |
|---|---|---|---|
| F-31 Direct Messaging | Member opens a pseudonymous profile → "Message" → if the recipient allows `requests_only`, a message request is sent → the recipient accepts or declines → the conversation opens; block or report from the thread. | Check `privacy_settings.message_permissions` and blocks → insert `message_requests` → on accept create `conversations` + members → `messages` inserts rate-limited (20/10 min) → safety check per message → notification. | Anonymous labels are never messageable; no attachments in v1; safety notice on the first message; admins cannot browse messages. |
| F-32 Career Damage Report | Member opens My Damage → Report → adds tracked applications and goals → sees private counts and progress notes. | CRUD on `career_applications`, `career_goals` (owner only) → aggregate counts on read. | Never public, never in exports to third parties, no mental-health inference, no ranking. |
| F-33 Meme Generator | Choose template → add text → preview (safety check on text) → export PNG or share as a `meme` post. | `GET /meme-templates` → `POST /memes` renders server-side with sharp → creates `media` (clean) → optional `POST /posts` type `meme`. | PII/harassment checks on overlay text; templates admin-curated; no faces of real private individuals. |
| F-34 Hall of Shame | Moderator nominates a recurring pattern from anonymized moderated stories → editorial review → publish a pattern highlight linking to example posts (with consent of pseudonymous authors). | `hall_of_shame_entries` created by moderators → admin approval → public listing. | Patterns only; never names individuals or employees; not a leaderboard. |
| F-35 Relevant feed | Member switches to Relevant → ranked feed → one click back to Latest. | Candidate generation (last 14 days) → score = topic match × recency decay × meaningful-reply factor × diversity penalty → cache per user for 5 min. | No anger/controversy signals; no sensitive attributes; repeated distressing content is down-weighted. |
| F-36 AI-assisted categorization | Composer suggests a category and content warnings → author accepts or ignores. | `POST /safety/suggest` calls the model with text only (no identity) → returns suggestions → nothing is stored unless the author accepts. | Suggestions only; never auto-moderates; provider DPA and EU processing required. |
| F-37 Premium | Member views plans → checkout via a billing provider → premium features unlocked. | Billing webhook → `subscriptions` row → entitlement checks in guards. | Safety and privacy features never paywalled; payment identity never linked to anonymous content. |
| F-38 Employer analytics | Verified representative opens Analytics → sees aggregated theme trends. | Nightly aggregates over approved reviews and job reports → suppress segments with < 10 contributors. | k-anonymity ≥ 10; no individual content beyond what is public; no suppression tools. |
| F-39 MFA | Member enables TOTP → scans QR → confirms code → saves recovery codes → login asks for a code. | `mfa_factors` with encrypted secret → `POST /auth/login` returns `mfa_required` → `POST /auth/login/mfa`. | Mandatory for staff roles once shipped. |
| F-40 Mobile apps | Same journeys as web, with native navigation. | Same `/api/v1`; session cookie via secure native storage. | Same privacy rules; no extra trackers. |
| F-41 Learning paths | Member starts a path → completes steps (curated lessons and resources) → progress saved privately. | `learning_paths`, `learning_path_steps`, `learning_progress`. | Progress is private. |
| F-42 Push notifications | Member enables push → browser/device subscription stored → notifications delivered respecting preferences and quiet hours. | `push_subscriptions` → the `notifications` processor fans out to the Web Push/APNs/FCM provider. | Same non-manipulative copy rules; no anonymous labels in push text. |
| F-43 Survivor events | Community moderator creates an online event → members RSVP as pseudonym or as an anonymous count → reminder notification → event happens on an external moderated platform. | `events`, `event_rsvps` → reminder jobs. | Attendee lists show pseudonyms only by opt-in. |

---

## W-32 — Error and edge cases

| Scenario | Expected behavior | Where handled |
|---|---|---|
| Session expired mid-compose | API returns 401; the web app keeps the draft in local storage, opens the login dialog, retries the submit after login | `lib/api/client.ts`, composer autosave |
| CSRF token missing/rotated | 403 CSRF_INVALID → client fetches `/auth/csrf` once and retries | API client |
| Double submit (double click, retry) | `Idempotency-Key` returns the original 201 response | Idempotency interceptor |
| Post deleted while viewing | Actions return 404/410 → toast "This post is no longer available" | API + UI |
| Comment on a post that became held/removed | 404 | engagement service |
| Reply to a deleted comment | Allowed if the parent row exists (soft-deleted); shown under "[deleted]" | engagement service |
| Reaction when blocked by the author | 403 BLOCKED with a generic message | engagement service |
| Username changed between page loads | Old `/u/old` → 404; post AuthorRefs already show the new name | profile service |
| Member restricted while writing | 403 POSTING_RESTRICTED with an `until` date → banner | VerifiedGuard |
| Account suspended with active sessions | Sessions revoked; next request 401 → login shows the suspension page with an appeal link | moderation service |
| Vault DB unavailable | Confession/review/anonymous job report creation → 503; pseudonymous features continue | VaultService health |
| Redis unavailable | Authenticated requests 503; visitor reads served from the DB | SessionGuard, throttler |
| Email provider failure | Jobs retry 5× with backoff; the user can resend verification after 60 s | email processor |
| Safety rules updated after publish | Post-publish `safety-scan` may create a moderation item; content stays visible unless high severity (then auto-limited pending review) | safety-scan processor |
| Company merged while writing a review | Submit to the old ID → 301-equivalent: API 409 with `details.mergedInto`; the UI redirects | company service |
| Pending company approved later | Attached content is published at approval time | admin service |
| Duplicate report | 409 ALREADY_EXISTS → UI "You've already reported this" | moderation service |
| Moderator tries to appeal-decide own action | 403 `same_moderator` | moderation service |
| Two moderators claim the same item | Second gets 409 → UI refreshes the assignee | moderation service |
| Reveal requested without permission | 403 + audit `vault.reveal_denied` + alert | VaultPermissionGuard |
| Deletion requested with open escalation | Escalated content retained (hidden) until resolved; everything else deleted | deletion processor |
| Draft reaches 180 days | Deleted with its vault row; a reminder was sent at 173 days | maintenance job |
| Poll vote after closing | 400 `poll_closed` | content service |
| Upload infected | Object deleted, `scan_status = infected`, user sees "This file couldn't be used", security alert | media-scan processor |
| Feed cursor from an old deploy | Invalid cursor → 400 INVALID_CURSOR → client restarts from page 1 | feed service |
| Network offline | TanStack Query pauses mutations; banner "You're offline — we'll retry when you're back" | web |
