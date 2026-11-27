# Tuli — Base UI Kit, Safety, Confessions & Member Space

## 1. Role summary

Tuli owns the **base UI kit** (`src/components/ui/**`, every component in TEAM_CONTRACT §8.1–§8.2). The kit's props are fixed by the contract, so teammates code against them from day 1 with local placeholders.

Tuli also owns the **safety components** other features embed: `SafetyCheckPanel`, `useSafetyCheck`, `ReportDialog` and `BlockMuteMenuItems`.

On top of that, Tuli owns the **privacy-sensitive member journeys**:
- Office Confessions (wizard and list);
- the public Anti-Profile and profile settings (including username change);
- privacy settings, and blocked & muted management;
- My Damage, the private dashboard;
- the Report & appeal center;
- the legal pages (Privacy Policy, Terms, Community Guidelines).

All features run on Tuli's own mock services.

## 2. Owned folders and files

All paths are under `apps/web/`.

| Path | Contents |
|---|---|
| `components.json` | shadcn/ui config |
| `src/components/ui/**` | Base kit (every component in contract §8.1–§8.2) |
| `src/app/dev/ui/**` | Kit preview page (dev builds only) |
| `src/features/safety/**` | SafetyCheckPanel, FindingList, useSafetyCheck, ReportDialog, BlockMuteMenuItems; safety mock API |
| `src/features/confessions/**` | ConfessionWizard (3 steps), IdentityModePicker, ConfessionPreview, ConfessionsListPage; confessions mock API |
| `src/features/profile/**` | PublicProfilePage, ProfileHeader, LessonsList, SkillsList, Timeline, ProfileSettingsPage, editors, UsernameChangeSection; profile mock API |
| `src/features/privacy/**` | PrivacySettingsPage, BlockedMutedPage; privacy mock API |
| `src/features/my-damage/**` | MyDamagePage (My posts, Drafts, Saved tabs); my-damage mock API |
| `src/features/reports/**` | ReportsCenterPage (Reports I made, Decisions about me), AppealDialog; reports mock API |
| `src/features/legal/**` | LegalPage and policy content (privacy, terms, guidelines) |
| `src/app/(marketing)/legal/**` | Policy routes |
| `src/app/(app)/confessions/**`, `src/app/(app)/u/**`, `src/app/(app)/me/**`, `src/app/(app)/reports/**`, `src/app/(app)/settings/profile/**`, `src/app/(app)/settings/privacy/**`, `src/app/(app)/settings/blocked/**` | Route files |
| `tests/e2e/member/**` | E2E specs |

## 3. Pages and screens

### 3.1 Base UI kit + `/dev/ui` preview
- **Purpose:** accessible, token-themed building blocks for every screen (UI_UX C.5). **FR/NFR:** NFR-A1–A5.
- **Components:** Button, IconButton, Input, Textarea, FormField, Select, Combobox, ChipMultiSelect, TagInput, Checkbox, RadioGroup, Switch, RatingInput, Card, Badge, Chip, Avatar, AuthorBadge, RelativeTime, Modal, ConfirmDialog, DropdownMenu, Tooltip, Popover, Tabs, Toast/Toaster/useToast, Spinner, Skeleton, EmptyState, ErrorState, Banner, Stepper, LoadMore, VisuallyHidden, PageHeader, DataTable (props exactly as contract §8).
- **UI details:** shadcn/ui (new-york) generated, then re-themed with tokens; no raw hex values; focus ring `focus-ring` 2 px + 2 px offset everywhere.
- **States:** each component's preview shows default, hover, focus, disabled, loading/invalid where relevant.
- **Responsive:** DataTable stacks into cards below `md`; Modal becomes a full-height sheet below `sm`.
- **Mock data:** none (preview uses inline sample props).
- **Actions:** `/dev/ui` lists every component with all variants so teammates can compare against their placeholders.

### 3.2 Safety components (cross-feature)
- **Purpose:** pre-publication privacy checks, reporting, blocking/muting. **US:** US-034, US-045, US-018, US-019. **FR:** FR-45, FR-44, FR-16, FR-17.
- **Components:**
  - `SafetyCheckPanel`: summary line by risk level ("Looks good" / "A few things to check" / "This will be reviewed before it's public" / "Remove this before posting"), `FindingList` (rule message, field, "Jump to text", acknowledgement checkbox for medium findings, blocking marker), and `aria-live` updates.
  - `useSafetyCheck`: debounced run on demand.
  - `ReportDialog`: reason radio list (9 reasons with plain-language descriptions), details textarea ≤ 1,000, submit; success "Thanks — our moderators will review this"; `ALREADY_EXISTS` → "You've already reported this".
  - `BlockMuteMenuItems`: block member, block anonymous label (explains "This only hides content under this label"), mute member, mute category; confirmations via ConfirmDialog.
- **States:** checking (spinner in panel); error ("We couldn't run the safety check. You can try again."); rate limit.
- **Mock data:** `safetyApi.check(input: SafetyCheckInput): SafetyCheckResult`. A rule engine in the mock detects emails, phone numbers, `AKIA…` keys (blocking), Luhn-valid card numbers (blocking), "my manager <Capitalized Name>" and two capitalized words (named individual, medium), and "the questions were" (interview dump). `safetyApi.report(input: ReportInput) → { reportId, status: 'open', message }`, `safetyApi.block(target) → Block`, `safetyApi.mute(target) → Mute`.

### 3.3 S-09 Confession submission — `/confessions/new`
- **Purpose:** anonymous storytelling with preview and confirmation. **US:** US-033, US-034, US-035. **FR:** FR-31, FR-32, FR-33.
- **Components:** `FlagGate confessions.enabled` (fallback "Confessions open soon — we're finishing our safety tools first."), `RequireAuth verified`, `ConfessionWizard` with `Stepper`:
  1. `WriteStep`: title, body (Textarea autoGrow, ≤ 20,000), experience type, category, desired response, content warnings, optional industry/country/year+quarter, tags; persistent tip panel.
  2. `CheckStep`: SafetyCheckPanel with `context="confession"`, `ConfessionPreview` using `MarkdownRenderer`/`ContentWarningGate` from `@/features/posts`, and a sample label note "your actual label is created when you publish"; `IdentityModePicker` (Anonymous to members — default from privacy settings / Public pseudonym @username) with a "What this means" disclosure; visibility (Members only default / Public).
  3. `ConfirmStep`: checklist ("I've removed names and identifying details", "I understand anonymity isn't absolute"); Publish; Save as private draft.
- **UI details:** Save draft is available on every step; Back keeps the data; publishing requires both checklist items.
- **States:** step validation errors; `SAFETY_CHECK_BLOCKED` keeps you on step 2; held → success screen "Your confession is waiting for a quick review (usually within 24 hours)" with a "View it" link (author view); published → success with the generated label + link; 429 → "You've reached today's confession limit"; restricted → banner.
- **Responsive:** single column; step titles collapse to "Step 2 of 3" on mobile; sticky action bar.
- **Mock data:** `confessionsApi.create(input: CreateConfessionInput) → { confession: PostDetail; moderationNotice }` (label from a list of office objects + 4-digit number, unique per confession; named individual → held); `confessionsApi.update(id, { publish: true, previewConfirmed: true })`.
- **Actions:** next/back/save draft/publish; "View it" → `/posts/[id]` (Shoaib's route via `routes.post`).

### 3.4 S-10 Confessions list — `/confessions`
- **Purpose:** browse confessions. **US:** US-033 (display). **FR:** FR-31.
- **Components:** PageHeader with "Confess anonymously" CTA, filters (experience type, category), list of `PostCard` (from `@/features/posts`), LoadMore.
- **States:** loading; empty "No confessions yet. The office is either perfect or very quiet."; error; members-only items locked for visitors (PostCard locked variant); flag off → coming-soon panel.
- **Mock data:** `confessionsApi.list({ experienceType, category, cursor }) → Page<PostSummary>` (12 confessions, 8 anonymous with distinct labels).

### 3.5 S-12 Public Anti-Profile — `/u/[username]`
- **Purpose:** a pseudonymous career reality profile. **US:** US-016. **FR:** FR-14, FR-10.
- **Components:** `ProfileHeader` (Avatar, username, display name, headline, career-status chip e.g. "Open to Escape", industry, country, "Member since"), tabs: Story (bio "What went wrong", `LessonsList` "Lessons and setbacks", `SkillsList` "Skills still under development", `Timeline` "Career survival history"), Posts (`PostCard` list), Comments (`ProfileCommentRow`: excerpt + link to the post), Communities (chips); overflow menu (Report profile → `ReportDialog` with `targetType: 'profile'`, Block, Mute). **No counts anywhere.**
- **States:** loading; restricted (`hidden` → "This profile is private"; `members` for visitors → "Sign in to see this profile"); not found / old username → "This member isn't here" (404 component); empty sections hidden for visitors, owner sees "Add your first lesson" prompts linking to `/settings/profile`.
- **Responsive:** header stacks on mobile; timeline is a vertical list at all sizes.
- **Mock data:** `profileApi.getProfile(username): PublicProfile` (public: `rejected_but_rising`; members-only: `survived_standups`; hidden: `deploy_on_friday`), `getProfilePosts(username, cursor): Page<PostSummary>`, `getProfileComments(username, cursor): Page<{ commentId, postId, postTitle, excerpt, createdAt }>`.

### 3.6 S-13 My Damage — `/me`
- **Purpose:** private member hub (plan screen 9). **US:** none owned here: it displays results of US-032 (Shoaib's save action), US-033 and US-035 (Tuli's confession stories). **FR:** FR-30, FR-32, FR-33.
- **Components:** `WelcomeCard` (new members: 3 starter actions), tabs **My posts** (PostCards with moderation-status chips and the "Only you can see that this is yours" chip on anonymous items), **Drafts** (draft rows with "Deletes on {date}", Continue → `/confessions/new?draft=<id>`, Delete), **Saved** (PostCards; unavailable items show "This post is no longer available" + Remove), **Decisions & reports** (link card to `/reports`).
- **States:** loading per tab; empties "Nothing saved yet. Tap the bookmark on stories you'll want later." / "No drafts. Brave." / "You haven't posted yet. The Mess awaits."; error.
- **Mock data:** `myDamageApi.getMyPosts({ status: 'published' | 'draft', cursor }) → Page<PostSummary>` (with `ownershipNote`, `moderationStatus`), `getSaved(cursor) → Page<PostSummary | { id; unavailable: true }>`, `removeSaved(id)`, `deleteDraft(id)`.

### 3.7 S-22 Report & appeal center — `/reports`
- **Purpose:** transparency on reports and decisions. **US:** US-046, US-047. **FR:** FR-44, FR-49.
- **Components:** tabs **Reports I made** (`ReportRow`: target summary, reason, status chip, dates, outcome message) and **Decisions about me** (`DecisionCard`: action type, target summary, policy reference linking to `/legal/guidelines#cg-x-y`, member message, appeal status/deadline, Appeal button → `AppealDialog` with reason ≤ 2,000 and deadline note).
- **States:** loading; empty "You haven't reported anything. Thanks for keeping it civil." / "No decisions about your content. Keep it up."; `ALREADY_EXISTS` → disabled with "You've already appealed this decision"; window closed → disabled with date; success toast "Appeal sent. A different moderator will review it."
- **Mock data:** `reportsApi.getMyReports(cursor) → Page<MyReport>` (4: open, actioned, no_violation, actioned), `getMyDecisions(cursor) → Page<MyModerationAction>` (3: one appealable, one appealed, one past window), `appeal(actionId, reason) → { appealId, status: 'pending' }`.

### 3.8 S-25 Settings → Profile — `/settings/profile`
- **Purpose:** edit the Anti-Profile, avatar and username. **US:** US-012, US-014, US-015. **FR:** FR-9, FR-12, FR-13, FR-68 (avatar).
- **Components:** `ProfileFieldsForm` (display name, headline ≤ 120, bio ≤ 600 with SafetyCheckPanel `context="profile"`, industry select, country select, career status select, `AvatarUploader` (Should, F-29: JPEG/PNG/WebP ≤ 5 MB, crop to square preview, "Checking image…" state, remove avatar; falls back to the monogram)), `LessonsEditor` (reorderable list with Move up/down buttons, max 10, ≤ 280 each), `SkillsEditor` (TagInput, max 20), `TimelineEditor` (add/edit/remove entries: start/end year+month, role category, industry, note), `UsernameChangeSection` (rules: once per 30 days, old name reserved 90 days; availability check; next-available date).
- **States:** loading; per-section save with success toast "Profile updated"; field errors; `SAFETY_CONFIRMATION_REQUIRED` → acknowledgement in the panel; username `change_too_soon` → date shown; `CONFLICT` → taken.
- **Responsive:** sections stack; timeline entries become cards on mobile.
- **Mock data:** `profileApi.getMyProfile() → PublicProfile & MeProfile`, `updateProfile(patch)`, `replaceLessons(string[])`, `replaceSkills(string[])`, `replaceTimeline(TimelineEntry[])`, `checkUsername(u) → UsernameAvailability`, `changeUsername(u) → { username, nextChangeAvailableAt }`, `uploadAvatar(file) → MediaItem` (pending → clean after 1.5 s), `removeAvatar()`; industries list (contract §7.3).

### 3.9 S-26 Settings → Privacy — `/settings/privacy`
- **Purpose:** control visibility and defaults. **US:** US-017. **FR:** FR-15.
- **Components:** profile visibility RadioGroup with a live "how others see you" preview card, discoverability Switch, default confession identity, default post visibility, message permissions (disabled with "Messaging isn't available yet"), "What's never public" info box.
- **States:** loading; saving; success toast; error.
- **Mock data:** `privacyApi.get(): PrivacySettings`, `privacyApi.update(patch)`.

### 3.10 S-28 Settings → Blocked & muted — `/settings/blocked`
- **Purpose:** manage blocks and mutes. **US:** US-018, US-019. **FR:** FR-16, FR-17.
- **Components:** Tabs Blocked (members and anonymous labels) / Muted (members, categories, tags), each row with Unblock/Unmute.
- **States:** loading; empty "You haven't blocked anyone." / "Nothing muted."; error; success toasts.
- **Mock data:** `privacyApi.listBlocks(): Block[]`, `unblock(id)`, `listMutes(): Mute[]`, `unmute(id)`.

### 3.11 Legal pages — `/legal/privacy`, `/legal/terms`, `/legal/guidelines`
- **Purpose:** publish the policies linked from onboarding, the footer and moderation decisions. **US:** none directly (content supports Arko's onboarding story). **FR:** FR-11.
- **Components:** `LegalPage` (PageHeader, "Last updated", table of contents with anchor links, prose). Content comes from SECURITY.md §9–§11 and the PRD disclaimers. Guidelines list CG-1.1…CG-6.2 with anchors (`#cg-2-1`) so decision cards in S-22 and moderation messages deep-link.
- **States:** static content (no loading); unknown anchor → top of page.
- **Responsive:** TOC is a sticky sidebar from `lg` and a collapsible `<details>` on mobile.
- **Mock data:** none (content in `features/legal/content/*.ts`).
- **Actions:** TOC link → smooth scroll (instant under reduced motion) and focus the heading.

## 4. Task checklist (build order)

### Phase A — Base UI kit (shared-contract; nobody waits for it)
- [ ] **FE-T-01** · 2h · `components.json`, shadcn init, token theming conventions, `cn` usage; `Button`, `IconButton`. **AC:** all variants/sizes/states from contract §8.1.
- [ ] **FE-T-02** · 3h · `Input`, `Textarea` (autoGrow, counter), `FormField`, `Checkbox`, `RadioGroup`, `Switch`. **AC:** labels and `aria-describedby` wiring; invalid states.
- [ ] **FE-T-03** · 4h · `Select`, `Combobox` (async, create option), `ChipMultiSelect`, `TagInput`, `RatingInput`. **AC:** full keyboard support; max limits.
- [ ] **FE-T-04** · 3h · `Card`, `Badge`, `Chip`, `Avatar` (monogram palette), `AuthorBadge`, `RelativeTime`. **AC:** AuthorBadge never links anonymous authors.
- [ ] **FE-T-05** · 3h · `Modal`, `ConfirmDialog` (typed confirmation), `DropdownMenu`, `Tooltip`, `Popover`. **AC:** focus trap and restore; Esc behavior.
- [ ] **FE-T-06** · 3h · `Tabs`, `Toast`/`Toaster`/`useToast`, `Spinner`, `Skeleton`, `Banner`. **AC:** toast `role="status"`, pause on hover/focus.
- [ ] **FE-T-07** · 3h · `EmptyState`, `ErrorState` (requestId), `Stepper`, `LoadMore` (auto + button), `VisuallyHidden`, `PageHeader`, `DataTable` (mobile cards). **AC:** contract props exactly.
- [ ] **FE-T-08** · 2h · `/dev/ui` preview page listing every component and variant. **AC:** page renders all components with no console errors.

### Phase B — Safety components
- [ ] **FE-T-09** · 3h · `safety` mock API with the rule engine and fixtures. **AC:** detects the cases in §3.2 with correct ranges and severities.
- [ ] **FE-T-10** · 4h · `SafetyCheckPanel` + `FindingList` + `useSafetyCheck`. **AC:** US-034 criteria 1–4; acknowledgement callback; jump-to callback.
- [ ] **FE-T-11** · 3h · `ReportDialog`. **AC:** US-045 criteria 1–3.
- [ ] **FE-T-12** · 2h · `BlockMuteMenuItems`. **AC:** US-018 criteria 1–2, US-019 criterion 2 (via `onDone`).

### Phase C — Confessions
- [ ] **FE-T-13** · 2h · `confessions` mock API + fixtures (12 confessions, drafts, label generator).
- [ ] **FE-T-14** · 3h · Wizard step 1 (Write) with schema and tips. **AC:** required fields; period rules (quarter needs a year).
- [ ] **FE-T-15** · 4h · Wizard step 2 (Safety check & preview, IdentityModePicker, visibility). **AC:** US-034 criteria; preview matches the final rendering.
- [ ] **FE-T-16** · 3h · Wizard step 3 (Confirm, publish, save draft), success/held screens, draft resume (`?draft=`), flag gate. **AC:** US-033 criteria 1–5, US-035 criteria 1 and 3.
- [ ] **FE-T-17** · 2h · Confessions list S-10. **AC:** filters, locked items, empty and error states.

### Phase D — Profile, privacy, My Damage, reports, legal
- [ ] **FE-T-18** · 3h · `profile` mock API + fixtures (3 visibility cases, posts, comments, my profile).
- [ ] **FE-T-19** · 4h · Public Anti-Profile S-12 (header, Story tab, Posts/Comments/Communities tabs, restricted and not-found states). **AC:** US-016 criteria 1–5.
- [ ] **FE-T-20** · 3h · Settings → Profile fields + safety panel for the bio. **AC:** US-014 criteria 1–3.
- [ ] **FE-T-21** · 4h · Lessons, skills and timeline editors. **AC:** US-015 criteria 1–3.
- [ ] **FE-T-22** · 2h · Username change section. **AC:** US-012 criteria 1–2.
- [ ] **FE-T-23** · 2h · `AvatarUploader` in Settings → Profile (Should). **AC:** type/size validation, scan state, remove avatar, monogram fallback; FR-68 avatar path.
- [ ] **FE-T-24** · 2h · `privacy` mock API + Settings → Privacy S-26. **AC:** US-017 criteria 1–2.
- [ ] **FE-T-25** · 2h · Settings → Blocked & muted S-28. **AC:** US-018 criterion 3, US-019 criterion 3.
- [ ] **FE-T-26** · 3h · `my-damage` mock API + My Damage S-13 (My posts, Drafts, Saved). **AC:** unavailable saved posts handled (US-032 criteria 2–3 display); drafts show expiry date (US-035 criterion 2).
- [ ] **FE-T-27** · 3h · `reports` mock API + Report & appeal center S-22 + AppealDialog. **AC:** US-046 criteria, US-047 criteria 1–2.
- [ ] **FE-T-28** · 3h · Legal pages (privacy, terms, guidelines with CG anchors). **AC:** content matches SECURITY §9–§11; `#cg-x-y` anchors work; TOC accessible.

### Phase E — Testing and accessibility
- [ ] **FE-T-29** · 3h · Kit tests: Button, Input/FormField, Combobox, Modal/ConfirmDialog, Toast, Tabs, DataTable, Avatar/AuthorBadge.
- [ ] **FE-T-30** · 3h · Safety tests: SafetyCheckPanel (risk levels, acknowledgement, blocking), ReportDialog, BlockMuteMenuItems.
- [ ] **FE-T-31** · 3h · Feature tests: ConfessionWizard flow, profile visibility states, editors limits, AppealDialog.
- [ ] **FE-T-32** · 3h · E2E (`tests/e2e/member`): anonymous confession (no username visible anywhere on the page), profile privacy states, report + appeal; axe on S-09, S-12, S-13, S-22, S-25, `/dev/ui`. **AC:** green on chromium + mobile-chrome.
- [ ] **FE-T-33** · 2h · Accessibility and responsive pass on all owned screens and kit. **AC:** 0 axe serious/critical; 320 px OK.

## 5. Mock data and mock services

| File | Exports | Returns |
|---|---|---|
| `src/features/safety/api/mock.ts` | `mockSafetyApi`: `check`, `report`, `block`, `mute` | `SafetyCheckResult`, `{reportId,status,message}`, `Block`, `Mute` |
| `src/features/confessions/api/mock.ts` | `mockConfessionsApi`: `create`, `update`, `remove`, `list`, `get` | `{confession: PostDetail; moderationNotice}`, `Page<PostSummary>`, `PostDetail` |
| `src/features/profile/api/mock.ts` | `mockProfileApi`: `getProfile`, `getProfilePosts`, `getProfileComments`, `getMyProfile`, `updateProfile`, `replaceLessons`, `replaceSkills`, `replaceTimeline`, `checkUsername`, `changeUsername`, `uploadAvatar`, `removeAvatar` | `PublicProfile`, `Page<PostSummary>`, `Page<…>`, `UsernameAvailability` |
| `src/features/privacy/api/mock.ts` | `mockPrivacyApi`: `get`, `update`, `listBlocks`, `unblock`, `listMutes`, `unmute` | `PrivacySettings`, `Block[]`, `Mute[]` |
| `src/features/my-damage/api/mock.ts` | `mockMyDamageApi`: `getMyPosts`, `getSaved`, `removeSaved`, `deleteDraft` | `Page<PostSummary>` |
| `src/features/reports/api/mock.ts` | `mockReportsApi`: `getMyReports`, `getMyDecisions`, `appeal` | `Page<MyReport>`, `Page<MyModerationAction>`, `{appealId,status}` |

Fixtures use the canonical users and post IDs (contract §7.3). Confessions 31–35 match Shoaib's post IDs, so "View it" links resolve after integration.

## 6. Testing tasks

| Test file | What to test |
|---|---|
| `components/ui/__tests__/*.test.tsx` | Variants render; disabled/loading semantics; focus trap; typed confirmation; toast live region; DataTable keyboard and mobile mode |
| `safety/__tests__/safety-check-panel.test.tsx` | Summary per risk level, medium acknowledgement required, blocking disables, jump-to callback |
| `safety/__tests__/report-dialog.test.tsx` | Reason required, details limit, success and `ALREADY_EXISTS` |
| `confessions/__tests__/confession-wizard.test.tsx` | Step validation, identity default from privacy, checklist gating, held vs published result, draft save |
| `profile/__tests__/public-profile.test.tsx` | Public/members/hidden/not-found; no counts rendered; anonymous items never listed |
| `profile/__tests__/editors.test.tsx` | 10 lessons / 20 skills limits, timeline validation, username change rules |
| `reports/__tests__/appeal-dialog.test.tsx` | Eligible vs closed vs already appealed; reason limit |

## 7. Accessibility checklist

- [ ] Every kit component meets contract §8 accessibility notes; verified in `/dev/ui` with keyboard only.
- [ ] Focus ring visible on every interactive kit element; no `outline: none` without a replacement.
- [ ] Dialogs: labelled by title, described by description, focus trap, Esc, focus return.
- [ ] SafetyCheckPanel: summary in `aria-live="polite"`; findings are a list; acknowledgement checkboxes labelled with the finding message.
- [ ] Wizard: `aria-current="step"`, step heading receives focus on change, error summary per step.
- [ ] IdentityModePicker: radio group with descriptions linked via `aria-describedby`.
- [ ] Profile tabs: Radix Tabs arrow keys; restricted states announced as headings.
- [ ] Editors: reorder via buttons (not drag only); each row's controls labelled with the item text.
- [ ] Contrast through tokens only; monogram palette checked at ≥ 4.5:1.

## 8. Integration notes (mock → real API)

| Mock function | Real endpoint |
|---|---|
| `safetyApi.check` | `POST /safety/check` |
| `safetyApi.report` | `POST /reports` |
| `safetyApi.block` / `mute` | `POST /me/blocks` / `POST /me/mutes` |
| `confessionsApi.create` | `POST /confessions` |
| `confessionsApi.list` / `get` | `GET /confessions` / `GET /confessions/:confessionId` |
| `confessionsApi.update` / `remove` | `PATCH` / `DELETE /confessions/:confessionId` |
| `profileApi.getProfile` / `getProfilePosts` / `getProfileComments` | `GET /profiles/:username` / `…/posts` / `…/comments` |
| `profileApi.getMyProfile` | `GET /me` (profile part) |
| `profileApi.updateProfile` | `PATCH /me/profile` |
| `profileApi.replaceLessons` / `replaceSkills` / `replaceTimeline` | `PUT /me/profile/lessons` / `skills` / `timeline` |
| `profileApi.checkUsername` / `changeUsername` | `GET /auth/username-availability` / `PATCH /me/username` |
| `profileApi.uploadAvatar` / `removeAvatar` | `POST /media/uploads` (`purpose: avatar`) → PUT presigned URL → `POST /media/:mediaId/complete` → `PATCH /me/profile { avatarMediaId }` / `PATCH /me/profile { avatarMediaId: null }` |
| `privacyApi.get` / `update` | `GET` / `PATCH /me/privacy` |
| `privacyApi.listBlocks` / `unblock` | `GET /me/blocks` / `DELETE /me/blocks/:blockId` |
| `privacyApi.listMutes` / `unmute` | `GET /me/mutes` / `DELETE /me/mutes/:muteId` |
| `myDamageApi.getMyPosts` | `GET /me/posts?status=` |
| `myDamageApi.getSaved` / `removeSaved` | `GET /me/saved` / `DELETE /posts/:postId/save` |
| `myDamageApi.deleteDraft` | `DELETE /confessions/:confessionId` |
| `reportsApi.getMyReports` | `GET /me/reports` |
| `reportsApi.getMyDecisions` | `GET /me/moderation-actions` |
| `reportsApi.appeal` | `POST /moderation/actions/:actionId/appeal` |

## 9. Independence check

This work depends on **no other teammate's work**. Tuli owns the base kit, so there is nothing to wait for there. Local placeholders (same paths and props, excluded from git) are used only for:
- `@/shared/**` and the scaffold (Arko): copied from contract §2.3, §5–§10;
- `PostCard`, `PostCardSkeleton`, `MarkdownRenderer`, `ContentWarningGate` from `@/features/posts` (Shoaib): placeholders that render the title/excerpt/body text with the contract props.

## 10. Total estimated hours

**94 hours** across 33 tasks (Base UI kit 23 h · Safety components 12 h · Confessions 14 h · Profile, privacy, My Damage, reports, legal 31 h · Testing and accessibility 14 h).
