# LinkedOut — Product Requirements Document (PRD)

| Field | Value |
|---|---|
| Product | LinkedOut — *Your career. Your chaos.* |
| Document version | 1.0 |
| Source of truth | [LinkedOut_Detailed_Product_and_Engineering_Plan.md](LinkedOut_Detailed_Product_and_Engineering_Plan.md) |
| Date | 27 November 2026 |
| Related | [FEATURES.md](FEATURES.md) · [USER_STORIES.md](USER_STORIES.md) · [ROADMAP.md](ROADMAP.md) · [ASSUMPTIONS.md](ASSUMPTIONS.md) |

---

## 1. Overview

LinkedOut is an anonymous-first professional community and workplace transparency platform. Members share the parts of working life that polished professional networks hide: failed interviews, rejections, layoffs, toxic management patterns, unpaid overtime, burnout, mistakes, and the lessons learned from them. Humor and candid storytelling lower the barrier to talking about hard experiences. Privacy, evidence labelling and human moderation keep the platform from becoming a rumor mill, harassment forum or public-shaming tool.

**Product promise:** *A place to share the professional reality behind the polished profile — honestly, safely, and with a sense of humor.*

**Guiding principle:** *LinkedIn is where people showcase their professional identity. LinkedOut is where people discuss the professional experiences behind it.*

### 1.1 Core principles

| # | Principle | What it means for requirements |
|---|---|---|
| 1 | Honesty over personal branding | No vanity metrics, no "success" badges; setbacks are first-class content. |
| 2 | Support over ridicule | Reactions and copy are supportive; harassment is prohibited and moderated. |
| 3 | Experience over rumor | Workplace content is labelled *Unverified firsthand account* unless verified; allegations are never shown as facts. |
| 4 | Privacy by design | Public pseudonym is separate from the private account; we never promise absolute anonymity. |
| 5 | Useful outcomes | Posts declare a desired response (support, advice, discussion, just sharing); lessons are structured. |
| 6 | Fairness | Companies get a right of reply; members get appeals; corrections are supported. |

---

## 2. Problem Statement

1. **Performative success culture.** Mainstream professional networks reward highlight reels. People facing rejection, layoffs or a bad first job see only success stories and feel isolated.
2. **No safe place to be candid.** Posting honestly under a real-name professional identity carries career risk. People stay silent or vent in unmoderated spaces where identity leaks and harassment are common.
3. **Opaque hiring and workplaces.** Candidates cannot easily learn whether a company ghosts applicants, sets unpaid test assignments, or changes offers. Existing review sites focus on star ratings and offer little structured, contextual, firsthand information.
4. **Lessons are lost.** Hard-won career lessons live in private chats and are not organized so others can learn from them.
5. **Employers lack a fair channel.** Organizations have no structured, moderated way to respond to recurring feedback without being able to suppress it.

**LinkedOut's solution:** a pseudonymous community with structured story formats (stories, advice requests, lessons, confessions, workplace reviews, job nightmare reports). It adds pre-publication privacy checks, clear verification labels, minimum-sample rules for company aggregates, a right of reply for verified company representatives, and an auditable moderation and appeals process.

---

## 3. Goals and Non-Goals

### 3.1 Goals (MVP, first public release)

| ID | Goal | Measured by (see §8) |
|---|---|---|
| G-1 | Let people share career setbacks without exposing their legal identity. | Zero anonymous-identity exposure incidents; trust survey ≥ 4.0/5 on "I feel safe posting". |
| G-2 | Turn venting into learning and support. | ≥ 60% of advice requests receive ≥ 1 comment within 24 h. |
| G-3 | Give candidates structured, contextual company and interview information. | ≥ 150 active company pages with ≥ 5 reviews within 90 days of launch. |
| G-4 | Keep the community safe and fair. | Median time-to-first-moderator-action on high-priority reports ≤ 4 h; appeal reversal rate tracked and reported monthly. |
| G-5 | Ship a maintainable product with a small team. | MVP launched within 16 weeks as a modular monolith with ≥ 80% backend unit coverage on domain services. |

### 3.2 Non-Goals

| ID | Non-goal | Reason |
|---|---|---|
| NG-1 | Guarantee that anonymous posts are untraceable. | Legally and technically dishonest (plan §6.5, §14). |
| NG-2 | Determine the truth of allegations or publish "truth scores". | Explicitly out of scope (plan §17). |
| NG-3 | Public leaderboards of failures, rejections, hours or suffering. | Shames people and rewards extreme disclosure (plan §6.2, §6.13). |
| NG-4 | Microservices, ML feed ranking, real-time chat or video streaming in MVP. | Premature complexity (plan §17). |
| NG-5 | Let companies pay to remove or suppress criticism. | Monetization boundary (plan §16). |
| NG-6 | Provide legal, medical, mental-health or employment advice. | Platform is peer support only; disclaimers shown (plan §2). |
| NG-7 | Native mobile apps in MVP. | Responsive web first (plan §17 "Later"). |

---

## 4. Personas

All personas are fictional composites.

| ID | Persona | Segment | Context | Needs | Frustrations | Primary features |
|---|---|---|---|---|---|---|
| P-1 | **Priya, 22** | Student / fresh graduate | Final-year computer-science student; 40 applications, 3 interviews, 0 offers. | Know rejection is normal; get concrete interview advice. | Peers only post offer letters. | The Mess, Advice posts, Learned the Hard Way, Job Nightmare Board |
| P-2 | **Marcus, 27** | Job seeker | Laid off after 3 years; ghosted twice after final rounds. | Research companies' hiring behavior before investing time. | Glossy career pages; star ratings with no context. | Workplace Reality Pages, Job Nightmare Board, Search |
| P-3 | **Aisha, 24** | Early-career developer | First job at a startup with unrealistic deadlines and unpaid weekend work. | Share what is happening safely; learn whether it is normal. | Fear of being identified by their manager. | Office Confessions (anonymous to members), Communities |
| P-4 | **Daniel, 38** | Experienced employee | Team lead recovering from burnout. | Help others avoid the same mistakes; discuss patterns. | Professional networks punish vulnerability. | Learned the Hard Way, Anti-Profile, Communities |
| P-5 | **Sofia, 31** | Career changer | Moving from teaching to UX design. | Learn from people who changed paths. | Fear of starting over. | Communities, Lessons, Search |
| P-6 | **Ken, 45** | Company representative (HR) | Employer-brand lead at a mid-size company that appears in reviews. | Respond publicly and constructively to recurring feedback. | Cannot reply on other platforms; sees reviews as unfair. | Company Representative Verification & Responses |
| P-7 | **Lena, 29** | Platform moderator / T&S lead | Part-time trust and safety lead in the founding team. | A clear, prioritized queue, policy guidance and audit trail. | Unstructured reports; no context. | Moderation Queue, Appeals, Audit Log, Identity Vault |

---

## 5. Scope

Priority labels: **MVP** = required for first public release; **Should** = shortly after launch (v1.1); **Future** = later releases. Full feature definitions live in [FEATURES.md](FEATURES.md).

### 5.1 Feature scope summary

| Feature ID | Feature | Priority |
|---|---|---|
| F-01 | Accounts & Authentication | MVP |
| F-02 | OAuth Sign-in (Google) | Should |
| F-03 | Public Pseudonym & Identity Separation | MVP |
| F-04 | Onboarding & Privacy Explanation | MVP |
| F-05 | Anti-Profile | MVP |
| F-06 | Privacy Settings | MVP |
| F-07 | Block & Mute | MVP |
| F-08 | Account Deletion | MVP |
| F-09 | Data Export | Should |
| F-10 | Posts (stories, advice requests) with Categories, Tags & Content Warnings | MVP |
| F-11 | The Mess — Latest Feed & Filters | MVP |
| F-12 | Comments | MVP |
| F-13 | Reactions | MVP |
| F-14 | Saved Posts | MVP |
| F-15 | Office Confessions | MVP |
| F-16 | Learned the Hard Way | MVP |
| F-17 | Workplace Reality Pages (company directory & workplace reviews) | MVP |
| F-18 | Job Nightmare Board | MVP |
| F-19 | Search | MVP |
| F-20 | Content Reporting | MVP |
| F-21 | Automated Safety Checks & Moderation Queue | MVP |
| F-22 | Appeals | MVP |
| F-23 | Audit Logging & Identity Vault | MVP |
| F-24 | Admin Dashboard (basic) | MVP |
| F-25 | Company Representative Verification & Responses | Should |
| F-26 | Communities (Trauma Network) | Should |
| F-27 | Notifications (in-app & email) | Should |
| F-28 | Polls | Should |
| F-29 | Image & Meme Posts (media upload) | Should |
| F-30 | Topic Following & Following Feed | Should |
| F-31 | Direct Messaging (peer support conversations) | Future |
| F-32 | Career Damage Report | Future |
| F-33 | Corporate Meme Generator | Future |
| F-34 | Hall of Shame (pattern highlights) | Future |
| F-35 | Relevant Feed Ranking | Future |
| F-36 | AI-assisted Categorization | Future |
| F-37 | Premium Membership | Future |
| F-38 | Employer Analytics & Trend Reports | Future |
| F-39 | Multi-factor Authentication | Future |
| F-40 | Mobile Applications | Future |
| F-41 | Learning Paths | Future |
| F-42 | Push Notifications | Future |
| F-43 | Survivor Events & Meetups | Future |

### 5.2 Explicitly out of scope for MVP

Microservices; complex machine-learning feed ranking; real-time chat; video streaming; public failure leaderboards; unmoderated company accusation pages; automated truth scores for allegations; paid features; advertising.

---

## 6. Functional Requirements

Each requirement is traceable to a feature (F-ID), user stories ([USER_STORIES.md](USER_STORIES.md)) and tasks ([TASKS.md](TASKS.md)).

### 6.1 Identity, registration and privacy

| ID | Requirement | Feature | Priority |
|---|---|---|---|
| FR-1 | The system shall let a visitor register with email, password (min 12 chars, checked against a breached-password list), a unique public username, date-of-birth confirmation of age ≥ 16, and acceptance of the Terms, Privacy Policy and Community Guidelines. | F-01 | MVP |
| FR-2 | The system shall send a single-use email verification link valid for 24 h; unverified accounts can browse and save but cannot publish, comment, react or report until verified. | F-01 | MVP |
| FR-3 | The system shall let a member log in with email and password and log out, using server-side sessions stored in Redis with an httpOnly, Secure, SameSite=Lax cookie; idle timeout 14 days, absolute timeout 30 days. | F-01 | MVP |
| FR-4 | The system shall provide password reset via a single-use emailed link valid for 60 minutes. The response must not reveal whether an email is registered, and all other sessions are revoked on success. | F-01 | MVP |
| FR-5 | The system shall list a member's active sessions (device label, browser, approximate city-level location derived from IP at login, last active) and allow revoking one or all other sessions. | F-01 | MVP |
| FR-6 | The system shall rate-limit authentication endpoints and temporarily lock login for an email after 10 failed attempts in 15 minutes, without revealing whether the account exists. | F-01 | MVP |
| FR-7 | The system shall let a visitor sign up or log in with Google OAuth (OpenID Connect). OAuth sign-ups must still choose a username and accept policies; the Google display name and photo are never imported into the public profile. | F-02 | Should |
| FR-8 | The system shall require a unique public username (3–30 chars, `[a-z0-9_]`, case-insensitive, not on the reserved list) and never derive it from the email or legal name. | F-03 | MVP |
| FR-9 | The system shall let a member change their username at most once every 30 days. The old username is reserved for 90 days and cannot be claimed by another member, and username history is retained for moderation. | F-03 | MVP |
| FR-10 | Public API responses shall never contain private account fields (email, legal name, phone, IP, internal user ID, session data, identity-mapping data); enforced by response serializers and automated contract tests. | F-03 | MVP |
| FR-11 | The system shall show a mandatory onboarding flow after first verification. It explains pseudonymity vs anonymity, what data is collected and why, what may be disclosed under lawful request, and the Community Guidelines, and requires explicit acknowledgement before first publish. | F-04 | MVP |
| FR-12 | The system shall let a member edit their Anti-Profile: avatar (Should, depends on F-29), headline (≤ 120 chars), bio (≤ 600 chars), industry, broad location (country or region only), and career status (`exploring`, `studying`, `employed`, `open_to_escape`, `between_jobs`, `career_changing`, `prefer_not_to_say`). | F-05 | MVP |
| FR-13 | The system shall let a member add up to 10 career lessons (≤ 280 chars each), up to 20 skills being learned, and an optional experience timeline (up to 20 entries: period, role category, industry, free-text note ≤ 280 chars; no employer names required). | F-05 | MVP |
| FR-14 | The system shall show a public Anti-Profile with the username, profile fields, pseudonymous posts and comments, and community memberships (Should), according to the member's visibility setting. Anonymous confessions, saved posts and drafts are never shown, and no hardship counts or rankings are displayed. | F-05 | MVP |
| FR-15 | The system shall let a member set profile visibility (`public`, `members`, `hidden`), discoverability in member search (on/off), default identity mode for confessions, and default post visibility. Message permissions are stored for F-31. | F-06 | MVP |
| FR-16 | The system shall let a member block another member, or an anonymous label, from a profile, post or comment. Blocked parties cannot comment on or react to the blocker's content, and the blocker no longer sees their content. Blocking an anonymous label hides only content under that label and never reveals the underlying account. | F-07 | MVP |
| FR-17 | The system shall let a member mute a member, a category or a tag. Muted content is excluded from feeds and search for that member, and the muted party is not notified. | F-07 | MVP |
| FR-18 | The system shall let a member request account deletion with re-authentication. The request is cancellable for 30 days, after which the job removes or irreversibly anonymizes personal data and content. The member is shown exactly what is retained (moderation records, audit logs, legal holds) and for how long. | F-08 | MVP |
| FR-19 | The system shall let a member request a machine-readable export (JSON in a ZIP) of their account data, profile, content (including their own anonymous confessions), reactions, saves and settings, delivered by a 7-day download link. | F-09 | Should |

### 6.2 Content and feed

| ID | Requirement | Feature | Priority |
|---|---|---|---|
| FR-20 | The system shall let a verified member create a post of type `story` or `advice` with a title (≤ 150 chars), body (Markdown subset, ≤ 20,000 chars; > 3,000 chars renders as long-form with "Read more"), one category, up to 5 tags, a desired response (`support`, `advice`, `discussion`, `just_sharing`), language, and visibility (`public`, `members`). | F-10 | MVP |
| FR-21 | The system shall let an author edit (showing an "edited" marker and keeping revisions for moderation) or delete their own posts. Deleted posts are soft-deleted, show "[deleted]" in threads, and are purged after 30 days unless under legal hold. | F-10 | MVP |
| FR-22 | The system shall provide 12 seeded categories (Career Failures, Interview Disasters, Office Drama, Management Problems, Salary and Payment Issues, Unpaid Overtime, Internship Reality, Developer Problems, Startup Lessons, Layoffs and Job Loss, Career Changes, Workplace Wins and Recovery) and free-form, normalized tags. | F-10 | MVP |
| FR-23 | The system shall let authors add content warnings from a fixed list (`burnout`, `harassment`, `discrimination`, `job_loss`, `mental_health`, `strong_language`, `other`). Warned content is collapsed behind a reveal control in all listings. | F-10 | MVP |
| FR-24 | The system shall show The Mess, a reverse-chronological Latest feed with cursor pagination (default 20 items, max 50) of published posts the viewer may see. It excludes held/removed content and content from blocked and muted members or labels. | F-11 | MVP |
| FR-25 | The system shall let viewers filter the feed by category, post type and language, and hide categories persistently (members) or per session (visitors). Feed filter choices are reflected in the URL. | F-11 | MVP |
| FR-26 | The system shall let verified members comment on posts and reply to comments (threaded, max depth 3 levels rendered; deeper replies are flattened to depth 3) with bodies ≤ 5,000 chars. | F-12 | MVP |
| FR-27 | The system shall let comment authors edit and delete their own comments. Deleted comments with replies show "[deleted]" placeholders. | F-12 | MVP |
| FR-28 | The system shall let a post author pin one comment of their own as an "Author clarification" shown above other comments. | F-12 | MVP |
| FR-29 | The system shall let verified members give one reaction per post from six types: I'm Dead, Pain, Same Here, Melting, Corporate Circus, Survived. They can change or remove it. Per-type counts are shown on the post only and are never aggregated onto profiles. | F-13 | MVP |
| FR-30 | The system shall let members save and unsave posts and list their saved posts privately. Saves are never visible to others. | F-14 | MVP |

### 6.3 Core differentiators

| ID | Requirement | Feature | Priority |
|---|---|---|---|
| FR-31 | The system shall let a verified member submit an Office Confession with title, story body, optional industry, optional broad region, experience type, optional approximate period (quarter/year), desired response, content warnings and identity display mode (`pseudonym` or `anonymous`), through a three-step flow: write → safety check & preview → confirm. | F-15 | MVP |
| FR-32 | For anonymous mode, the system shall generate a unique per-confession anonymous label (e.g. "Anonymous Stapler #4821"). The label-to-account mapping is stored only in the Identity Vault, and public tables hold no author user ID. When replying in their own confession thread, the author appears under the same label with an "OP" badge. | F-15 | MVP |
| FR-33 | The system shall let members save confessions as private drafts visible only to the author, editable and publishable later. Drafts are deleted after 180 days of inactivity, with a 7-day prior email notice. | F-15 | MVP |
| FR-34 | The system shall let the author of any post or confession append dated corrections or clarifications (≤ 1,000 chars each, max 5) that are displayed prominently and cannot be deleted once published, except by moderators. | F-15 | MVP |
| FR-35 | The system shall support `lesson` posts ("Learned the Hard Way") with structured sections: what I did wrong, what happened, what I learned, what I would do differently, advice for others, and up to 5 helpful links. A dedicated listing filters by category. | F-16 | MVP |
| FR-36 | The system shall provide a company directory and Workplace Reality Pages showing name, industry, confirmed official website, description, workplace reviews, interview/job reports, theme summaries, and company responses (Should). | F-17 | MVP |
| FR-37 | The system shall let verified members submit a missing company (name, industry, website). It enters `pending` status and becomes public only after admin approval, with duplicate detection by normalized name and website domain. | F-17 | MVP |
| FR-38 | The system shall let verified members (account age ≥ 7 days) submit at most one active workplace review per company. A review has relationship type, approximate period, topic ratings 1–5 (work-life balance, management, communication, compensation & payment, career growth), firsthand narrative, firsthand attestation, and consent to public display of the narrative. | F-17 | MVP |
| FR-39 | The system shall label all reviews and job reports "Unverified firsthand account". Topic theme summaries (median and distribution) are shown only when ≥ 5 approved reviews from the last 24 months exist; otherwise "Not enough reports yet" is shown. | F-17 | MVP |
| FR-40 | The system shall let viewers filter company content by type (workplace, interview), topic (salary, management, work-life balance, communication, career growth), relationship type and period, sorted by newest. | F-17 | MVP |
| FR-41 | The system shall let verified members submit Job Nightmare reports. A report has a company (from directory or submitted for review), role category, approximate month, interview stage reached, report type, narrative, optional outcome, identity mode and visibility. Narratives are screened for interview-question dumps and recruiter personal details. | F-18 | MVP |
| FR-42 | The system shall show the Job Nightmare Board listing reports filterable by report type, company, role category and period, and show them on the relevant company page. | F-18 | MVP |
| FR-43 | The system shall provide full-text search across posts, lessons, confessions, job reports and companies (PostgreSQL full-text + trigram), and communities once F-26 ships. Results respect visibility, moderation status, blocks and mutes; anonymous content is never searchable by author. | F-19 | MVP |

### 6.4 Trust, safety and administration

| ID | Requirement | Feature | Priority |
|---|---|---|---|
| FR-44 | The system shall let verified members report posts, comments, profiles, reviews, job reports, company responses and communities. A report has one of 9 reasons (personal information, harassment or threats, false or manipulated content, confidential information, spam, impersonation, discrimination or hate, illegal content, other policy violation) and optional details (≤ 1,000 chars). Members can track the status of their reports. | F-20 | MVP |
| FR-45 | The system shall run automated safety checks on every submission: PII and secret detection, named-individual heuristics, threat/harassment term lists, and new-account velocity rules. Each check yields a risk level: `none`/`low` (publish, with inline warnings shown pre-publish), `medium` (author must confirm after editing suggestions; confessions/reviews/job reports are held), or `high` (held for human review). | F-21 | MVP |
| FR-46 | The system shall provide a moderation queue that merges member reports and automated holds into one moderation item per target. Items are prioritized by severity (threats, doxxing first), report count and age, and filterable by status, reason and content type. | F-21 | MVP |
| FR-47 | The system shall let platform moderators apply proportionate actions: no action, request edits, add content warning, limit visibility, remove content, temporarily restrict posting (1/7/30 days), suspend account, or escalate for specialist/legal review. A reason and policy reference are required, and the affected member and reporters are notified by email with an enforcement explanation. | F-21 | MVP |
| FR-48 | The system shall detect likely coordinated review campaigns: > 5 reviews for one company within 24 h, or > 3 from accounts younger than 14 days within 7 days, or > 80% textual similarity between reviews. Matching reviews are flagged into the queue, and theme summaries are frozen until review. | F-21 | MVP |
| FR-49 | The system shall let an affected member appeal a moderation action once, within 30 days, with a reason (≤ 2,000 chars). A different moderator than the original decides (`upheld`, `reversed`, `modified`), and the member is notified with an explanation. | F-22 | MVP |
| FR-50 | The system shall write immutable audit logs for authentication events, role changes, moderation actions, appeal decisions, admin changes and every Identity Vault access. Logs are retained for 2 years and readable only by administrators. | F-23 | MVP |
| FR-51 | The system shall restrict anonymous-identity resolution to members holding the `identity_vault.reveal` permission. A reveal requires a written justification and a second staff approval (four-eyes), and every reveal is audit-logged. Revealed identity is shown only to the requester for 24 h. | F-23 | MVP |
| FR-52 | The system shall let administrators search users by email or username, view account status, assign platform roles (`member`, `platform_moderator`, `admin`), and grant or revoke the `identity_vault.reveal` permission. Admins cannot read anonymous mappings without FR-51. | F-24 | MVP |
| FR-53 | The system shall let administrators create, rename, reorder and archive categories. | F-24 | MVP |
| FR-54 | The system shall let administrators approve, edit, reject, merge and suspend company pages. | F-24 | MVP |
| FR-55 | The system shall show administrators platform metrics (sign-ups, verified members, posts per day, reports per 1,000 posts, queue size, median moderation response time, appeal reversal rate). | F-24 | MVP |

### 6.5 Should-have (v1.1)

| ID | Requirement | Feature | Priority |
|---|---|---|---|
| FR-56 | The system shall let a member request association with a company page as a representative, by verifying an email address on the company's confirmed website domain and giving a job title. | F-25 | Should |
| FR-57 | The system shall let administrators approve or reject representative requests. Approved representatives get an "Official representative" badge on responses, and memberships expire after 12 months unless renewed. | F-25 | Should |
| FR-58 | The system shall let verified representatives post one public response per review or job report on their company (≤ 3,000 chars), subject to moderation. Representatives can never see author identity and cannot remove or hide content. | F-25 | Should |
| FR-59 | The system shall let verified members (account age ≥ 14 days) create communities with name, slug, description, visibility (`public`, `private`) and rules, and let anyone discover public communities. | F-26 | Should |
| FR-60 | The system shall let members join and leave public communities instantly, and request to join private communities (approved by community moderators). | F-26 | Should |
| FR-61 | The system shall show a community feed and let community members post into a community. Private community content is visible only to members and is excluded from public search. | F-26 | Should |
| FR-62 | The system shall let community owners appoint community moderators, who can edit rules, pin up to 5 resources, review a community-scoped report queue, and remove content from the community or ban members from it. Platform-wide actions remain with platform moderators. | F-26 | Should |
| FR-63 | The system shall let community owners create private communities whose member lists are visible only to members. | F-26 | Should |
| FR-64 | The system shall deliver in-app notifications for: comment on my post, reply to my comment, reactions to my post (batched hourly), community invitation, community announcement, response to my report, moderation decision, and followed-topic update (daily digest). | F-27 | Should |
| FR-65 | The system shall let members configure per-type in-app and email notification preferences. Moderation decisions are always delivered by email. | F-27 | Should |
| FR-66 | The system shall support quiet hours (start/end in the member's time zone) during which emails are deferred. Notification copy must follow the non-manipulative copy guidelines in [UI_UX.md](UI_UX.md). | F-27 | Should |
| FR-67 | The system shall let members create `poll` posts with 2–6 options and a duration of 1–14 days, and let verified members vote once. Results are shown after voting or closing, and individual votes are never shown. | F-28 | Should |
| FR-68 | The system shall let members attach up to 4 images (JPEG, PNG, WebP, GIF; ≤ 5 MB each) to posts and set an avatar. Uploads go via pre-signed URLs, EXIF is stripped, files are malware-scanned (ClamAV) and re-encoded, and images are held until scanned. | F-29 | Should |
| FR-69 | The system shall let members follow categories and tags. | F-30 | Should |
| FR-70 | The system shall provide a Following feed mode showing posts from followed categories, tags and joined communities. | F-30 | Should |

### 6.6 Future

| ID | Requirement | Feature | Priority |
|---|---|---|---|
| FR-71 | One-to-one peer support conversations with message requests, block/report, no attachments in v1, spam limits, safety notices and user-controlled permissions. Anonymous confession authors cannot be messaged. | F-31 | Future |
| FR-72 | Private Career Damage Report dashboard: applications tracked, interview stages completed, lessons saved, skills in development, career goals, private progress notes. Never public, no mental-health inference. | F-32 | Future |
| FR-73 | Meme generator with moderated templates, text editing, preview, export and optional sharing with automated PII/harassment checks. | F-33 | Future |
| FR-74 | Hall of Shame: curated, anonymized, moderated recurring workplace patterns; never names individuals. | F-34 | Future |
| FR-75 | Relevant feed mode using topic relevance, recency, meaningful replies, explicit interests, quality and diversity signals; never anger or controversy. | F-35 | Future |
| FR-76 | AI-assisted category and content-warning suggestions that the author confirms; AI never makes final moderation decisions. | F-36 | Future |
| FR-77 | Optional premium membership: advanced filters, saved-search alerts, private analytics, profile themes and Anti-CV card themes, ad-free experience. Safety and privacy features stay free. | F-37 | Future |
| FR-78 | Employer analytics and trend reports built from aggregated, k-anonymous (k ≥ 10) data; no individual content beyond what is public. | F-38 | Future |
| FR-79 | TOTP multi-factor authentication with recovery codes; mandatory for staff roles. | F-39 | Future |
| FR-80 | iOS and Android apps consuming the same `/api/v1`. | F-40 | Future |
| FR-81 | Curated learning paths (first job survival, interview preparation, workplace communication, salary negotiation, software engineering mistakes, career transitions). | F-41 | Future |
| FR-82 | Web and mobile push notifications respecting preferences and quiet hours. | F-42 | Future |
| FR-83 | Survivor events and moderated storytelling meetups (online first) with RSVP. | F-43 | Future |

---

## 7. Non-Functional Requirements

### 7.1 Performance

| ID | Requirement |
|---|---|
| NFR-P1 | API p95 latency ≤ 300 ms for reads and ≤ 500 ms for writes at 50 requests/second sustained (excluding email/background work). |
| NFR-P2 | Feed first page served in ≤ 150 ms p95 from cache for visitors (30 s Redis cache). |
| NFR-P3 | Web Core Web Vitals at p75 on mid-range mobile over 4G: LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1. |
| NFR-P4 | Search p95 ≤ 500 ms for datasets up to 1 million posts. |
| NFR-P5 | Initial JavaScript for the feed route ≤ 200 KB gzipped. |

### 7.2 Security and privacy

| ID | Requirement |
|---|---|
| NFR-S1 | Passwords hashed with Argon2id (memory 19 MiB, iterations 2, parallelism 1 minimum, per OWASP). |
| NFR-S2 | TLS 1.2+ everywhere; HSTS with preload; encryption at rest for database, backups and object storage. |
| NFR-S3 | OWASP ASVS Level 2 controls for authentication, session management and access control; full coverage map in [SECURITY.md](SECURITY.md). |
| NFR-S4 | No private identity data in logs, analytics events, error reports or URLs; automated log-redaction tests. |
| NFR-S5 | Identity Vault stored in a separate PostgreSQL schema accessible only by a dedicated database role. |
| NFR-S6 | GDPR-aligned data rights (access, export, rectification, erasure, objection); retention schedule in [SECURITY.md](SECURITY.md). |
| NFR-S7 | Zero critical or high vulnerabilities open in dependencies at release. |

### 7.3 Scalability

| ID | Requirement |
|---|---|
| NFR-SC1 | MVP sized for 10,000 monthly active members, 500 concurrent users and 50 requests/second; architecture must scale horizontally (stateless API, sessions in Redis) to 100,000 MAU without redesign. |
| NFR-SC2 | Background jobs run in a separate worker process that scales independently. |
| NFR-SC3 | Scalability claims are made only after load tests in [TESTING.md](TESTING.md) pass against defined workloads. |

### 7.4 Accessibility

| ID | Requirement |
|---|---|
| NFR-A1 | WCAG 2.2 Level AA across all screens. |
| NFR-A2 | Full keyboard operability with visible focus indicators, and skip-to-content links. |
| NFR-A3 | Text contrast ≥ 4.5:1 (normal) and ≥ 3:1 (large text and UI components). |
| NFR-A4 | Screen-reader labels for every reaction, icon button and form control; live regions for async status. |
| NFR-A5 | Respect `prefers-reduced-motion`; no auto-playing media. |

### 7.5 Availability and reliability

| ID | Requirement |
|---|---|
| NFR-AV1 | 99.5% monthly availability for the web app and API (MVP), measured by external uptime checks. |
| NFR-AV2 | Recovery point objective (RPO) ≤ 1 hour using point-in-time recovery; recovery time objective (RTO) ≤ 4 hours. |
| NFR-AV3 | Backups restored to staging and verified monthly. |
| NFR-AV4 | Graceful degradation: if Redis is unavailable, the API returns 503 for authenticated writes rather than bypassing rate limits. |

### 7.6 Maintainability and observability

| ID | Requirement |
|---|---|
| NFR-M1 | Modular monolith with enforced module boundaries (lint rule on cross-module imports). |
| NFR-M2 | Backend domain services ≥ 80% line coverage; frontend components ≥ 70%. |
| NFR-M3 | Structured JSON logs with request IDs; OpenTelemetry traces for every request; error tracking with PII scrubbing. |
| NFR-M4 | All schema changes via versioned migrations, applied automatically in CI/CD. |

### 7.7 Internationalization

| ID | Requirement |
|---|---|
| NFR-I1 | UI in English (en) for MVP with all strings externalized for later translation. |
| NFR-I2 | Content may be in any language; authors tag the language (default `en`), and the feed language filter uses it. |

---

## 8. KPIs and Success Metrics

| Area | KPI | Definition | MVP target (90 days post-launch) |
|---|---|---|---|
| Activation | Registration completion rate | Verified accounts ÷ registration starts | ≥ 60% |
| Activation | Onboarding completion | Members completing onboarding ÷ verified | ≥ 85% |
| Activation | First story | Members who read ≥ 3 posts or create 1 within 7 days ÷ verified | ≥ 70% read, ≥ 25% create |
| Activation | Time to first meaningful interaction | Median time from verification to first comment, save or post | ≤ 24 h |
| Engagement | Meaningful comments per post | Comments ≥ 40 chars not removed, per published post | ≥ 1.5 |
| Engagement | Returning members | Week-4 retention of verified cohort | ≥ 25% |
| Engagement | Stories saved | Saves per weekly active member | ≥ 1.0 |
| Engagement | Advice answered | Advice posts with ≥ 1 comment within 24 h | ≥ 60% |
| Engagement | Community participation (v1.1) | Weekly active members who post or comment in a community | ≥ 20% |
| Trust & safety | Reports per 1,000 posts | — | Tracked; alert if > 50 |
| Trust & safety | Median moderation response time | Report created → first action | High priority ≤ 4 h; normal ≤ 24 h |
| Trust & safety | Appeal reversal rate | Reversed or modified ÷ decided appeals | Tracked; investigate if > 30% |
| Trust & safety | Repeat abuse rate | Members with ≥ 2 upheld violations in 90 days ÷ actioned members | ≤ 15% |
| Trust & safety | Privacy incidents | Confirmed anonymous-identity or PII exposure incidents | 0 |
| Trust & safety | Member trust survey | "I feel safe posting on LinkedOut" (1–5) | ≥ 4.0 |
| Product value | Helpful stories | Members answering "a story helped me this month" | ≥ 50% of surveyed |
| Product value | Useful company info | Company page visitors who rate the page useful | ≥ 60% |
| Product value | Company response timeliness (v1.1) | Median time to official response on verified pages | ≤ 7 days |
| Guardrail | Harmful-content exposure | Views of later-removed content ÷ total views | ≤ 0.5% |
| Guardrail | Harassment reports | Harassment reports per 1,000 active members | Tracked, downward trend |
| Guardrail | Moderation workload | Open items per moderator-hour | ≤ 15 |
| Guardrail | Negativity balance | Share of posts with desired response `advice` or type `lesson` | ≥ 30% |

---

## 9. Risks and Mitigations

| ID | Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|---|
| R-1 | Defamatory allegations | High | Legal & reputational | Guidelines discouraging naming individuals; named-individual heuristics hold content; "Unverified" labels; corrections; right of reply; legal escalation path. | T&S lead |
| R-2 | Doxxing | Medium | Personal safety | PII detection pre-publish; high-priority queue; immediate removal; restricted identity data. | T&S lead |
| R-3 | Fake company reviews | High | Loss of trust | Account-age gates, one review per company, campaign detection (FR-48), minimum sample rule (FR-39). | Backend |
| R-4 | Coordinated attacks | Medium | Manipulation | Rate limits, velocity rules, community rules against coordinated attacks, freeze of theme summaries. | Backend + T&S |
| R-5 | Excessive negativity | High | Retention & wellbeing | Desired-response field, lessons format, Workplace Wins category, supportive reactions, content warnings, mute controls. | Product |
| R-6 | Anonymous identity leakage | Low | Severe privacy harm | Identity Vault schema, no author ID on anonymous rows, serializer contract tests, four-eyes reveal, log redaction, security tests. | Backend |
| R-7 | Spam | High | Low quality | Email verification, rate limits, new-account limits, link limits (max 3 links for accounts < 7 days old). | Backend |
| R-8 | Legal requests | Medium | Compliance | Documented legal request process, counsel contact, transparency report, minimal data retention. | Product owner |
| R-9 | Moderation overload | Medium | Slow response | Prioritized queue, automated holds, beta size cap, moderator rota, alert at queue > 100. | T&S lead |
| R-10 | Premature complexity | Medium | Slow delivery | Modular monolith, staged delivery, strict MVP scope. | Product owner |
| R-11 | Brand/trademark conflict with "LinkedIn" | Medium | Forced rebrand | Trademark search and legal review before public launch (open question). | Product owner |
| R-12 | Small team bus factor | High | Delivery risk | Documentation, CLAUDE.md, pairing, explicit role assignment. | Product owner |

---

## 10. Constraints

1. **Team:** 2–3 people (student or early-stage team) covering product, design, frontend, backend, QA, trust & safety and DevOps; moderation and privacy ownership must be explicitly assigned.
2. **Timeline:** MVP in 16 weeks starting Monday 30 November 2026 (target public launch week of 15 March 2027).
3. **Budget:** managed hosting on free/low tiers during development; production monthly infrastructure budget ≤ USD 150 at MVP scale.
4. **Architecture:** single modular monolith (NestJS API + worker), single frontend (Next.js), one backend stack only.
5. **Compliance:** GDPR used as the privacy baseline for all users regardless of location; minimum age 16.
6. **Content:** English-language UI at launch; launch audience is students, graduates, job seekers and early-career tech workers.
7. **Policy:** anonymous posting must not be enabled in production before reporting, moderation, appeals and abuse-prevention features are complete (plan §26).

---

## 11. Release Criteria (MVP)

The MVP is releasable when every MVP feature in §5.1 meets its acceptance criteria in [USER_STORIES.md](USER_STORIES.md), the launch checklist in [ROADMAP.md](ROADMAP.md#4-mvp-launch-criteria) is complete, and the security tests in [TESTING.md](TESTING.md) pass with zero critical or high findings.
