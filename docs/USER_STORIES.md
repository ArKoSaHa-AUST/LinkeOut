# LinkedOut — User Stories

Stories are grouped by role (see roles in [SECURITY.md](SECURITY.md#2-roles-and-permissions)). Each story has an ID, priority (MVP / Should / Future), linked feature (F-ID, [FEATURES.md](FEATURES.md)) and requirements (FR-ID, [PRD.md](PRD.md)), and Given/When/Then acceptance criteria.

**Roles:** Visitor · Member · Community moderator · Platform moderator · Company representative · Administrator. "Verified member" means a member whose email is verified and who has completed onboarding.

## Index

| Role | Stories |
|---|---|
| Visitor | US-001 – US-005 |
| Member — account & privacy | US-006 – US-021 |
| Member — posts, feed & interaction | US-022 – US-032 |
| Member — confessions, lessons, companies, jobs, search | US-033 – US-044 |
| Member — safety | US-045 – US-047 |
| Member — Should-have features | US-048 – US-057 |
| Community moderator | US-058 – US-059 |
| Platform moderator & T&S | US-060 – US-066 |
| Company representative | US-067 – US-069 |
| Administrator | US-070 – US-075 |
| Future | US-076 – US-088 |

---

## 1. Visitor

### US-001 — Browse the public feed
**Priority:** MVP · **Feature:** F-11 · **Requirements:** FR-24, FR-25

As a **visitor**, I want to browse public stories in The Mess so that I can see what LinkedOut is about before creating an account.

1. **Given** I am not logged in, **when** I open `/mess`, **then** I see the Latest feed containing only posts with visibility `public` and moderation status `clear` (posts with status `limited` are hidden from visitors and from feeds), newest first, 20 per page.
2. **Given** I scroll to the end of the list, **when** more posts exist, **then** the next page loads using the cursor and no post is duplicated.
3. **Given** a post has a content warning, **when** it appears in the feed, **then** its body is collapsed behind a "Show anyway" control naming the warning.
4. **Given** I select the category "Interview Disasters", **when** the feed reloads, **then** only posts in that category appear and the URL contains `?category=interview-disasters`.
5. **Given** I try to react, comment or save, **when** I click the control, **then** I am prompted to sign up or log in and return to the same post afterwards.

### US-002 — Read a public post and its comments
**Priority:** MVP · **Feature:** F-10, F-12 · **Requirements:** FR-20, FR-26

As a **visitor**, I want to open a post and read its comments so that I can learn from the discussion.

1. **Given** a public, clear post, **when** I open `/posts/{postId}`, **then** I see title, body, category, tags, author pseudonym or anonymous label, reaction counts by type, corrections and the threaded comments.
2. **Given** the post visibility is `members`, **when** I open it as a visitor, **then** I see a sign-in prompt and no body text.
3. **Given** the post is removed or held, **when** I open it, **then** I see a 404 page with no information about why.

### US-003 — View a Workplace Reality Page
**Priority:** MVP · **Feature:** F-17 · **Requirements:** FR-36, FR-39, FR-40

As a **visitor**, I want to view a company's Workplace Reality Page so that I can research an employer.

1. **Given** an active company page, **when** I open `/companies/{slug}`, **then** I see name, industry, official website (only if confirmed), description, theme summaries and the latest public reviews and job reports.
2. **Given** a company has fewer than 5 approved reviews in the last 24 months, **when** I view theme summaries, **then** I see "Not enough reports yet" instead of ratings.
3. **Given** any review or job report, **when** it is displayed, **then** it shows the label "Unverified firsthand account", the relationship type and the approximate period.
4. **Given** I filter by "Interview", **when** the list reloads, **then** only interview/job reports appear.

### US-004 — Search public content
**Priority:** MVP · **Feature:** F-19 · **Requirements:** FR-43

As a **visitor**, I want to search posts and companies so that I can find experiences relevant to me.

1. **Given** I enter "unpaid test assignment" in search, **when** I submit, **then** I see matching public posts, lessons, job reports and companies grouped by type, ranked by relevance.
2. **Given** a matching post is `members` visibility, held or removed, **when** results are computed, **then** it is not included.
3. **Given** my query is shorter than 2 characters, **when** I submit, **then** I see a validation message and no request is sent.

### US-005 — Create an account
**Priority:** MVP · **Feature:** F-01, F-03 · **Requirements:** FR-1, FR-8

As a **visitor**, I want to register with my email and a pseudonym so that I can participate without using my real name.

1. **Given** I provide a valid email, a 12+ character password not found in the breached-password list, an available username, age ≥ 16 confirmation and policy acceptance, **when** I submit, **then** my account is created with status `pending_verification` and a verification email is sent.
2. **Given** the username is taken or reserved, **when** I type it, **then** I see "That username isn't available" within 500 ms (debounced availability check).
3. **Given** the email is already registered, **when** I submit, **then** I see the same success screen ("Check your inbox") and the existing owner receives an "Someone tried to register with your email" message, so that registration does not reveal account existence.
4. **Given** I have made 5 registration attempts from my IP in the last hour, **when** I submit again, **then** I receive HTTP 429 with a retry time.

---

## 2. Member — Account & Privacy

### US-006 — Verify my email
**Priority:** MVP · **Feature:** F-01 · **Requirements:** FR-2

As a **member**, I want to verify my email so that I can start posting.

1. **Given** I click a valid verification link within 24 h, **when** the page loads, **then** my account becomes `active`, I am logged in and redirected to onboarding.
2. **Given** the link is expired or already used, **when** I open it, **then** I see "This link has expired" with a "Resend email" button.
3. **Given** I am unverified, **when** I try to publish, comment, react or report, **then** the action is blocked with a banner "Verify your email to join the conversation".
4. **Given** I request a resend, **when** I have requested 3 in the last hour, **then** I get HTTP 429.

### US-007 — Log in and log out
**Priority:** MVP · **Feature:** F-01 · **Requirements:** FR-3, FR-6

As a **member**, I want to log in and out securely so that only I can use my account.

1. **Given** correct credentials, **when** I log in, **then** a new session is created, the `lo_sid` cookie is set httpOnly/Secure/SameSite=Lax, and I land on `/mess`.
2. **Given** incorrect credentials, **when** I log in, **then** I see "Email or password is incorrect" without indication of which.
3. **Given** 10 failed attempts for my email in 15 minutes, **when** I try again, **then** login is locked for 15 minutes and I receive an email notifying me with a password-reset link.
4. **Given** I am logged in, **when** I click Log out, **then** the session is deleted server-side, the cookie is cleared, and protected pages redirect to `/login`.

### US-008 — Reset my password
**Priority:** MVP · **Feature:** F-01 · **Requirements:** FR-4

As a **member**, I want to reset a forgotten password so that I can recover my account.

1. **Given** I submit my email on `/forgot-password`, **when** the request completes, **then** I always see "If that email is registered, we've sent a link" regardless of existence.
2. **Given** I open a valid reset link within 60 minutes, **when** I set a new valid password, **then** the password is changed, all my sessions are revoked, I receive a confirmation email and I am logged in with a new session.
3. **Given** the link is used or expired, **when** I open it, **then** I see an error with a link to request a new one.

### US-009 — Manage my sessions and devices
**Priority:** MVP · **Feature:** F-01 · **Requirements:** FR-5

As a **member**, I want to see and revoke my active sessions so that I can secure my account if I lose a device.

1. **Given** I open Settings → Sessions, **when** the page loads, **then** I see each session's device label, browser, approximate city/country, created and last-active time, with my current session marked.
2. **Given** I click "Sign out" on another session, **when** confirmed, **then** that session is deleted and its next request returns 401.
3. **Given** I click "Sign out all other sessions", **when** confirmed, **then** all sessions except the current one are deleted and an audit log entry `auth.sessions_revoked_all` is written.

### US-010 — Sign in with Google
**Priority:** Should · **Feature:** F-02 · **Requirements:** FR-7

As a **visitor**, I want to sign in with Google so that I do not need another password.

1. **Given** I complete Google consent for the first time, **when** I return, **then** I must choose a username and accept policies before an account is created; my Google name and photo are not imported.
2. **Given** my Google email matches an existing password account, **when** I sign in with Google, **then** I am asked to log in with my password once to link the accounts.
3. **Given** the OAuth `state` parameter does not match, **when** the callback is processed, **then** the request is rejected with 400 and no account is created.

### US-011 — Choose my public pseudonym
**Priority:** MVP · **Feature:** F-03 · **Requirements:** FR-8, FR-10

As a **member**, I want a public username separate from my identity so that people see my pseudonym, not my name or email.

1. **Given** I register, **when** I choose a username, **then** it must match `^[a-z0-9_]{3,30}$`, be unique case-insensitively, and not be on the reserved list (e.g. `admin`, `moderator`, `linkedout`, `support`, `official`).
2. **Given** any public API response that includes me as an author, **when** it is serialized, **then** it contains only `username`, `displayName`, `avatarUrl`, `headline` — never email, internal ID or IP.
3. **Given** the username contains my email local part, **when** I submit, **then** I see a warning "This looks like your email — choose something less identifying" (non-blocking).

### US-012 — Change my pseudonym
**Priority:** MVP · **Feature:** F-03 · **Requirements:** FR-9

As a **member**, I want to change my username so that I can stay unidentifiable if my pseudonym becomes known.

1. **Given** I have not changed my username in the last 30 days, **when** I submit a new valid username, **then** it is updated everywhere my pseudonymous content appears and the old name is reserved for 90 days.
2. **Given** I changed it fewer than 30 days ago, **when** I try again, **then** I see the date when I can next change it.
3. **Given** someone visits `/u/{oldUsername}` within 90 days, **when** the page loads, **then** they see "This member isn't here" (no redirect, to avoid linking old and new names).

### US-013 — Understand privacy during onboarding
**Priority:** MVP · **Feature:** F-04 · **Requirements:** FR-11

As a **new member**, I want a clear explanation of what is private and what is not so that I can decide what to share.

1. **Given** I verify my email for the first time, **when** I am redirected, **then** I see a 4-step onboarding: (1) pseudonym vs anonymous, (2) what data we collect and why, (3) what we may disclose under lawful request, (4) Community Guidelines.
2. **Given** I have not acknowledged step 4, **when** I try to publish, **then** I am returned to onboarding.
3. **Given** I finish onboarding, **when** I choose interests (categories), **then** my feed defaults to those categories as a suggested filter and `onboarding_completed_at` is set.

### US-014 — Edit my Anti-Profile
**Priority:** MVP · **Feature:** F-05 · **Requirements:** FR-12

As a **member**, I want to edit my Anti-Profile so that it reflects my real career experience.

1. **Given** I open Settings → Profile, **when** I update headline (≤ 120), bio (≤ 600), industry, broad location (country/region list only) and career status (including "Open to Escape"), **then** the changes save and appear on `/u/{username}`.
2. **Given** a field exceeds its limit, **when** I submit, **then** the field shows an inline error and nothing is saved.
3. **Given** my bio contains an email address or phone number, **when** I save, **then** I see a privacy warning and must confirm before saving.

### US-015 — Add lessons, skills and timeline
**Priority:** MVP · **Feature:** F-05 · **Requirements:** FR-13

As a **member**, I want to list career lessons, skills I'm learning and an experience timeline so that others can learn from my path.

1. **Given** I add a lesson, **when** I have fewer than 10, **then** it is saved and shown on my profile in my chosen order.
2. **Given** I add a timeline entry with period, role category and industry, **when** saved, **then** it appears in chronological order with no employer field required.
3. **Given** I try to add an 11th lesson or 21st skill, **when** I submit, **then** I see the limit message.

### US-016 — View another member's Anti-Profile
**Priority:** MVP · **Feature:** F-05 · **Requirements:** FR-14, FR-10

As a **member**, I want to view someone's Anti-Profile so that I can understand the context of their stories.

1. **Given** a member with profile visibility `public`, **when** anyone opens `/u/{username}`, **then** they see profile fields, pseudonymous posts and comments.
2. **Given** visibility `members`, **when** a visitor opens it, **then** they see only username and a sign-in prompt.
3. **Given** visibility `hidden`, **when** anyone except the owner opens it, **then** they see only username and "This profile is private".
4. **Given** the member has anonymous confessions, **when** anyone views their profile, **then** none are listed and no count of them is shown.
5. **Given** any profile, **when** rendered, **then** no counts of rejections, failures, reactions received or hours worked are displayed.

### US-017 — Control my privacy settings
**Priority:** MVP · **Feature:** F-06 · **Requirements:** FR-15

As a **member**, I want to control who can see my profile and how my content is attributed by default so that I stay in control.

1. **Given** I open Settings → Privacy, **when** I change profile visibility, discoverability, default confession identity mode and default post visibility, **then** they are saved and applied to new content and profile views immediately.
2. **Given** discoverability is off, **when** others search members, **then** I do not appear in member search results.

### US-018 — Block a member
**Priority:** MVP · **Feature:** F-07 · **Requirements:** FR-16

As a **member**, I want to block someone so that they cannot interact with me.

1. **Given** I block a member from their profile or content menu, **when** confirmed, **then** their posts and comments disappear from my feed, search and threads (shown as "Hidden: you blocked this member" in threads), and they cannot react to or comment on my content.
2. **Given** I block an anonymous label, **when** confirmed, **then** content under that label is hidden from me; I am never shown which account owns it, and the block does not hide that account's pseudonymous content.
3. **Given** I unblock from Settings → Blocked, **when** confirmed, **then** visibility is restored.
4. **Given** I am blocked, **when** I view the blocker's public content, **then** I can still read it but cannot interact, and I am not told that I am blocked.

### US-019 — Mute members, categories and tags
**Priority:** MVP · **Feature:** F-07 · **Requirements:** FR-17

As a **member**, I want to mute people or topics so that I can limit distressing content without confrontation.

1. **Given** I mute the category "Layoffs and Job Loss", **when** I load the feed or search, **then** posts in that category are excluded.
2. **Given** I mute a member, **when** they post, **then** I do not see it in feeds; they are not notified.
3. **Given** I unmute, **when** I reload, **then** the content reappears.

### US-020 — Request account deletion
**Priority:** MVP · **Feature:** F-08 · **Requirements:** FR-18

As a **member**, I want to delete my account and understand what is kept so that I can leave with confidence.

1. **Given** I open Settings → Account → Delete account, **when** the page loads, **then** I see a plain-language list of what is deleted and what is retained (moderation records 2 years, audit logs 2 years, data under legal hold) and the 30-day grace period.
2. **Given** I re-enter my password and confirm, **when** submitted, **then** my account status becomes `deletion_pending`, all sessions are revoked, my content is hidden immediately, and I receive an email with a cancellation link.
3. **Given** I log in within 30 days, **when** I click "Cancel deletion", **then** my account and content are restored.
4. **Given** 30 days pass, **when** the deletion job runs, **then** personal data is purged, content is deleted, and audit entries reference only a pseudonymous tombstone ID.

### US-021 — Export my data
**Priority:** Should · **Feature:** F-09 · **Requirements:** FR-19

As a **member**, I want to download my data so that I can keep a copy or move elsewhere.

1. **Given** I request an export, **when** the job completes (≤ 24 h), **then** I receive an email with a download link valid for 7 days to a ZIP of JSON files.
2. **Given** I already have an export in progress, **when** I request another, **then** I see "An export is already being prepared".
3. **Given** the export includes my anonymous confessions, **when** generated, **then** it is delivered only to my verified email after re-authentication.

---

## 3. Member — Posts, Feed & Interaction

### US-022 — Share a story
**Priority:** MVP · **Feature:** F-10 · **Requirements:** FR-20, FR-22, FR-45

As a **member**, I want to write a story about a career setback so that I can share it and feel less alone.

1. **Given** I click "Share a Story", **when** I fill title, body, category, up to 5 tags, desired response and visibility, **then** I can preview and publish.
2. **Given** the safety check finds no issues, **when** I publish, **then** the post is created with moderation status `clear` and appears at the top of the Latest feed.
3. **Given** the safety check finds an email or phone number, **when** I preview, **then** the text is highlighted with "This looks like personal information" and I can edit or confirm.
4. **Given** I have published 10 posts in the last hour, **when** I publish another, **then** I receive HTTP 429.

### US-023 — Ask for advice
**Priority:** MVP · **Feature:** F-10 · **Requirements:** FR-20

As a **member**, I want to post a question seeking advice so that experienced people can help me.

1. **Given** I click "Ask for Advice", **when** the composer opens, **then** post type is `advice` and desired response is preset to `advice`.
2. **Given** my advice post is published, **when** it appears in listings, **then** it shows an "Asking for advice" badge.

### US-024 — Add a content warning
**Priority:** MVP · **Feature:** F-10 · **Requirements:** FR-23

As a **member**, I want to add content warnings so that readers can choose whether to read difficult content.

1. **Given** I select warnings "burnout" and "mental_health", **when** published, **then** the post body is collapsed in all listings behind "Content warning: burnout, mental health — Show anyway".
2. **Given** a moderator adds a warning, **when** saved, **then** it is applied in the same way and I am notified of the change.

### US-025 — Edit or delete my post
**Priority:** MVP · **Feature:** F-10 · **Requirements:** FR-21

As a **member**, I want to edit or delete my posts so that I can fix mistakes or remove something I regret.

1. **Given** I edit my post, **when** saved, **then** the post shows "Edited" with time, re-runs safety checks, and a revision is stored for moderation.
2. **Given** I delete my post, **when** confirmed, **then** it disappears from feeds, the post URL returns 404, and comments become inaccessible.
3. **Given** I try to edit someone else's post via the API, **when** the request arrives, **then** it returns 403.

### US-026 — Browse the Latest feed with filters
**Priority:** MVP · **Feature:** F-11 · **Requirements:** FR-24, FR-25

As a **member**, I want to browse the newest posts filtered by category and type so that I find relevant stories.

1. **Given** I am logged in, **when** I open `/mess`, **then** I see public and members-only posts, newest first, excluding blocked/muted content.
2. **Given** I filter by type "Lessons" and category "Developer Problems", **when** applied, **then** only matching posts appear and the URL encodes both filters.

### US-027 — Hide categories and filter by language
**Priority:** MVP · **Feature:** F-11 · **Requirements:** FR-25

As a **member**, I want to hide categories and filter by language so that my feed suits me.

1. **Given** I hide "Office Drama" from the feed filter menu, **when** I reload on any device, **then** it remains hidden (stored as a category mute).
2. **Given** I set the language filter to "English", **when** the feed loads, **then** only posts tagged `en` appear.

### US-028 — Comment and reply
**Priority:** MVP · **Feature:** F-12 · **Requirements:** FR-26

As a **member**, I want to comment on posts and reply to comments so that I can support people and discuss.

1. **Given** I write a comment (≤ 5,000 chars), **when** I submit, **then** it appears in the thread immediately and the post's comment count increments.
2. **Given** I reply to a reply at depth 3, **when** submitted, **then** it is shown at depth 3 with "Replying to @user".
3. **Given** I am blocked by the post author, **when** I try to comment, **then** I receive 403 with "You can't comment on this post".
4. **Given** I post 30 comments within 10 minutes, **when** I submit another, **then** I receive 429.

### US-029 — Edit or delete my comment
**Priority:** MVP · **Feature:** F-12 · **Requirements:** FR-27

As a **member**, I want to edit or delete my comments so that I control what I said.

1. **Given** my comment, **when** I edit it, **then** it shows "Edited".
2. **Given** my comment has replies, **when** I delete it, **then** its body becomes "[deleted]" and replies remain.
3. **Given** my comment has no replies, **when** I delete it, **then** it disappears from the thread.

### US-030 — Pin an author clarification
**Priority:** MVP · **Feature:** F-12 · **Requirements:** FR-28

As a **post author**, I want to pin my own clarifying comment so that readers see important context first.

1. **Given** I am the post author and I wrote a comment on my post, **when** I select "Pin as clarification", **then** it appears above all comments with an "Author clarification" label.
2. **Given** a clarification is already pinned, **when** I pin another, **then** the previous one is unpinned.
3. **Given** a comment by another member, **when** I view its menu, **then** "Pin as clarification" is not available.

### US-031 — React to a post
**Priority:** MVP · **Feature:** F-13 · **Requirements:** FR-29

As a **member**, I want to react with I'm Dead, Pain, Same Here, Melting, Corporate Circus or Survived so that I can express support quickly.

1. **Given** I have not reacted, **when** I choose "Same Here", **then** the Same Here count increments by 1 and my reaction is highlighted.
2. **Given** I reacted "Pain", **when** I choose "Survived", **then** Pain decrements and Survived increments (one reaction per post).
3. **Given** I click my current reaction again, **when** processed, **then** it is removed.
4. **Given** any profile, **when** rendered, **then** no total of reactions received is shown.
5. **Given** I use a keyboard, **when** I open the reaction picker, **then** I can move between reactions with arrow keys and each has an accessible name.

### US-032 — Save posts
**Priority:** MVP · **Feature:** F-14 · **Requirements:** FR-30

As a **member**, I want to save useful posts so that I can come back to them.

1. **Given** I click Save on a post, **when** processed, **then** it appears in My Damage → Saved, newest-saved first.
2. **Given** a saved post is later deleted or removed, **when** I view Saved, **then** it shows "This post is no longer available" and can be removed from the list.
3. **Given** another member views my profile, **when** rendered, **then** my saved posts are not visible.

---

## 4. Member — Confessions, Lessons, Companies, Jobs, Search

### US-033 — Submit an anonymous Office Confession
**Priority:** MVP · **Feature:** F-15 · **Requirements:** FR-31, FR-32

As a **member**, I want to share a workplace story anonymously to other members so that I can be honest without being identified by colleagues.

1. **Given** I open `/confessions/new`, **when** I fill title, body, experience type, desired response, optional industry/region/period, content warnings and choose identity mode `anonymous`, **then** I proceed to the safety-check & preview step.
2. **Given** I confirm publication with no high-risk findings, **when** submitted, **then** the confession is published under a newly generated label such as "Anonymous Stapler #4821", and no response to any other user contains my username or user ID in connection with it.
3. **Given** I publish a second anonymous confession, **when** it appears, **then** it has a different label from my first.
4. **Given** I view My Damage → My posts, **when** loaded, **then** I see my anonymous confession marked "Only you can see that this is yours".
5. **Given** I have published 5 confessions in the last 24 h (1 if my account is < 24 h old), **when** I submit another, **then** I get 429.

### US-034 — See privacy warnings and preview before publishing
**Priority:** MVP · **Feature:** F-15, F-21 · **Requirements:** FR-31, FR-45

As a **member**, I want to be warned about identifying details before I publish so that I don't accidentally expose myself or others.

1. **Given** my confession contains "my manager John Smith", **when** the safety check runs, **then** the name is highlighted with "Naming individuals triggers a stricter review. Consider describing the role instead." and risk is at least `medium`.
2. **Given** risk is `medium` or `high`, **when** I confirm submission, **then** the confession is created with moderation status `held`, I see "Your confession is waiting for a quick review (usually within 24 hours)", and it appears in the moderation queue.
3. **Given** the text contains an AWS access key pattern, **when** checked, **then** I see "This looks like a secret or credential — remove it before posting" and publish is blocked until removed.
4. **Given** the preview step, **when** displayed, **then** it shows exactly how the confession will look to others, including my anonymous label.

### US-035 — Save a confession as a private draft
**Priority:** MVP · **Feature:** F-15 · **Requirements:** FR-33

As a **member**, I want to save a confession as a private draft so that I can think before sharing.

1. **Given** I click "Save as private draft", **when** saved, **then** it is visible only to me in My Damage → Drafts and is never in feeds or search.
2. **Given** a draft untouched for 173 days, **when** the reminder job runs, **then** I get an email that it will be deleted in 7 days.
3. **Given** I publish a draft, **when** confirmed, **then** it goes through the safety check and preview flow.

### US-036 — Add a correction to my post or confession
**Priority:** MVP · **Feature:** F-15 · **Requirements:** FR-34

As an **author**, I want to add a correction or clarification so that readers have accurate information.

1. **Given** my published confession, **when** I add a correction (≤ 1,000 chars), **then** it appears above the body with date and label "Correction from the author".
2. **Given** I have added 5 corrections, **when** I try another, **then** I see the limit message.
3. **Given** a published correction, **when** I try to delete it, **then** deletion is not available to me.

### US-037 — Write a Learned the Hard Way lesson
**Priority:** MVP · **Feature:** F-16 · **Requirements:** FR-35

As a **member**, I want to write a structured lesson so that my mistake helps someone else.

1. **Given** I choose "Lesson learned", **when** the composer opens, **then** I see sections: What I did wrong, What happened, What I learned, What I'd do differently, Advice for someone facing this, Helpful links (up to 5).
2. **Given** "What I learned" is empty, **when** I publish, **then** I see "Add what you learned — it's the heart of a lesson".
3. **Given** a link is not `https://`, **when** I submit, **then** it is rejected.

### US-038 — Browse lessons
**Priority:** MVP · **Feature:** F-16 · **Requirements:** FR-35

As a **member**, I want to browse Learned the Hard Way lessons by category so that I can learn before I make the same mistake.

1. **Given** I open `/lessons`, **when** loaded, **then** I see lesson posts, newest first, each previewing "What I learned".
2. **Given** I select category "Startup Lessons", **when** applied, **then** only lessons in that category appear.

### US-039 — Write a workplace review
**Priority:** MVP · **Feature:** F-17 · **Requirements:** FR-38, FR-45, FR-48

As a **member**, I want to write a structured workplace review so that others can make informed decisions.

1. **Given** my account is ≥ 7 days old and verified, **when** I submit relationship type, period, topic ratings, narrative (≥ 150 chars), firsthand attestation and display consent, **then** the review is created and shown on the company page labelled "Unverified firsthand account".
2. **Given** I already have an active review for the company, **when** I open the form, **then** I am offered to edit my existing review instead.
3. **Given** I do not consent to public display of the narrative, **when** published, **then** only my ratings contribute to theme summaries and the narrative is not shown publicly.
4. **Given** my account is < 7 days old, **when** I try, **then** I see "You can review companies 7 days after joining".
5. **Given** the narrative names an individual, **when** submitted, **then** it is held for stricter review.

### US-040 — Filter company reports
**Priority:** MVP · **Feature:** F-17 · **Requirements:** FR-40

As a **member**, I want to filter a company's reports by topic and relationship so that I see what matters to me.

1. **Given** I select topic "Salary/payment" and relationship "Intern", **when** applied, **then** only matching reviews appear, sorted newest first.
2. **Given** no results match, **when** applied, **then** I see "No reports match these filters" with a "Clear filters" button.

### US-041 — Submit a missing company
**Priority:** MVP · **Feature:** F-17 · **Requirements:** FR-37

As a **member**, I want to add a company that isn't listed so that I can report my experience there.

1. **Given** search finds no match, **when** I submit name, industry and website, **then** a company with status `pending` is created and I can attach my review/job report, which is published once the company is approved.
2. **Given** a company with the same normalized name or website domain exists, **when** I submit, **then** I am shown the existing company instead.

### US-042 — Submit a Job Nightmare report
**Priority:** MVP · **Feature:** F-18 · **Requirements:** FR-41

As a **job seeker**, I want to report my recruitment experience so that other candidates know what to expect.

1. **Given** I select a company, role category, month, stage reached (`application`, `screening`, `technical`, `onsite`, `final`, `offer`), report type and narrative, **when** I submit, **then** the report appears on the Job Nightmare Board and the company page.
2. **Given** my narrative contains a list of more than 5 question marks in succession or the phrase "the questions were", **when** checked, **then** I see "Please don't share confidential interview questions" and risk is `medium`.
3. **Given** report type "Positive or respectful hiring experience", **when** displayed, **then** it carries a "Good experience" badge.

### US-043 — Browse the Job Nightmare Board
**Priority:** MVP · **Feature:** F-18 · **Requirements:** FR-42

As a **job seeker**, I want to browse recruitment reports by type and company so that I can spot patterns.

1. **Given** I open `/job-nightmares`, **when** loaded, **then** I see reports newest first with company, role category, stage, type and period.
2. **Given** I filter by "Candidate ghosting", **when** applied, **then** only those reports appear.

### US-044 — Search with filters
**Priority:** MVP · **Feature:** F-19 · **Requirements:** FR-43

As a **member**, I want to search across content types with filters so that I find exactly what I need.

1. **Given** I search "offer withdrawn" with type "Job reports", **when** submitted, **then** only job reports appear with matched terms highlighted.
2. **Given** results include content from a member I muted, **when** computed, **then** it is excluded.
3. **Given** I search a username, **when** that member has discoverability off, **then** they are not in member results.

---

## 5. Member — Safety

### US-045 — Report content
**Priority:** MVP · **Feature:** F-20 · **Requirements:** FR-44

As a **member**, I want to report harmful content so that moderators can review it.

1. **Given** I click Report on a post, comment, profile, review, job report or company response, **when** I choose a reason and optional details, **then** a report is created, I see "Thanks — our moderators will review this", and the target's moderation item is created or updated.
2. **Given** I already reported this target, **when** I report again, **then** I see "You've already reported this".
3. **Given** reason "Harassment or threats" or "Personal information", **when** created, **then** the moderation item gets priority `high`.

### US-046 — Track my reports
**Priority:** MVP · **Feature:** F-20 · **Requirements:** FR-44

As a **member**, I want to see the status of reports I submitted so that I know they were handled.

1. **Given** I open `/reports`, **when** loaded, **then** I see each report's target summary, reason, status (`open`, `actioned`, `no_violation`) and decision date.
2. **Given** a report is resolved, **when** the decision is made, **then** I receive an email explaining the outcome without revealing details about the other member.

### US-047 — Appeal a moderation decision
**Priority:** MVP · **Feature:** F-22 · **Requirements:** FR-49

As a **member**, I want to appeal a decision about my content or account so that mistakes can be corrected.

1. **Given** a moderation action against my content within the last 30 days, **when** I open it in `/reports` → "Decisions about me" and submit an appeal reason, **then** an appeal is created with status `pending`.
2. **Given** I already appealed this action, **when** I try again, **then** the appeal button is disabled with "You've already appealed this decision".
3. **Given** my appeal is decided, **when** resolved, **then** I receive an email with the outcome (`upheld`, `reversed`, `modified`) and explanation, and reversed actions are undone automatically.

---

## 6. Member — Should-have Features

### US-048 — Create a community
**Priority:** Should · **Feature:** F-26 · **Requirements:** FR-59

As a **member**, I want to create a community such as "Fresh Graduate Survivors" so that people with shared experiences can support each other.

1. **Given** my account is ≥ 14 days old, **when** I submit name, slug, description, visibility and at least one rule, **then** the community is created and I am its owner.
2. **Given** the slug is taken, **when** I submit, **then** I see "That address is taken".
3. **Given** I already own 3 communities, **when** I try to create another, **then** I see the limit message.

### US-049 — Join and leave communities
**Priority:** Should · **Feature:** F-26 · **Requirements:** FR-60

As a **member**, I want to join and leave communities so that my feed includes groups I care about.

1. **Given** a public community, **when** I click Join, **then** I become a member immediately and its posts appear in my Following feed.
2. **Given** a private community, **when** I click "Request to join", **then** a join request is created for community moderators.
3. **Given** I am a member, **when** I click Leave, **then** my membership is removed.

### US-050 — Post in a community
**Priority:** Should · **Feature:** F-26 · **Requirements:** FR-61

As a **community member**, I want to post into a community so that I reach people with similar experiences.

1. **Given** I am a member, **when** I create a post and select the community, **then** it appears in the community feed and, for public communities, in The Mess.
2. **Given** the community is private, **when** a non-member searches, **then** the post does not appear.

### US-051 — Request to join a private community
**Priority:** Should · **Feature:** F-26 · **Requirements:** FR-63

As a **member**, I want to request to join a private community so that I can access a safer, smaller space.

1. **Given** I request to join, **when** a community moderator approves, **then** I become a member and receive a notification.
2. **Given** my request is declined, **when** processed, **then** I can request again after 30 days.

### US-052 — Receive in-app notifications
**Priority:** Should · **Feature:** F-27 · **Requirements:** FR-64

As a **member**, I want notifications about replies and decisions so that I don't miss responses.

1. **Given** someone comments on my post, **when** saved, **then** I get an in-app notification within 1 minute and the unread badge increments.
2. **Given** 12 people react to my post within an hour, **when** batched, **then** I get one notification "12 people reacted to your post".
3. **Given** I open the notification center, **when** I click "Mark all as read", **then** the unread count is 0.

### US-053 — Configure email notifications and quiet hours
**Priority:** Should · **Feature:** F-27 · **Requirements:** FR-65, FR-66

As a **member**, I want to control email notifications and set quiet hours so that LinkedOut doesn't intrude on my life.

1. **Given** I disable email for reactions, **when** someone reacts, **then** no email is sent.
2. **Given** quiet hours 22:00–08:00 in my time zone, **when** an email is triggered at 23:00, **then** it is deferred to 08:00.
3. **Given** a moderation decision, **when** it occurs, **then** an email is always sent regardless of preferences.

### US-054 — Create a poll
**Priority:** Should · **Feature:** F-28 · **Requirements:** FR-67

As a **member**, I want to create a poll so that I can ask the community a quick question.

1. **Given** I enter a question, 2–6 options and a duration of 1–14 days, **when** published, **then** a poll post appears in the feed.
2. **Given** I enter 1 option, **when** I submit, **then** I see "Add at least 2 options".

### US-055 — Vote in a poll
**Priority:** Should · **Feature:** F-28 · **Requirements:** FR-67

As a **member**, I want to vote in polls so that I can share my experience quickly.

1. **Given** an open poll, **when** I vote, **then** results (percentages and total votes) are shown and I cannot vote again.
2. **Given** a closed poll, **when** I open it, **then** I see final results and no vote controls.
3. **Given** any poll, **when** results are shown, **then** no individual voter is identifiable.

### US-056 — Add images to a post
**Priority:** Should · **Feature:** F-29 · **Requirements:** FR-68

As a **member**, I want to attach images or memes to posts so that I can express things visually.

1. **Given** I attach a 3 MB PNG, **when** upload completes, **then** I see "Checking image…" until the malware scan and re-encode complete, then a thumbnail.
2. **Given** a 7 MB file or a `.svg`, **when** I select it, **then** it is rejected client- and server-side with the allowed types and size.
3. **Given** a photo with GPS EXIF data, **when** processed, **then** the published image contains no EXIF metadata.
4. **Given** the image is published, **when** displayed, **then** it requires alt text (I am prompted to add it before publishing).

### US-057 — Follow topics and use the Following feed
**Priority:** Should · **Feature:** F-30 · **Requirements:** FR-69, FR-70

As a **member**, I want to follow categories and tags so that I see the topics I care about.

1. **Given** I follow the tag `#first-job`, **when** I open Following, **then** posts with that tag appear newest first.
2. **Given** I follow nothing and joined no communities, **when** I open Following, **then** I see suggestions of categories to follow.

---

## 7. Community Moderator

### US-058 — Moderate my community
**Priority:** Should · **Feature:** F-26 · **Requirements:** FR-62

As a **community moderator**, I want a queue of reports in my community so that I can keep it supportive.

1. **Given** content in my community is reported, **when** I open the community moderation tab, **then** I see the report with reason and content.
2. **Given** I remove a post from the community, **when** confirmed with a reason, **then** it is removed from the community feed, the author is notified, and the action is audit-logged.
3. **Given** content in a community I don't moderate, **when** I call the community moderation API for it, **then** I get 403.
4. **Given** a report of threats or doxxing, **when** created, **then** it is also routed to the platform queue automatically.

### US-059 — Maintain rules and pinned resources
**Priority:** Should · **Feature:** F-26 · **Requirements:** FR-62

As a **community moderator**, I want to edit rules and pin resources so that members know expectations and find help.

1. **Given** I edit the rules, **when** saved, **then** they are displayed on the community page and in the composer when posting there.
2. **Given** 5 resources are pinned, **when** I pin a 6th, **then** I see the limit message.

---

## 8. Platform Moderator & Trust and Safety

### US-060 — Review the moderation queue
**Priority:** MVP · **Feature:** F-21 · **Requirements:** FR-46

As a **platform moderator**, I want a prioritized queue so that the most harmful content is handled first.

1. **Given** I open `/mod`, **when** loaded, **then** I see open moderation items sorted by priority (high → normal → low), then oldest first, showing target type, reason(s), report count, automated findings and age.
2. **Given** 3 members report the same comment, **when** the queue loads, **then** there is one item with report count 3.
3. **Given** I open an item, **when** I claim it, **then** it is assigned to me and others see "Being reviewed by {moderator}".

### US-061 — Apply a moderation action
**Priority:** MVP · **Feature:** F-21 · **Requirements:** FR-47

As a **platform moderator**, I want to apply a proportionate action with a reason so that enforcement is fair and explained.

1. **Given** an item, **when** I choose "Remove content" with reason and policy section, **then** the content is removed, the author receives an email explaining which rule was broken and how to appeal, reporters are notified of the outcome, and an audit entry is written.
2. **Given** I choose "Temporarily restrict posting" for 7 days, **when** applied, **then** the member cannot post or comment until the restriction ends, and sees a banner with the end date.
3. **Given** I submit an action without a reason, **when** validated, **then** it is rejected.

### US-062 — Review held content
**Priority:** MVP · **Feature:** F-21 · **Requirements:** FR-45

As a **platform moderator**, I want to review automatically held content so that safe content is published quickly and risky content is not.

1. **Given** a held confession, **when** I approve it, **then** it is published with its original timestamp replaced by the approval time and the author is notified.
2. **Given** a held item, **when** I choose "Request edits" with guidance, **then** the author gets an email and can edit and resubmit.

### US-063 — Decide appeals
**Priority:** MVP · **Feature:** F-22 · **Requirements:** FR-49

As a **platform moderator**, I want to decide appeals so that wrong decisions are corrected.

1. **Given** a pending appeal for an action I took, **when** I open it, **then** decision controls are disabled with "A different moderator must decide this appeal".
2. **Given** I reverse the action, **when** saved, **then** the content or account restriction is restored and the member is notified.

### US-064 — Investigate coordinated review campaigns
**Priority:** MVP · **Feature:** F-21 · **Requirements:** FR-48

As a **platform moderator**, I want suspected review campaigns flagged so that company pages are not manipulated.

1. **Given** 6 reviews of the same company in 24 h, **when** the 6th is created, **then** a campaign item is added to the queue grouping all 6, and the company's theme summaries show "Summaries paused while we review recent activity".
2. **Given** I resolve the campaign as legitimate, **when** saved, **then** summaries resume.

### US-065 — Escalate for legal or specialist review
**Priority:** MVP · **Feature:** F-21 · **Requirements:** FR-47

As a **platform moderator**, I want to escalate complex allegations so that legal or specialist reviewers decide.

1. **Given** an item, **when** I choose "Escalate" with a note, **then** its status becomes `escalated`, it moves to the Escalations tab visible to admins, and content visibility can be limited while pending.

### US-066 — Resolve an anonymous identity under lawful need
**Priority:** MVP · **Feature:** F-23 · **Requirements:** FR-51, FR-50

As a **trust and safety officer** (moderator with `identity_vault.reveal`), I want to resolve an anonymous label to an account only with justification and approval so that serious threats can be acted on without casual access.

1. **Given** I hold the permission, **when** I request a reveal with a justification of ≥ 50 chars and a category (`credible_threat`, `legal_order`, `repeat_abuse_ban_evasion`), **then** a request is created with status `pending_approval`.
2. **Given** a second staff member with the permission approves, **when** approved, **then** I can view the account's username and account ID for 24 h, and both actions are audit-logged.
3. **Given** I do not hold the permission, **when** I call the reveal endpoint, **then** I get 403 and the attempt is audit-logged.
4. **Given** I try to approve my own request, **when** submitted, **then** it is rejected.

---

## 9. Company Representative

### US-067 — Request association with a company page
**Priority:** Should · **Feature:** F-25 · **Requirements:** FR-56, FR-57

As a **company representative**, I want to verify that I represent my company so that I can respond officially.

1. **Given** I am a verified member, **when** I submit a request with my work email on the company's confirmed domain and job title, **then** a verification link is sent to that work email.
2. **Given** I click the link and an admin approves, **when** approved, **then** I hold an active representative membership for 12 months.
3. **Given** the work email domain does not match the official website domain, **when** submitted, **then** the request is rejected with an explanation.

### US-068 — Post an official response
**Priority:** Should · **Feature:** F-25 · **Requirements:** FR-58

As a **company representative**, I want to respond to a review or job report so that readers see our perspective.

1. **Given** an active membership, **when** I submit a response (≤ 3,000 chars) to a review on my company, **then** it is safety-checked, shown under the review with an "Official representative" badge once clear, and the review author is notified.
2. **Given** a review already has a response from my company, **when** I try to add another, **then** I am offered to edit the existing one.
3. **Given** my response contains the review author's suspected name, **when** checked, **then** it is held for review.

### US-069 — Never see author identities
**Priority:** Should · **Feature:** F-25, F-03 · **Requirements:** FR-58, FR-10

As a **company representative**, I understand that I cannot access reviewer identities, so that members trust the platform.

1. **Given** any API response available to me, **when** inspected, **then** it contains no author user ID, email or vault data for reviews, reports or confessions.
2. **Given** I report a review, **when** submitted, **then** it follows the same moderation process as any member report; I have no remove/hide capability.

---

## 10. Administrator

### US-070 — Manage users and roles
**Priority:** MVP · **Feature:** F-24 · **Requirements:** FR-52

As an **administrator**, I want to manage roles so that staff have the access they need and no more.

1. **Given** I search by email or username, **when** results load, **then** I see account status, role, created date and moderation history summary — but never anonymous mappings.
2. **Given** I change a member's role to `platform_moderator`, **when** saved, **then** the change is audit-logged and effective on their next request.
3. **Given** I am the only admin, **when** I try to remove my own admin role, **then** it is rejected.

### US-071 — Manage categories
**Priority:** MVP · **Feature:** F-24 · **Requirements:** FR-53

As an **administrator**, I want to manage categories so that the taxonomy stays useful.

1. **Given** I archive a category, **when** saved, **then** it is no longer selectable for new posts but existing posts keep it.
2. **Given** I reorder categories, **when** saved, **then** the order applies to filters and composer.

### US-072 — Manage company pages
**Priority:** MVP · **Feature:** F-24 · **Requirements:** FR-54

As an **administrator**, I want to approve, merge and suspend company pages so that the directory is accurate.

1. **Given** a pending company, **when** I approve it, **then** it becomes active and attached pending reviews/reports are published if clear.
2. **Given** duplicates A and B, **when** I merge B into A, **then** all of B's reviews, reports and memberships move to A, and B's URL returns 301 to A.

### US-073 — Verify company representatives
**Priority:** Should · **Feature:** F-25 · **Requirements:** FR-57

As an **administrator**, I want to review representative requests so that only genuine representatives respond officially.

1. **Given** a request with verified work email, **when** I approve, **then** membership status becomes `verified` with expiry in 12 months.
2. **Given** I revoke a membership, **when** saved, **then** existing responses keep showing but lose the badge, and new responses are blocked.

### US-074 — View platform metrics
**Priority:** MVP · **Feature:** F-24 · **Requirements:** FR-55

As an **administrator**, I want key trust and growth metrics so that I can spot problems early.

1. **Given** I open `/admin`, **when** loaded, **then** I see sign-ups, verified members, posts/day, reports per 1,000 posts, open queue size, median moderation response time and appeal reversal rate for the last 7/30 days.

### US-075 — Review audit logs
**Priority:** MVP · **Feature:** F-23 · **Requirements:** FR-50

As an **administrator**, I want to review audit logs so that sensitive access is accountable.

1. **Given** I open `/admin/audit`, **when** I filter by action `identity_vault.reveal`, **then** I see every reveal request, approver, justification and time.
2. **Given** any audit entry, **when** I try to edit or delete it via any API, **then** no such endpoint exists and database permissions prevent update/delete.

---

## 11. Future Stories

| ID | Role | Story | Feature | FR | Key acceptance criterion (Given/When/Then) |
|---|---|---|---|---|---|
| US-076 | Member | As a member, I want one-to-one peer support conversations so that I can get private help. | F-31 | FR-71 | Given a member with message permission "requests only", when I message them, then a message request is created that they can accept or decline; anonymous confession authors cannot be messaged. |
| US-077 | Member | As a member, I want a private Career Damage Report so that I can reflect on my progress. | F-32 | FR-72 | Given I track 20 applications and 5 interviews, when I open My Damage → Report, then I see my private counts and goals, and no other user or API can read them. |
| US-078 | Member | As a member, I want to make a corporate meme from a template so that I can share humor safely. | F-33 | FR-73 | Given I add text naming a private individual, when I preview, then the PII check warns me and sharing requires moderation. |
| US-079 | Member | As a member, I want to browse Hall of Shame pattern highlights so that I can recognize common workplace anti-patterns. | F-34 | FR-74 | Given a highlight, when displayed, then it describes a pattern from anonymized moderated stories and names no individual. |
| US-080 | Member | As a member, I want a Relevant feed so that I see the most useful stories. | F-35 | FR-75 | Given I switch to Relevant, when loaded, then ranking uses topic relevance, recency, meaningful replies and diversity, and I can switch back to Latest in one click. |
| US-081 | Member | As a member, I want to enable two-factor authentication so that my account is safer. | F-39 | FR-79 | Given MFA is enabled, when I log in, then I must enter a valid TOTP code or recovery code. |
| US-082 | Member | As a member, I want optional premium features so that I can customize my experience. | F-37 | FR-77 | Given I am not premium, when I use block, mute, report or privacy settings, then they are fully available. |
| US-083 | Company representative | As a representative, I want aggregated trend reports so that I can address recurring issues. | F-38 | FR-78 | Given fewer than 10 contributing reviews for a segment, when the report renders, then that segment is suppressed. |
| US-084 | Member | As a member, I want category and content-warning suggestions so that posting is faster. | F-36 | FR-76 | Given I write a post, when suggestions appear, then they are only applied after I accept them. |
| US-085 | Member | As a member, I want a mobile app so that I can use LinkedOut on the go. | F-40 | FR-80 | Given the app, when I log in, then it uses the same `/api/v1` and the same privacy rules. |
| US-086 | Member | As a member, I want guided learning paths so that I can prepare for interviews and first jobs. | F-41 | FR-81 | Given I start "Interview preparation", when I complete a step, then my progress is saved privately. |
| US-087 | Member | As a member, I want push notifications so that I hear about replies promptly. | F-42 | FR-82 | Given quiet hours, when a push is triggered, then it is deferred. |
| US-088 | Member | As a member, I want to RSVP to survivor meetups so that I can connect with others. | F-43 | FR-83 | Given an event, when I RSVP, then I appear only as my pseudonym or as "1 anonymous attendee" per my choice. |
