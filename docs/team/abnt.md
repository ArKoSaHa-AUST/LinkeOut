# Abnt — Workplace Reality, Job Nightmares, Communities & Admin

## 1. Role summary

Abnt owns the **workplace transparency and community features**, plus the **admin console**:
- the company directory, Workplace Reality Pages (theme summaries, reviews, job reports), the review form, company submission, and the representative request with official responses;
- the Job Nightmare Board and report form;
- communities (directory, community page, creation, community moderation tab);
- the admin dashboard and every admin sub-page.

Abnt also exports `CompanyCard`, `JobReportCard` and `CommunityCard` for Shoaib's search page; their props are fixed in the contract. Everything runs on Abnt's own mock services.

## 2. Owned folders and files

All paths are under `apps/web/`.

| Path | Contents |
|---|---|
| `src/features/companies/**` | Directory, CompanyCard, CompanyHeader, ThemeSummary, ReviewCard, ReviewFilters, ReviewForm, SubmitCompanyForm, RepresentPage, CompanyResponseEditor; companies mock API |
| `src/features/job-nightmares/**` | BoardPage, JobReportCard, JobReportDetail, JobReportForm, CompanyCombobox; job-nightmares mock API |
| `src/features/communities/**` | Directory, CommunityCard, CommunityPage, CreateCommunityForm, CommunityModTab; communities mock API |
| `src/features/admin/**` | AdminDashboard, Users, Categories, Companies, Representatives, Escalations, FeatureFlags, AuditLog, Invites; admin mock API |
| `src/app/(app)/companies/**`, `src/app/(app)/job-nightmares/**`, `src/app/(app)/communities/**`, `src/app/admin/**` | Route files |
| `tests/e2e/workplace/**` | E2E specs |

## 3. Pages and screens

### 3.1 S-15 Company directory — `/companies`
- **Purpose:** find a company. **US:** US-003 (entry), US-041 (entry). **FR:** FR-36.
- **Components:** PageHeader, search input (debounced 300 ms, min 2 chars), industry Select, `CompanyCard` list (name, industry, review and job report counts; with `?intent=review` a "Review this company" button), "Can't find it? Add a company" link, LoadMore.
- **States:** loading list skeleton; no results "No company called '{q}' yet." + "Add it"; error.
- **Responsive:** cards 1 column on mobile, 2 columns from `md` (within 680 px they stay 1 column at `lg`).
- **Mock data:** `companiesApi.list({ q, industry, cursor }) → Page<CompanySummary>` (12 companies including acme-widgets, globex-analytics; initech-systems is pending and never listed).
- **Actions:** type → URL `?q=`; card → `/companies/[slug]`; Review → `/companies/[slug]/review`.

### 3.2 S-16 Workplace Reality Page — `/companies/[slug]`
- **Purpose:** balanced employer information. **US:** US-003, US-040, US-068 (response display). **FR:** FR-36, FR-39, FR-40, FR-58.
- **Components:** `CompanyHeader` (name, industry, confirmed website with external-link icon, description, "Official representative active" badge), persistent `Banner` "Reviews and reports are unverified firsthand accounts from members. Read them as experiences, not facts.", `ThemeSummary` (5 topics: median + 5-bar distribution with text alternative, n and "last 24 months"; "Not enough reports yet" when insufficient; "Summaries paused while we review recent activity" when paused), Tabs **Workplace** | **Interview & hiring**, `ReviewFilters` (topic, relationship, period from/to), `ReviewCard` (relationship, period, title, narrative or "The author chose not to show details publicly", ratings, "Unverified firsthand account" badge, company response block with Official badge, Report → `ReportDialog`, "Request correction" → `ReportDialog` preset to `false_or_manipulated`), `JobReportCard` list, CTAs "Write a review" / "Report a job nightmare", "Represent this company?" link, and for representatives a "Respond" button per review → `CompanyResponseEditor`.
- **States:** loading header + summary + list skeletons; not found / pending (non-submitter) → 404; pending (submitter) → "Waiting for approval" banner; merged → redirect to the target slug; empty reviews "No workplace reports yet. If you've worked here, your experience helps the next person."; filter empty "No reports match these filters" + Clear; error.
- **Responsive:** theme summary bars become a stacked list on mobile; filters in a sheet on mobile.
- **Mock data:** `companiesApi.getCompany(idOrSlug) → Company` (acme-widgets: themes available, 7 reviews, representative active; globex-analytics: insufficient; `paused-co`: paused; initech-systems: pending, submitter `rejected_but_rising`; `old-acme` → merged into acme-widgets), `listReviews(companyId, filters) → Page<Review>`, `listCompanyJobReports(companyId) → Page<JobReport>`.
- **Actions:** filter → URL; tab switch → URL `?tab=`; report → ReportDialog; respond (rep) → editor.

### 3.3 S-17 Write workplace review — `/companies/[slug]/review`
- **Purpose:** structured firsthand review. **US:** US-039. **FR:** FR-38.
- **Components:** `RequireAuth verified`, `ReviewForm`: relationship Select, period start/end years with an "I still work here" Switch, 5 `RatingInput` rows (skippable), title, narrative (min 150 with counter), tips panel "Describe practices and patterns, not individuals", firsthand attestation Checkbox (required), display consent Switch with explanation, `SafetyCheckPanel context="workplace_review"`, Submit.
- **States:** account < 7 days (`ACCOUNT_TOO_NEW`, mock user `unverified`-adjacent `fresh_intern_42` after verification) → explanation panel with date; already reviewed → redirect into edit mode with existing values; held → confirmation with review notice; success → company page with toast "Thanks. Your review helps the next person."; validation errors inline.
- **Mock data:** `companiesApi.getMyReview(companyId) → Review | null`, `createReview(companyId, input: ReviewInput) → Review`, `updateReview(reviewId, input)`, `deleteReview(reviewId)`.
- **Actions:** submit → create/update; Delete review (edit mode) → ConfirmDialog.

### 3.4 S-18 Submit a company — `/companies/new`
- **Purpose:** add a missing company. **US:** US-041. **FR:** FR-37.
- **Components:** form (name, industry, website), live duplicate suggestions ("Did you mean Acme Widgets?").
- **States:** `CONFLICT` with `details[0].existing` → "This company already exists" + link; success → "Thanks! We'll check it soon. You can already write your review — it'll appear once the company is approved." + "Write review" button.
- **Mock data:** `companiesApi.submitCompany(input: CompanySubmitInput) → { id, slug, name, pageStatus: 'pending' }`, `suggestDuplicates(name) → CompanyRef[]`.

### 3.5 S-32 Representative request + responses — `/companies/[slug]/represent` (Should)
- **Purpose:** company right of reply. **US:** US-067, US-068, US-069. **FR:** FR-56, FR-57, FR-58.
- **Components:** explainer ("You can respond. You can't remove reviews or see who wrote them."), form (work email, job title), status panel (pending_email → "Check your work inbox", pending_review → "An admin will review your request", verified → "You can now respond on reviews"); `CompanyResponseEditor` (Modal on S-16: textarea ≤ 3,000, SafetyCheckPanel `context="company_response"`, edit existing).
- **States:** `domain_mismatch` → inline error with the expected domain; website not confirmed → explanation; existing request → status panel; response `CONFLICT` → "Edit your existing response"; held response → notice.
- **Mock data:** `companiesApi.requestRepresentation(companyId, { workEmail, jobTitle }) → RepresentationRequest`, `getMyRepresentation(companyId)`, `createResponse(companyId, { targetType, targetId, body, acknowledgedFindings }) → CompanyResponse`, `updateResponse(id, body)`, `deleteResponse(id)`.

### 3.6 S-19 Job Nightmare Board — `/job-nightmares`, `/job-nightmares/[jobReportId]`
- **Purpose:** recruitment experience reports. **US:** US-043. **FR:** FR-42.
- **Components:** report-type chips with icons (8 types), company filter (`CompanyCombobox`), role category Select, period from/to, `JobReportCard` (company link, AuthorBadge, role category, stage-reached chips, type badge, outcome, month, "Unverified firsthand account", excerpt; "Good experience" success badge for positive reports; menu: Report, Block label/member via `BlockMuteMenuItems`), "Report a job nightmare" CTA; detail page with the full narrative, company response (Should), report link and company link.
- **States:** loading; empty "No nightmares in this category. Sweet dreams (for now)."; error; detail not found → 404.
- **Mock data:** `jobReportsApi.list({ companyId, reportType, roleCategory, from, to, cursor }) → Page<JobReport>` (16 reports covering all 8 types, anonymous and pseudonymous), `get(id) → JobReport`.

### 3.7 S-20 Job Nightmare report form — `/job-nightmares/new`
- **Purpose:** submit a recruitment experience. **US:** US-042. **FR:** FR-41.
- **Components:** `RequireAuth verified`; `CompanyCombobox` (search directory, or "Add a new company" which expands inline name/industry/website fields), role category, month picker (no future months), stage-reached stepper selector, report type RadioGroup, outcome Select (optional), narrative (≥ 100) with tip "Don't share confidential interview questions or recruiter details", identity mode (Anonymous default / Pseudonym), visibility, `SafetyCheckPanel context="job_report"`, Submit.
- **States:** validation; `CONFLICT` on new company → "Use the existing company" action; held notice; success → detail page + toast.
- **Mock data:** `jobReportsApi.create(input: JobReportInput) → JobReport`, `searchCompanies(q) → CompanyRef[]`.

### 3.8 S-33 Communities directory — `/communities` (Should)
- **Purpose:** discover communities. **US:** US-049 (entry). **FR:** FR-59, FR-60.
- **Components:** `FlagGate communities.enabled`, search, `CommunityCard` (name, description, member count, Public/Private badge, Join / Request to join / Joined / Requested), featured seeded communities, "Create a community" CTA.
- **States:** loading; empty search "No community yet. Start one?"; error; join success toasts; `BLOCKED`/banned → disabled with a note.
- **Mock data:** `communitiesApi.list({ q, cursor }) → Page<CommunitySummary>` (7 canonical communities), `join(id, message?)`, `leave(id)`.

### 3.9 S-34 Community page — `/communities/[slug]` (Should)
- **Purpose:** a community feed with rules and resources. **US:** US-049, US-050, US-051, US-058, US-059. **FR:** FR-60–FR-63.
- **Components:** header (name, description, visibility, member count, Join/Leave/Request), `RulesPanel`, `PinnedResources`, community feed (`PostCard` list from `@/features/posts`), "Post in this community" → `routes.newPost()` + `?community=<id>`, moderators list; **Moderation tab** (`CommunityModTab`, community moderators only): scoped queue (items with reason, content preview, actions No action / Remove from community / Add content warning / Ban from community, each needing a reason), join requests (approve/decline), rules editor (≤ 10), resources editor (≤ 5), announcement form, invite by username.
- **States:** private for non-members → header, rules and "Request to join" only; request pending → "Requested"; declined < 30 days → disabled with a retry date; leave as owner → "Transfer ownership first"; loading/empty/error per section.
- **Responsive:** rules and resources move above the feed on mobile; mod tab tables become cards.
- **Mock data:** `communitiesApi.get(slug) → Community`, `getPosts(id, cursor) → Page<PostSummary>`, `getMembers(id)`, `getModQueue(id) → Page<QueueItem>`, `applyModAction(id, itemId, { actionType, reason })`, `listJoinRequests(id) → JoinRequest[]`, `decideJoinRequest(id, requestId, decision)`, `updateRules(id, rules)`, `addResource(id, r)`, `removeResource(id, resourceId)`, `postAnnouncement(id, { title, body })`, `invite(id, username)`.

### 3.10 S-35 Create community — `/communities/new` (Should)
- **Purpose:** start a support community. **US:** US-048. **FR:** FR-59.
- **Components:** name, slug (auto from name, editable, availability), description, visibility RadioGroup, rules editor (≥ 1), acknowledgement Checkbox "Communities are for support, not coordinated attacks".
- **States:** `ACCOUNT_TOO_NEW` (< 14 days) panel; `CONFLICT` slug "That address is taken"; `owned_limit` "You already own 3 communities"; success → community page.
- **Mock data:** `communitiesApi.create(input: CreateCommunityInput) → Community`, `checkSlug(slug) → { available: boolean }`.

### 3.11 S-30 Admin dashboard — `/admin`
- **Purpose:** trust and growth overview. **US:** US-074. **FR:** FR-55.
- **Components:** `RequireRole admin`; KPI tiles (sign-ups, verified members, posts/day, reports per 1,000 posts, open queue, median response time high/normal, appeal reversal rate) with a 7d/30d toggle; links to sub-pages; system status card (queue depths, worker heartbeat, from mock).
- **States:** loading tiles; error; non-admin → 404.
- **Mock data:** `adminApi.getMetrics(window) → AdminMetrics`, `getSystemStatus() → { queues: Array<{ name: string; depth: number; oldestSeconds: number }>; workerHeartbeatAt: string }`.

### 3.12 S-31 Admin sub-pages — `/admin/*`
- **Purpose:** manage users, taxonomy, companies, representatives, escalations, flags, audit, invites. **US:** US-070, US-071, US-072, US-073, US-075. **FR:** FR-50, FR-52, FR-53, FR-54, FR-57.
- **Components:**
  - **Users:** search by email/username, DataTable, role Select with ConfirmDialog, permission toggle `identity_vault.reveal` (only for moderators/admins), `last_admin` error.
  - **Categories:** DataTable with Move up/down + drag, add/edit Modal, archive toggle.
  - **Companies:** pending queue with duplicate hints, approve/reject/edit Modal (name, industry, website, website confirmed, description), merge Modal (choose target via combobox).
  - **Representatives** (Should): pending list with work email domain and verification date, approve/reject/revoke.
  - **Escalations:** list of escalated items linking to `/mod/items/[id]` via `routes.mod.item`.
  - **Feature flags:** Switch per flag; enabling `confessions.enabled` opens the launch-gate checklist Modal (all ROADMAP §4 items must be ticked).
  - **Audit log:** filters (action, actor, target type, from/to), read-only table, metadata JSON viewer in a Modal.
  - **Invites:** generate beta codes (count, max uses, expiry) and copy the list.
- **States:** loading/empty/error per page; success toasts; validation errors; destructive actions confirmed.
- **Responsive:** wide layout; tables become cards on mobile.
- **Mock data:** `adminApi.listUsers(q) → Page<AdminUser>`, `setRole`, `grantPermission`, `revokePermission`; `listCategories`, `createCategory`, `updateCategory`, `reorderCategories`; `listCompanies(status) → Page<AdminCompany>`, `updateCompany`, `mergeCompany`; `listMemberships(status) → Page<AdminMembership>`, `decideMembership`; `listEscalations() → Page<QueueItem>`; `listFlags() → FeatureFlag[]`, `setFlag(key, enabled, launchGateConfirmed?)`; `listAuditLogs(filters) → Page<AuditLogEntry>`; `createInvites(input) → BetaInviteBatch`.

## 4. Task checklist (build order)

### Phase A — Companies
- [ ] **FE-N-01** · 3h · `companies` mock API + fixtures (12 companies incl. pending/merged/paused, 25 reviews, responses, representation states). **AC:** shapes match `Company`/`Review`; themes computed only when n ≥ 5.
- [ ] **FE-N-02** · 3h · `CompanyCard` (exported) + directory S-15 with search, industry filter, intent mode. **AC:** US-041 entry flow; empty and error states.
- [ ] **FE-N-03** · 2h · `CompanyHeader` + notice banner + not-found/pending/merged handling.
- [ ] **FE-N-04** · 3h · `ThemeSummary` (available, insufficient, paused; text alternatives). **AC:** US-003 criteria 1–2; FR-48 paused banner.
- [ ] **FE-N-05** · 4h · `ReviewCard`, `ReviewFilters`, tabs, review list with LoadMore. **AC:** US-003 criteria 3–4, US-040 criteria 1–2.
- [ ] **FE-N-06** · 4h · `ReviewForm` S-17 (ratings, attestation, consent, safety panel, edit mode, account-age gate). **AC:** US-039 criteria 1–5.
- [ ] **FE-N-07** · 2h · Submit company S-18 with duplicate suggestions. **AC:** US-041 criteria 1–2.
- [ ] **FE-N-08** · 3h · Representative request S-32 (Should). **AC:** US-067 criteria 1–3 (statuses mocked).
- [ ] **FE-N-09** · 3h · `CompanyResponseEditor` + response display on cards (Should). **AC:** US-068 criteria 1–3; US-069 (no author data anywhere).

### Phase B — Job Nightmare Board
- [ ] **FE-N-10** · 2h · `job-nightmares` mock API + fixtures (16 reports, all types).
- [ ] **FE-N-11** · 3h · `JobReportCard` (exported) + board S-19 with filters. **AC:** US-043 criteria 1–2; "Good experience" badge.
- [ ] **FE-N-12** · 2h · Job report detail page. **AC:** company response block; 404.
- [ ] **FE-N-13** · 4h · `CompanyCombobox` + report form S-20 (inline new company, month picker, stage selector). **AC:** US-042 criteria 1–3.

### Phase C — Communities (Should)
- [ ] **FE-N-14** · 3h · `communities` mock API + fixtures (7 communities, members, join requests, mod queue, rules, resources).
- [ ] **FE-N-15** · 3h · `CommunityCard` (exported) + directory S-33 with join/request. **AC:** US-049 criteria 1–2.
- [ ] **FE-N-16** · 4h · Community page S-34 (header, rules, resources, feed, private view, leave rules). **AC:** US-049 criterion 3, US-050, US-051 criteria.
- [ ] **FE-N-17** · 3h · Create community S-35. **AC:** US-048 criteria 1–3.
- [ ] **FE-N-18** · 4h · `CommunityModTab`: scoped queue actions, join requests. **AC:** US-058 criteria 1–4 (403 simulated for non-moderators).
- [ ] **FE-N-19** · 3h · Rules editor, resources editor (max 5), announcements, invitations. **AC:** US-059 criteria 1–2.

### Phase D — Admin console
- [ ] **FE-N-20** · 3h · `admin` mock API + fixtures (metrics, 30 users, categories, companies, memberships, escalations, flags, 50 audit entries).
- [ ] **FE-N-21** · 3h · Admin dashboard S-30. **AC:** US-074 criterion 1; 7d/30d toggle.
- [ ] **FE-N-22** · 3h · Users page (search, role change, permission grant, last-admin error). **AC:** US-070 criteria 1–3.
- [ ] **FE-N-23** · 3h · Categories page (reorder with buttons and drag, add/edit/archive). **AC:** US-071 criteria 1–2.
- [ ] **FE-N-24** · 4h · Companies admin page (approve/reject/edit/merge). **AC:** US-072 criteria 1–2.
- [ ] **FE-N-25** · 2h · Representatives page (Should). **AC:** US-073 criteria 1–2.
- [ ] **FE-N-26** · 3h · Escalations + feature flags with the launch-gate Modal + invites. **AC:** gate blocks enabling until all items are ticked.
- [ ] **FE-N-27** · 3h · Audit log page with filters and the metadata viewer. **AC:** US-075 criteria 1–2 (read-only; no edit controls).

### Phase E — Testing and accessibility
- [ ] **FE-N-28** · 3h · Component tests: ThemeSummary, ReviewCard, ReviewForm, CompanyResponseEditor.
- [ ] **FE-N-29** · 3h · Component tests: JobReportForm/CompanyCombobox, CommunityModTab, CreateCommunityForm.
- [ ] **FE-N-30** · 3h · Component tests: admin Users, Categories reorder, FeatureFlags launch gate, AuditLog filters.
- [ ] **FE-N-31** · 3h · E2E (`tests/e2e/workplace`): research company + write review; job report; join community and post; admin approves a company; axe on S-15, S-16, S-17, S-19, S-20, S-34, S-30. **AC:** green on chromium + mobile-chrome.
- [ ] **FE-N-32** · 2h · Accessibility and responsive pass on all owned screens. **AC:** 0 axe serious/critical; 320 px OK.

## 5. Mock data and mock services

| File | Exports | Returns |
|---|---|---|
| `src/features/companies/api/mock.ts` | `mockCompaniesApi`: `list`, `getCompany`, `listReviews`, `listCompanyJobReports`, `getMyReview`, `createReview`, `updateReview`, `deleteReview`, `submitCompany`, `suggestDuplicates`, `requestRepresentation`, `getMyRepresentation`, `createResponse`, `updateResponse`, `deleteResponse` | `Page<CompanySummary>`, `Company`, `Page<Review>`, `Page<JobReport>`, `Review`, `CompanyRef[]`, `RepresentationRequest`, `CompanyResponse` |
| `src/features/job-nightmares/api/mock.ts` | `mockJobReportsApi`: `list`, `get`, `create`, `update`, `remove`, `searchCompanies` | `Page<JobReport>`, `JobReport`, `CompanyRef[]` |
| `src/features/communities/api/mock.ts` | `mockCommunitiesApi`: `list`, `get`, `create`, `checkSlug`, `join`, `leave`, `getPosts`, `getMembers`, `getModQueue`, `applyModAction`, `listJoinRequests`, `decideJoinRequest`, `updateRules`, `addResource`, `removeResource`, `postAnnouncement`, `invite` | `Page<CommunitySummary>`, `Community`, `Page<PostSummary>`, `CommunityMember[]`, `Page<QueueItem>`, `JoinRequest[]` |
| `src/features/admin/api/mock.ts` | `mockAdminApi`: `getMetrics`, `getSystemStatus`, `listUsers`, `setRole`, `grantPermission`, `revokePermission`, `listCategories`, `createCategory`, `updateCategory`, `reorderCategories`, `listCompanies`, `updateCompany`, `mergeCompany`, `listMemberships`, `decideMembership`, `listEscalations`, `listFlags`, `setFlag`, `listAuditLogs`, `createInvites` | `AdminMetrics`, `Page<AdminUser>`, `Category[]`, `Page<AdminCompany>`, `Page<AdminMembership>`, `Page<QueueItem>`, `FeatureFlag[]`, `Page<AuditLogEntry>`, `BetaInviteBatch` |

Fixtures use the canonical company slugs, users and community slugs (contract §7.3). Community feed fixtures reuse post IDs 1–40 so posts link to Shoaib's detail page after integration.

## 6. Testing tasks

| Test file | What to test |
|---|---|
| `companies/__tests__/theme-summary.test.tsx` | available / insufficient / paused rendering; text alternative for bars |
| `companies/__tests__/review-card.test.tsx` | Unverified label, hidden narrative, company response badge, report actions |
| `companies/__tests__/review-form.test.tsx` | Narrative ≥ 150, attestation required, account-age panel, edit mode, held notice |
| `companies/__tests__/response-editor.test.tsx` | Length limit, conflict → edit existing, safety acknowledgement |
| `job-nightmares/__tests__/job-report-form.test.tsx` | No future months, inline new company, duplicate conflict, identity mode default |
| `communities/__tests__/community-mod-tab.test.tsx` | Reason required, scoped 403, join request decisions, resource limit |
| `communities/__tests__/create-community.test.tsx` | Slug availability, owned limit, acknowledgement required |
| `admin/__tests__/users.test.tsx`, `categories.test.tsx`, `feature-flags.test.tsx`, `audit-log.test.tsx` | Last-admin error; reorder via buttons; launch gate blocks; filters update the table |

## 7. Accessibility checklist

- [ ] ThemeSummary: each distribution has a text alternative ("2 of 7 rated 1 (very poor)…"); medians are text, not just bar length.
- [ ] Badges (Unverified, Official representative, Good experience, Private) carry text and icons.
- [ ] RatingInput rows are radio groups labelled with the topic; skippable with a clear "Skip" option.
- [ ] CompanyCombobox follows the ARIA combobox pattern; the "Add a new company" option is announced.
- [ ] Month picker is native `type="month"` or a labelled select pair (no custom calendar without keyboard support).
- [ ] Admin tables are semantic with captions; reorder works with buttons (not drag only); destructive actions use ConfirmDialog.
- [ ] Launch-gate checklist items are real checkboxes; the Enable button explains why it is disabled.
- [ ] Contrast through tokens only; touch targets ≥ 44 px on mobile.

## 8. Integration notes (mock → real API)

| Mock function | Real endpoint |
|---|---|
| `companiesApi.list` / `getCompany` | `GET /companies` / `GET /companies/:companyIdOrSlug` |
| `companiesApi.listReviews` / `createReview` | `GET` / `POST /companies/:companyId/reviews` |
| `companiesApi.updateReview` / `deleteReview` | `PATCH` / `DELETE /reviews/:reviewId` |
| `companiesApi.getMyReview` | `Company.viewer.hasReviewed` + `GET /companies/:companyId/reviews` filtered by `viewer.isAuthor` |
| `companiesApi.listCompanyJobReports` | `GET /job-reports?companyId=` |
| `companiesApi.submitCompany` / `suggestDuplicates` | `POST /companies` / `GET /companies?q=` |
| `companiesApi.requestRepresentation` / `getMyRepresentation` | `POST /companies/:companyId/representation-requests` / `GET /me/company-memberships` |
| `companiesApi.createResponse` / `updateResponse` / `deleteResponse` | `POST /companies/:companyId/responses` / `PATCH` / `DELETE /company-responses/:responseId` |
| `jobReportsApi.list` / `get` / `create` / `update` / `remove` | `GET /job-reports` / `GET`, `PATCH`, `DELETE /job-reports/:jobReportId` / `POST /job-reports` |
| `jobReportsApi.searchCompanies` | `GET /companies?q=` |
| `communitiesApi.list` / `get` / `create` / `join` / `leave` | `GET /communities` / `GET /communities/:idOrSlug` / `POST /communities` / `POST /communities/:id/join` / `DELETE /communities/:id/membership` |
| `communitiesApi.checkSlug` | `GET /communities/:slug` (404 = available) |
| `communitiesApi.getPosts` / `getMembers` | `GET /communities/:id/posts` / `GET /communities/:id/members` |
| `communitiesApi.getModQueue` / `applyModAction` | `GET /communities/:id/moderation/queue` / `POST /communities/:id/moderation/items/:itemId/actions` |
| `communitiesApi.listJoinRequests` / `decideJoinRequest` | `GET /communities/:id/join-requests` / `POST …/join-requests/:requestId/decision` |
| `communitiesApi.updateRules` / `addResource` / `removeResource` | `PUT /communities/:id/rules` / `POST` / `DELETE …/pinned-resources/:resourceId` |
| `communitiesApi.postAnnouncement` / `invite` | `POST /communities/:id/announcements` / `POST /communities/:id/invitations` |
| `adminApi.getMetrics` | `GET /admin/metrics?window=` |
| `adminApi.getSystemStatus` | `GET /health/ready` (+ queue metrics; admin-only extension noted for integration) |
| `adminApi.listUsers` / `setRole` / `grantPermission` / `revokePermission` | `GET /admin/users` / `PATCH /admin/users/:userId/role` / `POST /admin/users/:userId/permissions` / `DELETE …/permissions/:permission` |
| `adminApi.listCategories` / `createCategory` / `updateCategory` / `reorderCategories` | `GET` / `POST /admin/categories` / `PATCH /admin/categories/:categoryId` / `PUT /admin/categories/order` |
| `adminApi.listCompanies` / `updateCompany` / `mergeCompany` | `GET /admin/companies` / `PATCH /admin/companies/:companyId` / `POST /admin/companies/:companyId/merge` |
| `adminApi.listMemberships` / `decideMembership` | `GET /admin/company-memberships` / `POST /admin/company-memberships/:membershipId/decision` |
| `adminApi.listEscalations` | `GET /admin/escalations` |
| `adminApi.listFlags` / `setFlag` | `GET /admin/feature-flags` / `PATCH /admin/feature-flags/:key` |
| `adminApi.listAuditLogs` | `GET /admin/audit-logs` |
| `adminApi.createInvites` | `POST /admin/beta-invites` |

## 9. Independence check

This work depends on **no other teammate's work**. Until the owners merge, local placeholders (same paths and props, excluded from git) are used for:
- `@/shared/**` and the scaffold (Arko): copied from contract §2.3, §5–§10;
- `@/components/ui/*` (Tuli): props from contract §8;
- `SafetyCheckPanel`, `ReportDialog`, `BlockMuteMenuItems` (Tuli): placeholders with the contract props;
- `PostCard`, `PostCardSkeleton` (Shoaib): placeholders that render title, excerpt and a link.

## 10. Total estimated hours

**96 hours** across 32 tasks (Companies 27 h · Job Nightmare Board 11 h · Communities 20 h · Admin console 24 h · Testing and accessibility 14 h).
