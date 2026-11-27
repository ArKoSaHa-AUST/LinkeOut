# LinkedOut — REST API Specification (v1)

| Item | Value |
|---|---|
| Base URL (production) | `https://api.linkedout.app/api/v1` |
| Base URL (staging) | `https://api.staging.linkedout.app/api/v1` |
| Base URL (local) | `http://localhost:4000/api/v1` |
| Format | JSON (`application/json; charset=utf-8`), UTF-8, camelCase fields, ISO-8601 UTC timestamps |
| OpenAPI | Generated from NestJS decorators at `/api/v1/docs` (non-production) and `/api/v1/openapi.json` |
| Shared types | Zod schemas in `packages/shared/src/schemas` are the source for request validation and frontend types |

Related: [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md) · [SECURITY.md](SECURITY.md) · [WORKFLOW.md](WORKFLOW.md)

---

## 1. Conventions

### 1.1 Authentication levels

| Level | Meaning |
|---|---|
| **Public** | No session needed. |
| **Optional** | Works without a session; returns viewer-specific fields (`viewer`) and members-only content when a session exists. |
| **Member** | Valid session (`lo_sid` cookie). Account may be unverified. |
| **Verified** | Member with `email_verified_at` set, `onboarding_completed_at` set, status `active`, and not posting-restricted (for create/update actions). |
| **Moderator** | `platform_role` ∈ {`platform_moderator`, `admin`}. |
| **Community moderator** | `community_members.role` ∈ {`moderator`, `owner`} for the community in the path. |
| **Representative** | Active, verified `company_memberships` row for the company in the path. |
| **Vault** | Moderator holding active `identity_vault.reveal` staff permission. |
| **Admin** | `platform_role = admin`. |

The server derives identity only from the session. User IDs, roles or company IDs in request bodies are never trusted for authorization.

### 1.2 Headers

| Header | Direction | Required | Description |
|---|---|---|---|
| `Cookie: lo_sid=…` | Request | For authenticated endpoints | Session cookie (httpOnly, Secure, SameSite=Lax, Domain `.linkedout.app`). |
| `Cookie: lo_csrf=…` + `X-CSRF-Token: …` | Request | All `POST`, `PUT`, `PATCH`, `DELETE` | Double-submit CSRF token; value from `GET /auth/csrf`. Mismatch → `403 CSRF_INVALID`. |
| `Content-Type: application/json` | Request | Bodies | |
| `Idempotency-Key: <uuid>` | Request | Optional on create endpoints (`POST /posts`, `/confessions`, `/companies/:id/reviews`, `/job-reports`, `/reports`) | Duplicate key within 24 h returns the original response. |
| `X-Request-Id` | Both | Optional in request; always in response | Correlation ID (`req_…`). |
| `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset` | Response | Always on rate-limited routes | IETF RateLimit header fields. |
| `Retry-After` | Response | On 429 | Seconds. |
| `Cache-Control` | Response | | `no-store` for authenticated responses; `public, max-age=30` for visitor feed first page. |

### 1.3 Response envelopes

Single resource:
```json
{ "data": { } }
```

Collection (cursor pagination):
```json
{
  "data": [ ],
  "page": { "nextCursor": "MjAyNy0wMS0xMlQwODoxNTowMFp8MDE5M2IwYWE", "hasMore": true, "limit": 20 }
}
```

### 1.4 Pagination

- Query params: `limit` (default 20, min 1, max 50) and `cursor` (opaque base64url string from `page.nextCursor`).
- Ordering is stable (`published_at DESC, id DESC` unless stated). Invalid cursor → `400 INVALID_CURSOR`.
- Admin lists use the same cursor scheme.

### 1.5 Standard error format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Some fields need attention.",
    "details": [
      { "field": "title", "issue": "too_long", "message": "Title must be 150 characters or fewer." }
    ],
    "requestId": "req_01JH8Z3K2W9Q"
  }
}
```

| HTTP | Code | When |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Body/query fails schema validation. |
| 400 | `INVALID_CURSOR` | Malformed pagination cursor. |
| 400 | `TOKEN_INVALID` | Verification/reset token unknown, used or expired. |
| 401 | `UNAUTHENTICATED` | Missing/expired session. |
| 401 | `INVALID_CREDENTIALS` | Wrong email or password. |
| 403 | `FORBIDDEN` | Authenticated but not permitted. |
| 403 | `CSRF_INVALID` | CSRF token missing or mismatched. |
| 403 | `EMAIL_NOT_VERIFIED` | Verified-level endpoint, email unverified. |
| 403 | `ONBOARDING_REQUIRED` | Onboarding not completed. |
| 403 | `POSTING_RESTRICTED` | Member under `restrict_posting`; `details[0].until` gives end time. |
| 403 | `ACCOUNT_SUSPENDED` | Account suspended. |
| 403 | `ACCOUNT_TOO_NEW` | Account-age gate (reviews 7 days, communities 14 days). |
| 403 | `BLOCKED` | Interaction with a member who blocked you (message is generic). |
| 403 | `FEATURE_DISABLED` | Feature flag off (e.g. confessions before launch gate). |
| 404 | `NOT_FOUND` | Resource missing or not visible to the caller (also used instead of 403 where existence must not leak). |
| 409 | `CONFLICT` | Uniqueness conflict (e.g. username taken, already reviewed). |
| 409 | `ALREADY_EXISTS` | Duplicate report/appeal/save where idempotent behavior is not used. |
| 410 | `GONE` | Content deleted. |
| 413 | `PAYLOAD_TOO_LARGE` | Body > 256 KB or upload > 5 MB. |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | Wrong content type. |
| 422 | `SAFETY_CHECK_BLOCKED` | Content contains blocking findings (secrets/credentials); `details` lists ranges. |
| 422 | `SAFETY_CONFIRMATION_REQUIRED` | Medium-risk findings not acknowledged (`acknowledgedFindings` missing). |
| 423 | `ACCOUNT_LOCKED` | Login locked after failed attempts; generic message. |
| 429 | `RATE_LIMITED` | Rate limit exceeded. |
| 500 | `INTERNAL_ERROR` | Unexpected error (no stack traces in body). |
| 503 | `SERVICE_UNAVAILABLE` | Dependency down (e.g. Redis) or maintenance. |

### 1.6 Versioning

- URI versioning: `/api/v1`. Additive changes (new fields, endpoints, enum values documented as extensible) ship in v1.
- Breaking changes require `/api/v2`. v1 receives `Deprecation: true` and `Sunset: <date>` headers at least 90 days before removal.
- Clients must ignore unknown fields.

### 1.7 Rate limits

Implemented with `@nestjs/throttler` + Redis storage (sliding window). Keys: IP for unauthenticated calls, user ID for authenticated ones.

| Scope | Limit |
|---|---|
| Global unauthenticated | 120 requests / minute / IP |
| Global authenticated | 300 requests / minute / user |
| `POST /auth/register` | 5 / hour / IP |
| `POST /auth/login` | 10 / 15 min / (IP + email); lockout after 10 failures per email |
| `POST /auth/forgot-password`, `POST /auth/resend-verification` | 3 / hour / email and 10 / hour / IP |
| `GET /auth/username-availability` | 30 / minute / IP |
| `POST /posts` | 10 / hour, 30 / day / user |
| `POST /confessions` | 5 / 24 h / user (1 / 24 h if account < 24 h old) |
| `POST /posts/:id/comments` | 30 / 10 min / user |
| `POST /posts/:id/reactions` | 120 / minute / user |
| `POST /companies/:id/reviews` | 3 / day / user |
| `POST /companies` | 5 / day / user |
| `POST /job-reports` | 5 / day / user |
| `POST /reports` | 20 / hour / user |
| `POST /safety/check` | 60 / hour / user |
| `GET /search` | 60 / minute / user or IP |
| `POST /media/uploads` | 30 / hour / user |
| Links in content | Accounts < 7 days old: max 3 links per post/comment (validation error, not 429) |

### 1.8 Shared schemas

**AuthorRef** — never contains internal IDs or private fields.
```json
{ "kind": "member", "username": "rejected_but_rising", "displayName": "Still Compiling", "avatarUrl": null, "headline": "CS grad · 40 applications" }
```
```json
{ "kind": "anonymous", "label": "Anonymous Stapler #4821", "isOp": false }
```
```json
{ "kind": "deleted" }
```

**PostSummary**
```json
{
  "id": "0193b0aa-1c2d-7e3f-9a10-11aa22bb33cc",
  "postType": "story",
  "title": "Five rounds, one take-home, and then silence",
  "excerpt": "I spent a weekend on the take-home…",
  "isLongForm": false,
  "author": { "kind": "member", "username": "rejected_but_rising", "displayName": "Still Compiling", "avatarUrl": null, "headline": null },
  "category": { "id": 2, "slug": "interview-disasters", "name": "Interview Disasters" },
  "tags": [ { "slug": "take-home", "name": "take-home" } ],
  "desiredResponse": "advice",
  "language": "en",
  "visibility": "public",
  "contentWarnings": [],
  "reactionCounts": { "im_dead": 3, "pain": 12, "same_here": 27, "melting": 2, "corporate_circus": 5, "survived": 4 },
  "commentCount": 9,
  "community": null,
  "publishedAt": "2027-01-12T08:15:00Z",
  "editedAt": null,
  "viewer": { "reaction": "same_here", "saved": true, "isAuthor": false }
}
```

**PostDetail** = PostSummary plus:
```json
{
  "body": "Full markdown body…",
  "corrections": [ { "id": "…", "body": "Update: they replied after 3 weeks.", "createdAt": "2027-01-20T09:00:00Z" } ],
  "pinnedComment": null,
  "confession": null,
  "lesson": null,
  "poll": null,
  "media": [],
  "moderationNotice": null,
  "viewer": { "reaction": null, "saved": false, "isAuthor": true, "canEdit": true, "canDelete": true }
}
```
- `confession`: `{ "experienceType": "workload_overtime", "industry": {"slug":"technology","name":"Technology"}, "broadRegion": "IN", "periodYear": 2026, "periodQuarter": 3 }`
- `lesson`: `{ "whatIDidWrong": "…", "whatHappened": "…", "whatILearned": "…", "whatIdDoDifferently": "…", "advice": "…", "links": [ { "title": "…", "url": "https://…" } ] }`
- `poll` (Should): `{ "closesAt": "…", "isClosed": false, "totalVotes": 42, "options": [ { "id": "…", "label": "Yes", "voteCount": 30, "percentage": 71.4 } ], "viewerVoteOptionId": "…" }` — `voteCount`/`percentage` are `null` until the viewer votes or the poll closes.
- `moderationNotice` (author only): `{ "status": "held", "message": "Waiting for a quick review (usually within 24 hours)." }`

**Comment**
```json
{
  "id": "0193b2aa-0000-7000-8000-000000000042",
  "postId": "0193b0aa-1c2d-7e3f-9a10-11aa22bb33cc",
  "parentCommentId": null,
  "depth": 0,
  "author": { "kind": "member", "username": "survived_standups", "displayName": null, "avatarUrl": null, "headline": null },
  "body": "Same thing happened to me — follow up once after a week, then let it go.",
  "contentWarnings": [],
  "isDeleted": false,
  "isPinnedClarification": false,
  "createdAt": "2027-01-12T09:00:00Z",
  "editedAt": null,
  "replyCount": 2,
  "viewer": { "isAuthor": false, "canEdit": false }
}
```

**SafetyFinding**
```json
{ "rule": "pii.email_address", "severity": "medium", "field": "body", "start": 120, "end": 141, "message": "This looks like an email address. Remove it unless it's yours and you want it public.", "blocking": false }
```

---

## 2. Health

| Method & path | Auth | Description | Responses |
|---|---|---|---|
| `GET /health/live` | Public | Process is running. | `200 {"status":"ok"}` |
| `GET /health/ready` | Public (internal network) | DB, Redis and queue reachable. | `200 {"status":"ok","checks":{"db":"ok","redis":"ok"}}` · `503` with failing check names |

---

## 3. Authentication (F-01, F-02, F-03, F-04)

### `GET /auth/csrf`
Auth: Public. Issues the `lo_csrf` cookie and returns the token.
Response `200`:
```json
{ "data": { "csrfToken": "b1c2…f9" } }
```

### `GET /auth/username-availability?username=rejected_but_rising`
Auth: Public · Rate limit 30/min/IP.
Response `200`:
```json
{ "data": { "username": "rejected_but_rising", "available": false, "reason": "taken" } }
```
`reason` ∈ `taken`, `reserved`, `invalid_format`, `null`. Errors: `400 VALIDATION_ERROR`, `429`.

### `POST /auth/register`
Auth: Public · CSRF · Rate limit 5/h/IP. Creates account (`pending_verification`), profile, privacy settings; sends verification email. If `beta.invite_only` flag is on, `inviteCode` is required.
Request:
```json
{
  "email": "priya@example.com",
  "password": "correct-horse-battery-staple-42",
  "username": "rejected_but_rising",
  "ageConfirmed": true,
  "acceptTerms": true,
  "termsVersion": "2026-12-01",
  "inviteCode": "BETA-7K2Q"
}
```
Response `202` (same body for new and already-registered emails):
```json
{ "data": { "status": "verification_sent", "message": "Check your inbox to verify your email." } }
```
Errors: `400 VALIDATION_ERROR` (password < 12 chars, found in breached list — `issue: "breached_password"`, username format, age/terms not true, invite invalid), `409 CONFLICT` (username taken), `429`.

### `POST /auth/resend-verification`
Auth: Public · CSRF · Rate limit 3/h/email. Request `{ "email": "priya@example.com" }`. Response `202 { "data": { "status": "sent_if_applicable" } }`.

### `POST /auth/verify-email`
Auth: Public · CSRF. Consumes a verification token, activates the account and creates a session.
Request `{ "token": "9f8e7d…" }`
Response `200` + `Set-Cookie: lo_sid=…`:
```json
{ "data": { "user": { "username": "rejected_but_rising", "emailVerified": true, "onboardingCompleted": false } } }
```
Errors: `400 TOKEN_INVALID`.

### `POST /auth/login`
Auth: Public · CSRF · Rate limit 10/15 min.
Request:
```json
{ "email": "priya@example.com", "password": "correct-horse-battery-staple-42", "rememberDevice": true }
```
Response `200` + `Set-Cookie: lo_sid=…` (new session ID; any pre-login session is discarded):
```json
{ "data": { "user": { "username": "rejected_but_rising", "emailVerified": true, "onboardingCompleted": true, "platformRole": "member" } } }
```
Errors: `401 INVALID_CREDENTIALS`, `423 ACCOUNT_LOCKED`, `403 ACCOUNT_SUSPENDED` (with appeal link), `429`.

### `POST /auth/logout`
Auth: Member · CSRF. Deletes the session in Redis and PostgreSQL; clears cookie. Response `204`.

### `POST /auth/forgot-password`
Auth: Public · CSRF · Rate limit 3/h/email. Request `{ "email": "priya@example.com" }`. Response `202 { "data": { "status": "sent_if_applicable" } }` (always).

### `POST /auth/reset-password`
Auth: Public · CSRF.
Request `{ "token": "4a5b…", "newPassword": "a-much-better-passphrase-2027" }`
Response `200` + new session cookie: `{ "data": { "status": "password_reset" } }`. All other sessions revoked; confirmation email sent.
Errors: `400 TOKEN_INVALID`, `400 VALIDATION_ERROR` (`breached_password`, `too_short`).

### `GET /auth/oauth/google` (Should, F-02)
Auth: Public. Redirects (`302`) to Google with `state`, `nonce` and PKCE `code_challenge` stored in a short-lived signed cookie `lo_oauth` (10 min).

### `GET /auth/oauth/google/callback?code=…&state=…` (Should)
Auth: Public. Validates state/nonce/PKCE and ID token. Existing linked account → session + `302` to `/mess`. Matching email with password account → `302` to `/login?link=google`. New user → `302` to `/signup/complete` with a signed `lo_oauth_pending` cookie.
Errors (rendered as redirect to `/login?error=oauth_failed`): invalid state → 400.

### `POST /auth/oauth/complete-signup` (Should)
Auth: Public (requires `lo_oauth_pending` cookie) · CSRF.
Request `{ "username": "career_changer_ux", "ageConfirmed": true, "acceptTerms": true, "termsVersion": "2026-12-01" }`
Response `201` + session cookie: `{ "data": { "user": { "username": "career_changer_ux", "emailVerified": true, "onboardingCompleted": false } } }`. Errors: `400`, `409 CONFLICT`.

---

## 4. Current member — account, profile, privacy (F-01, F-03–F-09)

### `GET /me`
Auth: Member. The only endpoint that returns the member's own email.
```json
{
  "data": {
    "email": "priya@example.com",
    "emailVerified": true,
    "accountStatus": "active",
    "platformRole": "member",
    "permissions": [],
    "onboardingCompleted": true,
    "postingRestrictedUntil": null,
    "profile": { "username": "rejected_but_rising", "displayName": "Still Compiling", "headline": "…", "bio": "…", "industry": { "slug": "technology", "name": "Technology" }, "broadLocation": "IN", "careerStatus": "open_to_escape", "avatarUrl": null, "usernameChangeAvailableAt": "2027-02-14T00:00:00Z" },
    "deletionScheduledFor": null,
    "featureFlags": { "confessions.enabled": true, "communities.enabled": false }
  }
}
```

### `POST /me/onboarding/complete`
Auth: Member (verified email) · CSRF.
Request `{ "acknowledgedSteps": ["pseudonymity", "data_collection", "lawful_disclosure", "community_guidelines"], "interestCategoryIds": [2, 8, 11] }`
Response `200 { "data": { "onboardingCompleted": true } }`. Errors: `400` if any step missing.

### `PATCH /me/profile`
Auth: Member · CSRF. All fields optional; `null` clears.
Request:
```json
{ "displayName": "Still Compiling", "headline": "CS grad · 40 applications", "bio": "…", "industryId": 1, "broadLocation": "IN", "careerStatus": "open_to_escape", "avatarMediaId": null }
```
Response `200 { "data": { "profile": { … } } }`. Errors: `400` (lengths, unknown industry, non-ISO country, avatar not owned/clean).
If the bio contains PII findings, request must include `"acknowledgedFindings": ["pii.email_address"]`, otherwise `422 SAFETY_CONFIRMATION_REQUIRED` with findings in `details`.

### `PATCH /me/username`
Auth: Member · CSRF. Request `{ "username": "quietly_recompiling" }`. Response `200 { "data": { "username": "quietly_recompiling", "nextChangeAvailableAt": "2027-03-15T10:00:00Z" } }`. Errors: `400`, `409 CONFLICT` (taken/reserved), `403 FORBIDDEN` with `details[0].issue = "change_too_soon"`.

### `PUT /me/profile/lessons` · `PUT /me/profile/skills` · `PUT /me/profile/timeline`
Auth: Member · CSRF. Replace the full ordered list.
```json
{ "lessons": [ "Ask about on-call before accepting.", "A take-home over 4 hours is a red flag." ] }
```
```json
{ "skills": [ "Saying no", "TypeScript", "Salary negotiation" ] }
```
```json
{ "timeline": [ { "startYear": 2024, "startMonth": 6, "endYear": 2026, "endMonth": 1, "roleCategory": "software_engineering", "industryId": 1, "note": "First job. Learned to estimate ×3." } ] }
```
Response `200` with the saved list. Errors: `400` (limits: 10 lessons ≤ 280 chars, 20 skills ≤ 50 chars, 20 timeline entries).

### `GET /me/privacy` · `PATCH /me/privacy`
Auth: Member (PATCH: CSRF).
```json
{ "profileVisibility": "members", "discoverable": false, "defaultConfessionIdentity": "anonymous", "defaultPostVisibility": "public", "messagePermissions": "requests_only" }
```
Response `200 { "data": { …same shape… } }`. Errors: `400`.

### `GET /me/sessions`
Auth: Member.
```json
{ "data": [ { "sessionId": "0193…", "deviceLabel": "Firefox on Linux", "approxLocation": "Kolkata, IN", "createdAt": "…", "lastSeenAt": "…", "isCurrent": true } ] }
```

### `DELETE /me/sessions/:sessionId`
Auth: Member · CSRF. Response `204`. Errors: `404` (not own session).

### `DELETE /me/sessions`
Auth: Member · CSRF. Revokes all sessions except the current one. Response `200 { "data": { "revokedCount": 3 } }`.

### `GET /me/posts?status=published|draft&type=confession&cursor=`
Auth: Member. Lists the member's own posts **including anonymous confessions** (resolved via the vault). Each item is a PostSummary plus `"ownershipNote": "Only you can see that this is yours"` for anonymous ones.

### `GET /me/saved?cursor=`
Auth: Member. Returns PostSummary list ordered by save time; unavailable posts appear as `{ "id": "…", "unavailable": true }`.

### `GET /me/blocks` · `POST /me/blocks` · `DELETE /me/blocks/:blockId`
Auth: Member (writes: CSRF).
POST request — exactly one of:
```json
{ "username": "loud_manager_fan" }
```
```json
{ "postId": "0193b1f0-5a6b-7c8d-8e9f-a0b1c2d3e4f5" }
```
(`postId` / `jobReportId` blocks the anonymous label on that content.)
Response `201 { "data": { "blockId": "…", "target": { "kind": "member", "username": "loud_manager_fan" } } }` or `{ "kind": "anonymous", "label": "Anonymous Stapler #4821" }`.
GET returns the same `target` shapes. Errors: `400` (self-block), `404`, `409 ALREADY_EXISTS`.

### `GET /me/mutes` · `POST /me/mutes` · `DELETE /me/mutes/:muteId`
Request — exactly one of `{ "username": "…" }`, `{ "categoryId": 3 }`, `{ "tagSlug": "layoffs" }`.
Response `201 { "data": { "muteId": "…", "target": { "kind": "category", "id": 3, "name": "Office Drama" } } }`. Errors: `400`, `404`, `409`.

### `DELETE /me/account` (F-08)
Auth: Member · CSRF.
Request `{ "password": "correct-horse-battery-staple-42", "confirm": "DELETE" }` (OAuth-only accounts: `{ "confirm": "DELETE", "reauthToken": "…" }` from a fresh Google login).
Response `202`:
```json
{ "data": { "status": "deletion_pending", "scheduledFor": "2027-02-19T10:00:00Z", "retained": [ { "item": "Moderation records about your content", "period": "2 years" }, { "item": "Security audit logs", "period": "2 years" }, { "item": "Data subject to a legal hold", "period": "Until the hold is lifted" } ] } }
```
All sessions revoked. Errors: `401 INVALID_CREDENTIALS`, `409 CONFLICT` (already pending).

### `POST /me/account/cancel-deletion`
Auth: Public with credentials (account in `deletion_pending` can log in only to cancel) · CSRF. Request `{ "email": "…", "password": "…" }`. Response `200 { "data": { "status": "active" } }` + session. Errors: `401`, `400` (not pending / past date).

### `POST /me/exports` · `GET /me/exports/:exportId` (Should, F-09)
Auth: Member · CSRF (POST). POST requires `{ "password": "…" }`.
POST response `202 { "data": { "exportId": "…", "status": "pending" } }`. GET response `200 { "data": { "exportId": "…", "status": "completed", "downloadUrl": "https://files.linkedout.app/exports/…?X-Amz-Expires=604800…", "expiresAt": "…" } }`. Errors: `409 CONFLICT` (in progress), `404`.

### `GET /me/notification-preferences` · `PATCH /me/notification-preferences` (Should, F-27)
```json
{
  "inApp": { "comment_on_post": true, "reply_to_comment": true, "reactions_on_post": true, "community_invitation": true, "community_announcement": true, "report_update": true, "moderation_decision": true, "followed_topic_digest": true, "company_response": true },
  "email": { "comment_on_post": true, "reply_to_comment": true, "reactions_on_post": false, "community_invitation": false, "community_announcement": false, "report_update": true, "moderation_decision": true, "followed_topic_digest": false, "company_response": true },
  "quietHours": { "start": "22:00", "end": "08:00" },
  "timezone": "Asia/Kolkata"
}
```
Errors: `400` (setting `email.moderation_decision` to false → `issue: "required_notification"`).

### `GET /me/follows` · `POST /me/follows` · `DELETE /me/follows/:followId` (Should, F-30)
POST `{ "categoryId": 8 }` or `{ "tagSlug": "first-job" }` → `201 { "data": { "followId": "…", "target": { "kind": "tag", "slug": "first-job" } } }`.

### `GET /me/company-memberships` (Should, F-25)
`200 { "data": [ { "membershipId": "…", "company": { "id": "…", "slug": "acme-widgets", "name": "Acme Widgets" }, "status": "verified", "expiresAt": "2028-01-01T00:00:00Z" } ] }`

---

## 5. Profiles (F-05)

### `GET /profiles/:username`
Auth: Optional. Applies `profile_visibility`.
Response `200`:
```json
{
  "data": {
    "username": "rejected_but_rising",
    "displayName": "Still Compiling",
    "avatarUrl": null,
    "headline": "CS grad · 40 applications · 1 lesson per rejection",
    "bio": "…",
    "industry": { "slug": "technology", "name": "Technology" },
    "broadLocation": "IN",
    "careerStatus": "open_to_escape",
    "lessons": [ "Ask about on-call before accepting." ],
    "skills": [ "Saying no", "TypeScript" ],
    "timeline": [ { "startYear": 2024, "startMonth": 6, "endYear": 2026, "endMonth": 1, "roleCategory": "software_engineering", "industry": { "slug": "technology", "name": "Technology" }, "note": "…" } ],
    "communities": [ { "slug": "fresh-graduate-survivors", "name": "Fresh Graduate Survivors" } ],
    "memberSince": "2026-12",
    "visibility": "public",
    "viewer": { "isSelf": false, "blocked": false, "muted": false }
  }
}
```
Restricted views: `members` visibility for visitors and `hidden` for non-owners return `{ "username": "…", "visibility": "hidden", "restricted": true }`. Unknown or reserved-after-change usernames → `404`.

### `GET /profiles/:username/posts?cursor=` · `GET /profiles/:username/comments?cursor=`
Auth: Optional. Pseudonymous published content only (never anonymous). Same visibility rules; restricted → `200` with empty `data` and `restricted: true` in `page`.

---

## 6. Taxonomy (F-10)

| Endpoint | Auth | Response |
|---|---|---|
| `GET /categories` | Public | `{ "data": [ { "id": 1, "slug": "career-failures", "name": "Career Failures", "description": "…", "position": 1 } ] }` (archived excluded) |
| `GET /industries` | Public | `{ "data": [ { "id": 1, "slug": "technology", "name": "Technology" } ] }` |
| `GET /tags?q=first` | Public | Top 10 by usage: `{ "data": [ { "slug": "first-job", "name": "first-job", "usageCount": 213 } ] }` |

---

## 7. Safety check (F-21)

### `POST /safety/check`
Auth: Verified · CSRF · Rate limit 60/h. Runs the same checks the create endpoints run, without saving anything. Used by the preview steps.
Request:
```json
{ "context": "confession", "identityMode": "anonymous", "fields": { "title": "…", "body": "My manager John Smith told me to call +91 98765 43210…" } }
```
`context` ∈ `post`, `confession`, `lesson`, `comment`, `workplace_review`, `job_report`, `company_response`, `profile`.
Response `200`:
```json
{
  "data": {
    "riskLevel": "medium",
    "willBeHeld": true,
    "findings": [
      { "rule": "named_individual", "severity": "medium", "field": "body", "start": 3, "end": 23, "message": "Naming individuals triggers a stricter review. Consider describing the role instead.", "blocking": false },
      { "rule": "pii.phone_number", "severity": "medium", "field": "body", "start": 40, "end": 55, "message": "This looks like a phone number.", "blocking": false }
    ]
  }
}
```

**Create endpoints and safety:** every create/update endpoint that accepts free text re-runs the check server-side. If any finding is `blocking` → `422 SAFETY_CHECK_BLOCKED`. If risk is `medium` and the request does not include `"acknowledgedFindings": ["named_individual","pii.phone_number"]` matching the current rules → `422 SAFETY_CONFIRMATION_REQUIRED`. Content is saved with `moderationStatus: "held"` when `willBeHeld` is true.

---

## 8. Posts and feed (F-10–F-14, F-16, F-28, F-29)

### `GET /feed`
Auth: Optional. Query:

| Param | Type | Notes |
|---|---|---|
| `mode` | `latest` (default) \| `following` (Should, Member) | `relevant` reserved for F-35. |
| `category` | category slug, repeatable | |
| `type` | `story`,`advice`,`lesson`,`confession`,`poll`,`meme`, repeatable | |
| `language` | BCP-47 primary tag | |
| `excludeCategory` | slug, repeatable | Visitors' session-level hide; members use mutes. |
| `cursor`, `limit` | | |

Response `200`: PostSummary collection. Excludes held/removed/limited/deleted posts, drafts, private-community posts the viewer can't see, and content from blocked/muted members, aliases, categories and tags. Errors: `400 INVALID_CURSOR`, `400 VALIDATION_ERROR` (unknown slug).

### `POST /posts`
Auth: Verified · CSRF · Rate limit 10/h. Creates `story`, `advice`, `lesson`, `poll` (Should) or `meme` (Should). Confessions use `/confessions`.
Request (story):
```json
{
  "postType": "story",
  "title": "Five rounds, one take-home, and then silence",
  "body": "I spent a weekend on the take-home…",
  "categoryId": 2,
  "tags": ["take-home", "ghosting"],
  "desiredResponse": "advice",
  "language": "en",
  "visibility": "public",
  "contentWarnings": [],
  "communityId": null,
  "acknowledgedFindings": []
}
```
Lesson additions:
```json
{
  "postType": "lesson",
  "lesson": {
    "whatIDidWrong": "Accepted the offer without asking about on-call.",
    "whatHappened": "Paged every other night for 3 months.",
    "whatILearned": "On-call expectations belong in the offer conversation.",
    "whatIdDoDifferently": "Ask for the on-call rota and compensation in writing.",
    "advice": "Ask: 'How often were people paged last quarter?'",
    "links": [ { "title": "Questions to ask before accepting an offer", "url": "https://example.org/questions" } ]
  }
}
```
(`body` is optional for lessons; the server composes it from sections for search.)
Poll additions (Should): `{ "postType": "poll", "poll": { "options": ["Yes", "No", "Still waiting"], "durationDays": 7 } }`.
Media additions (Should): `{ "mediaIds": ["0193…"] }` — each must be owned by caller, `scan_status = clean`, and have alt text.
Response `201 { "data": PostDetail }` (with `moderationNotice` if held).
Errors: `400` (title ≤ 150, body ≤ 20,000, ≤ 5 tags, unknown category, invalid community/visibility combination, lesson `whatILearned` required, poll 2–6 options, links must be https), `403 EMAIL_NOT_VERIFIED | ONBOARDING_REQUIRED | POSTING_RESTRICTED`, `403 FORBIDDEN` (not a member of community), `422 SAFETY_*`, `429`.

### `GET /posts/:postId`
Auth: Optional. Response `200 { "data": PostDetail }`. Visibility: `members` posts return `401 UNAUTHENTICATED` for visitors (UI shows sign-in prompt); held/removed/draft posts return `404` to everyone except the author (who sees `moderationNotice`). Also used for confessions (same ID space).

### `PATCH /posts/:postId`
Auth: Verified · CSRF · Author only. Editable: `title`, `body`, `categoryId`, `tags`, `desiredResponse`, `language`, `contentWarnings`, `visibility`, `lesson`, `acknowledgedFindings`. Poll options are not editable after the first vote. Response `200 { "data": PostDetail }` with `editedAt`. Stores a revision; re-runs safety checks (may move to `held`). Errors: `403 FORBIDDEN` (not author), `404`, `400`, `422`.

### `DELETE /posts/:postId`
Auth: Verified · CSRF · Author only. Soft delete. Response `204`. Errors: `403`, `404`.

### `POST /posts/:postId/corrections`
Auth: Verified · CSRF · Author only (anonymous authors verified via vault). Request `{ "body": "Correction: it was 4 rounds, not 5." }`. Response `201 { "data": { "id": "…", "body": "…", "createdAt": "…" } }`. Errors: `400` (≤ 1,000 chars; max 5 → `issue: "limit_reached"`), `403`, `404`, `422`.

### `POST /posts/:postId/reactions`
Auth: Verified · CSRF · Rate limit 120/min. Upserts the caller's single reaction.
Request `{ "reactionType": "same_here" }`
Response `200 { "data": { "reactionType": "same_here", "reactionCounts": { "im_dead": 3, "pain": 12, "same_here": 28, "melting": 2, "corporate_circus": 5, "survived": 4 } } }`.
Errors: `400` (unknown type), `403 BLOCKED`, `404`.

### `DELETE /posts/:postId/reactions`
Auth: Verified · CSRF. Response `200 { "data": { "reactionType": null, "reactionCounts": { … } } }`.

### `PUT /posts/:postId/save` · `DELETE /posts/:postId/save`
Auth: Member · CSRF. Idempotent. Response `204`. Errors: `404`.

### `PUT /posts/:postId/pinned-comment` · `DELETE /posts/:postId/pinned-comment`
Auth: Verified · CSRF · Post author. PUT `{ "commentId": "…" }` — comment must belong to this post and be authored by the post author (same user or same alias). Response `200 { "data": { "pinnedCommentId": "…" } }` / `204`. Errors: `400 VALIDATION_ERROR` (`issue: "not_own_comment"`), `403`, `404`.

### `POST /posts/:postId/poll/votes` (Should, F-28)
Auth: Verified · CSRF. Request `{ "optionId": "…" }`. Response `200 { "data": PostDetail.poll }`. Errors: `409 ALREADY_EXISTS` (voted), `400` (poll closed → `issue: "poll_closed"`), `404`.

### `GET /lessons` (F-16)
Auth: Optional. Query `category`, `cursor`, `limit`. Response: PostSummary collection where `postType = "lesson"`, each with `lessonPreview: { "whatILearned": "…(first 280 chars)" }`.

---

## 9. Comments (F-12)

### `GET /posts/:postId/comments?cursor=&limit=&sort=oldest|newest`
Auth: Optional. Returns top-level comments (default `oldest`) with up to 3 most recent replies each nested in `replies`; `replyCount` indicates more. Pinned clarification is returned separately in PostDetail. Blocked/muted authors' comments come back as `{ "id": "…", "hiddenReason": "blocked" }`.

### `GET /comments/:commentId/replies?cursor=`
Auth: Optional. Paginated replies for a comment.

### `POST /posts/:postId/comments`
Auth: Verified · CSRF · Rate limit 30/10 min.
Request:
```json
{ "body": "Follow up once after a week, then let it go.", "parentCommentId": null, "contentWarnings": [], "identityMode": "pseudonym", "acknowledgedFindings": [] }
```
`identityMode: "anonymous"` is accepted only when the caller owns the anonymous post (vault check); the comment then shows the post's label with `isOp: true`.
Response `201 { "data": Comment }`.
Errors: `400` (≤ 5,000 chars, parent not on this post), `403 BLOCKED`, `403 FORBIDDEN` (anonymous mode not owner), `404`, `422`, `429`.

### `PATCH /comments/:commentId`
Auth: Verified · CSRF · Author. Request `{ "body": "…", "contentWarnings": [] }`. Response `200 { "data": Comment }`. Errors: `403`, `404`, `422`.

### `DELETE /comments/:commentId`
Auth: Verified · CSRF · Author. Response `204`.

---

## 10. Office Confessions (F-15)

All confession endpoints return `403 FEATURE_DISABLED` while `confessions.enabled` is false.

### `POST /confessions`
Auth: Verified · CSRF · Rate limit 5/24 h (1 if account < 24 h).
Request:
```json
{
  "title": "Weekend deploys are 'optional' the way breathing is optional",
  "body": "Every Friday at 6pm…",
  "experienceType": "workload_overtime",
  "industryId": 1,
  "broadRegion": "IN",
  "periodYear": 2026,
  "periodQuarter": 3,
  "categoryId": 6,
  "tags": ["on-call"],
  "desiredResponse": "support",
  "contentWarnings": ["burnout"],
  "identityMode": "anonymous",
  "visibility": "members",
  "saveAsDraft": false,
  "acknowledgedFindings": ["named_individual"],
  "previewConfirmed": true
}
```
`previewConfirmed` must be `true` unless `saveAsDraft` is `true`.
Response `201`:
```json
{
  "data": {
    "confession": { "id": "0193b1f0-5a6b-7c8d-8e9f-a0b1c2d3e4f5", "postType": "confession", "author": { "kind": "anonymous", "label": "Anonymous Stapler #4821", "isOp": false }, "publicationStatus": "published", "moderationStatus": "held", "…": "PostDetail fields" },
    "moderationNotice": { "status": "held", "message": "Your confession is waiting for a quick review (usually within 24 hours)." }
  }
}
```
The response never includes the caller's username for anonymous confessions.
Errors: `400` (missing required fields, `previewConfirmed` false), `403 FEATURE_DISABLED | POSTING_RESTRICTED | EMAIL_NOT_VERIFIED`, `422 SAFETY_CHECK_BLOCKED | SAFETY_CONFIRMATION_REQUIRED`, `429`.

### `GET /confessions?experienceType=&category=&cursor=`
Auth: Optional (`members`-visibility confessions require Member). PostSummary collection with `postType = "confession"`.

### `GET /confessions/:confessionId`
Auth: Optional. Alias of `GET /posts/:postId` restricted to confessions; `404` if the ID is not a confession.

### `PATCH /confessions/:confessionId`
Auth: Verified · CSRF · Owner (vault check). Edits fields or publishes a draft (`{ "publish": true, "previewConfirmed": true }`). Identity mode cannot change after publication (`400 issue: "identity_mode_locked"`). Response `200 { "data": PostDetail }`.

### `DELETE /confessions/:confessionId`
Auth: Verified · CSRF · Owner. Soft delete; the alias mapping is deleted at purge time. Response `204`.

---

## 11. Companies and Workplace Reality Pages (F-17, F-25)

### `GET /companies?q=&industry=&cursor=`
Auth: Public. Active companies; `q` uses trigram similarity on name (min 2 chars).
```json
{ "data": [ { "id": "0193a000-1111-7222-8333-444455556666", "slug": "acme-widgets", "name": "Acme Widgets", "industry": { "slug": "technology", "name": "Technology" }, "reviewCount": 7, "jobReportCount": 4 } ], "page": { "nextCursor": null, "hasMore": false, "limit": 20 } }
```

### `POST /companies`
Auth: Verified · CSRF · Rate limit 5/day. Submit a missing company.
Request `{ "name": "Initech Systems", "industryId": 1, "officialWebsite": "https://initech.example" }`
Response `201 { "data": { "id": "…", "slug": "initech-systems", "name": "Initech Systems", "pageStatus": "pending" } }`.
Duplicate → `409 CONFLICT` with `details[0].existing = { "id": "…", "slug": "…", "name": "…" }`.

### `GET /companies/:companyIdOrSlug`
Auth: Optional. Accepts UUID or slug. Merged companies → `301` to the target slug. Pending companies → `404` except to the submitter.
```json
{
  "data": {
    "id": "0193a000-1111-7222-8333-444455556666",
    "slug": "acme-widgets",
    "name": "Acme Widgets",
    "industry": { "slug": "technology", "name": "Technology" },
    "officialWebsite": "https://acme-widgets.test",
    "description": "Maker of widgets and widget accessories.",
    "verificationNotice": "Reviews and reports are unverified firsthand accounts from members.",
    "themes": {
      "status": "available",
      "windowMonths": 24,
      "topics": {
        "work_life_balance": { "n": 7, "median": 2, "distribution": [2, 3, 1, 1, 0] },
        "management": { "n": 7, "median": 3, "distribution": [1, 1, 3, 2, 0] },
        "communication": { "n": 6, "median": 3, "distribution": [0, 2, 2, 2, 0] },
        "compensation": { "n": 5, "median": 3, "distribution": [0, 1, 3, 1, 0] },
        "career_growth": { "n": 4, "median": null, "distribution": null, "insufficient": true }
      }
    },
    "counts": { "reviews": 7, "jobReports": 4 },
    "hasVerifiedRepresentative": true,
    "viewer": { "hasReviewed": false, "isRepresentative": false }
  }
}
```
`themes.status` ∈ `available`, `insufficient_data` (< 5 reviews overall), `paused` (campaign review).

### `GET /companies/:companyId/reviews?relationship=&topic=&periodFrom=&periodTo=&cursor=`
Auth: Optional.
```json
{
  "data": [
    {
      "id": "0193c000-aaaa-7bbb-8ccc-000000000001",
      "author": { "kind": "anonymous", "label": "Former employee" },
      "relationshipType": "former_employee",
      "period": { "startYear": 2024, "endYear": 2026 },
      "title": "Great peers, chaotic planning",
      "narrative": "Sprint goals changed weekly…",
      "narrativeHidden": false,
      "ratings": { "work_life_balance": 2, "management": 3 },
      "evidenceStatus": "unverified",
      "label": "Unverified firsthand account",
      "companyResponse": { "id": "…", "body": "Thanks for the feedback — we've since…", "badge": "Official representative", "createdAt": "…" },
      "publishedAt": "2027-01-05T12:00:00Z",
      "viewer": { "isAuthor": false }
    }
  ],
  "page": { "nextCursor": null, "hasMore": false, "limit": 20 }
}
```
If `display_consent` is false: `narrative: null, narrativeHidden: true`.

### `POST /companies/:companyId/reviews`
Auth: Verified · CSRF · Account ≥ 7 days · Rate limit 3/day.
Request:
```json
{
  "relationshipType": "former_employee",
  "periodStartYear": 2024,
  "periodEndYear": 2026,
  "title": "Great peers, chaotic planning",
  "narrative": "Sprint goals changed weekly and on-call was unpaid… (≥ 150 chars)",
  "ratings": { "work_life_balance": 2, "management": 3, "communication": 3, "compensation": 3, "career_growth": 2 },
  "firsthandAttested": true,
  "displayConsent": true,
  "acknowledgedFindings": []
}
```
Response `201 { "data": Review }` (held reviews include `moderationNotice`). Pending companies: review stored, published on approval.
Errors: `400`, `403 ACCOUNT_TOO_NEW`, `409 CONFLICT` (`details[0].existingReviewId`), `404`, `422`, `429`.

### `PATCH /reviews/:reviewId` · `DELETE /reviews/:reviewId`
Auth: Verified · CSRF · Owner (vault). PATCH accepts the same fields except `firsthandAttested` (must remain true). Response `200 { "data": Review }` / `204`.

### `POST /companies/:companyId/representation-requests` (Should, F-25)
Auth: Verified · CSRF.
Request `{ "workEmail": "ken.hr@acme-widgets.test", "jobTitle": "Employer Brand Lead" }`
Response `202 { "data": { "membershipId": "…", "status": "pending_email" } }`. Errors: `400` (`issue: "domain_mismatch"` or company website not confirmed), `409` (existing active request).

### `POST /companies/representation-requests/verify-email` (Should)
Auth: Member · CSRF. Request `{ "token": "…" }`. Response `200 { "data": { "membershipId": "…", "status": "pending_review" } }`. Errors: `400 TOKEN_INVALID`.

### `POST /companies/:companyId/responses` (Should)
Auth: Representative · CSRF.
Request `{ "targetType": "workplace_review", "targetId": "0193c000-…", "body": "Thanks for the feedback — we've since introduced paid on-call.", "acknowledgedFindings": [] }`
Response `201 { "data": { "id": "…", "targetType": "workplace_review", "targetId": "…", "body": "…", "badge": "Official representative", "moderationStatus": "clear", "createdAt": "…" } }`.
Errors: `403 FORBIDDEN` (not representative of this company), `404` (target not on this company), `409 CONFLICT` (already responded), `422`.

### `PATCH /company-responses/:responseId` · `DELETE /company-responses/:responseId` (Should)
Auth: Representative of the response's company · CSRF. Response `200` / `204`.

---

## 12. Job Nightmare Board (F-18)

### `GET /job-reports?companyId=&reportType=&roleCategory=&from=2026-01&to=2026-12&cursor=`
Auth: Optional.
```json
{
  "data": [
    {
      "id": "0193d000-bbbb-7ccc-8ddd-000000000001",
      "company": { "id": "…", "slug": "acme-widgets", "name": "Acme Widgets" },
      "author": { "kind": "anonymous", "label": "Anonymous Lanyard #0917", "isOp": false },
      "roleCategory": "software_engineering",
      "reportType": "unpaid_test_assignment",
      "experienceMonth": "2026-10",
      "stageReached": "technical",
      "outcome": "ghosted",
      "narrative": "The 'two-hour' assignment needed…",
      "label": "Unverified firsthand account",
      "companyResponse": null,
      "publishedAt": "2026-11-02T10:00:00Z"
    }
  ],
  "page": { "nextCursor": "…", "hasMore": true, "limit": 20 }
}
```

### `POST /job-reports`
Auth: Verified · CSRF · Rate limit 5/day.
Request:
```json
{
  "companyId": "0193a000-1111-7222-8333-444455556666",
  "newCompany": null,
  "roleCategory": "software_engineering",
  "reportType": "unpaid_test_assignment",
  "experienceMonth": "2026-10",
  "stageReached": "technical",
  "outcome": "ghosted",
  "narrative": "The 'two-hour' assignment needed a full backend… (≥ 100 chars)",
  "identityMode": "anonymous",
  "visibility": "public",
  "acknowledgedFindings": []
}
```
Provide either `companyId` or `newCompany: { "name", "industryId", "officialWebsite" }` (creates a pending company).
Response `201 { "data": JobReport }`. Errors: `400` (future month, narrative length), `404` (company), `409` (new company duplicate, with existing), `422`, `429`.

### `GET /job-reports/:jobReportId` · `PATCH /job-reports/:jobReportId` · `DELETE /job-reports/:jobReportId`
GET: Optional. PATCH/DELETE: Verified · CSRF · Owner (user id or vault). Identity mode is locked after publication. Responses `200` / `200` / `204`.

---

## 13. Search (F-19)

### `GET /search?q=&type=&category=&cursor=&limit=`
Auth: Optional · Rate limit 60/min. `q` 2–100 chars. `type` ∈ `all` (default), `posts`, `lessons`, `confessions`, `job_reports`, `companies`, `members`, `communities` (Should).
Response for `type=all` (top 5 per group, no cursor):
```json
{
  "data": {
    "posts": [ { "…": "PostSummary", "highlight": "…<mark>unpaid</mark> <mark>test</mark> assignment…" } ],
    "lessons": [],
    "confessions": [],
    "jobReports": [ { "…": "JobReport", "highlight": "…" } ],
    "companies": [ { "id": "…", "slug": "acme-widgets", "name": "Acme Widgets", "reviewCount": 7 } ],
    "members": [ { "username": "unpaid_tester", "displayName": null, "headline": "…" } ],
    "communities": []
  }
}
```
For a specific `type`, a paginated collection of that item type. `members` include only `discoverable = true`, non-hidden profiles. Highlights use `ts_headline` with HTML-escaped text and only `<mark>` tags.
Errors: `400 VALIDATION_ERROR` (`q` too short), `429`.

---

## 14. Reports (F-20)

### `POST /reports`
Auth: Verified · CSRF · Rate limit 20/h.
Request:
```json
{ "targetType": "comment", "targetId": "0193b2aa-0000-7000-8000-000000000042", "reason": "personal_information", "details": "It includes my coworker's phone number." }
```
For profiles: `{ "targetType": "profile", "username": "…" }` (no user IDs). For communities: `targetId` is the community id.
Response `201 { "data": { "reportId": "…", "status": "open", "message": "Thanks — our moderators will review this." } }`.
Errors: `400`, `404` (target not visible), `409 ALREADY_EXISTS`, `429`.

### `GET /me/reports?cursor=`
Auth: Member.
```json
{ "data": [ { "reportId": "…", "target": { "type": "comment", "summary": "Comment on “Five rounds, one take-home…”" }, "reason": "personal_information", "status": "actioned", "createdAt": "…", "resolvedAt": "…", "outcomeMessage": "We removed the content for breaking our personal-information rule." } ] }
```

### `GET /me/moderation-actions?cursor=` (F-22)
Auth: Member. Actions affecting the caller's content or account (including anonymous content, resolved via vault).
```json
{ "data": [ { "actionId": "…", "actionType": "remove_content", "target": { "type": "comment", "summary": "Your comment on “…”" }, "policyReference": "CG-2.1", "memberMessage": "…", "createdAt": "…", "appeal": { "eligible": true, "deadline": "2027-02-19T14:03:12Z", "status": null } } ] }
```

---

## 15. Moderation (F-21, F-22, F-26)

> The plan's `POST /moderation/:targetId/action` is implemented as `POST /moderation/items/:itemId/actions`, because a bare target ID does not identify the target type. The plan's `POST /moderation/:actionId/appeal` is implemented as `POST /moderation/actions/:actionId/appeal`. See [ASSUMPTIONS.md](ASSUMPTIONS.md#5-api).

### `GET /moderation/queue?status=open&priority=&reason=&targetType=&assigned=me|unassigned|any&cursor=`
Auth: Moderator. Ordered by `priority_score DESC, created_at ASC`.
```json
{
  "data": [
    {
      "itemId": "0193e000-cccc-7ddd-8eee-000000000001",
      "targetType": "confession",
      "targetId": "0193b1f0-5a6b-7c8d-8e9f-a0b1c2d3e4f5",
      "priority": "high",
      "status": "open",
      "automatedHold": true,
      "reasons": [ { "reason": "harassment_or_threats", "count": 2 } ],
      "reportCount": 2,
      "automatedFindings": [ { "rule": "named_individual", "severity": "medium" } ],
      "preview": { "title": "…", "excerpt": "…", "author": { "kind": "anonymous", "label": "Anonymous Stapler #4821" } },
      "ageMinutes": 47,
      "assignedTo": null
    }
  ],
  "page": { "nextCursor": "…", "hasMore": true, "limit": 20 }
}
```

### `GET /moderation/items/:itemId`
Auth: Moderator. Full context: target content (including revisions), all reports with details (reporter shown as username only for pseudonymous reporters), automated findings with highlighted ranges, the author's moderation history summary (counts by action type; for anonymous authors the history is aggregated via vault without revealing identity), prior actions on the target, and related campaign items.

### `POST /moderation/items/:itemId/claim` · `POST /moderation/items/:itemId/release`
Auth: Moderator · CSRF. Response `200 { "data": { "assignedTo": "lena_mod" } }`. Errors: `409 CONFLICT` (claimed by another moderator).

### `POST /moderation/items/:itemId/actions`
Auth: Moderator · CSRF.
Request:
```json
{
  "actionType": "remove_content",
  "reason": "Comment contained a third party's phone number.",
  "policyReference": "CG-2.1",
  "memberMessage": "We removed your comment because it included someone's personal phone number (Community Guidelines 2.1). You can appeal within 30 days.",
  "durationDays": null,
  "contentWarnings": null,
  "escalationNote": null,
  "notifyReporters": true
}
```
Rules: `restrict_posting` requires `durationDays` ∈ {1, 7, 30}; `add_content_warning` requires `contentWarnings`; `escalate` requires `escalationNote` and sets item status `escalated`; `no_action` on a held item publishes it. `suspend_account` and `restrict_posting` on anonymous content are applied to the owning account via `IdentityVaultService.resolveForEnforcement` without returning identity.
Response `201 { "data": { "actionId": "…", "itemStatus": "resolved", "targetModerationStatus": "removed" } }`.
Errors: `400`, `403`, `404`, `409 CONFLICT` (item already resolved).

### `POST /moderation/actions/:actionId/appeal`
Auth: Member · CSRF. Caller must be the affected member (user ID or vault owner).
Request `{ "reason": "The number was a public business line, not personal." }`
Response `201 { "data": { "appealId": "…", "status": "pending" } }`.
Errors: `403` (not affected member), `409 ALREADY_EXISTS`, `400` (`issue: "appeal_window_closed"`).

### `GET /moderation/appeals?status=pending&cursor=`
Auth: Moderator. Lists appeals with action summary; `canDecide: false` when the caller took the original action.

### `POST /moderation/appeals/:appealId/decision`
Auth: Moderator (not the original moderator) · CSRF.
Request `{ "outcome": "reversed", "decisionNote": "The number belongs to a public business listing.", "modifiedAction": null }`
`modified` requires `modifiedAction` (same shape as an action request). Reversal restores the target and lifts restrictions.
Response `200 { "data": { "appealId": "…", "status": "reversed" } }`. Errors: `403` (`issue: "same_moderator"`), `409` (already decided).

### Community moderation (Should, F-26)
| Method & path | Auth | Description |
|---|---|---|
| `GET /communities/:communityId/moderation/queue` | Community moderator | Scoped items (`scope = community`). Same item shape. |
| `POST /communities/:communityId/moderation/items/:itemId/actions` | Community moderator · CSRF | Allowed `actionType`: `no_action`, `remove_content` (from community), `add_content_warning`, `ban_from_community` (community-only action stored with `scope=community`; maps to `community_members.banned_at`). Threat/doxxing items are always mirrored to the platform queue. |

---

## 16. Trust & Safety — Identity Vault (F-23)

### `POST /trust-safety/identity-reveals`
Auth: Vault · CSRF.
Request `{ "aliasSource": { "type": "post", "id": "0193b1f0-…" }, "moderationItemId": "0193e000-…", "category": "credible_threat", "justification": "Post contains a specific threat of violence against a named coworker with date and location (≥ 50 chars)." }`
Response `201 { "data": { "revealId": "…", "status": "pending_approval" } }`. Audit: `vault.reveal_requested`.
Errors: `403` (no permission; attempt audit-logged), `400` (justification < 50).

### `POST /trust-safety/identity-reveals/:revealId/approve` · `POST /trust-safety/identity-reveals/:revealId/reject`
Auth: Vault (different user from requester) · CSRF. Request `{ "note": "Confirmed threat criteria met." }`. Response `200 { "data": { "status": "approved", "accessExpiresAt": "…" } }`. Errors: `403` (`issue: "self_approval"`), `409` (not pending). Audit: `vault.reveal_approved` / `vault.reveal_rejected`.

### `GET /trust-safety/identity-reveals/:revealId`
Auth: Vault, requester only, within the 24 h access window.
`200 { "data": { "status": "approved", "account": { "userId": "…", "username": "…", "accountStatus": "active", "createdAt": "…" }, "accessExpiresAt": "…" } }`. The email is not returned; legal-order cases obtain it through the documented legal process in [SECURITY.md](SECURITY.md#10-legal-requests). Every read is audit-logged (`vault.reveal_viewed`). After expiry → `403`.

---

## 17. Communities (Should, F-26)

All return `403 FEATURE_DISABLED` while `communities.enabled` is false.

| Method & path | Auth | Request | Response / errors |
|---|---|---|---|
| `GET /communities?q=&cursor=` | Optional | — | `200` list `{ id, slug, name, description, visibility, memberCount, viewer: { role, joinRequestStatus } }`; private communities are listed (name/description) but content hidden. |
| `POST /communities` | Verified · CSRF · age ≥ 14 d | `{ "name": "Fresh Graduate Survivors", "slug": "fresh-graduate-survivors", "description": "…", "visibility": "public", "rules": [ { "title": "Support, don't compete", "body": "…" } ] }` | `201` Community. `400`, `403 ACCOUNT_TOO_NEW`, `409 CONFLICT` (slug), `400 issue: owned_limit` (max 3). |
| `GET /communities/:communityIdOrSlug` | Optional | — | `200` `{ …, rules: [], pinnedResources: [], moderators: [ { username } ] }`. |
| `PATCH /communities/:communityId` | Community owner · CSRF | `{ "name"?, "description"?, "visibility"? }` | `200`. |
| `POST /communities/:communityId/join` | Verified · CSRF | `{ "message": "…" }` (private only) | Public: `200 { "role": "member" }`; private: `202 { "joinRequestStatus": "pending" }`. `403` banned, `409` already member, `400 issue: retry_after` (declined < 30 days). |
| `DELETE /communities/:communityId/membership` | Member · CSRF | — | `204`. Owners must transfer ownership first (`400 issue: owner_cannot_leave`). |
| `GET /communities/:communityId/posts?cursor=` | Optional (private: members) | — | PostSummary collection. Non-members of private → `404`. |
| `GET /communities/:communityId/members?cursor=` | Members (private) / Optional (public) | — | `[ { username, role, joinedAt } ]`. |
| `GET /communities/:communityId/join-requests` | Community moderator | — | `[ { requestId, username, message, createdAt } ]`. |
| `POST /communities/:communityId/join-requests/:requestId/decision` | Community moderator · CSRF | `{ "decision": "approved" }` | `200`. |
| `PUT /communities/:communityId/rules` | Community moderator · CSRF | `{ "rules": [ { "title": "…", "body": "…" } ] }` (≤ 10) | `200`. |
| `POST /communities/:communityId/pinned-resources` | Community moderator · CSRF | `{ "title": "Salary negotiation guide", "url": "https://…" }` | `201`; `400 issue: limit_reached` (5). |
| `DELETE /communities/:communityId/pinned-resources/:resourceId` | Community moderator · CSRF | — | `204`. |
| `PUT /communities/:communityId/moderators/:username` | Community owner · CSRF | — | `200` (user must be a member). |
| `DELETE /communities/:communityId/moderators/:username` | Community owner · CSRF | — | `204`. |
| `POST /communities/:communityId/announcements` | Community moderator · CSRF | `{ "title": "…", "body": "…" }` | `201` — creates a pinned community post and `community_announcement` notifications. |
| `POST /communities/:communityId/invitations` | Community moderator · CSRF | `{ "username": "…" }` | `201` — `community_invitation` notification; respects blocks. |

---

## 18. Notifications (Should, F-27)

| Method & path | Auth | Response |
|---|---|---|
| `GET /notifications?cursor=&unreadOnly=true` | Member | `[ { "id": "…", "type": "comment_on_post", "title": "New comment on your post", "body": "survived_standups commented on “Five rounds…”", "link": "/posts/0193b0aa…#comment-0193b2aa…", "actorCount": 1, "readAt": null, "createdAt": "…" } ]` |
| `GET /notifications/unread-count` | Member | `{ "data": { "count": 4 } }` (polled every 60 s by the web app) |
| `POST /notifications/:notificationId/read` | Member · CSRF | `204` |
| `POST /notifications/read-all` | Member · CSRF | `200 { "data": { "updated": 4 } }` |

Notifications about the recipient's anonymous content never include the anonymous label in email subjects; links are resolved server-side from encrypted references.

---

## 19. Media (Should, F-29)

### `POST /media/uploads`
Auth: Verified · CSRF · Rate limit 30/h.
Request `{ "purpose": "post_image", "mimeType": "image/png", "sizeBytes": 3145728, "altText": "Meme: a dog in a burning room saying 'this sprint is fine'" }`
Response `201`:
```json
{ "data": { "mediaId": "…", "upload": { "url": "https://8c1f4e2a9b7d.r2.cloudflarestorage.com/linkedout-media/uploads/0193f1c2-…?X-Amz-Signature=…", "method": "PUT", "headers": { "Content-Type": "image/png" }, "expiresAt": "…(10 min)" } } }
```
Errors: `400` (alt text required for `post_image`), `413`, `415`.

### `POST /media/:mediaId/complete`
Auth: Owner · CSRF. Queues validation (magic bytes), EXIF strip, re-encode and ClamAV scan. Response `202 { "data": { "scanStatus": "pending" } }`.

### `GET /media/:mediaId`
Auth: Owner (pending) / Optional (clean and attached to visible content). `200 { "data": { "mediaId": "…", "scanStatus": "clean", "url": "https://files.linkedout.app/m/…", "width": 1200, "height": 800, "altText": "…" } }`. Infected → `scanStatus: "infected"` and the object is deleted.

---

## 20. Administration (F-24, F-25)

All `/admin/*` endpoints: Auth Admin; writes require CSRF and are audit-logged.

| Method & path | Request | Response / notes |
|---|---|---|
| `GET /admin/metrics?window=7d\|30d` | — | `{ "signups": 412, "verifiedMembers": 301, "postsPerDay": 58.3, "reportsPer1000Posts": 14.2, "openQueue": 23, "medianModerationResponseMinutes": { "high": 96, "normal": 610 }, "appealReversalRate": 0.18 }` |
| `GET /admin/users?q=&status=&role=&cursor=` | — | `[ { "userId": "…", "email": "…", "username": "…", "accountStatus": "active", "platformRole": "member", "permissions": [], "createdAt": "…", "moderationSummary": { "removeContent": 1 } } ]` — never anonymous mappings. |
| `PATCH /admin/users/:userId/role` | `{ "platformRole": "platform_moderator" }` | `200`. `400 issue: last_admin`. |
| `POST /admin/users/:userId/permissions` | `{ "permission": "identity_vault.reveal" }` | `201`. Requires target to be moderator/admin. |
| `DELETE /admin/users/:userId/permissions/:permission` | — | `204`. |
| `GET /admin/categories` · `POST /admin/categories` · `PATCH /admin/categories/:categoryId` | `{ "slug": "…", "name": "…", "description": "…", "position": 13, "isArchived": false }` | `200`/`201`. `409` slug. |
| `PUT /admin/categories/order` | `{ "categoryIds": [1, 2, 3, …] }` | `200`. |
| `GET /admin/companies?status=pending&cursor=` | — | Pending companies with submitter username and possible duplicates. |
| `PATCH /admin/companies/:companyId` | `{ "pageStatus": "active", "name"?, "industryId"?, "officialWebsite"?, "websiteConfirmed": true, "description"? }` | `200`. Approval publishes attached clear reviews/reports. |
| `POST /admin/companies/:companyId/merge` | `{ "intoCompanyId": "…" }` | `200`. Moves reviews, reports, memberships; source becomes `merged`. |
| `GET /admin/company-memberships?status=pending_review` (Should) | — | `[ { "membershipId", "company", "username", "jobTitle", "workEmailDomain", "workEmailVerifiedAt" } ]` |
| `POST /admin/company-memberships/:membershipId/decision` (Should) | `{ "decision": "verified" \| "rejected" \| "revoked", "note": "…" }` | `200`. `verified` sets `expiresAt = now + 12 months`. |
| `GET /admin/escalations?cursor=` | — | Moderation items with status `escalated`. |
| `GET /admin/audit-logs?action=&actorUsername=&targetType=&from=&to=&cursor=` | — | `[ { "id", "actorType", "actor": { "username" }, "action", "targetType", "targetId", "metadata", "requestId", "createdAt" } ]` (read-only; no update/delete endpoints exist). |
| `GET /admin/feature-flags` · `PATCH /admin/feature-flags/:key` | `{ "enabled": true }` | `200`. Enabling `confessions.enabled` in production requires the launch-gate checklist confirmation `{ "enabled": true, "launchGateConfirmed": true }`. |
| `POST /admin/beta-invites` | `{ "count": 50, "maxUses": 1, "expiresInDays": 30 }` | `201 { "codes": ["BETA-7K2Q", …] }` |

---

## 21. Future endpoints (reserved, not implemented in MVP or v1.1)

| Feature | Endpoints |
|---|---|
| F-31 Direct Messaging | `GET /conversations`, `POST /message-requests` `{ "username", "body" }`, `POST /message-requests/:id/decision`, `GET /conversations/:id/messages`, `POST /conversations/:id/messages`, `DELETE /messages/:id` |
| F-32 Career Damage Report | `GET /me/career-report`, `GET/POST/PATCH/DELETE /me/career/applications`, `GET/POST/PATCH/DELETE /me/career/goals` |
| F-33 Meme Generator | `GET /meme-templates`, `POST /memes` (renders server-side, returns mediaId) |
| F-34 Hall of Shame | `GET /hall-of-shame` (curated pattern highlights), `POST /admin/hall-of-shame` |
| F-35 Relevant feed | `GET /feed?mode=relevant` |
| F-36 AI categorization | `POST /safety/suggest` → `{ suggestedCategoryId, suggestedContentWarnings }` |
| F-37 Premium | `GET /billing/plans`, `POST /billing/checkout`, `POST /billing/webhook` |
| F-38 Employer analytics | `GET /companies/:companyId/analytics` (Representative, k ≥ 10) |
| F-39 MFA | `POST /me/mfa/totp/setup`, `POST /me/mfa/totp/verify`, `DELETE /me/mfa/totp`, `POST /auth/login/mfa` |
| F-40 Mobile apps | No new endpoints: native clients use the same `/api/v1` with the session cookie held in secure native storage; `X-Client: ios/<version>` / `android/<version>` header for telemetry and minimum-version checks (`426 Upgrade Required` when unsupported) |
| F-41 Learning paths | `GET /learning-paths`, `GET /learning-paths/:slug`, `POST /learning-paths/:slug/progress` |
| F-42 Push | `POST /me/push-subscriptions`, `DELETE /me/push-subscriptions/:id` |
| F-43 Events | `GET /events`, `POST /events` (Community moderator), `POST /events/:id/rsvp` `{ "displayMode": "pseudonym" \| "anonymous_count" }` |

---

## 22. Endpoint → feature traceability

| Feature | Endpoint groups |
|---|---|
| F-01 | §3 auth, `/me/sessions` |
| F-02 | `/auth/oauth/*` |
| F-03 | `/auth/username-availability`, `PATCH /me/username`, AuthorRef |
| F-04 | `POST /me/onboarding/complete` |
| F-05 | `/profiles/*`, `PATCH /me/profile`, `PUT /me/profile/*` |
| F-06 | `/me/privacy` |
| F-07 | `/me/blocks`, `/me/mutes` |
| F-08 | `DELETE /me/account`, `POST /me/account/cancel-deletion` |
| F-09 | `/me/exports` |
| F-10 | `/posts`, `/categories`, `/tags`, `/industries`, `/posts/:id/corrections` |
| F-11 | `GET /feed` |
| F-12 | comments §9, `/posts/:id/pinned-comment` |
| F-13 | `/posts/:id/reactions` |
| F-14 | `/posts/:id/save`, `/me/saved` |
| F-15 | `/confessions/*` |
| F-16 | `POST /posts` (lesson), `GET /lessons` |
| F-17 | `/companies*`, `/reviews/*` |
| F-18 | `/job-reports*` |
| F-19 | `GET /search` |
| F-20 | `POST /reports`, `GET /me/reports` |
| F-21 | `POST /safety/check`, `/moderation/queue`, `/moderation/items/*` |
| F-22 | `/moderation/actions/:id/appeal`, `/moderation/appeals*`, `/me/moderation-actions` |
| F-23 | `/trust-safety/identity-reveals*`, `/admin/audit-logs` |
| F-24 | `/admin/*` |
| F-25 | representation requests, `/companies/:id/responses`, `/company-responses/*`, `/admin/company-memberships*` |
| F-26 | §17, community moderation §15 |
| F-27 | §18, `/me/notification-preferences` |
| F-28 | `POST /posts` (poll), `/posts/:id/poll/votes` |
| F-29 | §19 |
| F-30 | `/me/follows`, `GET /feed?mode=following` |
| F-31–F-43 | §21 |
