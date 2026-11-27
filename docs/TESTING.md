# LinkedOut — Testing Strategy

Related: [SECURITY.md](SECURITY.md) · [USER_STORIES.md](USER_STORIES.md) · [TECH_STACK.md](TECH_STACK.md#5-testing-and-quality) · [DEPLOYMENT.md](DEPLOYMENT.md#4-cicd-pipeline)

## 1. Strategy overview

```mermaid
flowchart TB
  U[Unit tests<br/>Jest / Vitest — fast, many] --> I[Integration tests<br/>Supertest + Testcontainers Postgres/Redis]
  I --> C[Contract tests<br/>public DTO private-field checks]
  C --> S[Security tests<br/>IDOR, identity leakage, CSRF, rate limits]
  S --> E[E2E tests<br/>Playwright + axe — critical journeys]
  E --> L[Load tests<br/>k6 against staging — release gates]
```

| Level | Scope | Tools | Runs |
|---|---|---|---|
| Unit | Services, safety rules, guards, serializers, utilities, React components, hooks | Jest 30 (API), Vitest 3 + React Testing Library (web, shared) | Every commit (pre-push) and every PR |
| Integration | HTTP endpoints through the Nest app with real PostgreSQL 17 and Redis 7 containers, migrations applied, DB roles created | Supertest, Testcontainers | Every PR |
| Contract | Every public response DTO against a deny-list of private fields | Jest + JSON schema walk | Every PR |
| Security | Access control, anonymity, CSRF, rate limits, injection, uploads | Jest/Supertest specs, OWASP ZAP baseline, CodeQL, Trivy | Every PR (specs); nightly (ZAP) |
| E2E | Critical user journeys in real browsers against a full local stack (docker compose + built apps) | Playwright (Chromium, Firefox, WebKit, Pixel 7 + iPhone 14 viewports), @axe-core/playwright | Every PR (Chromium + mobile Chromium); nightly (all browsers) |
| Accessibility | Automated axe checks in E2E; manual screen reader passes | axe, NVDA, VoiceOver | Every PR (axe); before beta and launch (manual) |
| Performance | Lighthouse CI budgets on key pages | Lighthouse CI | Every PR touching `apps/web` |
| Load | Throughput/latency against NFRs | k6 | Before beta, before launch, and before scalability claims |

### 1.1 Test data

- Factories in `apps/api/test/fixtures` (`createUser`, `createVerifiedMember`, `createPost`, `createAnonymousConfession`, `createCompany`, `createModerator`, …) build valid entities through services, not raw SQL, so invariants hold.
- Each integration test file gets a fresh schema (Testcontainers + `prisma migrate deploy`). Tests in a file share a transaction that rolls back after each test where possible.
- E2E uses `pnpm db:seed` dev fixtures plus per-test users created via the API.
- Emails are captured by Mailpit in E2E (its API reads verification links).

## 2. Coverage targets

| Area | Line coverage | Branch coverage | Notes |
|---|---|---|---|
| `apps/api/src/modules/*` services and rules | ≥ 80% | ≥ 75% | CI fails below threshold (NFR-M2) |
| `safety/rules`, `vault`, `moderation`, `identity` | ≥ 90% | ≥ 85% | Critical trust paths |
| `apps/web/src/components` | ≥ 70% | ≥ 60% | |
| `packages/shared` schemas | ≥ 90% | — | Schema edge cases |
| E2E | 100% of MVP user journeys in §5 | — | Each MVP story has ≥ 1 automated acceptance test (unit, integration or E2E) |

## 3. Unit and integration focus areas (plan §19)

| Plan item | Test focus | Level |
|---|---|---|
| Authentication | Password policy, Argon2 verify/rehash, lockout counter, token expiry/single use, session TTL math | Unit + integration |
| Profile permissions | Visibility matrix (public/members/hidden × visitor/member/owner) | Unit + integration |
| Post visibility | Feed/search/detail predicates for visibility, moderation status, drafts, deletion, blocks, mutes, private communities | Integration |
| Reaction rules | One per user per post, change, toggle off, counter consistency under concurrency | Integration (parallel requests) |
| Comment ownership | Edit/delete own only; OP anonymous comment only for vault owner; depth capping | Unit + integration |
| Company response permissions | Only verified, non-expired representative of the same company; no removal powers | Integration |
| Moderation transitions | Item status machine (open → in_review → resolved/escalated), action effects and reversals, appeal by different moderator | Unit (state machine) + integration |
| Registration → profile creation | Register → verify → profile + privacy defaults exist | Integration |
| Post creation → feed | New post appears first in Latest; held post does not | Integration |
| Confession → anonymous rendering | No username/user ID anywhere in any response; label stable per confession, different across confessions | Integration + contract |
| Report → moderation queue | Reports aggregate into one item; priority rules | Integration |
| Company response publication | Response safety-checked, badge shown, held when PII | Integration |
| Account deletion behavior | Grace period, cancellation, purge job results, retention of moderation/audit records | Integration (job invoked directly) |

## 4. Security tests

| ID | Area (plan §19) | Steps | Expected result |
|---|---|---|---|
| SEC-01 | Unauthorized profile access | As visitor and as another member, GET `/profiles/:u` for `members` and `hidden` profiles; GET `/me` without a session | Restricted payload only; `/me` 401 |
| SEC-02 | IDOR — content | Member B PATCH/DELETE member A's post, comment, review, job report, correction, pinned comment, confession | 403 or 404; no change in DB |
| SEC-03 | IDOR — sessions, blocks, mutes, exports, reports | Member B accesses A's `sessionId`, `blockId`, `muteId`, `exportId`; lists `/me/reports` | 404; only own items listed |
| SEC-04 | Private-field leakage (contract) | Walk every public response schema and recorded integration responses for `email`, `userId`, `authorUserId`, `ip`, `passwordHash`, `aliasId`, `sessionId` (except owner session list), `workEmail` | No occurrences |
| SEC-05 | Anonymous identity leakage | Create an anonymous confession, job report and review as A; fetch every endpoint that can return them as visitor, member, company rep, moderator, admin (`/feed`, `/posts/:id`, `/confessions`, `/search`, `/profiles/A/*`, `/companies/:id/reviews`, `/moderation/items/:id`, `/admin/users?q=A`) | No response links A's username/user ID to the content; A's profile shows no trace or count |
| SEC-06 | XSS | Submit `<script>`, `<img onerror>`, `javascript:` links, Markdown link injection, payloads in username/headline/tags/search query; render in E2E | Rendered as inert text; no script execution (Playwright `page.on('dialog')` never fires); CSP violations reported 0 |
| SEC-07 | SQL injection | Fuzz query params (`q`, `cursor`, filters) with SQLi payloads | 400 or normal results; no DB errors; no `$queryRawUnsafe` in codebase (lint) |
| SEC-08 | CSRF | State-changing requests without the header, with a mismatched token, from a foreign origin | 403 CSRF_INVALID; CORS blocks the foreign origin |
| SEC-09 | Security headers & config | Request API and web root; check headers; check Swagger disabled in prod config | Headers per SECURITY §13; `/api/v1/docs` 404 in production |
| SEC-10 | Token handling | Reuse verification/reset tokens; use expired; brute-force token guessing rate | 400 TOKEN_INVALID; tokens stored hashed (DB inspection) |
| SEC-11 | Vault isolation (DB level) | Connect as `lo_app` and `SELECT` from `vault.anonymous_identity_aliases`; attempt `UPDATE`/`DELETE` on `audit_logs` | Permission denied; trigger raises exception |
| SEC-12 | Sensitive data in logs | Capture pino output during register, login, confession, review, report, reveal flows | No emails, passwords, tokens, cookies, bodies; no user ID logged together with an anonymous target ID |
| SEC-13 | Privilege escalation | Member calls `/moderation/*`, `/admin/*`, `/trust-safety/*`; community mod acts on another community; representative responds for another company; moderator decides own appeal; officer self-approves reveal | 403 in all cases; denied vault attempts audit-logged |
| SEC-14 | Rate-limit bypass | Exceed login, register, posts, comments, reports limits; rotate `X-Forwarded-For`; parallel bursts across 2 API instances | 429 enforced; spoofed headers ignored (trust proxy = Cloudflare/Render only) |
| SEC-15 | Audit integrity | Perform role change, moderation action, reveal; verify audit rows; attempt update via API | Rows exist; no update/delete endpoints; DB trigger blocks modification |
| SEC-16 | Fail-closed behavior | Stop Redis container; call authenticated write; stop vault connection; create confession | 503 SERVICE_UNAVAILABLE; no write happens |
| SEC-17 | File upload abuse (Should) | Upload SVG, polyglot JPEG/HTML, oversized file, EICAR test file, image with GPS EXIF | Rejected or quarantined; EICAR → infected; output has no EXIF |
| SEC-18 | Account enumeration | Compare register/forgot/resend/login responses and timings for existing vs non-existing emails (100 samples) | Identical bodies; timing difference < 50 ms median |
| SEC-19 | Session security | Cookie flags; session fixation (pre-set `lo_sid`); logout invalidation; password reset revokes other sessions | Flags correct; new ID on login; old sessions 401 |
| SEC-20 | Alias block oracle | A blocks B's anonymous label; check whether B's pseudonymous content disappears for A | Pseudonymous content still visible (no inference possible) |

## 5. Functional test cases

Legend: **U** unit, **I** integration, **E** end-to-end. Each row is automated unless marked **M** (manual).

| ID | Feature | Story | Steps | Expected result | Level |
|---|---|---|---|---|---|
| TC-001 | F-01 Register | US-005 | Submit valid registration | 202; user `pending_verification`; profile + privacy rows; verification email in Mailpit | I, E |
| TC-002 | F-01 Register | US-005 | Register with a breached password (`password123456`) | 400 `breached_password` | I |
| TC-003 | F-01 Register | US-005 | Register an existing email | 202 identical body; "someone tried" email sent; no new rows | I |
| TC-004 | F-01 Register | US-005 | 6 registrations from one IP in an hour | 6th returns 429 | I |
| TC-005 | F-01 Verify | US-006 | Open a valid link | Account active; session cookie; redirect to onboarding | E |
| TC-006 | F-01 Verify | US-006 | Open an expired link (time-travel 25 h) | TOKEN_INVALID; resend works | I |
| TC-007 | F-01 Unverified | US-006 | Unverified user POST /posts | 403 EMAIL_NOT_VERIFIED | I |
| TC-008 | F-01 Login | US-007 | Login with correct credentials | 200; cookie flags HttpOnly/Secure/SameSite=Lax | I, E |
| TC-009 | F-01 Lockout | US-007 | 10 wrong passwords then the correct one | 423 locked; email notice; success after 15 min | I |
| TC-010 | F-01 Logout | US-007 | Logout then GET /me | 401 | I |
| TC-011 | F-01 Reset | US-008 | Full forgot → reset flow | Password changed; other sessions revoked; confirmation email | E |
| TC-012 | F-01 Sessions | US-009 | Two sessions; revoke the other from S-27 | Other session's next request 401 | E |
| TC-013 | F-03 Username | US-011 | Register with `admin` or `Admin` | 409 reserved | I |
| TC-014 | F-03 Username change | US-012 | Change username twice within 30 days | 2nd → 403 change_too_soon | I |
| TC-015 | F-03 Old username | US-012 | Visit `/u/oldname` after change | 404 page; old name unclaimable for 90 days | I, E |
| TC-016 | F-04 Onboarding | US-013 | Verified user without onboarding tries to publish | 403 ONBOARDING_REQUIRED → UI redirects | I, E |
| TC-017 | F-04 Onboarding | US-013 | Complete 4 steps | `onboarding_completed_at` set; lands on /mess | E |
| TC-018 | F-05 Profile edit | US-014 | Update headline/bio/status `open_to_escape` | Visible on /u/:username | E |
| TC-019 | F-05 Limits | US-015 | Add an 11th lesson | 400 limit | I |
| TC-020 | F-05 Visibility | US-016 | Hidden profile viewed by another member | Restricted view only | I |
| TC-021 | F-05 No counts | US-016 | Inspect profile response & UI | No reaction totals, failure counts or anonymous items | I, E |
| TC-022 | F-06 Privacy | US-017 | Discoverable off → member search | Not returned | I |
| TC-023 | F-07 Block member | US-018 | A blocks B; B comments on A's post | 403 BLOCKED; A no longer sees B's posts | I |
| TC-024 | F-07 Block alias | US-018 | A blocks an anonymous label | Only that label's content hidden; no identity revealed | I |
| TC-025 | F-07 Mute category | US-019 | Mute "Layoffs and Job Loss" | Feed and search exclude it | I |
| TC-026 | F-08 Delete | US-020 | Request deletion; view profile as another user | Profile 404; content hidden; sessions revoked | I |
| TC-027 | F-08 Cancel | US-020 | Cancel within 30 days | Account and content restored | I |
| TC-028 | F-08 Purge | US-020 | Run the deletion job after 30 days | Content, vault rows and profile gone; moderation/audit rows kept with tombstone | I |
| TC-029 | F-10 Create story | US-022 | Publish a clean story | 201; appears first in Latest | I, E |
| TC-030 | F-10 Safety warn | US-022 | Body contains a phone number | Preview highlights; 422 CONFIRMATION_REQUIRED without ack; 201 with ack | I, E |
| TC-031 | F-10 Secret block | US-034 | Body contains `AKIAIOSFODNN7EXAMPLE` | 422 SAFETY_CHECK_BLOCKED | I |
| TC-032 | F-10 Advice | US-023 | Ask for Advice | `postType=advice`; badge shown | E |
| TC-033 | F-10 Content warning | US-024 | Post with CW burnout | Collapsed in feed; reveal shows body; screen reader can't read hidden body | E |
| TC-034 | F-10 Edit | US-025 | Edit own post | `editedAt` set; revision stored | I |
| TC-035 | F-10 Edit other | US-025 | Edit another's post | 403 | I |
| TC-036 | F-10 Delete | US-025 | Delete own post | 404 afterwards; purged after 30 days by job | I |
| TC-037 | F-11 Feed pagination | US-001 | Seed 45 posts; page through with limit 20 | 20/20/5; no duplicates; `hasMore` false at end | I |
| TC-038 | F-11 Filters | US-026 | Filter type=lesson & category | Only matching; URL reflects filters | E |
| TC-039 | F-11 Visitor cache | US-001 | Two visitor requests within 30 s | Second served from Redis | I |
| TC-040 | F-11 Language | US-027 | Filter language=en | Only `en` posts | I |
| TC-041 | F-12 Comment | US-028 | Comment and reply | Thread shows both; comment_count 2 | I, E |
| TC-042 | F-12 Depth | US-028 | Reply at depth 3 | Stored depth 2 with "Replying to" | I |
| TC-043 | F-12 Delete with replies | US-029 | Delete a parent comment | Body "[deleted]", replies remain | I |
| TC-044 | F-12 Pin | US-030 | Author pins own comment; tries to pin another's | First ok; second 400 not_own_comment | I |
| TC-045 | F-13 React | US-031 | React, change, remove | Counts correct after each step | I |
| TC-046 | F-13 Concurrency | US-031 | 50 parallel reactions by distinct users | Sum of counts equals rows | I |
| TC-047 | F-13 Keyboard | US-031 | Use arrow keys + Space on ReactionBar | Reaction applied; accessible names announced | E |
| TC-048 | F-14 Save | US-032 | Save and view Saved | Listed; not visible to others | I, E |
| TC-049 | F-15 Anonymous confession | US-033 | Publish anonymous confession | Label shown; no username anywhere (see SEC-05) | I, E |
| TC-050 | F-15 Unique labels | US-033 | Same user publishes 2 anonymous confessions | Different labels | I |
| TC-051 | F-15 Held | US-034 | Confession naming "John Smith" | Held; notice shown; item in queue | I, E |
| TC-052 | F-15 Draft | US-035 | Save draft | Visible only in My Damage Drafts; not in feed/search | I |
| TC-053 | F-15 Draft expiry | US-035 | Time-travel 173 d and 180 d; run maintenance | Reminder email, then deletion incl. vault row | I |
| TC-054 | F-15 Correction | US-036 | Add correction; try 6th; try deleting | Shown; 6th 400; no delete endpoint for author | I |
| TC-055 | F-15 Flag off | US-033 | `confessions.enabled=false` | 403 FEATURE_DISABLED; UI coming-soon panel | I, E |
| TC-056 | F-15 OP comment | US-033 | Author replies anonymously in own confession | Comment shows same label + OP; other users cannot use anonymous mode there | I |
| TC-057 | F-16 Lesson | US-037 | Publish lesson without "what I learned" | 400; with it, 201 and listed in /lessons | I, E |
| TC-058 | F-16 Lesson links | US-037 | Add `http://` link | 400 https only | I |
| TC-059 | F-17 Company page | US-003 | Company with 4 reviews | "Not enough reports yet" | I, E |
| TC-060 | F-17 Themes | US-003 | Company with 7 reviews | Medians and distributions correct | I |
| TC-061 | F-17 Review gate | US-039 | Account aged 3 days writes a review | 403 ACCOUNT_TOO_NEW | I |
| TC-062 | F-17 One review | US-039 | Second review same company | 409 with existingReviewId | I |
| TC-063 | F-17 Consent | US-039 | `displayConsent=false` | Narrative null publicly; ratings counted | I |
| TC-064 | F-17 Filters | US-040 | Filter topic + relationship | Only matching reviews | I |
| TC-065 | F-17 Submit company | US-041 | Submit a duplicate by domain | 409 with existing | I |
| TC-066 | F-17 Pending | US-041 | Review on a pending company, then approve | Hidden until approval, then published | I |
| TC-067 | F-18 Job report | US-042 | Submit report | Appears on board + company page with label | I, E |
| TC-068 | F-18 Interview dump | US-042 | Narrative with 6 question lines | Medium finding; held after ack | I |
| TC-069 | F-18 Filters | US-043 | Filter `candidate_ghosting` | Only that type | I |
| TC-070 | F-19 Search | US-004 | Search "unpaid test assignment" | Grouped results with highlights | I, E |
| TC-071 | F-19 Exclusions | US-044 | Matching held, members-only (as visitor), muted, blocked items | Excluded | I |
| TC-072 | F-19 Short query | US-004 | `q=a` | 400; UI hint | I, E |
| TC-073 | F-20 Report | US-045 | Report a comment for personal info | Item priority high; report_count 1 | I |
| TC-074 | F-20 Aggregation | US-060 | 3 users report the same comment | One item, report_count 3 | I |
| TC-075 | F-20 Duplicate | US-045 | Same user reports twice | 409 | I |
| TC-076 | F-20 Status | US-046 | Moderator resolves; reporter checks /reports | Status actioned; outcome email | I, E |
| TC-077 | F-21 Queue order | US-060 | Items of high/normal/low and ages | Sorted by priority_score then age | I |
| TC-078 | F-21 Claim race | US-060 | Two moderators claim simultaneously | One 200, one 409 | I |
| TC-079 | F-21 Remove | US-061 | Remove content with reason | Status removed; member emailed with policy + appeal link; audit row | I, E |
| TC-080 | F-21 Restrict | US-061 | Restrict posting 7 days | Member POST /posts → 403 POSTING_RESTRICTED until date | I |
| TC-081 | F-21 Restrict anonymous author | US-061 | Restrict the author of an anonymous confession | Owning account restricted; moderator response contains no identity | I |
| TC-082 | F-21 Approve held | US-062 | no_action on held confession | Published; author notified | I |
| TC-083 | F-21 Campaign | US-064 | 6 reviews for one company in 24 h | Campaign item; summaries paused | I |
| TC-084 | F-21 Escalate | US-065 | Escalate with note | Item escalated; visible in admin escalations | I |
| TC-085 | F-22 Appeal | US-047 | Member appeals within window | Appeal pending; listed in /mod/appeals | I, E |
| TC-086 | F-22 Same moderator | US-063 | Original moderator decides | 403 same_moderator | I |
| TC-087 | F-22 Reverse | US-063 | Different moderator reverses removal | Content restored; email sent | I |
| TC-088 | F-22 Window | US-047 | Appeal after 31 days | 400 appeal_window_closed | I |
| TC-089 | F-23 Reveal | US-066 | Request + approve by second officer + view | Account summary returned; 3 audit rows; expires after 24 h | I |
| TC-090 | F-23 Self-approve | US-066 | Requester approves own request | 403 self_approval | I |
| TC-091 | F-23 Audit view | US-075 | Admin filters audit by `vault.reveal_approved` | Entries shown; read-only | I, E |
| TC-092 | F-24 Roles | US-070 | Admin promotes member to moderator | Effective immediately; audit row | I |
| TC-093 | F-24 Last admin | US-070 | Only admin demotes self | 400 last_admin | I |
| TC-094 | F-24 Categories | US-071 | Archive a category | Not selectable; existing posts keep it | I |
| TC-095 | F-24 Merge | US-072 | Merge duplicate companies | Content moved; old slug redirects | I |
| TC-096 | F-24 Metrics | US-074 | Seeded data → GET /admin/metrics | Values match fixture expectations | I |
| TC-097 | F-24 Launch gate | — | Enable confessions without `launchGateConfirmed` | 400 | I |
| TC-098 | F-02 Google (Should) | US-010 | OAuth new user (mock IdP) | Username step required; no Google name imported | I, E |
| TC-099 | F-09 Export (Should) | US-021 | Request export | ZIP with JSON files incl. own anonymous content; link expires in 7 days | I |
| TC-100 | F-25 Rep request (Should) | US-067 | Domain mismatch | 400 domain_mismatch | I |
| TC-101 | F-25 Response (Should) | US-068 | Verified rep responds | Badge shown; second response → 409 | I, E |
| TC-102 | F-25 No identity (Should) | US-069 | Rep fetches reviews | No author data (SEC-05) | I |
| TC-103 | F-26 Create community (Should) | US-048 | Account < 14 days creates | 403 ACCOUNT_TOO_NEW | I |
| TC-104 | F-26 Private (Should) | US-051 | Non-member reads a private community post | 404; excluded from search | I |
| TC-105 | F-26 Scoped mod (Should) | US-058 | Community mod acts on another community | 403 | I |
| TC-106 | F-27 Notification (Should) | US-052 | Comment on a post | Recipient notification within 1 min; unread count +1 | I, E |
| TC-107 | F-27 Batching (Should) | US-052 | 12 reactions in an hour | One notification with actorCount 12 | I |
| TC-108 | F-27 Quiet hours (Should) | US-053 | Trigger email inside quiet hours | Email job delayed to quiet end | I |
| TC-109 | F-28 Poll (Should) | US-054, US-055 | Create poll; vote; vote again; view after close | Results after vote; 409 second vote; final results | I, E |
| TC-110 | F-29 Upload (Should) | US-056 | Upload PNG with EXIF | Clean, re-encoded, no EXIF; alt text required | I, E |
| TC-111 | F-30 Following (Should) | US-057 | Follow a tag; open Following | Only followed-topic posts | I |
| TC-112 | Accessibility | — | axe on S-01, S-02, S-03, S-06, S-07, S-08, S-09, S-11, S-12, S-16, S-21, S-22, S-23 | 0 serious/critical violations | E |
| TC-113 | Accessibility | — | Keyboard-only completion of Journey A | Possible without a mouse; visible focus throughout | E |
| TC-114 | Accessibility | — | NVDA + Firefox and VoiceOver + Safari on Journeys A–C | All content and controls announced correctly | M |
| TC-115 | Responsive | — | 320 px, 768 px, 1280 px viewports on key screens | No horizontal scroll; bottom tab bar on mobile | E |
| TC-116 | UX | — | Privacy explanations understandable (5 beta users, think-aloud) | ≥ 4/5 explain pseudonym vs anonymous correctly | M |

## 6. Load and performance tests

Load tests run with k6 against **staging** sized like production, using seeded data (100k posts, 500k comments, 2k companies). The thresholds below are the pass criteria.

| Script | Scenario | Load profile | Thresholds |
|---|---|---|---|
| `feed.k6.js` | Visitor + member feed pagination (5 pages deep, filters) | Ramp to 40 rps for 10 min | p95 < 300 ms (members), p95 < 150 ms (visitor first page), errors < 0.5% |
| `post-create.k6.js` | Verified members creating posts with safety checks | 5 rps for 10 min | p95 < 500 ms, errors < 0.5% |
| `comment-burst.k6.js` | 200 users commenting on one hot post within 60 s | Spike | p95 < 500 ms; comment_count consistent; no deadlocks |
| `search.k6.js` | Mixed queries over 100k posts | 10 rps for 10 min | p95 < 500 ms |
| `notifications.k6.js` (Should) | 10k notification jobs enqueued | Burst | All processed < 5 min; queue age alert not triggered |
| `media-upload.k6.js` (Should) | Presign + upload 3 MB images | 2 rps for 5 min | Presign p95 < 300 ms; scan completes < 30 s p95 |
| Soak | Mixed realistic traffic (70% reads) | 50 rps for 2 h | No memory growth > 10%; error rate < 0.5% |

Results go into `docs/runbooks/load-test-results.md` with the date, commit and staging size. The platform is not described as scalable beyond the last passing profile (plan §19).

**Lighthouse CI budgets** (mobile, simulated 4G): landing, `/mess`, post detail and company page each need performance ≥ 90, accessibility = 100, LCP ≤ 2.5 s, CLS ≤ 0.1, JS for `/mess` ≤ 200 KB gzipped.

## 7. Beta feedback validation (plan §18 Phase 6)

| Question | Method | Success signal |
|---|---|---|
| Is the concept understandable? | Onboarding completion + 5-question survey | ≥ 80% can describe LinkedOut's purpose |
| Does posting feel safe? | Trust survey item (1–5) | ≥ 4.0 |
| Are stories useful? | "A story helped me" monthly prompt | ≥ 50% yes |
| Is moderation fair? | Appeal reversal rate; survey of actioned members | Reversal < 30%; ≥ 3.5/5 fairness |
| Is the feed too negative? | Share of advice/lesson/wins posts; mood survey | ≥ 30% constructive content; ≤ 20% report feeling worse |
| Do users return? | Week-2 and week-4 retention | ≥ 35% / ≥ 25% |

## 8. Definition of Done for tests

A task in [TASKS.md](TASKS.md) is done only when:
1. New logic has unit tests, and new endpoints have integration tests covering success, validation, authorization and rate-limit paths.
2. Any new public response field is added to the contract test allow-list, which is reviewed for private data.
3. Affected E2E journeys pass locally and in CI.
4. Coverage thresholds still pass.
5. No new axe violations are introduced.
