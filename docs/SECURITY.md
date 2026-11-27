# LinkedOut — Security, Privacy and Trust

Privacy and security are foundational requirements (plan §14), not enhancements. This document is normative. Code that contradicts it is a bug.

Related: [ARCHITECTURE.md](ARCHITECTURE.md) · [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md) · [API_SPEC.md](API_SPEC.md) · [TESTING.md](TESTING.md#4-security-tests) · [DEPLOYMENT.md](DEPLOYMENT.md#11-operational-runbooks)

## 1. Threat model summary

| Asset | Threat actors | Key threats | Primary controls |
|---|---|---|---|
| Link between anonymous content and an account | Employers, coworkers, curious staff, attackers with DB access | De-anonymization via API leaks, logs, timing, side channels, insider browsing | Vault schema + DB role isolation, no author IDs on anonymous rows, serializer whitelists, log redaction, four-eyes reveal, audit logs |
| Account credentials and sessions | Credential stuffers, phishers | Account takeover | Argon2id, breached-password check, rate limits + lockout, httpOnly session cookies, session revocation, MFA (Future) |
| Personal data (email, IP) | Attackers, data brokers | Breach, over-collection | Data minimization, encryption at rest, retention schedule, restricted DB roles |
| Platform integrity | Spammers, review brigades, harassers | Spam, fake reviews, coordinated attacks, doxxing | Verification gates, account-age gates, velocity rules, campaign detection, moderation queue |
| Third parties named in content | Members posting about others | Defamation, doxxing, harassment | Named-individual detection, PII detection, stricter review, right of reply, corrections, legal escalation |
| Infrastructure | Attackers | Injection, SSRF, supply-chain compromise, secret leaks | Parameterized queries, input validation, dependency scanning, image scanning, secret scanning, least-privilege credentials |

## 2. Roles and permissions

### 2.1 Roles

| Role | How assigned | Scope |
|---|---|---|
| Visitor | No session | Public content within visibility rules |
| Member | Registered account (`platform_role = member`); "verified member" once email verified + onboarding complete | Own content and settings |
| Community moderator | `community_members.role ∈ {moderator, owner}` (Should) | One community |
| Platform moderator | `platform_role = platform_moderator` (assigned by admin) | Platform-wide moderation |
| Trust & Safety officer | Platform moderator or admin with staff permission `identity_vault.reveal` | Identity reveal workflow only |
| Company representative | Verified `company_memberships` row (Should) | One company page's responses |
| Administrator | `platform_role = admin` | Platform configuration and access management |

Least privilege applies throughout: roles do not inherit vault access, and admins cannot read anonymous mappings without the reveal workflow (plan §7).

### 2.2 Permissions matrix

✅ allowed · 🟡 own/scoped only · ❌ denied

| Capability | Visitor | Member (unverified) | Verified member | Community mod | Platform mod | T&S officer | Company rep | Admin |
|---|---|---|---|---|---|---|---|---|
| Read public content | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Read members-only content | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Read private community content | ❌ | ❌ | 🟡 member | 🟡 own community | ✅ (when moderating) | ✅ | ❌ unless member | ✅ |
| Create posts, confessions, comments, reactions | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Edit/delete content | ❌ | ❌ | 🟡 own | 🟡 own | 🟡 own | 🟡 own | 🟡 own | 🟡 own |
| Save posts | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Report content | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Write workplace reviews | ❌ | ❌ | ✅ (age ≥ 7 d) | ✅ | ✅ | ✅ | ❌ for own company | ✅ |
| Respond officially on company page | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | 🟡 own company | ❌ |
| Remove or hide company-page criticism | ❌ | ❌ | ❌ | ❌ | ✅ (policy violations only) | ✅ | ❌ | ✅ |
| Moderate community content | ❌ | ❌ | ❌ | 🟡 own community | ✅ | ✅ | ❌ | ✅ |
| View platform moderation queue | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ |
| Apply platform moderation actions | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ |
| Decide appeals | ❌ | ❌ | ❌ | ❌ | ✅ (not own action) | ✅ (not own action) | ❌ | ✅ (not own action) |
| Request / approve identity reveal | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ (four-eyes, not self) | ❌ | ❌ unless granted |
| View private account data (email) of others | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | 🟡 via admin user search (audit-logged) |
| View private messages (Future) | ❌ | ❌ | 🟡 own | 🟡 own | ❌ | ❌ | ❌ | ❌ |
| Manage roles/permissions, categories, companies, flags | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Read audit logs | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

### 2.3 Authorization implementation

1. **Guards** (`SessionGuard`, `VerifiedGuard`, `RolesGuard`, `CommunityRoleGuard`, `RepresentativeGuard`, `VaultPermissionGuard`) check coarse access. **Services** check fine-grained ownership (e.g. `post.author_user_id === currentUser.id`, or `vault.isOwner(user, aliasId)` for anonymous content).
2. All controllers are deny-by-default. Public routes need the `@Public()` decorator, and a CI test fails if a controller method has neither `@Public()` nor an auth guard.
3. IDs in paths are looked up **with** the caller's visibility constraints. Resources the caller cannot see return `404`, not `403`, so existence doesn't leak.
4. Role and permission changes take effect immediately by rewriting the Redis session entries.
5. Admin user search that shows emails is audit-logged (`admin.user_viewed`).

## 3. Authentication

| Control | Implementation |
|---|---|
| Registration | Email + password; username chosen independently; age ≥ 16 confirmation; policy acceptance with version stored |
| Password policy | Minimum 12 characters, maximum 128; no composition rules; rejected if found in Have I Been Pwned (k-anonymity range API; fail-open with logging if HIBP is unavailable, re-checked at next login); rejected if it contains the username or email local part |
| Password hashing | Argon2id, memory 19 MiB, time cost 2, parallelism 1, 16-byte salt (PHC string); parameters re-evaluated yearly; rehash on login when parameters change |
| Login | Constant-time verification; a dummy hash is verified when the email is unknown; generic error message |
| Brute force | Rate limit 10 / 15 min per IP+email; lockout after 10 failures per email for 15 min, with an email notice; Cloudflare WAF rate rules as an outer layer |
| Email verification | 32-byte random token (base64url), SHA-256 stored, 24 h expiry, single use |
| Password reset | 32-byte random token, SHA-256 stored, 60 min expiry, single use; all sessions revoked on success; confirmation email; response never reveals account existence |
| OAuth (Should) | Google OIDC with PKCE (S256), `state` and `nonce`; ID token signature and audience validated; email must be `email_verified`; linking requires password confirmation |
| MFA (Future, F-39) | TOTP (RFC 6238) + 10 single-use recovery codes; mandatory for staff roles once shipped |
| Account enumeration | Registration, forgot-password and resend endpoints return identical responses; timing equalized by doing email-sending work in the queue |

## 4. Sessions and tokens

| Item | Value |
|---|---|
| Session token | 32 bytes from `crypto.randomBytes`, base64url; never stored server-side in raw form |
| Storage | Redis `sess:{sha256(token)}` (authoritative for requests) + PostgreSQL `sessions` (listing, audit, revocation) |
| Cookie | `lo_sid`; `HttpOnly; Secure; SameSite=Lax; Path=/; Domain=.linkedout.app`; `Max-Age` 30 days if "remember device" else a session cookie (server idle limit still applies) |
| Expiry | Idle 14 days (sliding), absolute 30 days |
| Rotation | New session ID on login, password change, privilege change; old session deleted |
| Revocation | Logout, "sign out other sessions", password reset, suspension, deletion request, role change |
| CSRF | Double-submit token: `lo_csrf` cookie (`Secure; SameSite=Lax`, readable by JS) + `X-CSRF-Token` header must match; token HMAC-bound to session ID when logged in; plus `SameSite=Lax` and strict CORS origin allow-list |
| Other tokens | Email verification (24 h), password reset (60 min), representative email (24 h), export download links (pre-signed, 7 days), media upload URLs (pre-signed PUT, 10 min) |
| JWTs | Not used |

## 5. Input validation and output encoding

| Area | Control |
|---|---|
| Request validation | Zod schemas (shared package) on every body, query and param; unknown keys stripped; body size limit 256 KB; strict content-type checks |
| Database | Prisma parameterized queries only; `$queryRaw` allowed only with tagged-template parameters (lint rule bans `$queryRawUnsafe`) |
| Free text | Treated as untrusted (plan §11): stored as Markdown/plain text, rendered via `react-markdown` + `rehype-sanitize` allow-list; no raw HTML; links get `rel="nofollow ugc noopener noreferrer"` and `target="_blank"` |
| Search highlights | `ts_headline` output is HTML-escaped server-side except `<mark>` tags, inserted from fixed delimiters |
| URLs in content | Only `https:` (and `http:` displayed as text, not links); max 3 links for accounts < 7 days old; no link previews/unfurling in MVP (avoids SSRF) |
| File uploads (Should) | Allowed MIME types `image/jpeg`, `image/png`, `image/webp`, `image/gif`; ≤ 5 MB; magic-byte validation; re-encode with sharp (strips EXIF, GPS, embedded payloads); ClamAV scan; random storage keys; served from a separate domain `files.linkedout.app` with `Content-Disposition: inline`, `X-Content-Type-Options: nosniff`; SVG rejected |
| Emails | React Email templates escape all interpolations; anonymous labels never in subjects |
| Safety checks | See §8 |

## 6. Encryption

| Data | In transit | At rest |
|---|---|---|
| Browser ↔ Cloudflare ↔ Vercel / Render | TLS 1.2+ (TLS 1.3 preferred); HSTS `max-age=63072000; includeSubDomains; preload` | — |
| API ↔ PostgreSQL / Redis | TLS within Render private network (`sslmode=require`) | Render managed disk encryption (AES-256) |
| Object storage (R2) | TLS | Encrypted at rest by Cloudflare (AES-256) |
| Backups | TLS | Encrypted by provider; offsite copies in R2 additionally encrypted with `age` using the backup public key (private key offline with two founders) |
| Sensitive notification references | — | AES-256-GCM application-level encryption with `VAULT_ENCRYPTION_KEY` (rotated yearly; key ID prefix in ciphertext) |
| Passwords | — | Argon2id hashes (not encryption) |
| Session/verification tokens | — | SHA-256 hashes |

## 7. Rate limiting and abuse prevention

The rate limits are defined in [API_SPEC.md §1.7](API_SPEC.md#17-rate-limits). The layers are:

1. **Cloudflare WAF:** managed rules, bot fight mode on auth routes, a 600 req/min per IP ceiling.
2. **API throttler:** Redis-backed per-route limits.
3. **Domain rules:** account-age gates (reviews ≥ 7 days, communities ≥ 14 days, link limits < 7 days, 1 confession/day < 24 h); a verified email for all writes; one review per company; campaign detection (FR-48).
4. **Spam signals:** duplicate content hashes across posts within 24 h, so the 3rd identical post is held.
5. **Beta controls:** invite-only registration flag during the closed beta.

If Redis is unavailable, authenticated writes fail closed (`503`). Rate limits are never bypassed.

## 8. Anonymous publishing safeguards (plan §14)

| Safeguard | Implementation |
|---|---|
| Separate pseudonyms from account identity | `users` (private) vs `profiles` (public); public DTOs never include `users` fields or internal IDs (contract tests SEC-04). |
| Per-confession anonymous labels | New label per confession, job report and review; labels of the same author are not linkable through the API. |
| Restricted identity mappings | `vault` schema; only role `lo_vault`; `IdentityVaultService` exposes narrow methods; the app role has no vault privileges (DB-level test SEC-11). |
| Audited staff access | Every vault call except ownership checks writes an audit entry; reveals need four-eyes approval and a justification, and expire after 24 h. |
| No internal identifiers exposed | Only UUIDs of content; members identified publicly by username; anonymous content has no author reference at all. |
| Warnings against identifying details | SafetyCheckPanel + tips; preview step; named-individual heuristics. |
| No absolute anonymity claims | Copy rules (UI_UX C.7); onboarding step 3; Privacy Policy section on lawful disclosure. |
| Timing and side channels | Anonymous content is excluded from profile pages, counts and activity; notification references are encrypted; blocking an alias never reveals or hides pseudonymous content; feeds don't expose "posted by someone you follow" signals for anonymous content. |
| Logs and telemetry | Redaction (ARCHITECTURE §4.4); no user IDs in spans for anonymous flows; Sentry `beforeSend` strips request bodies and cookies. |
| Company representatives | Never receive author data (US-069; SEC-05). |
| Enforcement against anonymous authors | `resolveForEnforcement` applies restrictions inside the service; moderators see only the alias; appeals for anonymous content store the alias, not the user ID. |

### 8.1 Automated safety check rules

| Rule ID | Detects | Severity | Blocking |
|---|---|---|---|
| `secrets.credential` | API keys and tokens (AWS `AKIA…`, GitHub `ghp_…`/`github_pat_…`, Slack `xox…`, Stripe `sk_live_…`, private key headers, JWT-shaped strings, high-entropy strings ≥ 32 chars) | high | Yes |
| `pii.email_address` | Email addresses (except the author's own, if public) | medium | No |
| `pii.phone_number` | Phone numbers (libphonenumber, any region) | medium | No |
| `pii.government_id` | SSN-, Aadhaar-, PAN- and passport-shaped numbers | high | No (held) |
| `pii.payment_card` | Luhn-valid 13–19 digit numbers | high | Yes |
| `pii.street_address` | Street-address patterns (number + street-type word) | medium | No |
| `pii.social_profile` | Links to `linkedin.com/in/`, `x.com/`, `twitter.com/`, `instagram.com/`, `facebook.com/` profiles and `@handle` mentions of non-members | medium | No |
| `named_individual` | Capitalized first+last name sequences not matching company/category names, or role words followed by a capitalized name ("my manager Priya") | medium | No (held in confessions, reviews, job reports) |
| `threat_terms` | Curated threat/incitement phrase list (en) | high | No (held) |
| `harassment_terms` | Slurs and harassment list (en) | high | No (held) |
| `confidential_markers` | "confidential", "internal only", "NDA", "do not distribute" near document-like text | medium | No |
| `interview_dump` | ≥ 5 question lines or phrases like "the questions were" in job reports | medium | No |
| `velocity.new_account` | Account < 24 h creating its 2nd+ post, or > 3 links | low | No |
| `duplicate_content` | Same content hash posted ≥ 3 times in 24 h | medium | No (held) |

Risk level = max severity. **High** → held. **Medium** → confirmation required, and held for confessions, reviews, job reports and company responses. **Low** → warnings only. Rule lists are versioned in `apps/api/src/modules/safety/rules/` and reviewed monthly by the T&S lead.

## 9. Privacy and data retention

GDPR is the baseline for all users regardless of location (see [ASSUMPTIONS.md](ASSUMPTIONS.md#7-privacy-and-legal)).

### 9.1 Data inventory and lawful basis

| Data | Purpose | Lawful basis | Visible publicly |
|---|---|---|---|
| Email | Login, security, required notices | Contract | Never |
| Password hash | Authentication | Contract | Never |
| Username, profile fields | Public pseudonymous identity | Contract | Per visibility setting |
| Posts, comments, reviews, reports | Core service | Contract | Per visibility |
| Anonymous alias mapping | Ownership, abuse prevention, lawful requests | Legitimate interest (safety) | Never |
| IP address, user agent | Security, abuse prevention, session list | Legitimate interest | Never |
| Approximate session location | Session list for the member | Legitimate interest | Never |
| Reactions, saves, poll votes | Core service | Contract | Reaction counts only; saves and votes never |
| Moderation records | Safety, appeals, legal defense | Legitimate interest / legal obligation | Never (outcome messages to the affected parties only) |
| Audit logs | Accountability | Legitimate interest | Never |
| Representative work email domain | Verification | Consent | Never |

No precise location, phone number, legal name, date of birth, or third-party trackers are collected (plan §6.1).

### 9.2 Retention schedule

| Data | Retention |
|---|---|
| Active account data | While the account exists |
| Soft-deleted content | Purged 30 days after deletion (unless legal hold) |
| Confession drafts | 180 days of inactivity (7-day warning) |
| Session records | IP nulled after 90 days; session rows deleted 90 days after expiry/revocation |
| Auth tokens | 7 days after expiry |
| Notifications | 180 days |
| Moderation items, actions, appeals, reports | 2 years after resolution |
| Audit logs | 2 years |
| Username history | 2 years |
| Data exports | Download link and file deleted after 7 days |
| Backups | PITR 7 days; daily logical backups 30 days; monthly backups 12 months (deleted accounts disappear from backups when they age out; restores re-apply deletions from the `account_deletion_requests` log) |
| Legal holds | Until released by counsel |

### 9.3 Account deletion procedure (FR-18)

1. The request sets `deletion_pending`, revokes sessions, and hides the profile and all content immediately.
2. Grace period: 30 days with cancellation.
3. The deletion job then:
   - hard-deletes posts, comments, reactions, saves, votes, media files, follows, blocks, mutes, the profile, privacy and notification settings, sessions, tokens, OAuth links, and vault aliases (with their anonymous content);
   - replaces the email with a tombstone;
   - nulls `reporter_user_id` on reports;
   - keeps moderation actions and audit entries for 2 years, referencing only the tombstone user ID.
4. Content under an active escalation or legal hold is retained, hidden, until the hold is released.
5. A confirmation email is sent before the email address is destroyed.

### 9.4 Data subject rights

| Right | How |
|---|---|
| Access & portability | Data export (F-09, Should); before it ships, `privacy@linkedout.app` handles requests manually within 30 days |
| Rectification | Profile settings; corrections on posts |
| Erasure | Account deletion (F-08); per-item delete |
| Objection / restriction | `privacy@linkedout.app` |
| Transparency | Privacy Policy, onboarding, semi-annual transparency report (moderation and legal-request counts) |

## 10. Legal requests

1. All requests go to `legal@linkedout.app` and are logged in a restricted tracker.
2. The founder acting as legal contact verifies the requester and the legal basis with counsel. Informal requests (an employer asking "who wrote this") are refused.
3. For a valid order, two T&S officers execute the identity reveal workflow (FR-51). Only the minimum data specified is produced, and the email is retrieved by an admin via an audited database query.
4. The affected member is notified unless legally prohibited.
5. Each request is counted in the transparency report.
6. Defamation complaints follow the moderation path: the content is reviewed against the guidelines, the right of reply is offered, and the content may be limited pending escalation. Content is never removed solely because it is negative (plan §7).

## 11. Moderation policy codes

Moderation actions cite these Community Guidelines (CG) sections in `policy_reference`.

| Code | Rule (plan §14 prohibited list) |
|---|---|
| CG-1.1 | Threats and incitement to violence |
| CG-1.2 | Targeted harassment and bullying |
| CG-1.3 | Hate-based abuse and discrimination |
| CG-2.1 | Private personal data (doxxing, contact details, addresses, IDs) |
| CG-2.2 | Non-consensual intimate material |
| CG-3.1 | Confidential company or customer information |
| CG-3.2 | Confidential interview materials and proprietary assessments |
| CG-4.1 | Impersonation (including posing as a company representative) |
| CG-4.2 | Fabricated evidence and knowingly false allegations presented as fact |
| CG-4.3 | Coordinated review manipulation and brigading |
| CG-5.1 | Spam and commercial solicitation |
| CG-5.2 | Illegal content |
| CG-6.1 | Naming private individuals in allegations without necessity (stricter review; edits requested first) |
| CG-6.2 | Graphic content without content warnings |

## 12. Secrets management

| Rule | Implementation |
|---|---|
| No secrets in source control | `.env` git-ignored; gitleaks pre-commit + CI; GitHub secret scanning push protection |
| Storage | Production/staging secrets in Render environment groups and Vercel environment variables (encrypted); CI secrets in GitHub Actions environments with required reviewers for `production` |
| Separation | Distinct credentials per environment; `VAULT_DATABASE_URL` only on api and worker services; backup bucket credentials only in the backup job |
| Validation | `env.schema.ts` validates at boot; the process exits on missing/invalid values |
| Rotation | Every 12 months and immediately on staff offboarding or suspected exposure: DB passwords, `SESSION_SECRET`/`CSRF_SECRET`, `VAULT_ENCRYPTION_KEY` (with key ID for decrypt-old/encrypt-new), API keys (Resend, Sentry, R2) |
| Access | Two founders hold production secrets access; access reviewed quarterly |

## 13. Security headers

| Header | API | Web |
|---|---|---|
| `Strict-Transport-Security` | ✅ | ✅ |
| `Content-Security-Policy` | `default-src 'none'; frame-ancestors 'none'` | `default-src 'self'; script-src 'self' 'nonce-{n}' 'strict-dynamic'; style-src 'self' 'unsafe-inline'; img-src 'self' https://files.linkedout.app data:; connect-src 'self' https://api.linkedout.app https://*.ingest.sentry.io; frame-ancestors 'none'; base-uri 'self'; form-action 'self'` |
| `X-Content-Type-Options: nosniff` | ✅ | ✅ |
| `Referrer-Policy: strict-origin-when-cross-origin` | ✅ | ✅ |
| `Permissions-Policy: camera=(), microphone=(), geolocation=(), interest-cohort=()` | ✅ | ✅ |
| `Cross-Origin-Opener-Policy: same-origin` | ✅ | ✅ |
| CORS | Allow-list per environment, credentials true, methods GET/POST/PUT/PATCH/DELETE | n/a |

## 14. OWASP Top 10 (2025) coverage

| # | Category | LinkedOut controls | Tests |
|---|---|---|---|
| A01 | Broken Access Control | Deny-by-default guards, ownership checks in services, 404 for invisible resources, vault isolation, community/representative scoping, no client-supplied identity trusted | SEC-01, SEC-02, SEC-03, SEC-05, SEC-13 |
| A02 | Security Misconfiguration | Helmet/CSP headers, env validation, no debug endpoints in prod (Swagger disabled), least-privilege DB roles, IaC (`render.yaml`) reviewed in PRs | SEC-09, ZAP baseline |
| A03 | Software Supply Chain Failures | pnpm lockfile, Renovate, Dependabot alerts, `pnpm audit` gate, Trivy image scans, pinned GitHub Action SHAs, provenance for images, minimal base images | CI security workflow |
| A04 | Cryptographic Failures | TLS everywhere, HSTS preload, Argon2id, hashed tokens, AES-256-GCM for sensitive references, provider encryption at rest | SEC-10 |
| A05 | Injection | Prisma parameterization, lint ban on unsafe raw SQL, Zod validation, Markdown sanitization, escaped search highlights, no shell execution with user input | SEC-06, SEC-07 |
| A06 | Insecure Design | Threat model (§1), privacy-by-design vault, abuse-case stories, four-eyes reveals, rate limits designed per route, launch gate for confessions | Design reviews per phase |
| A07 | Authentication Failures | Breached-password check, lockout, generic errors, session rotation, secure cookies, reset token hygiene, MFA later | SEC-08, SEC-14 |
| A08 | Software or Data Integrity Failures | Signed commits encouraged, protected `main`, required reviews, CI-only deployments, migrations reviewed, append-only audit logs | Branch protection, SEC-15 |
| A09 | Security Logging and Alerting Failures | Audit logs for sensitive actions, structured logs with request IDs, alerts on auth anomalies (login failure spikes), vault access alerts, without logging PII | SEC-12, alert rules |
| A10 | Mishandling of Exceptional Conditions | Global exception filter (no stack traces), fail-closed on Redis/vault outages, transactional writes with compensation for vault steps, idempotency keys | SEC-16, integration tests |

Also covered: SSRF (no server-side URL fetching from user input in MVP; presigned uploads go direct to R2) and clickjacking (`frame-ancestors 'none'`).

## 15. Security operations

| Activity | Cadence |
|---|---|
| Dependency updates (Renovate) | Weekly; critical CVEs within 48 h |
| SAST (CodeQL), secret scan, Trivy | Every PR + nightly |
| DAST (OWASP ZAP baseline vs staging) | Nightly; full scan before beta and before launch |
| Access review (prod secrets, admin roles, vault permissions) | Quarterly |
| Restore test | Monthly (DEPLOYMENT §8) |
| External security review / pentest | Before public launch (week 15); then yearly |
| Incident response | Runbooks in DEPLOYMENT §11; breach notification to the supervisory authority within 72 h where required, and to affected members without undue delay |
| Vulnerability disclosure | `security@linkedout.app` and `/.well-known/security.txt`; acknowledgement within 3 business days |
