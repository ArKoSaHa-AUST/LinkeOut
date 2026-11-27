# LinkedOut — UI/UX Specification

Related: [FEATURES.md](FEATURES.md) · [USER_STORIES.md](USER_STORIES.md) · [WORKFLOW.md](WORKFLOW.md) · [API_SPEC.md](API_SPEC.md)

**Design intent (plan §4):** dark-first, clean cards, restrained humor, never cruel. Humor lives in copy, reactions and empty states. The interface itself stays calm, readable and high-contrast, and screens should not look aggressive or depressing.

---

## Part A — Information architecture

### A.1 Navigation

**Desktop (≥ 1024 px):** left side navigation (240 px) + content column (max 680 px) + context aside (320 px, hidden < 1280 px).

| Nav item (plan §9) | Route | Feature | Visible to |
|---|---|---|---|
| The Mess | `/mess` | F-11 | Everyone |
| Trauma Network | `/communities` | F-26 | Everyone (once `communities.enabled`) |
| Job Nightmares | `/job-nightmares` | F-18 | Everyone |
| Confessions | `/confessions` | F-15 | Everyone (once `confessions.enabled`) |
| Company Reality | `/companies` | F-17 | Everyone |
| Learned the Hard Way | `/lessons` | F-16 | Everyone |
| Notifications | `/notifications` | F-27 | Members (Should; until then hidden) |
| My Damage | `/me` | S-13 | Members |
| Moderation | `/mod` | F-21 | Moderators |
| Admin | `/admin` | F-24 | Admins |

**Primary actions** (a "Share" split button at the top of the side nav; floating action button on mobile):

| Action | Opens |
|---|---|
| Share a Story | `/post/new?type=story` |
| Ask for Advice | `/post/new?type=advice` |
| Write a Lesson | `/post/new?type=lesson` |
| Confess Anonymously | `/confessions/new` |
| Report a Workplace Experience | `/companies?intent=review` → choose company → `/companies/[slug]/review` |
| Report a Job Nightmare | `/job-nightmares/new` |
| Create a Community (Should) | `/communities/new` |

**Mobile (< 768 px):** top bar (logo, search icon, notifications bell) + bottom tab bar with 5 items: The Mess, Confessions, **Share** (center, opens action sheet), Companies, My Damage. Other sections live in the "More" drawer opened from the avatar.

**Footer (all public pages):** Community Guidelines, Privacy Policy, Terms, "Not legal, medical or mental-health advice" disclaimer, Help/contact, © LinkedOut.

### A.2 Screen inventory

| ID | Screen | Route | Plan screen # | Features | Priority |
|---|---|---|---|---|---|
| S-01 | Landing | `/` | 1 | F-11 | MVP |
| S-02 | Sign up | `/signup` | 2 | F-01, F-03 | MVP |
| S-02b | Complete OAuth sign-up | `/signup/complete` | 2 | F-02 | Should |
| S-03 | Log in | `/login` | 2 | F-01 | MVP |
| S-04 | Verify email | `/verify-email` | 2 | F-01 | MVP |
| S-05 | Forgot / reset password | `/forgot-password`, `/reset-password` | 2 | F-01 | MVP |
| S-06 | Onboarding & privacy explanation | `/onboarding` | 3 | F-04 | MVP |
| S-07 | The Mess (main feed) | `/mess` | 4 | F-11, F-13, F-14, F-30 | MVP |
| S-08 | Create post | `/post/new` | 5 | F-10, F-16, F-28, F-29 | MVP |
| S-09 | Confession submission | `/confessions/new` | 6 | F-15, F-21 | MVP |
| S-10 | Confessions list | `/confessions` | — | F-15 | MVP |
| S-11 | Post detail & comments | `/posts/[postId]` | 7 | F-10, F-12, F-13, F-15 | MVP |
| S-12 | Public Anti-Profile | `/u/[username]` | 8 | F-05 | MVP |
| S-13 | My Damage (private member dashboard) | `/me` | 9 | F-14, F-15, F-20, F-22 | MVP |
| S-14 | Learned the Hard Way | `/lessons` | — | F-16 | MVP |
| S-15 | Company directory | `/companies` | — | F-17 | MVP |
| S-16 | Workplace Reality Page | `/companies/[slug]` | 10 | F-17, F-18, F-25 | MVP |
| S-17 | Write workplace review | `/companies/[slug]/review` | — | F-17 | MVP |
| S-18 | Submit a company | `/companies/new` | — | F-17 | MVP |
| S-19 | Job Nightmare Board (+ report detail) | `/job-nightmares`, `/job-nightmares/[id]` | — | F-18 | MVP |
| S-20 | Job Nightmare report form | `/job-nightmares/new` | 11 | F-18 | MVP |
| S-21 | Search results | `/search` | 12 | F-19 | MVP |
| S-22 | Report & appeal center | `/reports` | 16 | F-20, F-22 | MVP |
| S-23 | Moderator dashboard | `/mod`, `/mod/appeals` | 17 | F-21, F-22 | MVP |
| S-24 | Moderation item detail | `/mod/items/[itemId]` | 17 | F-21, F-23 | MVP |
| S-25 | Settings → Profile | `/settings/profile` | — | F-03, F-05 | MVP |
| S-26 | Settings → Privacy | `/settings/privacy` | 15 | F-06 | MVP |
| S-27 | Settings → Sessions | `/settings/sessions` | — | F-01 | MVP |
| S-28 | Settings → Blocked & muted | `/settings/blocked` | — | F-07 | MVP |
| S-29 | Settings → Account & data | `/settings/account` | — | F-08, F-09 | MVP |
| S-30 | Admin dashboard | `/admin` | 18 | F-24 | MVP |
| S-31 | Admin sub-pages | `/admin/*` | 18 | F-23, F-24, F-25 | MVP |
| S-32 | Representative request | `/companies/[slug]/represent` | — | F-25 | Should |
| S-33 | Communities directory | `/communities` | — | F-26 | Should |
| S-34 | Community page | `/communities/[slug]` | 13 | F-26 | Should |
| S-35 | Create community | `/communities/new` | — | F-26 | Should |
| S-36 | Notification center | `/notifications` | 14 | F-27 | Should |
| S-37 | Settings → Notifications | `/settings/notifications` | — | F-27 | Should |
| S-38 | System pages (404, error, maintenance, suspended) | various | — | — | MVP |

---

## Part B — Screens

Each screen lists its **Purpose**, **Components**, **Actions**, **Navigation**, and **States** (Loading / Empty / Error). Global rules for every screen:
- Loading uses skeletons matching the final layout. No spinners for content areas, and no layout shift.
- Errors use `ErrorState` with a plain-language message, a retry button, and the `requestId` in small text for support.
- Rate-limit errors (429) show "You're going a bit fast — try again in {n} seconds."

### S-01 Landing
- **Purpose:** Explain LinkedOut in one screen, show real public stories, and convert visitors.
- **Components:** Hero (headline "Your career. Your chaos.", sub-line "LinkedIn shows the promotion. LinkedOut tells the story behind it."), primary CTA "Join LinkedOut", secondary CTA "Read the Mess"; "How privacy works" 3-card strip (Pseudonym · Anonymous to members · What we never promise); a live preview of the latest 5 public posts (PostCard compact); "What LinkedOut is / isn't" two-column list (plan §2); footer.
- **Actions:** Sign up, log in, open a post, read the guidelines.
- **Navigation:** Logged-in users are redirected to `/mess`.
- **States:** Loading — skeleton of 5 cards. Empty (no public posts yet) — "The Mess is suspiciously clean. Be the first to share." Error — the preview is hidden and the static content remains.

### S-02 Sign up
- **Purpose:** Create an account with a pseudonym (FR-1, FR-8).
- **Components:** Form: email, password (strength meter + "checked against known breaches" note), username with live availability (debounced 400 ms) and an "Avoid your real name or email" hint, "I am 16 or older" checkbox, Terms/Privacy/Guidelines checkbox with links, invite code field (only while `beta.invite_only`), Google button (Should), submit.
- **Actions:** Submit, go to log in.
- **Navigation:** Success → "Check your inbox" panel with a resend link (60 s cooldown).
- **States:** Loading — button spinner with the inputs disabled. Error — inline field errors (`aria-describedby`), form-level alert for 429/500. Username taken — "That username isn't available. Try {suggestion}."

### S-02b Complete OAuth sign-up (Should)
- **Purpose:** Pick a username and accept policies after Google sign-in.
- **Components:** Username field, age and policy checkboxes, note "We don't import your Google name or photo."
- **States:** Expired pending cookie → error with "Start again".

### S-03 Log in
- **Purpose:** Authenticate (FR-3).
- **Components:** Email, password (show/hide), "Remember this device", submit, "Forgot password?", Google button (Should), link to sign up.
- **Navigation:** Success → return URL or `/mess`. Unverified → `/verify-email`. Onboarding incomplete → `/onboarding`. Account `deletion_pending` → panel "Your account is scheduled for deletion on {date}" with a "Cancel deletion" button.
- **States:** Error — generic "Email or password is incorrect." Locked — "Too many attempts. We've emailed you a reset link. Try again in 15 minutes."

### S-04 Verify email
- **Purpose:** Consume the token and explain the verification state.
- **Components:** Status panel (verifying → success → redirect), expired-link panel with "Resend email".
- **States:** Loading — "Verifying…" with `aria-live="polite"`.

### S-05 Forgot / reset password
- **Purpose:** Recover an account (FR-4).
- **Components:** Forgot: email field and submit, then a neutral confirmation. Reset: new password with a strength meter, confirmation field, submit.
- **States:** Invalid token panel with "Request a new link".

### S-06 Onboarding & privacy explanation
- **Purpose:** Informed consent before participating (FR-11).
- **Components:** 4-step stepper with progress (`aria-current="step"`):
  1. **Pseudonym vs anonymous.** Illustrated comparison: "Public pseudonym: people see @your_username" / "Anonymous to members: people see a label like *Anonymous Stapler #4821*. We still know it's you, and so does the law if it asks properly." / "Private draft: only you."
  2. **What we collect and why.** A table of email (login, security), IP (security, 90 days), content, and settings, plus the sentence "We never show your email, IP or real name."
  3. **What we may disclose.** Lawful requests and credible threats of harm; the four-eyes review; the transparency report.
  4. **Community Guidelines.** Summary of the prohibited list (plan §14) and "Prefer patterns over naming people." An "I understand" checkbox is required.
  5. **Pick interests** (optional): category chips.
- **Actions:** Next/back; finish → `/mess`.
- **States:** Error saving → retry; progress is kept in local storage.

### S-07 The Mess (main feed)
- **Purpose:** Discovery and conversation (F-11).
- **Components:** Mode tabs "Latest" | "Following" (Should; "Relevant" appears with F-35); FeedFilters bar (category multi-select chips, type select, language select, "Hide category" in each chip's menu); FeedList of PostCards; infinite scroll with a "Load more" fallback button; aside (≥ 1280 px): "Workplace Wins this week" (latest posts in that category), Community Guidelines snippet, "Feeling heavy? Mute topics" shortcut.
- **PostCard:** author (avatar/monogram + username, or an anonymous badge with the label), relative time, category chip, type badge (Advice / Lesson / Confession / Poll), title, 3-line excerpt or ContentWarningGate, tags, ReactionBar (6 reactions with counts), comment count, Save, overflow menu (Report, Block, Mute author, Mute category, Copy link).
- **Actions:** Open a post, react, save, filter, report, block, mute, share (copy link).
- **Navigation:** Card → S-11. Author → S-12 (pseudonymous only).
- **States:** Loading — 5 card skeletons. Empty (filters) — "Nothing here yet. Either this category is thriving or everyone's in a meeting." + "Clear filters". Empty (Following, Should) — suggested categories to follow. Error — ErrorState with retry; already-loaded items stay.

### S-08 Create post
- **Purpose:** Compose stories, advice requests, lessons and polls (F-10, F-16, F-28).
- **Components:** Type switcher (Story / Advice / Lesson / Poll*); title (counter /150); Markdown editor with toolbar (bold, italic, list, quote, link) and a Preview tab; Lesson mode replaces the body with 6 labelled sections ("What I learned" is required); Poll mode adds 2–6 option inputs and a duration select*; category select (required); tag input with autocomplete (max 5); desired response radio (Support / Advice / Discussion / Just sharing); content warning multi-select; visibility (Public / Members only); language select; image picker (Should, alt text required); community select (Should); **SafetyCheckPanel** (runs on preview and before publish, lists findings with "Jump to text", highlights ranges in the editor); "Publish" button. (*Should)
- **Actions:** Preview, run the safety check, publish, discard (confirm dialog if dirty).
- **Navigation:** Publish → S-11 with the toast "Posted. The Mess thanks you." Held → S-11 with a moderation notice banner.
- **States:** Loading (submitting) — publish button disabled and the editor read-only. Error — 422 findings shown in the SafetyCheckPanel; other errors inline. Unverified/onboarding — composer replaced by a call-to-action panel.

### S-09 Confession submission
- **Purpose:** Safe anonymous storytelling with a preview and confirmation (F-15, Journey B).
- **Components:** 3-step wizard:
  1. **Write.** Title, story body, experience type (required), category, desired response, content warnings, optional industry / broad region (country) / period (year + quarter), tags. A persistent tip panel: "Don't include names, emails, internal project names or anything confidential."
  2. **Safety check & preview.** SafetyCheckPanel with findings, the rendered preview exactly as others will see it (including the anonymous label, generated at this step and shown as "e.g. Anonymous Stapler #4821 — your actual label is created when you publish"), identity mode picker (Anonymous to members / Public pseudonym @username), visibility (Members only default / Public), "What this means" expandable explainer.
  3. **Confirm.** Summary checklist ("I've removed names and identifying details", "I understand anonymity isn't absolute"), "Publish" and "Save as private draft".
- **Actions:** Next/back, save draft at any step, publish.
- **Navigation:** Published → S-11. Held → S-11 with "Your confession is waiting for a quick review (usually within 24 hours)." Draft → S-13 Drafts tab.
- **States:** Feature disabled → "Confessions open soon — we're finishing our safety tools first." Loading/error as S-08.

### S-10 Confessions list
- **Purpose:** Browse confessions.
- **Components:** Filters (experience type, category), PostCard list with the Confession badge, "Confess anonymously" CTA.
- **States:** Empty — "No confessions yet. The office is either perfect or very quiet." Members-only items for visitors — a card with a "Sign in to read" overlay.

### S-11 Post detail & comments
- **Purpose:** Read a post in full and discuss it (F-10, F-12, F-13).
- **Components:** Breadcrumb (category); header (author, time, edited marker, type badge, "Asking for advice" badge); moderation notice banner (author only, when held/limited); ContentWarningGate; body (long-form with "Read more" for > 3,000 chars); Lesson sections layout; Confession meta row (experience type, industry, region, period); Corrections block ("Correction from the author", dated); poll widget (Should); images (Should); ReactionBar (large); Save; Share; overflow menu (Edit, Delete, Add correction — author; Report, Block — others); Pinned "Author clarification" card; CommentThread (sort: Oldest / Newest; depth ≤ 3 with indentation and "Replying to @x" beyond that; "[deleted]" and "Hidden: you blocked this member" placeholders; OP badge for anonymous authors); CommentForm (sticky on mobile) with a content-warning toggle.
- **Actions:** React, save, comment, reply, edit/delete own, pin a clarification (author), add a correction (author), report, block.
- **States:** Loading — header + body skeleton, then a comments skeleton. Removed/held/not found — S-38 404. Members-only to visitors — title + "Sign in to read". Comments empty — "No replies yet. Be the person you needed when this happened to you."

### S-12 Public Anti-Profile
- **Purpose:** Show a member's pseudonymous career reality (F-05).
- **Components:** ProfileHeader (monogram/avatar, username, display name, headline, career status chip e.g. "Open to Escape", industry, country, "Member since"); tabs: Story (bio "What went wrong", lessons list "Lessons and setbacks", skills "Skills still under development", timeline "Career survival history"), Posts, Comments, Communities (Should); overflow (Report profile, Block, Mute). **No counts of reactions, rejections, posts or followers.**
- **States:** Restricted — "This profile is private" (hidden) or "Sign in to see this profile" (members-only). Empty sections are omitted for visitors; the owner sees "Add your first lesson" prompts.

### S-13 My Damage (private member dashboard)
- **Purpose:** The member's private hub (plan screen 9). It is renamed Career Damage Report when F-32 ships.
- **Components:** Tabs: **My posts** (all, including anonymous confessions with the "Only you can see that this is yours" chip and moderation status chips), **Drafts** (confession drafts with "Deletes on {date}"), **Saved** (saved posts), **Decisions & reports** (link to S-22). A welcome card for new members with 3 starter actions.
- **States:** Empty Saved — "Nothing saved yet. Tap the bookmark on stories you'll want later." Empty Drafts — "No drafts. Brave."

### S-14 Learned the Hard Way
- **Purpose:** Browse structured lessons (F-16).
- **Components:** Intro banner ("Mistakes, converted into advice"), category filter chips, LessonCards (title, "What I learned" preview, category, author, reactions), "Write a lesson" CTA.
- **States:** Empty — "No lessons here yet. Someone has definitely learned something, though."

### S-15 Company directory
- **Purpose:** Find a company (F-17, Journey C).
- **Components:** Search box (trigram), industry filter, CompanyCards (name, industry, report counts), "Can't find it? Add a company" link. With `?intent=review`, the cards show a "Review this company" button.
- **States:** No results — "No company called '{q}' yet." + "Add it". Loading — list skeleton.

### S-16 Workplace Reality Page
- **Purpose:** Balanced, structured employer information (F-17, F-18, F-25).
- **Components:** CompanyHeader (name, industry, confirmed website link with an external-link icon, neutral description, "Official representative active" badge when applicable); persistent notice "Reviews and reports are unverified firsthand accounts from members. Read them as experiences, not facts."; ThemeSummary (5 topics with median and a 5-bar distribution, n and "last 24 months"; "Not enough reports yet" if n < 5; a "Summaries paused while we review recent activity" banner if paused); tabs **Workplace** | **Interview & hiring**; filters (topic, relationship, period); ReviewCards (relationship, period, title, narrative or "The author chose not to show details publicly", ratings, "Unverified firsthand account" label, company response block with the Official badge, Report and "Request correction" links); JobReportCards; CTAs "Write a review" and "Report a job nightmare"; "Represent this company?" link (Should).
- **States:** Pending company — 404 for others; the submitter sees "Waiting for approval". Merged — redirect. Empty reviews — "No workplace reports yet. If you've worked here, your experience helps the next person."

### S-17 Write workplace review
- **Purpose:** Structured firsthand review (FR-38).
- **Components:** Relationship select, period start/end years ("I still work here" toggle), 5 topic rating rows (1–5 radio groups with text labels "Very poor … Very good", each skippable), title, narrative (min 150 chars with a counter), tips panel ("Describe practices and patterns, not individuals"), firsthand attestation checkbox (required), display consent toggle with an explanation, SafetyCheckPanel, submit.
- **States:** Account < 7 days — explanatory panel with the date when allowed. Already reviewed — redirect to the edit form for the existing review. Held — confirmation with the review notice.

### S-18 Submit a company
- **Purpose:** Add a missing company (FR-37).
- **Components:** Name, industry, website URL; live duplicate suggestions ("Did you mean Acme Widgets?").
- **States:** Success — "Thanks! We'll check it soon. You can already write your review — it'll appear once the company is approved."

### S-19 Job Nightmare Board
- **Purpose:** Browse recruitment experiences (F-18).
- **Components:** Filters (report type chips with icons, company search, role category, period), JobReportCards (company, role category, stage reached timeline chips, report type, outcome, period, label, narrative excerpt), "Good experience" badge for positive reports, "Report a job nightmare" CTA. The detail page `/job-nightmares/[id]` shows the full narrative, company response (Should), report link and company page link.
- **States:** Empty — "No nightmares in this category. Sweet dreams (for now)."

### S-20 Job Nightmare report form
- **Purpose:** Submit a recruitment experience (FR-41).
- **Components:** Company combobox (directory search, or "Add a new company" inline fields), role category, month picker (no future months), stage reached stepper, report type, outcome (optional), narrative with the tip "Don't share confidential interview questions or recruiter details", identity mode (Anonymous default / Pseudonym), visibility, SafetyCheckPanel, submit.
- **States:** As S-08.

### S-21 Search results
- **Purpose:** Find content (F-19).
- **Components:** Search input (≥ 2 chars), type tabs (All, Posts, Lessons, Confessions, Job reports, Companies, Members, Communities*), category filter, grouped results for "All" with "See all" per group, highlighted matches (`<mark>` styled with an underline and background, not color only).
- **States:** Empty — "No results for '{q}'. Try fewer words or a different spelling." Too short — inline hint.

### S-22 Report & appeal center
- **Purpose:** Transparency on reports filed and decisions received (F-20, F-22).
- **Components:** Tabs **Reports I made** (target summary, reason, status chip, decision date, outcome message) and **Decisions about me** (action type, target summary, policy reference with a link to the guideline section, member message, appeal status; an "Appeal" button opens AppealDialog with a reason textarea (≤ 2,000) and the deadline).
- **States:** Empty — "You haven't reported anything. Thanks for keeping it civil." / "No decisions about your content. Keep it up."

### S-23 Moderator dashboard
- **Purpose:** Prioritized moderation work (F-21, F-22).
- **Components:** Tabs **Queue**, **Mine**, **Escalated**, **Appeals**; filters (priority, reason, target type, automated vs reported); QueueTable (priority pill, target type, reasons with counts, automated findings icons, preview excerpt, age, assignee) with keyboard navigation (J/K to move, Enter to open); header stats (open items, high-priority items, oldest age, my resolved today); Appeals list showing "Can't decide (you took the original action)".
- **States:** Empty queue — "Queue clear. Go drink some water." Error — retry.

### S-24 Moderation item detail
- **Purpose:** Decide with full context (FR-46, FR-47, FR-51).
- **Components:** Content panel (rendered content with finding highlights, revisions diff toggle, corrections); Reports panel (each report: reason, details, time); Automated findings panel; Author history summary (counts by action type; anonymous authors show "History aggregated — identity hidden"); Related items (campaign members); ActionForm (action type radio with descriptions, required reason, policy reference select from Community Guidelines sections, member message pre-filled from a template and editable, duration select for restrictions, content-warning select, escalation note, "Notify reporters" checkbox); Claim/Release button; **RevealPanel** (visible only with `identity_vault.reveal`): request form (category, justification ≥ 50 chars), pending status, approval controls for a second officer, the revealed result with a 24 h countdown.
- **States:** Claimed by another moderator — read-only with "Being reviewed by @x".

### S-25 Settings → Profile
- **Components:** Display name, headline, bio (with PII warning), industry, country, career status, avatar (Should), lessons editor (reorderable list, max 10), skills chips (max 20), timeline editor; username change section with rules and the next-available date.
- **States:** Save success toast "Profile updated"; field errors inline.

### S-26 Settings → Privacy
- **Components:** Profile visibility radio (Public / Members only / Hidden) with an example preview, discoverability toggle, default confession identity, default post visibility, message permissions (disabled with the note "Messaging isn't available yet"), "What's never public" info box.

### S-27 Settings → Sessions
- **Components:** Session list (device, browser, approximate location, last active, "This device" badge), "Sign out" per row, "Sign out all other sessions" (confirm dialog).

### S-28 Settings → Blocked & muted
- **Components:** Tabs Blocked (members and anonymous labels) and Muted (members, categories, tags), each with an Unblock/Unmute button.
- **States:** Empty — "You haven't blocked anyone." / "Nothing muted."

### S-29 Settings → Account & data
- **Components:** Email (read-only with a "Change email" note: contact support in MVP), change password, Data export (Should): "Request export" with status, Delete account: the retained-data table, password re-entry, a typed confirmation "DELETE", and a destructive button.
- **States:** Deletion pending — banner with the date and "Cancel deletion".

### S-30 Admin dashboard
- **Components:** KPI tiles (sign-ups, verified members, posts/day, reports per 1,000 posts, open queue, median response time high/normal, appeal reversal rate) with a 7/30-day toggle; links to sub-pages; system status (queue depths from `/health/ready` plus the worker heartbeat).

### S-31 Admin sub-pages
- **Users:** search, role select, permission grant/revoke (with confirmation and audit note).
- **Categories:** table with drag-to-reorder (plus keyboard "Move up/down" buttons), add/edit/archive.
- **Companies:** pending queue with duplicate hints, approve/reject/edit/merge, website-confirmed toggle.
- **Representatives (Should):** pending requests, approve/reject/revoke.
- **Escalations:** escalated items.
- **Feature flags:** toggles; `confessions.enabled` requires the launch-gate checklist modal.
- **Audit log:** filterable read-only table with a metadata JSON viewer.

### S-32 Representative request (Should)
- **Components:** Explanation of what representatives can and cannot do ("You can respond. You can't remove reviews or see who wrote them."), work email, job title, submit; status after submission.

### S-33 Communities directory (Should)
- **Components:** Search, CommunityCards (name, description, member count, Public/Private badge, Join / Request to join), "Create a community" CTA. Seeded examples are featured.
- **States:** Empty search — "No community yet. Start one?"

### S-34 Community page (Should)
- **Components:** Header (name, description, visibility, member count, Join/Leave), Rules panel, Pinned resources, community feed (PostCards), "Post in this community" composer shortcut, Moderators list; Moderation tab for community moderators (scoped queue, join requests, rules editor, resources editor, announcements, invitations).
- **States:** Private to non-members — header, rules and "Request to join" only.

### S-35 Create community (Should)
- **Components:** Name, slug (auto from name, editable, availability check), description, visibility, rules editor (≥ 1 rule), a "Communities are for support, not coordinated attacks" acknowledgement.

### S-36 Notification center (Should)
- **Components:** Filter (All / Unread), notification rows (icon by type, text, time, unread dot), "Mark all as read", link to preferences.
- **States:** Empty — "All quiet. Enjoy it."

### S-37 Settings → Notifications (Should)
- **Components:** Matrix of notification types × (In-app, Email) toggles; moderation decision email locked on with an explanation; quiet hours start/end and time zone.

### S-38 System pages
- **404:** "This page went the way of our 2019 career plans." + links home.
- **Error:** "Something broke on our side. It's not you (this time)." + retry + request ID.
- **Maintenance:** "We're doing some quick maintenance. Back shortly."
- **Suspended account:** explanation, policy reference, appeal link.

### Future screens (designed during each Phase 10 spike)

| ID | Screen | Route | Feature | Purpose and key components | States to design |
|---|---|---|---|---|---|
| S-40 | Messages inbox & thread | `/messages`, `/messages/[id]` | F-31 | Conversation list, message requests tab (Accept/Decline), thread with safety notice banner, block/report menu, composer (text only) | Empty inbox, pending request, blocked, rate-limited |
| S-41 | Career Damage Report | `/me/report` | F-32 | Private tiles (applications tracked, interview stages, lessons saved, skills in development), goals list, progress notes; "Only you can see this" banner | First-use empty state with gentle prompts; no red "failure" framing |
| S-42 | Meme generator | `/memes/new` | F-33 | Template gallery, text layers, live preview, safety findings, Export / Share as post | Template load error, safety warning |
| S-43 | Hall of Shame | `/hall-of-shame` | F-34 | Pattern highlight cards ("The 6 pm Friday deploy"), linked example stories, how patterns are chosen | Empty state |
| S-44 | Relevant feed tab | `/mess?mode=relevant` | F-35 | Third tab in S-07 with a "Why am I seeing this?" popover | Same as S-07 |
| S-45 | Composer suggestions | within S-08 | F-36 | Suggested category and warnings as dismissible chips labelled "Suggestion" | Suggestions unavailable (silent) |
| S-46 | Premium | `/premium`, `/settings/billing` | F-37 | Plan comparison, "Safety features are always free" note, billing management | Payment failure, cancelled |
| S-47 | Company analytics | `/companies/[slug]/analytics` | F-38 | Trend charts per topic with suppressed segments labelled "Not enough data" | Insufficient data |
| S-48 | Two-factor setup | `/settings/security` | F-39 | QR code, manual key, code input, recovery codes download | Wrong code, lost device |
| S-49 | Mobile apps | native | F-40 | Same IA with bottom tab bar | Offline |
| S-50 | Learning paths | `/learn`, `/learn/[slug]` | F-41 | Path cards, step checklist, private progress | Not started, completed |
| S-51 | Push settings | `/settings/notifications` | F-42 | Push toggle per type, browser permission explainer | Permission denied |
| S-52 | Events | `/communities/[slug]/events`, `/events/[id]` | F-43 | Event cards, RSVP with display-mode choice, reminders | Full, cancelled, past |

---

## Part C — Design system

### C.1 Color tokens

The plan palette is kept. Two additions are needed for accessibility: `text-secondary` (`#A3A3A3`), because the plan's ash gray `#737373` measures 3.8:1 on `#171717` and fails AA for body text, and dark text on red buttons, because white on `#F87171` is 2.8:1. See [ASSUMPTIONS.md](ASSUMPTIONS.md#6-design).

| Token | Hex | Use | Contrast on `bg` #171717 |
|---|---|---|---|
| `bg` | `#171717` | App background (Midnight charcoal) | — |
| `surface` | `#1F1F1F` | Cards, inputs | — |
| `surface-raised` | `#262626` | Menus, dialogs, ContentWarningGate | — |
| `surface-hover` | `#2E2E2E` | Hover state of cards/rows | — |
| `divider` | `#2E2E2E` | Decorative separators | decorative |
| `border-strong` | `#737373` | Input borders, focusable control outlines (Ash gray) | 3.8:1 on `bg`, 3.5:1 on `surface` (UI component ✓) |
| `text-primary` | `#F5F5F5` | Main text (Off-white) | 16.4:1 |
| `text-secondary` | `#A3A3A3` | Metadata, helper text | 7.1:1 |
| `text-muted` | `#737373` | Placeholders, disabled text, large decorative text only | 3.8:1 (large text only) |
| `accent` | `#F87171` | Primary buttons, active nav, links (Failure red) | 6.5:1 |
| `accent-hover` | `#FCA5A5` | Hover | 9.5:1 |
| `accent-pressed` | `#EF4444` | Active/pressed | 4.8:1 |
| `on-accent` | `#171717` | Text/icons on accent backgrounds | 6.5:1 on `#F87171` |
| `danger` | `#DC2626` | Destructive confirm buttons (white text 4.8:1) | — |
| `success` | `#4ADE80` | Success toasts, "Good experience" badge, Survived | 10.3:1 |
| `warning` | `#FBBF24` | Content warnings, "Unverified" label outline, held notices | 10.7:1 |
| `info` | `#60A5FA` | Informational banners, Official representative badge | 7.1:1 |
| `anon` | `#A78BFA` | Anonymous label badge | 6.6:1 |
| `focus-ring` | `#FBBF24` | 2 px outline + 2 px offset on all focusable elements | 10.7:1 |
| `mark-bg` | `#3F3A1E` | Search highlight background (with underline) | text-primary on it 10.5:1 |

Rules: never use color alone to convey meaning (badges always carry text and an icon); red is reserved for primary actions and brand accents, never for "your career is failing" framing; light theme is Future (tokens are named semantically to allow it).

### C.2 Typography

| Token | Font | Size / line-height | Weight | Use |
|---|---|---|---|---|
| `display-xl` | Bricolage Grotesque | 48 / 52 px | 800 | Landing hero |
| `display-lg` | Bricolage Grotesque | 36 / 42 px | 700 | Page titles on marketing pages |
| `h1` | Bricolage Grotesque | 30 / 36 px | 700 | Screen titles |
| `h2` | Inter | 24 / 30 px | 600 | Section headings, post title on detail |
| `h3` | Inter | 20 / 26 px | 600 | Card titles |
| `body-lg` | Inter | 18 / 29 px | 400 | Post body on detail (max 68ch) |
| `body` | Inter | 16 / 26 px | 400 | Default text |
| `body-sm` | Inter | 14 / 21 px | 400/500 | Metadata, helper text |
| `caption` | Inter | 12 / 17 px | 500 | Badges, timestamps (never the only carrier of essential info) |
| `mono` | JetBrains Mono | 14 / 21 px | 400 | Code blocks, request IDs |

Fonts are self-hosted via `next/font` with `display: swap`. Text resizes to 200% without loss of content. Minimum body size is 16 px on mobile.

### C.3 Spacing, radii, elevation

| Token | Value |
|---|---|
| Spacing scale (4 px base) | `0`=0, `1`=4, `2`=8, `3`=12, `4`=16, `5`=20, `6`=24, `8`=32, `10`=40, `12`=48, `16`=64 px |
| Card padding | 16 px mobile, 20 px desktop |
| Gap between feed cards | 12 px |
| Radii | `sm` 6 px (chips, inputs), `md` 10 px (cards, buttons), `lg` 16 px (dialogs), `full` (avatars, pills) |
| Elevation | Surfaces get lighter as they rise (`surface` → `surface-raised`) plus a 1 px `divider` border; one shadow token for dialogs: `0 12px 32px rgba(0,0,0,0.5)` |
| Hit targets | ≥ 44 × 44 px on touch devices; ≥ 24 × 24 px with spacing on desktop (WCAG 2.5.8) |
| Motion | 150 ms ease-out for hover/press, 200 ms for dialogs; disabled under `prefers-reduced-motion` |

### C.4 Breakpoints and layout

| Name | Min width | Layout |
|---|---|---|
| `base` | 0 | Single column, 16 px gutters, bottom tab bar, sticky top bar |
| `sm` | 640 px | Single column, 24 px gutters |
| `md` | 768 px | Single column with wider cards; top nav replaces bottom bar |
| `lg` | 1024 px | Side nav (240 px) + content (max 680 px) |
| `xl` | 1280 px | Side nav + content + aside (320 px) |
| `2xl` | 1536 px | Same as xl, centered max width 1440 px |

### C.5 Components

| Component | Variants / states | Accessibility notes |
|---|---|---|
| Button | primary (accent bg, on-accent text), secondary (surface + border-strong), ghost, danger, icon; sizes sm/md/lg; states hover, pressed, focus, disabled, loading | `aria-busy` when loading; icon buttons need `aria-label` |
| Input / Textarea | default, focus, error (warning-colored message + icon), disabled; character counter | Labels always visible; errors linked via `aria-describedby`; counters announced politely at 90% |
| Select / Combobox | single, multi (chips), async (company search) | Radix/cmdk combobox pattern; type-ahead |
| Checkbox / Radio / Switch | default, focus, disabled | Native semantics |
| Chip | category, tag, filter (selectable), status | Selected state uses icon + text, not color only |
| Badge | Advice, Lesson, Confession, Poll, OP, Anonymous (anon), Official representative (info), Unverified firsthand account (warning outline), Good experience (success), Held/Limited/Removed (author views) | Text always present |
| PostCard | default, compact (landing), with ContentWarningGate, members-only locked | Whole card is not a link; the title is the link (avoids nested interactive elements) |
| ReactionBar / ReactionPicker | idle, selected, disabled (visitor/unverified), counts | `role="radiogroup"`-like toolbar: arrow keys move, Space selects; each reaction has an accessible name, e.g. "Same Here, 27 reactions, selected" |
| ContentWarningGate | collapsed, revealed | Button "Show content: burnout, mental health"; focus moves to the revealed content |
| CommentThread | depth 0–2, deleted, hidden (blocked), pinned clarification, OP | Nested `<ul>` structure; "Reply" buttons labelled with the author |
| SafetyCheckPanel | none, low, medium, high, blocking | `aria-live="polite"` summary; each finding has a "Jump to text" button that focuses and selects the range |
| Dialog / AlertDialog | default, destructive | Focus trap, Esc closes (except destructive confirms), returns focus |
| Toast | success, info, warning, error | `role="status"`; never the only place critical info appears |
| Tabs | default | Radix Tabs (arrow-key navigation) |
| EmptyState | with illustration (simple line icons), copy, action | Illustrations `aria-hidden` |
| ErrorState | with retry, request ID | `role="alert"` |
| Skeleton | line, card, avatar | `aria-hidden`; container has `aria-busy="true"` |
| ThemeSummary | available, insufficient, paused | Each distribution bar has a text alternative "2 of 7 rated 1 (very poor)…" |
| Avatar / Monogram | image, monogram (first 2 letters of username on a deterministic color from a 6-color palette with ≥ 4.5:1 text contrast), anonymous (mask icon) | Decorative when the username is adjacent |
| Stepper | onboarding, confession wizard | `aria-current="step"`; step titles announced |

### C.6 Reactions

| Type | Label | Icon (lucide) + emoji fallback | Tooltip |
|---|---|---|---|
| `im_dead` | I'm Dead | `skull` 💀 | Funny or absurd |
| `pain` | Pain | `heart-crack` 😣 | That sounds difficult |
| `same_here` | Same Here | `hand` 🙋 | Similar experience |
| `melting` | Melting | `flame` 🫠 | Overwhelming situation |
| `corporate_circus` | Corporate Circus | `tent` 🎪 | Workplace absurdity |
| `survived` | Survived | `shield-check` 🛡️ | Respect and encouragement |

Counts appear only on the post. No profile shows totals.

### C.7 Voice, tone and copy rules

1. **Witty, never cruel.** Humor targets situations and systems, never people or the reader.
2. **Supportive defaults.** Empty states and confirmations reassure ("You're not the only one").
3. **Plain language for privacy.** No legalese in UI explanations; link to the full policy.
4. **Honest about anonymity.** Never use "untraceable", "100% anonymous" or "no one will ever know".
5. **Non-manipulative notifications (FR-66).** Notifications are factual. They never imply career failure, urgency, FOMO or negative feedback waiting ("Your career is slipping", "You won't believe what they said about you"). Allowed: "New reply to your post", "Moderation decision on your comment".
6. **No shaming numbers.** No streaks, "rejection counts" or "pain scores".
7. **Disclaimers.** Advice posts show the footer note "Peer advice, not professional legal, medical or mental-health advice." Content warnings for mental health include a link to a crisis resources page.

---

## Part D — Accessibility checklist (WCAG 2.2 AA)

| Area | Requirement | Verified by |
|---|---|---|
| Semantics | Landmarks (`header`, `nav`, `main`, `aside`, `footer`), one `h1` per page, logical heading order | axe + manual |
| Keyboard | All actions reachable; visible focus ring (`focus-ring`); skip link "Skip to content"; no keyboard traps; moderator queue shortcuts can be disabled | Playwright keyboard specs |
| Focus management | Route changes move focus to `h1`; dialogs trap and restore focus; revealed content receives focus | E2E |
| Contrast | Tokens in C.1 meet 4.5:1 text and 3:1 UI | Token unit test computes contrast ratios |
| Forms | Visible labels, inline errors, error summary on submit with links to fields, `autocomplete` attributes (email, new-password, current-password, username) | axe + RTL tests |
| Screen readers | Accessible names for reactions, icon buttons, badges; `aria-live` for safety findings, toasts, unread count changes | Manual NVDA + VoiceOver passes before beta and launch |
| Motion | `prefers-reduced-motion` disables transitions; no auto-play | Manual |
| Content warnings | Gated content hidden from screen readers until revealed (not just visually) | RTL tests |
| Zoom/reflow | Works at 320 px width and 200% zoom without horizontal scrolling (except code blocks) | Playwright viewport tests |
| Target size | ≥ 24 px minimum (2.5.8), 44 px on touch | Manual audit |
| Language | `lang="en"` on `<html>`; posts in other languages get `lang` attribute from post language | Unit test |
| Time limits | Session expiry warning not required (14-day idle); drafts autosave | — |
