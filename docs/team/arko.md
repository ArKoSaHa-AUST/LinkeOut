# Arko — Platform, Authentication, Account & Moderation

## 1. Role summary

Arko owns the **platform layer and the identity journeys**:
- the Next.js scaffold and config, design tokens, shared types, routes, flags, mock helpers and the mock session (all shared-contract code, built exactly as [TEAM_CONTRACT.md](TEAM_CONTRACT.md) specifies, so nobody needs to wait for it);
- the app shell (navigation, mobile tab bar, footer);
- every authentication screen (sign up, log in, verify, reset, Google completion) and onboarding;
- the account settings pages (sessions, account & data);
- system pages (404, error, maintenance, suspended);
- the **moderator workspace** (queue, item detail with action form, appeals, identity-reveal panel).

Everything runs on mock data in Arko's own feature folders.

## 2. Owned folders and files

All paths are under `apps/web/`. Arko commits only these.

| Path | Contents |
|---|---|
| `package.json`, `pnpm-lock.yaml`, `next.config.ts`, `tsconfig.json`, `postcss.config.mjs`, `eslint.config.mjs`, `.prettierrc`, `vitest.config.ts`, `vitest.setup.ts`, `playwright.config.ts`, `.env.local.example` | Scaffold and config |
| `public/**` | favicon, OG image, robots.txt |
| `src/middleware.ts` | UX redirects (logged-in user on `/` → `/mess`; `/settings` → `/settings/profile`) |
| `src/styles/tokens.css` | Design tokens (TEAM_CONTRACT §4) |
| `src/shared/**` | types, config (routes, flags, env, nav), lib (mock, api client, utils), auth (provider, guards, mock users, dev switcher), hooks |
| `src/components/layout/**` | AppShell, SideNav, TopNav, MobileTabBar, MoreDrawer, ShareMenu, Footer, SkipLink, PageAside |
| `src/app/layout.tsx`, `src/app/providers.tsx`, `src/app/globals.css`, `src/app/not-found.tsx`, `src/app/error.tsx` | Root app files |
| `src/app/(auth)/**` | `signup`, `signup/complete`, `login`, `verify-email`, `forgot-password`, `reset-password` |
| `src/app/onboarding/**`, `src/app/maintenance/**`, `src/app/suspended/**` | |
| `src/app/(app)/layout.tsx` | Shell layout |
| `src/app/(app)/settings/layout.tsx`, `src/app/(app)/settings/page.tsx`, `src/app/(app)/settings/sessions/**`, `src/app/(app)/settings/account/**` | Settings frame + 2 pages |
| `src/app/mod/**` | Moderator area |
| `src/features/auth/**`, `src/features/onboarding/**`, `src/features/account/**`, `src/features/system/**`, `src/features/moderation/**` | Feature folders |
| `tests/e2e/platform/**` | E2E specs |

## 3. Pages and screens

### 3.1 App shell (wraps every `(app)` page)
- **Purpose:** consistent navigation and layout (UI_UX A.1). **US/FR:** NFR-A2 (keyboard and skip link), supports all stories.
- **Components:** `AppShell` (grid: SideNav | main | aside slot), `SideNav` (logo, Share split button → `ShareMenu`, nav items from `nav.ts` with active state, role-gated Moderation/Admin, flag-gated Trauma Network/Confessions/Notifications), `TopNav` (md: horizontal nav; base: logo, search icon → `/search`, `NotificationBell` from `@/features/notifications`, avatar menu), `MobileTabBar` (The Mess, Confessions, Share (center), Companies, My Damage), `MoreDrawer` (remaining nav items, settings, log out), `ShareMenu` (Share a Story, Ask for Advice, Write a Lesson, Confess Anonymously, Report a Workplace Experience, Report a Job Nightmare, Create a Community), `Footer` (guidelines, privacy, terms, disclaimer, help), `SkipLink`, `PageAside` portal.
- **UI details:** side nav 240 px from `lg`; content max 680 px; aside 320 px from `xl`; wide variant (1200 px, no aside) for `/admin/**` and `/mod/**`. Active item uses the `accent` left bar and `aria-current="page"`.
- **States:** session `loading` → nav skeleton; visitor → "Log in" and "Join" buttons instead of the avatar; unverified → persistent warning `Banner` "Verify your email to join the conversation" with a resend link.
- **Responsive:** base/sm: top bar + bottom tab bar; md: top nav; lg+: side nav; xl: aside.
- **Mock data:** `Me` from the mock session.
- **Actions:** Share → menu (visitors are sent to `/signup` with a return URL); avatar menu → My Damage, Settings, Log out (`signOut()` → toast "Logged out. Your chaos will be here when you're back." → `/`); route change → focus moves to the page `h1`.

### 3.2 S-02 Sign up — `/signup`
- **Purpose:** create an account with a pseudonym. **US:** US-005, US-011. **FR:** FR-1, FR-8.
- **Components:** `SignupForm` (email, password + `PasswordStrengthMeter`, username + `UsernameAvailability` (debounced 400 ms), age checkbox, policies checkbox with links, invite code field shown when `beta.invite_only`), `GoogleButton` (Should), `CheckInboxPanel` (resend with a 60 s cooldown).
- **UI details:** single card, max 440 px; hint "Avoid your real name or email" under username; warning (non-blocking) when the username contains the email local part; note "Checked against known data breaches" under password.
- **States:** loading (submit spinner, inputs disabled); field errors inline (`VALIDATION_ERROR` details incl. `breached_password`); `CONFLICT` → "That username isn't available. Try {suggestion}."; 429 → form alert with retry seconds; success → CheckInboxPanel (same for existing emails); `registration.open` false → "Sign-ups are paused" panel.
- **Responsive:** full-width card on mobile; centered card from `sm`.
- **Mock data:** `authApi.checkUsername(u): UsernameAvailability` (taken: `rejected_but_rising`, `survived_standups`; reserved: `admin`, `moderator`, `linkedout`, `support`, `official`); `authApi.register(input) → { status: 'verification_sent', message }`; breached passwords list `['password123456', 'qwertyuiop123']`.
- **Actions:** type username → availability check; submit → register → CheckInboxPanel; "Resend" → `resendVerification(email)`; "Log in instead" → `/login`.

### 3.3 S-02b Complete OAuth sign-up — `/signup/complete` (Should)
- **Purpose:** pick a username after Google. **US:** US-010. **FR:** FR-7.
- **Components:** `CompleteSignupForm` (username + availability, age, policies), note "We don't import your Google name or photo."
- **States:** loading; `?mock=error` → "This sign-in expired — start again" with link to `/login`; success → `/onboarding`.
- **Mock data:** `authApi.completeOAuthSignup({ username, ageConfirmed, acceptTerms, termsVersion }) → { user }`.
- **Actions:** submit → set the mock session to a new verified user without onboarding → `/onboarding`.

### 3.4 S-03 Log in — `/login`
- **Purpose:** authenticate. **US:** US-007. **FR:** FR-3, FR-6.
- **Components:** `LoginForm` (email, password with show/hide, "Remember this device"), links (Forgot password, Sign up), `GoogleButton` (Should), `DeletionPendingPanel`.
- **States:** loading; `INVALID_CREDENTIALS` → "Email or password is incorrect."; `ACCOUNT_LOCKED` → "Too many attempts. We've emailed you a reset link. Try again in 15 minutes."; `ACCOUNT_SUSPENDED` → redirect `/suspended`; deletion pending → panel with date and "Cancel deletion" (`cancelDeletion`); success → `next` param or `/mess` (unverified → `/verify-email`, onboarding incomplete → `/onboarding`).
- **Mock data:** `authApi.login({email,password,rememberDevice}) → { user: { username, emailVerified, onboardingCompleted, platformRole } }`. Credentials: any mock user's email `<key>@linkedout.local` with password `Password-For-Dev-Only-1`; `locked@linkedout.local` → 423; `suspended@linkedout.local` → 403; `leaving@linkedout.local` → deletion pending.
- **Actions:** submit → sets the mock session; Cancel deletion → toast "Welcome back. Deletion cancelled." → `/mess`.

### 3.5 S-04 Verify email — `/verify-email`
- **Purpose:** consume the token or explain status. **US:** US-006. **FR:** FR-2.
- **Components:** `VerifyStatus` (verifying → success → redirect to onboarding), `ExpiredLinkPanel` (Resend), `PendingVerificationPanel` (no token: "Check your inbox", resend).
- **States:** loading text with `aria-live="polite"`; `TOKEN_INVALID` (token `expired`) → expired panel; 429 on resend → message.
- **Mock data:** `authApi.verifyEmail(token)` (`valid-token` succeeds, anything else → `TOKEN_INVALID`).

### 3.6 S-05 Forgot / reset password — `/forgot-password`, `/reset-password`
- **Purpose:** account recovery. **US:** US-008. **FR:** FR-4.
- **Components:** `ForgotPasswordForm` → neutral confirmation; `ResetPasswordForm` (new password + meter + confirm).
- **States:** loading; always-neutral success on forgot; `TOKEN_INVALID` panel with "Request a new link"; `breached_password` / `too_short` inline errors; success → toast "Password updated. Other devices were signed out." → `/mess`.
- **Mock data:** `authApi.forgotPassword(email) → { status: 'sent_if_applicable' }`, `authApi.resetPassword({ token, newPassword })`.

### 3.7 S-06 Onboarding — `/onboarding`
- **Purpose:** informed consent before participating. **US:** US-013. **FR:** FR-11.
- **Components:** `OnboardingStepper` (`Stepper`), `StepPseudonymVsAnonymous` (3 comparison cards with the example label "Anonymous Stapler #4821"), `StepDataWeCollect` (table: email, IP 90 days, content, settings; "We never show your email, IP or real name"), `StepLawfulDisclosure` (lawful requests, four-eyes, transparency report), `StepGuidelines` (prohibited list summary, "Prefer patterns over naming people", required "I understand" checkbox), `StepInterests` (category chips, optional).
- **States:** progress saved in `localStorage['lo_onboarding_step']`; error saving → retry banner; finishing → `refresh()` session → `/mess` with toast "You're in. Welcome to the chaos."
- **Responsive:** one step per screen; mobile shows "Step 2 of 5" text; buttons stick to the bottom on mobile.
- **Mock data:** `onboardingApi.complete({ acknowledgedSteps, interestCategoryIds }) → { onboardingCompleted: true }`; local category list (ids 1–12, contract §7.3).
- **Actions:** Next/Back; Finish disabled until step 4 is acknowledged.

### 3.8 S-38 System pages — 404, error, `/maintenance`, `/suspended`
- **Components:** `NotFoundPage` ("This page went the way of our 2019 career plans."), `ErrorPage` (ErrorState with requestId + retry `reset()`), `MaintenancePage`, `SuspendedPage` (policy reference, member message, "Appeal this decision" → `/reports`).
- **Mock data:** suspended page reads `accountApi.getSuspension() → { policyReference, memberMessage, until: string | null }`.

### 3.9 S-27 Settings → Sessions — `/settings/sessions` (+ settings frame)
- **Purpose:** see and revoke sessions. **US:** US-009. **FR:** FR-5.
- **Components:** `SettingsLayout` (side tabs on md+: Profile, Privacy, Blocked & muted, Notifications, Sessions, Account & data; a select menu on mobile), `SessionsList` (`SessionRow`: device, location, created, last active, "This device" badge, Sign out), "Sign out all other sessions" (`ConfirmDialog`).
- **States:** loading rows; error; single session → note "Only this device is signed in."; success toasts.
- **Mock data:** `accountApi.listSessions(): SessionInfo[]` (3 sessions: Firefox on Linux · Kolkata, IN (current); Safari on iPhone · Dhaka, BD; Chrome on Windows · unknown location), `revokeSession(id)`, `revokeOtherSessions() → { revokedCount }`.

### 3.10 S-29 Settings → Account & data — `/settings/account`
- **Purpose:** password, export, deletion. **US:** US-020, US-021. **FR:** FR-18, FR-19.
- **Components:** `EmailReadonly` ("To change your email, contact support"), `ChangePasswordForm`, `DataExportSection` (Should: request with password → status pending/processing/completed with a download link and expiry), `DeleteAccountSection` (retained-data table, password field, `ConfirmDialog` with `typedConfirmation="DELETE"`), `DeletionPendingBanner` (date + "Cancel deletion").
- **States:** loading; export `CONFLICT` → "An export is already being prepared"; deletion `INVALID_CREDENTIALS` inline; after deletion → mock session cleared → `/` with toast "Your account is scheduled for deletion on {date}."
- **Mock data:** `accountApi.requestExport(password) → DataExport`, `getExport(id)` (pending → completed after 2 polls), `deleteAccount({ password, confirm }) → DeletionResult` (retained: moderation records 2 years, audit logs 2 years, legal hold until lifted), `cancelDeletion()`.

### 3.11 S-23 Moderator dashboard — `/mod`, `/mod/appeals`
- **Purpose:** prioritized moderation work. **US:** US-060, US-063, US-064, US-065. **FR:** FR-46, FR-48, FR-49.
- **Components:** `ModHeaderStats` (open, high priority, oldest age, my resolved today), `ModTabs` (Queue, Mine, Escalated, Appeals), `QueueFilters` (priority, reason, target type, automated vs reported, assigned), `QueueTable` (`DataTable`: priority pill, target type, reasons with counts, finding icons, preview excerpt with AuthorBadge, age, assignee); keyboard J/K/Enter with a "Keyboard shortcuts" toggle; `AppealsList` (appeal card: action summary, reason, "Can't decide (you took the original action)" when `canDecide` is false, Decide → `AppealDecisionDialog` with outcome radio, note, modified action form).
- **States:** loading table skeleton; empty "Queue clear. Go drink some water."; error; `RequireRole moderator` (others see 404).
- **Responsive:** wide layout; mobile: stacked cards, filters in a sheet.
- **Mock data:** `moderationApi.getQueue(filters): Page<QueueItem>` (8 items per contract §7.3 incl. a `review_campaign` item for acme-widgets), `getStats()`, `getAppeals(): Page<AppealQueueItem>` (2 pending, one with `canDecide: false` for `lena_mod`), `decideAppeal(id, input)`.
- **Actions:** row click/Enter → `/mod/items/[id]`; filter → URL params; decide appeal → toast + list refresh.

### 3.12 S-24 Moderation item detail — `/mod/items/[itemId]`
- **Purpose:** decide with full context. **US:** US-061, US-062, US-065, US-066. **FR:** FR-47, FR-51.
- **Components:** `ItemHeader` (target type, priority, status, claim state, Claim/Release), `ContentPanel` (rendered content via `MarkdownRenderer` from `@/features/posts` with finding ranges highlighted, revisions diff toggle, corrections), `ReportsPanel`, `FindingsPanel`, `AuthorHistory` ("History aggregated — identity hidden" for anonymous), `RelatedItems` (campaign members), `ActionForm` (action-type radio with descriptions; required reason; policy select from `POLICY_CODES` with guideline titles; member message pre-filled from templates; duration select 1/7/30 for restrictions; content-warning chips for `add_content_warning`; escalation note; "Notify reporters"), `RevealPanel` (only with `identity_vault.reveal`: request form with category + justification ≥ 50 chars; pending state; approve/reject when you are the second officer; revealed account summary with a 24 h countdown; self-approval disabled with an explanation).
- **States:** loading; not found → 404; claimed by another → read-only banner "Being reviewed by @x"; `CONFLICT` on claim → toast + refresh; submit success → toast "Action applied" → back to the queue; validation errors inline.
- **Responsive:** lg+: two columns (context left, action form right, sticky); mobile: tabs Content / Reports / Act.
- **Mock data:** `moderationApi.getItem(id): ModerationItemDetail`, `claim(id)`, `release(id)`, `applyAction(id, input) → { actionId, itemStatus, targetModerationStatus }`, `requestReveal(input) → RevealRequest`, `approveReveal(id)`, `rejectReveal(id)`, `getReveal(id)`.

## 4. Task checklist (build order)

Each task is 1–4 h. **AC** = acceptance criteria.

### Phase A — Platform (shared-contract code; nobody waits for it)
- [ ] **FE-A-01** · 3h · Scaffold Next.js app per contract §2.3, install dependencies, configure TS/ESLint/Prettier/Vitest/Playwright. **AC:** `pnpm dev`, `lint`, `typecheck`, `test`, `test:e2e` (one smoke spec) all run.
- [ ] **FE-A-02** · 2h · `tokens.css` + Tailwind 4 `@theme` mapping in `globals.css`; fonts via `next/font`. **AC:** every token in §4 is usable as a Tailwind class; contrast unit test passes for text tokens.
- [ ] **FE-A-03** · 3h · `src/shared/types/**` verbatim from contract §5. **AC:** `pnpm typecheck` passes; barrel exports all types.
- [ ] **FE-A-04** · 2h · `routes.ts`, `flags.ts` (+ `useFlag`), `env.ts`, `nav.ts`. **AC:** matches §6 and §9.1; unit test for `routes.*` output.
- [ ] **FE-A-05** · 3h · Mock helpers: `mockRequest`, `ApiError`, `paginate`, `getScenario`, `mockId`, `fixedNow`. **AC:** unit tests cover latency 0 in tests, every scenario, cursor paging and the error envelope.
- [ ] **FE-A-06** · 3h · `AuthProvider`, `useSession`, `mock-users.ts` (all users in §7.3 with full `Me` objects), `DevRoleSwitcher` (dev only). **AC:** switching users updates the session everywhere without a reload; visitor has `status: 'anonymous'`.
- [ ] **FE-A-07** · 2h · `RequireAuth`, `RequireRole`, `FlagGate`, `middleware.ts`. **AC:** behaviors in §10.3; tests for each branch.
- [ ] **FE-A-08** · 2h · `providers.tsx` (QueryClient, AuthProvider, Toaster, devtools), root `layout.tsx`, `not-found.tsx`, `error.tsx`. **AC:** app boots; Toaster import works (local copy of Tuli's kit until merge).
- [ ] **FE-A-09** · 2h · `useMediaQuery`, `useDebouncedValue`, `useUrlState`, `cn`, `format` (relative time, numbers, year-month). **AC:** unit tests.
- [ ] **FE-A-10** · 2h · Real HTTP client `src/shared/lib/api/client.ts` (credentials include, CSRF bootstrap via `GET /auth/csrf`, `X-CSRF-Token`, parsing into `ApiError`); not wired yet. **AC:** unit tests with a mocked `fetch`.

### Phase B — App shell
- [ ] **FE-A-11** · 4h · `AppShell` grid, `SideNav` with role/flag gating, active states, `PageAside` portal. **AC:** lg/xl layouts match UI_UX A.1; `aria-current` set.
- [ ] **FE-A-12** · 3h · `TopNav` (md), mobile top bar, `MobileTabBar`, `MoreDrawer`. **AC:** base/md layouts; drawer traps focus and closes on Esc.
- [ ] **FE-A-13** · 2h · `ShareMenu` (all 7 actions, visitor redirect), avatar menu, unverified banner. **AC:** actions route correctly via `routes`.
- [ ] **FE-A-14** · 2h · `Footer`, `SkipLink`, focus-to-`h1` on route change, wide layout variant for `/admin` and `/mod`. **AC:** skip link is the first tab stop; focus moves on navigation.

### Phase C — Authentication and onboarding
- [ ] **FE-A-15** · 3h · `auth` mock API + fixtures (users, breached list, tokens, scenarios). **AC:** functions and errors exactly as §3.2–§3.6.
- [ ] **FE-A-16** · 4h · S-02 Sign up form with validation schema, username availability, strength meter, invite field, CheckInboxPanel. **AC:** US-005 and US-011 criteria demonstrable with mocks.
- [ ] **FE-A-17** · 2h · S-02b complete OAuth sign-up + `GoogleButton` (mock redirect to `/signup/complete`). **AC:** US-010 criteria 1 and 3.
- [ ] **FE-A-18** · 3h · S-03 Log in with locked, suspended and deletion-pending states. **AC:** US-007 criteria; redirects by session state.
- [ ] **FE-A-19** · 3h · S-04 Verify email + S-05 forgot/reset. **AC:** US-006, US-008 criteria incl. neutral responses.
- [ ] **FE-A-20** · 4h · S-06 onboarding steps 1–3 + stepper + persisted progress. **AC:** stepper accessible; progress survives reload.
- [ ] **FE-A-21** · 2h · S-06 step 4 (guidelines acknowledgement) + step 5 (interests) + finish. **AC:** US-013 criteria; finish updates the session.

### Phase D — System and account pages
- [ ] **FE-A-22** · 2h · S-38 system pages (404, error, maintenance, suspended). **AC:** copy from UI_UX S-38; error page shows requestId.
- [ ] **FE-A-23** · 2h · `account` mock API + fixtures (sessions, export, deletion, suspension).
- [ ] **FE-A-24** · 3h · Settings layout + S-27 Sessions. **AC:** US-009 criteria; mobile settings select menu.
- [ ] **FE-A-25** · 4h · S-29 Account & data: change password, delete account flow, deletion-pending banner. **AC:** US-020 criteria 1–3.
- [ ] **FE-A-26** · 2h · S-29 data export section (Should). **AC:** US-021 criteria 1–2 with status polling.

### Phase E — Moderator workspace
- [ ] **FE-A-27** · 3h · `moderation` mock API + fixtures (8 queue items, item details, appeals, reveal requests, stats). **AC:** canonical IDs from §7.3; claim conflicts simulated.
- [ ] **FE-A-28** · 4h · S-23 queue: stats, tabs, filters (URL), QueueTable with keyboard shortcuts. **AC:** US-060 criteria 1–3.
- [ ] **FE-A-29** · 4h · S-24 item header, claim/release, content panel with finding highlights and revisions, reports/findings/history/related panels. **AC:** US-062 context visible; anonymous history hidden.
- [ ] **FE-A-30** · 4h · S-24 ActionForm with per-type validation and member-message templates. **AC:** US-061 criteria 1–3, US-065.
- [ ] **FE-A-31** · 3h · RevealPanel (request, pending, approve/reject, revealed with countdown, self-approval blocked). **AC:** US-066 criteria 1–4 (with `moderator` and `moderator2` mock users).
- [ ] **FE-A-32** · 3h · Appeals tab + AppealDecisionDialog. **AC:** US-063 criteria 1–2; `canDecide: false` disables the controls.

### Phase F — Testing and accessibility
- [ ] **FE-A-33** · 3h · Component tests: SignupForm, LoginForm, ResetPasswordForm, OnboardingStepper (see §6). **AC:** all pass, no act warnings.
- [ ] **FE-A-34** · 3h · Component tests: AppShell/nav gating, RequireAuth/RequireRole, SessionsList, DeleteAccountSection.
- [ ] **FE-A-35** · 3h · Component tests: QueueTable keyboard, ActionForm validation, RevealPanel, AppealDecisionDialog.
- [ ] **FE-A-36** · 3h · E2E (`tests/e2e/platform`): sign up → verify → onboarding; login states; moderator applies an action; axe checks on S-02, S-03, S-06, S-23, S-24. **AC:** green on chromium + mobile-chrome.
- [ ] **FE-A-37** · 2h · Accessibility and responsive pass on all owned screens (checklist §7). **AC:** 0 axe serious/critical; 320 px without horizontal scroll.

## 5. Mock data and mock services

| File | Exports | Returns |
|---|---|---|
| `src/shared/lib/mock/*` | `mockRequest`, `ApiError`, `paginate`, `getScenario`, `mockId`, `fixedNow` | Helpers for every teammate (contract §7.1) |
| `src/shared/auth/mock-users.ts` | `mockUsers: Record<MockUserKey, Me \| null>` | The 10 canonical users |
| `src/features/auth/api/mock.ts` | `mockAuthApi`: `checkUsername`, `register`, `resendVerification`, `verifyEmail`, `login`, `logout`, `forgotPassword`, `resetPassword`, `completeOAuthSignup`, `cancelDeletion` | `UsernameAvailability`, `{status,message}`, `{user}`, `void` |
| `src/features/onboarding/api/mock.ts` | `mockOnboardingApi.complete` | `{ onboardingCompleted: true }` |
| `src/features/account/api/mock.ts` | `mockAccountApi`: `listSessions`, `revokeSession`, `revokeOtherSessions`, `changePassword`, `requestExport`, `getExport`, `deleteAccount`, `getSuspension` | `SessionInfo[]`, `{revokedCount}`, `DataExport`, `DeletionResult` |
| `src/features/moderation/api/mock.ts` | `mockModerationApi`: `getQueue`, `getStats`, `getItem`, `claim`, `release`, `applyAction`, `getAppeals`, `decideAppeal`, `requestReveal`, `approveReveal`, `rejectReveal`, `getReveal` | `Page<QueueItem>`, `ModerationItemDetail`, `Page<AppealQueueItem>`, `RevealRequest` |

Fixture files: `src/features/<f>/mocks/fixtures.ts` for auth (credentials, tokens), account (3 sessions, export states), moderation (8 queue items, 8 details, 2 appeals, 2 reveal requests, stats).

## 6. Testing tasks

| Test file | What to test |
|---|---|
| `shared/lib/mock/__tests__/mock-request.test.ts` | Scenarios ok/empty/error/forbidden/rate_limited; paginate cursors; `ApiError` shape |
| `shared/auth/__tests__/require-auth.test.tsx` | Anonymous redirect, unverified banner, onboarding redirect, role 404, flag fallback |
| `components/layout/__tests__/side-nav.test.tsx` | Role/flag gating of items, `aria-current`, Share menu items |
| `features/auth/__tests__/signup-form.test.tsx` | Required fields, username availability states, breached password error, conflict suggestion, neutral success |
| `features/auth/__tests__/login-form.test.tsx` | Invalid credentials, locked, suspended redirect, deletion pending panel, `next` redirect |
| `features/onboarding/__tests__/onboarding.test.tsx` | Finish disabled until acknowledgement, progress persistence, stepper `aria-current` |
| `features/account/__tests__/sessions.test.tsx`, `delete-account.test.tsx` | Revoke one/all; typed "DELETE" gating; retained table rendering |
| `features/moderation/__tests__/queue-table.test.tsx` | J/K navigation, Enter opens, filters in URL, empty state |
| `features/moderation/__tests__/action-form.test.tsx` | Reason required; duration required for restrict; warnings required for add CW; escalation note |
| `features/moderation/__tests__/reveal-panel.test.tsx` | Hidden without permission; justification ≥ 50; self-approve disabled; countdown |

## 7. Accessibility checklist

- [ ] Skip link first in tab order; landmarks `header`, `nav` (labelled "Primary"), `main`, `aside`, `footer`.
- [ ] Focus moves to the page `h1` after navigation; dialogs and drawers trap and restore focus.
- [ ] All form fields have visible labels; errors linked with `aria-describedby`; an error summary with links on submit failure; `autocomplete` set (`email`, `username`, `new-password`, `current-password`).
- [ ] Password show/hide is a button with `aria-pressed`; the strength meter has a text equivalent.
- [ ] Stepper uses `aria-current="step"`; step changes announced with a live region.
- [ ] QueueTable is a semantic table with `scope="col"`; shortcuts can be turned off and are listed in a dialog; rows are focusable with visible focus.
- [ ] Priority and status pills carry text, not just color.
- [ ] RevealPanel countdown uses a polite live region at most once per minute.
- [ ] Contrast through tokens only; focus ring visible on every interactive element.

## 8. Integration notes (mock → real API)

| Mock function | Real endpoint (API_SPEC) |
|---|---|
| `authApi.checkUsername` | `GET /auth/username-availability` |
| `authApi.register` | `POST /auth/register` |
| `authApi.resendVerification` | `POST /auth/resend-verification` |
| `authApi.verifyEmail` | `POST /auth/verify-email` |
| `authApi.login` / `logout` | `POST /auth/login` / `POST /auth/logout` |
| `authApi.forgotPassword` / `resetPassword` | `POST /auth/forgot-password` / `POST /auth/reset-password` |
| `authApi.completeOAuthSignup` | `GET /auth/oauth/google` (redirect), `POST /auth/oauth/complete-signup` |
| `authApi.cancelDeletion` | `POST /me/account/cancel-deletion` |
| `AuthProvider` session load | `GET /me` (+ `GET /me/company-memberships` merged into `Me.companyMemberships`) |
| `onboardingApi.complete` | `POST /me/onboarding/complete` |
| `accountApi.listSessions` / `revokeSession` / `revokeOtherSessions` | `GET /me/sessions` / `DELETE /me/sessions/:sessionId` / `DELETE /me/sessions` |
| `accountApi.changePassword` | Reset flow (`POST /auth/forgot-password`) until a change-password endpoint is added to API_SPEC (logged in ASSUMPTIONS at integration) |
| `accountApi.requestExport` / `getExport` | `POST /me/exports` / `GET /me/exports/:exportId` |
| `accountApi.deleteAccount` | `DELETE /me/account` |
| `accountApi.getSuspension` | `GET /me/moderation-actions` (latest `suspend_account`) |
| `moderationApi.getQueue` / `getItem` | `GET /moderation/queue` / `GET /moderation/items/:itemId` |
| `moderationApi.claim` / `release` | `POST /moderation/items/:itemId/claim` / `…/release` |
| `moderationApi.applyAction` | `POST /moderation/items/:itemId/actions` |
| `moderationApi.getAppeals` / `decideAppeal` | `GET /moderation/appeals` / `POST /moderation/appeals/:appealId/decision` |
| `moderationApi.requestReveal` / `approveReveal` / `rejectReveal` / `getReveal` | `POST /trust-safety/identity-reveals`, `…/:revealId/approve`, `…/:revealId/reject`, `GET …/:revealId` |
| `moderationApi.getStats` | Derived from `GET /moderation/queue` counts (no extra endpoint) |

Also at integration: Arko switches `NEXT_PUBLIC_USE_MOCKS` handling, wires `src/shared/lib/api/client.ts`, and adds any dependency requests from teammates.

## 9. Independence check

This work depends on **no other teammate's work**. Arko owns the shared-contract platform code, so there is nothing to wait for. Local placeholders are used only for:
- `@/components/ui/*` (Tuli), built from contract §8.1–§8.2 props until Tuli's kit is merged;
- `NotificationBell` from `@/features/notifications` (Shoaib), as a placeholder icon link to `/notifications` with the same export name;
- `MarkdownRenderer` from `@/features/posts` (Shoaib), as a placeholder that renders text in a `<div>` with `white-space: pre-wrap`.

All three are excluded via `.git/info/exclude` and deleted before pulling the real versions.

## 10. Total estimated hours

**104 hours** across 37 tasks (Platform 24 h · App shell 11 h · Authentication and onboarding 21 h · System and account pages 13 h · Moderator workspace 21 h · Testing and accessibility 14 h).
