# LinkedOut Frontend — Team Contract

**Scope:** frontend only (`apps/web`), built against **mock data**. No backend work.
**Team:** Arko · Shoaib · Tuli · Abnt.
**Status:** binding agreement. Changing anything here needs a PR to this file approved by all four teammates.

This contract is what lets four people start on day 1 and never wait for each other. Everyone codes against the **paths, types, props and conventions defined here**, not against each other's code.

Related: [README.md](README.md) · [arko.md](arko.md) · [shoaib.md](shoaib.md) · [tuli.md](tuli.md) · [abnt.md](abnt.md) · [../UI_UX.md](../UI_UX.md) · [../API_SPEC.md](../API_SPEC.md)

---

## 1. The independence model

1. **Vertical ownership.** Each person owns whole features: routes, components, hooks, mock data and mock API. The ownership table is in §3.
2. **One owner per file.** You commit only inside paths you own. Paths you don't own are off-limits for commits.
3. **Mock first.** Every feature talks to `src/features/<feature>/api/index.ts`, which exports a mock implementation. The data has the exact shapes from [API_SPEC.md](../API_SPEC.md) (types in §5). Switching to the real API later changes one line per feature.
4. **Shared code is specified here, not waited for.** A small set of **shared-contract files** (scaffold, tokens, types, base UI kit, session, cross-feature components) has one owner each. Their public interface (path, exports, types, props) is fully defined in this document.
5. **The local-copy rule (how nobody waits):** if you need a shared-contract file that its owner hasn't merged yet, create a **local, uncommitted copy at the exact same path**, written from this contract, and list it in `.git/info/exclude` so it can never be committed. Your copy can be crude (e.g. a plain `<button>` that accepts the `ButtonProps` from §8). Because the path, exports and props are identical, **nothing in your code changes** when the owner's real file arrives. Before pulling the owner's merged work, delete your local copies (`git status --ignored` lists them).

```text
# .git/info/exclude (example for Shoaib before integration)
apps/web/src/components/ui/
apps/web/src/shared/
apps/web/src/features/safety/index.ts
apps/web/src/features/safety/components/
```

### 1.1 Unavoidable dependencies and their workarounds

| Dependency | Why it can't be avoided | Owner | Workaround so nobody is blocked |
|---|---|---|---|
| Next.js project scaffold and config | Only one `package.json`/config can exist | Arko | §2.3 gives the exact scaffold steps and config values. Everyone runs them locally on day 1 (≈ 30 min) and excludes the generated files; only Arko commits them. |
| Root layout and providers (`src/app/layout.tsx`, `src/app/providers.tsx`) | Next.js needs exactly one root layout | Arko | Local copy: a minimal root layout that wraps children in `<Providers>` (§10). |
| Shared types, routes, flags, mock helpers, session (`src/shared/**`) | Used by every feature | Arko | Full TypeScript in §5, §6, §9, §10. Copy them verbatim locally. |
| Base UI kit (`src/components/ui/**`) | Used by every screen | Tuli | Props in §8. Local copies can be minimal native elements with the same props. |
| Cross-feature components (PostCard, SafetyCheckPanel, ReportDialog, BlockMuteMenuItems, CompanyCard, JobReportCard, NotificationBell, AuthorBadge) | Used by more than one owner | See §8.3 | Props in §8.3. Local copies render simple placeholders with the same props. |
| App shell (navigation) | Wraps every page | Arko | Pages must render correctly **without** the shell (they are full-width blocks). The shell is added by `src/app/(app)/layout.tsx`, which only Arko owns. |

No other dependency exists. **No task in any personal file waits for another teammate.**

---

## 2. Tech stack and project setup

### 2.1 Versions (from [TECH_STACK.md](../TECH_STACK.md))

| Package | Version |
|---|---|
| Node.js | 24 LTS |
| pnpm | 10.x |
| TypeScript | 5.9 |
| Next.js (App Router) | 16.x |
| React / React DOM | 19.2 |
| Tailwind CSS | 4.1 (`@tailwindcss/postcss`) |
| shadcn/ui CLI | 3.x (Radix UI primitives 1.x) |
| @tanstack/react-query | 5.x (+ devtools) |
| react-hook-form + @hookform/resolvers | 7.x / 5.x |
| zod | 4.x |
| react-markdown + rehype-sanitize | 10.x / 6.x |
| lucide-react | 0.5xx |
| date-fns | 4.x |
| clsx + tailwind-merge | 2.x / 3.x |
| Vitest + @testing-library/react + user-event + jest-dom | 3.x / 16.x / 14.x / 6.x |
| jsdom | 25.x |
| Playwright + @axe-core/playwright | 1.5x / 4.10 |
| ESLint 9 (flat) + typescript-eslint + eslint-plugin-jsx-a11y + eslint-config-next | 9.x / 8.x / 6.x / 16.x |
| Prettier + prettier-plugin-tailwindcss | 3.x / 0.6x |

### 2.2 Frontend-only adjustments to FOLDER_STRUCTURE.md

During the frontend-only phase we use a **feature-folder** layout (`src/features/<feature>`) instead of `src/components/<domain>`. Copy lives in each feature's `copy.ts` instead of one shared `en.json`, so that no two people edit the same file. This is recorded in [../ASSUMPTIONS.md](../ASSUMPTIONS.md) (A-70).

The web app is a standalone Next.js project at `apps/web`. The `packages/*` workspace packages come later. Until then, types live in `src/shared/types` with the same names as `packages/shared`.

### 2.3 Scaffold (Arko commits it; everyone else creates it locally on day 1)

```bash
pnpm create next-app@16 apps/web --ts --eslint --tailwind --app --src-dir --import-alias "@/*" --use-pnpm --no-turbopack
cd apps/web
pnpm add @tanstack/react-query @tanstack/react-query-devtools react-hook-form @hookform/resolvers zod \
  react-markdown rehype-sanitize lucide-react date-fns clsx tailwind-merge
pnpm add -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/user-event \
  @testing-library/jest-dom @playwright/test @axe-core/playwright eslint-plugin-jsx-a11y \
  prettier prettier-plugin-tailwindcss
pnpm dlx shadcn@3 init   # style: new-york, base color: neutral, CSS variables: yes
```

**Config values everyone must match:**

| File | Required settings |
|---|---|
| `tsconfig.json` | `"strict": true`, `"noUncheckedIndexedAccess": true`, paths `"@/*": ["./src/*"]` |
| `next.config.ts` | `reactStrictMode: true`, `poweredByHeader: false`, `images.remotePatterns` for `picsum.photos` (mock images) |
| `.env.local` | `NEXT_PUBLIC_USE_MOCKS=true`, `NEXT_PUBLIC_MOCK_LATENCY_MS=400`, `NEXT_PUBLIC_API_URL=http://localhost:4000/api/v1` |
| `vitest.config.ts` | `environment: 'jsdom'`, `setupFiles: ['./vitest.setup.ts']`, alias `@` → `src`, `include: ['src/**/*.test.{ts,tsx}']` |
| `vitest.setup.ts` | `import '@testing-library/jest-dom/vitest'` |
| `playwright.config.ts` | `testDir: 'tests/e2e'`, `webServer: pnpm dev` on port 3000, projects `chromium` and `mobile-chrome` (Pixel 7) |
| `package.json` scripts | `dev`, `build`, `start`, `lint`, `typecheck` (`tsc --noEmit`), `test` (`vitest run`), `test:watch`, `test:e2e` (`playwright test`), `format` |

---

## 3. Folder structure and ownership

Every path is relative to `apps/web/`. **A** = Arko, **S** = Shoaib, **T** = Tuli, **N** = Abnt.

```text
apps/web/
├── package.json, pnpm-lock.yaml, next.config.ts, tsconfig.json,          [A]
│   postcss.config.mjs, eslint.config.mjs, .prettierrc, vitest.config.ts,
│   vitest.setup.ts, playwright.config.ts, .env.local.example
├── components.json                                                        [T]  shadcn config
├── public/                                                                [A]  favicon, og image, robots.txt
├── tests/e2e/
│   ├── platform/      auth, onboarding, settings, moderation journeys     [A]
│   ├── content/       feed, post, comments, search, notifications         [S]
│   ├── member/        confessions, profile, privacy, reports              [T]
│   └── workplace/     companies, jobs, communities, admin                 [N]
└── src/
    ├── middleware.ts                                                      [A]
    ├── styles/tokens.css                                                  [A]
    ├── shared/                                                            [A]  (shared-contract)
    │   ├── types/        index.ts + one file per domain (§5)
    │   ├── config/       routes.ts, flags.ts, env.ts, nav.ts
    │   ├── lib/
    │   │   ├── mock/     mock-request.ts, paginate.ts, scenario.ts, ids.ts, api-error.ts
    │   │   ├── api/      client.ts (real HTTP client, used at integration)
    │   │   └── utils/    cn.ts, format.ts
    │   ├── auth/         AuthProvider.tsx, useSession.ts, RequireAuth.tsx, RequireRole.tsx,
    │   │                 mock-users.ts, DevRoleSwitcher.tsx
    │   └── hooks/        useMediaQuery.ts, useDebouncedValue.ts, useUrlState.ts
    ├── components/
    │   ├── ui/                                                            [T]  (shared-contract base kit)
    │   └── layout/       AppShell, SideNav, TopNav, MobileTabBar,         [A]
    │                     MoreDrawer, ShareMenu, Footer, SkipLink
    ├── features/
    │   ├── auth/                  sign up, log in, verify, reset, OAuth    [A]
    │   ├── onboarding/            S-06                                     [A]
    │   ├── account/               sessions S-27, account & data S-29       [A]
    │   ├── legal/                 policy pages content                     [T]
    │   ├── system/                404, error, maintenance, suspended       [A]
    │   ├── moderation/            S-23, S-24, appeals tab, reveal panel    [A]
    │   ├── posts/                 PostCard, composer, post detail,         [S]
    │   │                          reactions, polls, media picker
    │   ├── comments/              CommentThread, CommentForm               [S]
    │   ├── feed/                  The Mess, filters, landing preview       [S]
    │   ├── lessons/               S-14                                     [S]
    │   ├── search/                S-21                                     [S]
    │   ├── notifications/         bell, S-36, S-37                         [S]
    │   ├── safety/                SafetyCheckPanel, ReportDialog,          [T]
    │   │                          BlockMuteMenuItems
    │   ├── confessions/           S-09, S-10                               [T]
    │   ├── profile/               S-12, S-25                               [T]
    │   ├── privacy/               S-26, S-28                               [T]
    │   ├── my-damage/             S-13                                     [T]
    │   ├── reports/               S-22                                     [T]
    │   ├── companies/             S-15–S-18, S-32, company responses       [N]
    │   ├── job-nightmares/        S-19, S-20                               [N]
    │   ├── communities/           S-33–S-35, community moderation          [N]
    │   └── admin/                 S-30, S-31                               [N]
    └── app/
        ├── layout.tsx, providers.tsx, globals.css, not-found.tsx, error.tsx   [A]
        ├── (marketing)/page.tsx                                          [S]  landing
        ├── (marketing)/legal/**                                          [T]
        ├── (auth)/**                                                     [A]
        ├── onboarding/**                                                 [A]
        ├── maintenance/**, suspended/**                                  [A]
        ├── dev/ui/**                                                     [T]  kit preview page
        ├── mod/**                                                        [A]
        ├── admin/**                                                      [N]
        └── (app)/
            ├── layout.tsx                                                [A]
            ├── mess/**, post/**, posts/**, lessons/**, search/**,         [S]
            │   notifications/**
            ├── confessions/**, u/**, me/**, reports/**                   [T]
            ├── companies/**, job-nightmares/**, communities/**           [N]
            └── settings/
                ├── layout.tsx, page.tsx, sessions/**, account/**         [A]
                ├── notifications/**                                      [S]
                └── profile/**, privacy/**, blocked/**                    [T]
```

### 3.1 Inside every feature folder (same layout for all)

```text
src/features/<feature>/
├── index.ts            # public exports ONLY (what other features may import)
├── copy.ts             # all user-facing strings for this feature
├── api/
│   ├── index.ts        # export const <feature>Api: <Feature>Api = mock<Feature>Api  ← the one-line switch
│   ├── types.ts        # the <Feature>Api interface (function signatures)
│   └── mock.ts         # mock implementation using fixtures + shared mock helpers
├── mocks/fixtures.ts   # deterministic mock data
├── hooks/              # TanStack Query hooks wrapping <feature>Api
├── components/         # feature components
├── pages/              # page-level components rendered by app/**/page.tsx
└── __tests__/          # Vitest + RTL tests
```

Route files in `src/app/**/page.tsx` stay thin: they import and render the feature's page component.

---

## 4. Design tokens

Owned by Arko in `src/styles/tokens.css` (CSS variables) and mapped in `globals.css` with Tailwind 4 `@theme`. **Use the Tailwind names below. Never write raw hex values in components.**

### 4.1 Colors

| Token / Tailwind name | Hex | Use |
|---|---|---|
| `bg` | `#171717` | App background |
| `surface` | `#1F1F1F` | Cards, inputs |
| `surface-raised` | `#262626` | Menus, dialogs, content-warning gate |
| `surface-hover` | `#2E2E2E` | Hover rows and cards |
| `divider` | `#2E2E2E` | Decorative borders |
| `border-strong` | `#737373` | Input borders and control outlines |
| `text-primary` | `#F5F5F5` | Main text |
| `text-secondary` | `#A3A3A3` | Metadata, helper text |
| `text-muted` | `#737373` | Placeholders, disabled, large decorative text only |
| `accent` | `#F87171` | Primary buttons, active nav, links |
| `accent-hover` | `#FCA5A5` | Hover |
| `accent-pressed` | `#EF4444` | Pressed |
| `on-accent` | `#171717` | Text on accent backgrounds |
| `danger` | `#DC2626` | Destructive confirm buttons (white text) |
| `success` | `#4ADE80` | Success, "Good experience", Survived |
| `warning` | `#FBBF24` | Content warnings, "Unverified" outline, held notices |
| `info` | `#60A5FA` | Info banners, Official representative badge |
| `anon` | `#A78BFA` | Anonymous label badge |
| `focus-ring` | `#FBBF24` | 2 px outline, 2 px offset |
| `mark-bg` | `#3F3A1E` | Search highlight background |
| Monogram palette (6) | `#7F1D1D`, `#1E3A8A`, `#14532D`, `#581C87`, `#78350F`, `#134E4A` | Avatar monogram backgrounds (white text ≥ 4.5:1); index = hash(username) % 6 |

### 4.2 Typography

| Token | Font | Size / line height | Weight |
|---|---|---|---|
| `display-xl` | Bricolage Grotesque | 48/52 px | 800 |
| `display-lg` | Bricolage Grotesque | 36/42 px | 700 |
| `h1` | Bricolage Grotesque | 30/36 px | 700 |
| `h2` | Inter | 24/30 px | 600 |
| `h3` | Inter | 20/26 px | 600 |
| `body-lg` | Inter | 18/29 px | 400 |
| `body` | Inter | 16/26 px | 400 |
| `body-sm` | Inter | 14/21 px | 400/500 |
| `caption` | Inter | 12/17 px | 500 |
| `mono` | JetBrains Mono | 14/21 px | 400 |

Fonts load via `next/font/google` in the root layout (CSS variables `--font-display`, `--font-sans`, `--font-mono`). Long post bodies use `max-w-[68ch]`.

### 4.3 Spacing, radius, shadow, motion

| Group | Values |
|---|---|
| Spacing (4 px base) | `0`, `1`=4, `2`=8, `3`=12, `4`=16, `5`=20, `6`=24, `8`=32, `10`=40, `12`=48, `16`=64 px (Tailwind defaults) |
| Radius | `sm` 6 px · `md` 10 px · `lg` 16 px · `full` |
| Shadow | `shadow-dialog`: `0 12px 32px rgba(0,0,0,0.5)` (dialogs and popovers only). Elevation otherwise comes from lighter surfaces plus a 1 px `divider` border. |
| Motion | 150 ms ease-out (hover/press), 200 ms (dialogs); none under `prefers-reduced-motion` |
| Hit targets | ≥ 44 × 44 px on touch; ≥ 24 × 24 px desktop |

### 4.4 Breakpoints and layout

| Name | Min width | Shell layout (Arko) | Page content rule (everyone) |
|---|---|---|---|
| base | 0 | Top bar + bottom tab bar, 16 px gutters | Single column, full width |
| `sm` | 640 px | Same, 24 px gutters | Single column |
| `md` | 768 px | Top nav, no bottom bar | Single column, wider cards; two-column forms allowed |
| `lg` | 1024 px | Side nav 240 px + content max 680 px | Page content fits in 680 px |
| `xl` | 1280 px | + aside 320 px | Optional aside via `<PageAside>` slot (§10.4) |
| `2xl` | 1536 px | Centered, max 1440 px | — |

Admin and moderator pages (`/admin/**`, `/mod/**`) use a wide layout: content max 1200 px, no aside.

---

## 5. Shared TypeScript types

Owned by Arko at `src/shared/types/`. **This text is normative:** copy it verbatim if you need a local copy. Field names and shapes match [API_SPEC.md](../API_SPEC.md); enums match [DATABASE_SCHEMA.md §3](../DATABASE_SCHEMA.md).

### 5.1 `common.ts`

```ts
export type UUID = string;
export type ISODateTime = string; // "2027-01-12T08:15:00Z"
export type YearMonth = string;   // "2026-10"

export interface PageInfo { nextCursor: string | null; hasMore: boolean; limit: number }
export interface Page<T> { data: T[]; page: PageInfo }
export interface One<T> { data: T }
export interface PageParams { cursor?: string | null; limit?: number }

export type ErrorCode =
  | 'VALIDATION_ERROR' | 'INVALID_CURSOR' | 'TOKEN_INVALID' | 'UNAUTHENTICATED' | 'INVALID_CREDENTIALS'
  | 'FORBIDDEN' | 'CSRF_INVALID' | 'EMAIL_NOT_VERIFIED' | 'ONBOARDING_REQUIRED' | 'POSTING_RESTRICTED'
  | 'ACCOUNT_SUSPENDED' | 'ACCOUNT_TOO_NEW' | 'BLOCKED' | 'FEATURE_DISABLED' | 'NOT_FOUND' | 'CONFLICT'
  | 'ALREADY_EXISTS' | 'GONE' | 'PAYLOAD_TOO_LARGE' | 'UNSUPPORTED_MEDIA_TYPE' | 'SAFETY_CHECK_BLOCKED'
  | 'SAFETY_CONFIRMATION_REQUIRED' | 'ACCOUNT_LOCKED' | 'RATE_LIMITED' | 'INTERNAL_ERROR' | 'SERVICE_UNAVAILABLE';

export interface ErrorDetail { field?: string; issue: string; message?: string; [k: string]: unknown }
export interface ApiErrorBody { error: { code: ErrorCode; message: string; details?: ErrorDetail[]; requestId: string } }
```

### 5.2 `enums.ts`

```ts
export type PlatformRole = 'member' | 'platform_moderator' | 'admin';
export type AccountStatus = 'pending_verification' | 'active' | 'restricted' | 'suspended' | 'deletion_pending' | 'deleted';
export type CareerStatus = 'exploring' | 'studying' | 'employed' | 'open_to_escape' | 'between_jobs' | 'career_changing' | 'prefer_not_to_say';
export type ProfileVisibility = 'public' | 'members' | 'hidden';
export type IdentityMode = 'pseudonym' | 'anonymous';
export type MessagePermission = 'everyone' | 'requests_only' | 'nobody';
export type PostType = 'story' | 'advice' | 'lesson' | 'confession' | 'poll' | 'meme';
export type DesiredResponse = 'support' | 'advice' | 'discussion' | 'just_sharing';
export type ContentVisibility = 'public' | 'members' | 'community';
export type ModerationStatus = 'clear' | 'held' | 'limited' | 'removed';
export type ContentWarning = 'burnout' | 'harassment' | 'discrimination' | 'job_loss' | 'mental_health' | 'strong_language' | 'other';
export type ReactionType = 'im_dead' | 'pain' | 'same_here' | 'melting' | 'corporate_circus' | 'survived';
export type ConfessionExperienceType = 'management' | 'workload_overtime' | 'pay_issue' | 'workplace_conflict' | 'discrimination_harassment' | 'layoff' | 'mistake' | 'ethics_concern' | 'internship' | 'other';
export type CompanyPageStatus = 'pending' | 'active' | 'rejected' | 'merged' | 'suspended';
export type ReviewRelationship = 'current_employee' | 'former_employee' | 'applicant' | 'intern' | 'other';
export type ReviewTopic = 'work_life_balance' | 'management' | 'communication' | 'compensation' | 'career_growth';
export type EvidenceStatus = 'unverified' | 'verified';
export type JobReportType = 'misleading_job_description' | 'unclear_salary_or_benefits' | 'interview_process_problems' | 'candidate_ghosting' | 'unpaid_test_assignment' | 'unexpected_work_requirements' | 'offer_changed_or_withdrawn' | 'positive_hiring_experience';
export type InterviewStage = 'application' | 'screening' | 'technical' | 'onsite' | 'final' | 'offer';
export type JobReportOutcome = 'rejected' | 'ghosted' | 'withdrew' | 'offer_declined' | 'offer_accepted' | 'offer_rescinded' | 'still_waiting';
export type RoleCategory = 'software_engineering' | 'data' | 'design' | 'product' | 'marketing' | 'sales' | 'operations' | 'finance' | 'hr' | 'customer_support' | 'internship' | 'other';
export type MembershipStatus = 'pending_email' | 'pending_review' | 'verified' | 'rejected' | 'revoked' | 'expired';
export type CommunityVisibility = 'public' | 'private';
export type CommunityRole = 'member' | 'moderator' | 'owner';
export type JoinRequestStatus = 'pending' | 'approved' | 'declined';
export type ReportTargetType = 'post' | 'comment' | 'profile' | 'workplace_review' | 'job_report' | 'company_response' | 'community';
export type ReportReason = 'personal_information' | 'harassment_or_threats' | 'false_or_manipulated' | 'confidential_information' | 'spam' | 'impersonation' | 'discrimination_or_hate' | 'illegal_content' | 'other_policy_violation';
export type ReportStatus = 'open' | 'actioned' | 'no_violation' | 'duplicate';
export type ModerationTargetType = ReportTargetType | 'confession' | 'review_campaign';
export type ModerationPriority = 'high' | 'normal' | 'low';
export type ModerationItemStatus = 'open' | 'in_review' | 'escalated' | 'resolved';
export type ModerationActionType = 'no_action' | 'request_edits' | 'add_content_warning' | 'limit_visibility' | 'remove_content' | 'restrict_posting' | 'suspend_account' | 'escalate';
export type AppealStatus = 'pending' | 'upheld' | 'reversed' | 'modified';
export type RevealCategory = 'credible_threat' | 'legal_order' | 'repeat_abuse_ban_evasion';
export type RevealStatus = 'pending_approval' | 'approved' | 'rejected' | 'expired';
export type NotificationType = 'comment_on_post' | 'reply_to_comment' | 'reactions_on_post' | 'community_invitation' | 'community_announcement' | 'report_update' | 'moderation_decision' | 'followed_topic_digest' | 'company_response';
export type SafetyContext = 'post' | 'confession' | 'lesson' | 'comment' | 'workplace_review' | 'job_report' | 'company_response' | 'profile';
export type RiskLevel = 'none' | 'low' | 'medium' | 'high';
export type FeatureFlagKey = 'confessions.enabled' | 'communities.enabled' | 'registration.open' | 'beta.invite_only';
```

### 5.3 `identity.ts`

```ts
import type { AccountStatus, PlatformRole, CareerStatus, ProfileVisibility, IdentityMode, ContentVisibility, MessagePermission, FeatureFlagKey, NotificationType } from './enums';
import type { ISODateTime, UUID } from './common';

export interface IndustryRef { slug: string; name: string }
export interface Industry extends IndustryRef { id: number }

export interface MeProfile {
  username: string; displayName: string | null; headline: string | null; bio: string | null;
  industry: IndustryRef | null; broadLocation: string | null; careerStatus: CareerStatus | null;
  avatarUrl: string | null; usernameChangeAvailableAt: ISODateTime | null;
}
export interface Me {
  email: string; emailVerified: boolean; accountStatus: AccountStatus; platformRole: PlatformRole;
  permissions: Array<'identity_vault.reveal'>; onboardingCompleted: boolean;
  postingRestrictedUntil: ISODateTime | null; profile: MeProfile;
  deletionScheduledFor: ISODateTime | null; featureFlags: Record<FeatureFlagKey, boolean>;
  companyMemberships: Array<{ companyId: UUID; slug: string; name: string }>; // verified representative memberships
}
export interface SessionInfo { sessionId: UUID; deviceLabel: string; approxLocation: string | null; createdAt: ISODateTime; lastSeenAt: ISODateTime; isCurrent: boolean }
export interface PrivacySettings { profileVisibility: ProfileVisibility; discoverable: boolean; defaultConfessionIdentity: IdentityMode; defaultPostVisibility: Exclude<ContentVisibility, 'community'>; messagePermissions: MessagePermission }
export interface NotificationPreferences { inApp: Record<NotificationType, boolean>; email: Record<NotificationType, boolean>; quietHours: { start: string; end: string } | null; timezone: string }
export interface UsernameAvailability { username: string; available: boolean; reason: 'taken' | 'reserved' | 'invalid_format' | null }
export interface RegisterInput { email: string; password: string; username: string; ageConfirmed: true; acceptTerms: true; termsVersion: string; inviteCode?: string }
export interface LoginInput { email: string; password: string; rememberDevice: boolean }
export interface DeletionResult { status: 'deletion_pending'; scheduledFor: ISODateTime; retained: Array<{ item: string; period: string }> }
export interface DataExport { exportId: UUID; status: 'pending' | 'processing' | 'completed' | 'failed'; downloadUrl: string | null; expiresAt: ISODateTime | null }
```

### 5.4 `profile.ts`

```ts
import type { CareerStatus, ProfileVisibility, RoleCategory } from './enums';
import type { IndustryRef } from './identity';

export interface TimelineEntry { startYear: number; startMonth: number | null; endYear: number | null; endMonth: number | null; roleCategory: RoleCategory; industry: IndustryRef | null; note: string | null }
export interface PublicProfile {
  username: string; displayName: string | null; avatarUrl: string | null; headline: string | null; bio: string | null;
  industry: IndustryRef | null; broadLocation: string | null; careerStatus: CareerStatus | null;
  lessons: string[]; skills: string[]; timeline: TimelineEntry[];
  communities: Array<{ slug: string; name: string }>; memberSince: string; visibility: ProfileVisibility;
  restricted?: boolean; viewer: { isSelf: boolean; blocked: boolean; muted: boolean };
}
export interface MemberSearchResult { username: string; displayName: string | null; headline: string | null }
```

### 5.5 `content.ts`

```ts
import type { UUID, ISODateTime } from './common';
import type { PostType, DesiredResponse, ContentVisibility, ContentWarning, ReactionType, ModerationStatus, ConfessionExperienceType } from './enums';
import type { IndustryRef } from './identity';

export type AuthorRef =
  | { kind: 'member'; username: string; displayName: string | null; avatarUrl: string | null; headline: string | null }
  | { kind: 'anonymous'; label: string; isOp: boolean }
  | { kind: 'deleted' };

export interface CategoryRef { id: number; slug: string; name: string }
export interface Category extends CategoryRef { description: string; position: number }
export interface TagRef { slug: string; name: string }
export interface Tag extends TagRef { usageCount: number }
export type ReactionCounts = Record<ReactionType, number>;

export interface PostSummary {
  id: UUID; postType: PostType; title: string; excerpt: string; isLongForm: boolean; author: AuthorRef;
  category: CategoryRef; tags: TagRef[]; desiredResponse: DesiredResponse; language: string;
  visibility: ContentVisibility; contentWarnings: ContentWarning[]; reactionCounts: ReactionCounts; commentCount: number;
  community: { id: UUID; slug: string; name: string } | null; publishedAt: ISODateTime; editedAt: ISODateTime | null;
  lessonPreview?: { whatILearned: string };
  highlight?: string;               // search results only; HTML with <mark> tags only
  ownershipNote?: string;           // /me/posts only
  moderationStatus?: ModerationStatus; // /me/posts only
  viewer: { reaction: ReactionType | null; saved: boolean; isAuthor: boolean };
}
export interface ConfessionMeta { experienceType: ConfessionExperienceType; industry: IndustryRef | null; broadRegion: string | null; periodYear: number | null; periodQuarter: 1 | 2 | 3 | 4 | null }
export interface LessonSections { whatIDidWrong: string | null; whatHappened: string | null; whatILearned: string; whatIdDoDifferently: string | null; advice: string | null; links: Array<{ title: string; url: string }> }
export interface PollOption { id: UUID; label: string; voteCount: number | null; percentage: number | null }
export interface Poll { closesAt: ISODateTime; isClosed: boolean; totalVotes: number; options: PollOption[]; viewerVoteOptionId: UUID | null }
export interface MediaItem { mediaId: UUID; scanStatus: 'pending' | 'clean' | 'infected' | 'failed'; url: string | null; width: number | null; height: number | null; altText: string }
export interface Correction { id: UUID; body: string; createdAt: ISODateTime }
export interface PostDetail extends Omit<PostSummary, 'viewer'> {
  body: string; corrections: Correction[]; pinnedComment: Comment | null;
  confession: ConfessionMeta | null; lesson: LessonSections | null; poll: Poll | null; media: MediaItem[];
  moderationNotice: { status: ModerationStatus; message: string } | null;
  viewer: { reaction: ReactionType | null; saved: boolean; isAuthor: boolean; canEdit: boolean; canDelete: boolean };
}
export interface Comment {
  id: UUID; postId: UUID; parentCommentId: UUID | null; depth: 0 | 1 | 2; author: AuthorRef; body: string;
  contentWarnings: ContentWarning[]; isDeleted: boolean; isPinnedClarification: boolean;
  createdAt: ISODateTime; editedAt: ISODateTime | null; replyCount: number; replies?: Comment[];
  hiddenReason?: 'blocked' | 'muted';
  viewer: { isAuthor: boolean; canEdit: boolean };
}
export interface CreatePostInput {
  postType: Exclude<PostType, 'confession'>; title: string; body?: string; categoryId: number; tags: string[];
  desiredResponse: DesiredResponse; language: string; visibility: Exclude<ContentVisibility, 'community'> | 'community';
  contentWarnings: ContentWarning[]; communityId: UUID | null; acknowledgedFindings: string[];
  lesson?: Omit<LessonSections, 'whatILearned'> & { whatILearned: string };
  poll?: { options: string[]; durationDays: number }; mediaIds?: UUID[];
}
export interface CreateConfessionInput {
  title: string; body: string; experienceType: ConfessionExperienceType; industryId: number | null; broadRegion: string | null;
  periodYear: number | null; periodQuarter: 1 | 2 | 3 | 4 | null; categoryId: number; tags: string[];
  desiredResponse: DesiredResponse; contentWarnings: ContentWarning[]; identityMode: 'pseudonym' | 'anonymous';
  visibility: 'public' | 'members'; saveAsDraft: boolean; acknowledgedFindings: string[]; previewConfirmed: boolean;
}
export interface FeedParams { mode?: 'latest' | 'following'; category?: string[]; type?: PostType[]; language?: string; excludeCategory?: string[]; cursor?: string | null; limit?: number }
export interface SearchAllResult { posts: PostSummary[]; lessons: PostSummary[]; confessions: PostSummary[]; jobReports: import('./workplace').JobReport[]; companies: import('./workplace').CompanySummary[]; members: import('./profile').MemberSearchResult[]; communities: import('./community').CommunitySummary[] }
export type SearchType = 'all' | 'posts' | 'lessons' | 'confessions' | 'job_reports' | 'companies' | 'members' | 'communities';
```

### 5.6 `safety.ts`

```ts
import type { RiskLevel, SafetyContext, IdentityMode, ReportTargetType, ReportReason, ReportStatus, ModerationActionType, AppealStatus } from './enums';
import type { UUID, ISODateTime } from './common';

export interface SafetyFinding { rule: string; severity: 'low' | 'medium' | 'high'; field: string; start: number; end: number; message: string; blocking: boolean }
export interface SafetyCheckResult { riskLevel: RiskLevel; willBeHeld: boolean; findings: SafetyFinding[] }
export interface SafetyCheckInput { context: SafetyContext; identityMode?: IdentityMode; fields: Record<string, string> }
export interface ReportInput { targetType: ReportTargetType; targetId?: UUID; username?: string; reason: ReportReason; details?: string }
export interface MyReport { reportId: UUID; target: { type: ReportTargetType; summary: string }; reason: ReportReason; status: ReportStatus; createdAt: ISODateTime; resolvedAt: ISODateTime | null; outcomeMessage: string | null }
export interface MyModerationAction { actionId: UUID; actionType: ModerationActionType; target: { type: string; summary: string }; policyReference: string; memberMessage: string; createdAt: ISODateTime; appeal: { eligible: boolean; deadline: ISODateTime; status: AppealStatus | null } }
export type BlockTarget = { kind: 'member'; username: string } | { kind: 'anonymous'; label: string };
export interface Block { blockId: UUID; target: BlockTarget; createdAt: ISODateTime }
export type MuteTarget = { kind: 'member'; username: string } | { kind: 'category'; id: number; name: string } | { kind: 'tag'; slug: string };
export interface Mute { muteId: UUID; target: MuteTarget; createdAt: ISODateTime }
export interface Follow { followId: UUID; target: { kind: 'category'; id: number; name: string } | { kind: 'tag'; slug: string } }
```

### 5.7 `workplace.ts`

```ts
import type { UUID, ISODateTime, YearMonth } from './common';
import type { ReviewRelationship, ReviewTopic, EvidenceStatus, JobReportType, InterviewStage, JobReportOutcome, RoleCategory, CompanyPageStatus, ModerationStatus, MembershipStatus } from './enums';
import type { IndustryRef } from './identity';
import type { AuthorRef } from './content';

export interface CompanyRef { id: UUID; slug: string; name: string }
export interface CompanySummary extends CompanyRef { industry: IndustryRef | null; reviewCount: number; jobReportCount: number }
export interface TopicTheme { n: number; median: number | null; distribution: [number, number, number, number, number] | null; insufficient?: boolean }
export interface CompanyThemes { status: 'available' | 'insufficient_data' | 'paused'; windowMonths: 24; topics: Record<ReviewTopic, TopicTheme> }
export interface Company extends CompanyRef {
  industry: IndustryRef | null; officialWebsite: string | null; description: string | null; verificationNotice: string;
  themes: CompanyThemes; counts: { reviews: number; jobReports: number }; hasVerifiedRepresentative: boolean;
  pageStatus?: CompanyPageStatus; viewer: { hasReviewed: boolean; isRepresentative: boolean };
}
export interface CompanyResponse { id: UUID; body: string; badge: 'Official representative' | null; createdAt: ISODateTime; moderationStatus?: ModerationStatus }
export interface Review {
  id: UUID; author: { kind: 'anonymous'; label: string }; relationshipType: ReviewRelationship;
  period: { startYear: number; endYear: number | null }; title: string; narrative: string | null; narrativeHidden: boolean;
  ratings: Partial<Record<ReviewTopic, 1 | 2 | 3 | 4 | 5>>; evidenceStatus: EvidenceStatus; label: 'Unverified firsthand account' | 'Verified';
  companyResponse: CompanyResponse | null; publishedAt: ISODateTime | null;
  moderationNotice?: { status: ModerationStatus; message: string } | null; viewer: { isAuthor: boolean };
}
export interface ReviewInput { relationshipType: ReviewRelationship; periodStartYear: number; periodEndYear: number | null; title: string; narrative: string; ratings: Partial<Record<ReviewTopic, number>>; firsthandAttested: true; displayConsent: boolean; acknowledgedFindings: string[] }
export interface JobReport {
  id: UUID; company: CompanyRef; author: AuthorRef; roleCategory: RoleCategory; reportType: JobReportType;
  experienceMonth: YearMonth; stageReached: InterviewStage; outcome: JobReportOutcome | null; narrative: string;
  label: 'Unverified firsthand account'; companyResponse: CompanyResponse | null; publishedAt: ISODateTime;
  highlight?: string;
}
export interface JobReportInput { companyId: UUID | null; newCompany: { name: string; industryId: number; officialWebsite: string | null } | null; roleCategory: RoleCategory; reportType: JobReportType; experienceMonth: YearMonth; stageReached: InterviewStage; outcome: JobReportOutcome | null; narrative: string; identityMode: 'pseudonym' | 'anonymous'; visibility: 'public' | 'members'; acknowledgedFindings: string[] }
export interface CompanySubmitInput { name: string; industryId: number; officialWebsite: string | null }
export interface RepresentationRequest { membershipId: UUID; status: MembershipStatus }
```

### 5.8 `community.ts`

```ts
import type { UUID, ISODateTime } from './common';
import type { CommunityVisibility, CommunityRole, JoinRequestStatus } from './enums';

export interface CommunitySummary { id: UUID; slug: string; name: string; description: string; visibility: CommunityVisibility; memberCount: number; viewer: { role: CommunityRole | null; joinRequestStatus: JoinRequestStatus | null } }
export interface CommunityRule { title: string; body: string }
export interface PinnedResource { resourceId: UUID; title: string; url: string }
export interface Community extends CommunitySummary { rules: CommunityRule[]; pinnedResources: PinnedResource[]; moderators: Array<{ username: string }>; createdAt: ISODateTime }
export interface JoinRequest { requestId: UUID; username: string; message: string | null; createdAt: ISODateTime }
export interface CommunityMember { username: string; role: CommunityRole; joinedAt: ISODateTime }
export interface CreateCommunityInput { name: string; slug: string; description: string; visibility: CommunityVisibility; rules: CommunityRule[] }
```

### 5.9 `moderation.ts` and `admin.ts`

```ts
// moderation.ts
import type { UUID, ISODateTime } from './common';
import type { ModerationTargetType, ModerationPriority, ModerationItemStatus, ReportReason, ModerationActionType, ContentWarning, AppealStatus, RevealCategory, RevealStatus, AccountStatus } from './enums';
import type { AuthorRef } from './content';
import type { SafetyFinding } from './safety';

export interface QueueItem {
  itemId: UUID; targetType: ModerationTargetType; targetId: UUID; priority: ModerationPriority; status: ModerationItemStatus;
  automatedHold: boolean; reasons: Array<{ reason: ReportReason; count: number }>; reportCount: number;
  automatedFindings: Array<Pick<SafetyFinding, 'rule' | 'severity'>>;
  preview: { title: string | null; excerpt: string; author: AuthorRef }; ageMinutes: number; assignedTo: string | null;
}
export interface ModerationItemDetail extends QueueItem {
  content: { title: string | null; body: string; revisions: Array<{ title: string | null; body: string; createdAt: ISODateTime }>; corrections: Array<{ body: string; createdAt: ISODateTime }> };
  findings: SafetyFinding[];
  reports: Array<{ reportId: UUID; reason: ReportReason; details: string | null; reporter: string | null; createdAt: ISODateTime }>;
  authorHistory: { identityHidden: boolean; counts: Partial<Record<ModerationActionType, number>> };
  priorActions: Array<{ actionId: UUID; actionType: ModerationActionType; moderator: string; createdAt: ISODateTime }>;
  relatedItems: Array<{ itemId: UUID; targetType: ModerationTargetType; preview: string }>;
  canReveal: boolean;
}
export interface ModerationActionInput { actionType: ModerationActionType; reason: string; policyReference: string; memberMessage: string; durationDays: 1 | 7 | 30 | null; contentWarnings: ContentWarning[] | null; escalationNote: string | null; notifyReporters: boolean }
export interface AppealQueueItem { appealId: UUID; status: AppealStatus; reason: string; action: { actionId: UUID; actionType: ModerationActionType; moderator: string; policyReference: string; targetSummary: string; createdAt: ISODateTime }; canDecide: boolean; createdAt: ISODateTime }
export interface AppealDecisionInput { outcome: Exclude<AppealStatus, 'pending'>; decisionNote: string; modifiedAction: ModerationActionInput | null }
export interface RevealRequest { revealId: UUID; status: RevealStatus; category: RevealCategory; requester: string; approver: string | null; accessExpiresAt: ISODateTime | null; account: { userId: UUID; username: string; accountStatus: AccountStatus; createdAt: ISODateTime } | null }
export const POLICY_CODES = ['CG-1.1','CG-1.2','CG-1.3','CG-2.1','CG-2.2','CG-3.1','CG-3.2','CG-4.1','CG-4.2','CG-4.3','CG-5.1','CG-5.2','CG-6.1','CG-6.2'] as const;
export type PolicyCode = typeof POLICY_CODES[number];
```

```ts
// admin.ts
import type { UUID, ISODateTime } from './common';
import type { AccountStatus, PlatformRole, MembershipStatus, FeatureFlagKey } from './enums';
import type { CompanyRef } from './workplace';

export interface AdminMetrics { signups: number; verifiedMembers: number; postsPerDay: number; reportsPer1000Posts: number; openQueue: number; medianModerationResponseMinutes: { high: number; normal: number }; appealReversalRate: number }
export interface AdminUser { userId: UUID; email: string; username: string; accountStatus: AccountStatus; platformRole: PlatformRole; permissions: Array<'identity_vault.reveal'>; createdAt: ISODateTime; moderationSummary: Record<string, number> }
export interface AdminCompany extends CompanyRef { pageStatus: 'pending' | 'active' | 'rejected' | 'merged' | 'suspended'; industryId: number | null; officialWebsite: string | null; websiteConfirmed: boolean; description: string | null; submittedBy: string | null; possibleDuplicates: CompanyRef[]; createdAt: ISODateTime }
export interface AdminMembership { membershipId: UUID; company: CompanyRef; username: string; jobTitle: string; workEmailDomain: string; workEmailVerifiedAt: ISODateTime | null; status: MembershipStatus }
export interface AuditLogEntry { id: UUID; actorType: 'user' | 'system'; actor: { username: string } | null; action: string; targetType: string | null; targetId: UUID | null; metadata: Record<string, unknown>; requestId: string | null; createdAt: ISODateTime }
export interface FeatureFlag { key: FeatureFlagKey; enabled: boolean; updatedAt: ISODateTime; updatedBy: string | null }
export interface BetaInviteBatch { codes: string[] }
```

### 5.10 `notification.ts` and `index.ts`

```ts
// notification.ts
import type { UUID, ISODateTime } from './common';
import type { NotificationType } from './enums';
export interface AppNotification { id: UUID; type: NotificationType; title: string; body: string; link: string; actorCount: number; readAt: ISODateTime | null; createdAt: ISODateTime }
```

```ts
// index.ts
export * from './common'; export * from './enums'; export * from './identity'; export * from './profile';
export * from './content'; export * from './safety'; export * from './workplace'; export * from './community';
export * from './moderation'; export * from './admin'; export * from './notification';
```

Import types with `import type { PostSummary } from '@/shared/types'`.

---

## 6. Route map

**Auth levels:** `public` = anyone · `member` = logged in (any status) · `verified` = email verified + onboarding complete · `moderator` = platform_moderator or admin · `admin` = admin. Pages for `member`+ wrap their content in `<RequireAuth level="…">` (§10.3). Flags hide routes when off.

| Route | Page (screen) | Owner | Auth | Flag |
|---|---|---|---|---|
| `/` | Landing (S-01) | Shoaib | public (members redirect to `/mess`) | — |
| `/legal/privacy`, `/legal/terms`, `/legal/guidelines` | Policy pages | Tuli | public | — |
| `/signup` | Sign up (S-02) | Arko | public | `registration.open` |
| `/signup/complete` | Complete OAuth sign-up (S-02b) | Arko | public | — |
| `/login` | Log in (S-03) | Arko | public | — |
| `/verify-email` | Verify email (S-04) | Arko | public | — |
| `/forgot-password`, `/reset-password` | Forgot / reset (S-05) | Arko | public | — |
| `/onboarding` | Onboarding (S-06) | Arko | member | — |
| `/maintenance`, `/suspended` | System pages (S-38) | Arko | public | — |
| 404 / error boundary | System pages (S-38) | Arko | — | — |
| `/mess` | The Mess (S-07) | Shoaib | public | — |
| `/post/new` | Create post (S-08) | Shoaib | verified | — |
| `/posts/[postId]` | Post detail (S-11) | Shoaib | public | — |
| `/lessons` | Learned the Hard Way (S-14) | Shoaib | public | — |
| `/search` | Search (S-21) | Shoaib | public | — |
| `/notifications` | Notification center (S-36) | Shoaib | member | — |
| `/settings/notifications` | Notification settings (S-37) | Shoaib | member | — |
| `/confessions` | Confessions list (S-10) | Tuli | public | `confessions.enabled` |
| `/confessions/new` | Confession wizard (S-09) | Tuli | verified | `confessions.enabled` |
| `/u/[username]` | Public Anti-Profile (S-12) | Tuli | public | — |
| `/me` | My Damage (S-13) | Tuli | member | — |
| `/reports` | Report & appeal center (S-22) | Tuli | member | — |
| `/settings/profile` | Settings → Profile (S-25) | Tuli | member | — |
| `/settings/privacy` | Settings → Privacy (S-26) | Tuli | member | — |
| `/settings/blocked` | Settings → Blocked & muted (S-28) | Tuli | member | — |
| `/dev/ui` | Base kit preview (dev only) | Tuli | public, dev builds only | — |
| `/settings` | Redirect to `/settings/profile` | Arko | member | — |
| `/settings/sessions` | Settings → Sessions (S-27) | Arko | member | — |
| `/settings/account` | Settings → Account & data (S-29) | Arko | member | — |
| `/mod` | Moderator queue (S-23) | Arko | moderator | — |
| `/mod/items/[itemId]` | Moderation item detail (S-24) | Arko | moderator | — |
| `/mod/appeals` | Appeals tab (S-23) | Arko | moderator | — |
| `/companies` | Company directory (S-15) | Abnt | public | — |
| `/companies/new` | Submit company (S-18) | Abnt | verified | — |
| `/companies/[slug]` | Workplace Reality Page (S-16) | Abnt | public | — |
| `/companies/[slug]/review` | Write review (S-17) | Abnt | verified | — |
| `/companies/[slug]/represent` | Representative request (S-32) | Abnt | verified | — |
| `/job-nightmares` | Job Nightmare Board (S-19) | Abnt | public | — |
| `/job-nightmares/new` | Job report form (S-20) | Abnt | verified | — |
| `/job-nightmares/[jobReportId]` | Job report detail (S-19) | Abnt | public | — |
| `/communities` | Communities directory (S-33) | Abnt | public | `communities.enabled` |
| `/communities/new` | Create community (S-35) | Abnt | verified | `communities.enabled` |
| `/communities/[slug]` | Community page (S-34) | Abnt | public | `communities.enabled` |
| `/admin` | Admin dashboard (S-30) | Abnt | admin | — |
| `/admin/users`, `/admin/categories`, `/admin/companies`, `/admin/representatives`, `/admin/escalations`, `/admin/feature-flags`, `/admin/audit`, `/admin/invites` | Admin sub-pages (S-31) | Abnt | admin | — |

`src/shared/config/routes.ts` (Arko) exports these as typed helpers. **Never hard-code paths**:

```ts
export const routes = {
  home: () => '/', mess: () => '/mess', newPost: (type?: 'story' | 'advice' | 'lesson' | 'poll') => type ? `/post/new?type=${type}` : '/post/new',
  post: (id: string) => `/posts/${id}`, lessons: () => '/lessons', search: (q?: string) => q ? `/search?q=${encodeURIComponent(q)}` : '/search',
  notifications: () => '/notifications', confessions: () => '/confessions', newConfession: () => '/confessions/new',
  profile: (u: string) => `/u/${u}`, me: () => '/me', reports: () => '/reports',
  settings: { profile: () => '/settings/profile', privacy: () => '/settings/privacy', blocked: () => '/settings/blocked', sessions: () => '/settings/sessions', account: () => '/settings/account', notifications: () => '/settings/notifications' },
  companies: () => '/companies', newCompany: () => '/companies/new', company: (slug: string) => `/companies/${slug}`,
  review: (slug: string) => `/companies/${slug}/review`, represent: (slug: string) => `/companies/${slug}/represent`,
  jobNightmares: () => '/job-nightmares', newJobReport: () => '/job-nightmares/new', jobReport: (id: string) => `/job-nightmares/${id}`,
  communities: () => '/communities', newCommunity: () => '/communities/new', community: (slug: string) => `/communities/${slug}`,
  signup: () => '/signup', login: (next?: string) => next ? `/login?next=${encodeURIComponent(next)}` : '/login',
  verifyEmail: () => '/verify-email', forgotPassword: () => '/forgot-password', onboarding: () => '/onboarding',
  legal: { privacy: () => '/legal/privacy', terms: () => '/legal/terms', guidelines: () => '/legal/guidelines' },
  mod: { queue: () => '/mod', item: (id: string) => `/mod/items/${id}`, appeals: () => '/mod/appeals' },
  admin: { home: () => '/admin', users: () => '/admin/users', categories: () => '/admin/categories', companies: () => '/admin/companies', representatives: () => '/admin/representatives', escalations: () => '/admin/escalations', flags: () => '/admin/feature-flags', audit: () => '/admin/audit', invites: () => '/admin/invites' },
} as const;
```

---

## 7. Mock data conventions and the mock service pattern

### 7.1 Shared mock helpers (`src/shared/lib/mock/`, owner Arko)

| Export | Signature | Behavior |
|---|---|---|
| `mockRequest` | `<T>(fn: () => T \| Promise<T>, opts?: { latencyMs?: number; scenarioKey?: string }) => Promise<T>` | Waits `latencyMs` (default `NEXT_PUBLIC_MOCK_LATENCY_MS`, 400; 0 in tests), checks the active scenario, then returns `fn()`. Throws `ApiError` for error scenarios. |
| `ApiError` | `class ApiError extends Error { status: number; body: ApiErrorBody }` | Mirrors the real error envelope. `ApiError.of(status, code, message, details?)` builds one. |
| `paginate` | `<T>(items: T[], params: PageParams) => Page<T>` | Cursor = base64 of the index; default limit 20, max 50. |
| `getScenario` | `(scenarioKey: string) => 'ok' \| 'empty' \| 'error' \| 'slow' \| 'forbidden' \| 'rate_limited'` | Reads `?mock=<scenario>` from the URL (applies to all keys) or `localStorage['lo_mock:<scenarioKey>']`. `empty` makes list functions return `[]`; `slow` adds 3 s; `error` throws a 500 `INTERNAL_ERROR`; `forbidden` throws 403; `rate_limited` throws 429. |
| `mockId` | `(prefix: string, n: number) => UUID` | Deterministic UUID-shaped ids, e.g. `mockId('post', 1)` → `00000000-0000-7000-8000-706f73740001`. |
| `fixedNow` | `const fixedNow = '2027-01-20T12:00:00Z'` | All fixture dates are relative to this, so screenshots and tests are stable. |

### 7.2 The pattern (every feature)

```ts
// src/features/posts/api/types.ts
import type { Page, PostSummary, PostDetail, FeedParams, CreatePostInput, ReactionType, ReactionCounts } from '@/shared/types';
export interface PostsApi {
  getFeed(params: FeedParams): Promise<Page<PostSummary>>;
  getPost(postId: string): Promise<PostDetail>;
  createPost(input: CreatePostInput): Promise<PostDetail>;
  react(postId: string, type: ReactionType): Promise<{ reactionType: ReactionType | null; reactionCounts: ReactionCounts }>;
  // …
}

// src/features/posts/api/mock.ts
import { mockRequest, paginate, ApiError } from '@/shared/lib/mock';
import { posts } from '../mocks/fixtures';
export const mockPostsApi: PostsApi = {
  getFeed: (p) => mockRequest(() => paginate(filterPosts(posts, p), p), { scenarioKey: 'posts.feed' }),
  // … mutations update the in-memory fixture arrays so the UI reflects them until reload
};

// src/features/posts/api/index.ts   ← THE ONLY LINE THAT CHANGES AT INTEGRATION
export const postsApi: PostsApi = mockPostsApi;
```

Rules:
1. **Shapes are exactly the §5 types.** A mock must never invent fields.
2. **Hooks call `<feature>Api`, never fixtures directly.** Components never import from `mocks/`.
3. **Mutations are realistic.** They validate the same rules as the API (lengths, limits, duplicate checks) and throw `ApiError` with the documented codes, so error UI can be built now.
4. **Every list function honors the `empty` and `error` scenarios**, so empty and error states can be demonstrated with `?mock=empty` or `?mock=error`.
5. **In-memory state** lives in module-level arrays and resets on page reload. Nothing is written to `localStorage` except the mock session (§10) and scenario overrides.
6. **No network calls** while `NEXT_PUBLIC_USE_MOCKS=true`.

### 7.3 Canonical fixtures (use these IDs and names so pages link correctly after integration)

| Entity | Canonical values |
|---|---|
| Mock users (`src/shared/auth/mock-users.ts`) | `visitor` (no session) · `unverified` → `fresh_intern_42` (pending_verification) · `member` → `rejected_but_rising` (Priya; verified, onboarding done) · `member2` → `survived_standups` (Marcus) · `member3` → `deploy_on_friday` (Aisha) · `moderator` → `lena_mod` (platform_moderator + `identity_vault.reveal`) · `moderator2` → `mod_two` (platform_moderator + reveal) · `admin` → `site_admin` (admin) · `rep` → `ken_at_acme` (member, verified representative of Acme Widgets) · `restricted` → `too_spicy` (posting restricted until 2027-01-27) |
| Categories | ids 1–12 in the order of DATABASE_SCHEMA §12: 1 `career-failures`, 2 `interview-disasters`, 3 `office-drama`, 4 `management-problems`, 5 `salary-and-payment`, 6 `unpaid-overtime`, 7 `internship-reality`, 8 `developer-problems`, 9 `startup-lessons`, 10 `layoffs-and-job-loss`, 11 `career-changes`, 12 `workplace-wins` |
| Industries | ids 1–20 in DATABASE_SCHEMA §5.1 order (1 = technology) |
| Posts | `mockId('post', 1…40)`; post 1 = "Five rounds, one take-home, and then silence" (story by `rejected_but_rising`); posts 31–35 confessions (33–35 anonymous: "Anonymous Stapler #4821", "Anonymous Lanyard #0917", "Anonymous Whiteboard #1203"); posts 21–26 lessons |
| Comments | `mockId('comment', n)`; comment 42 = the comment with a phone number used in moderation fixtures |
| Companies | `acme-widgets` (`mockId('company', 1)`, 7 reviews → themes available), `globex-analytics` (`mockId('company', 2)`, 3 reviews → insufficient), `initech-systems` (`mockId('company', 3)`, pending) |
| Reviews / job reports | `mockId('review', n)`, `mockId('jobreport', n)` |
| Communities | `fresh-graduate-survivors`, `failed-startup-survivors`, `developer-debugging-support`, `interview-rejection-society`, `career-changers`, `workplace-boundaries`, `monday-survivors` (private) |
| Moderation items | `mockId('moditem', 1…8)`; item 1 = held anonymous confession post 33 (named individual); item 2 = comment 42 reported 3× for personal information |

Each owner creates their own fixtures inside their feature folder using these canonical values. Small overlaps (e.g. category names) are deliberately duplicated so nobody imports another feature's fixtures.

---

## 8. Base UI components (shared-contract, owner Tuli)

Path: `src/components/ui/<kebab-name>.tsx`, each exporting the PascalCase component. Built on shadcn/ui + Radix and themed with §4 tokens. All components forward refs, accept `className`, and pass through native attributes. **Local copies made by other teammates only need the same names and props.**

### 8.1 Inputs and actions

| Component (file) | Props | Variants / sizes | States and behavior |
|---|---|---|---|
| `Button` (`button.tsx`) | `variant?: 'primary' \| 'secondary' \| 'ghost' \| 'danger' \| 'link'` (default primary) · `size?: 'sm' \| 'md' \| 'lg'` (md) · `loading?: boolean` · `leftIcon?`, `rightIcon?: ReactNode` · `asChild?: boolean` · `fullWidth?: boolean` · native button props | primary = accent bg + on-accent text; secondary = surface + border-strong; ghost = transparent; danger = `danger` bg + white; link = accent text underline on hover | hover, pressed, focus-visible ring, disabled (opacity 50%, `aria-disabled`), loading (spinner replaces left icon, `aria-busy`, click ignored) |
| `IconButton` (`icon-button.tsx`) | `icon: ReactNode` · `label: string` (required, becomes `aria-label`) · `variant?: 'ghost' \| 'secondary'` · `size?: 'sm' \| 'md'` | 32 px / 40 px squares (44 px touch target via padding) | Tooltip shows `label` on hover/focus |
| `Input` (`input.tsx`) | native input props · `invalid?: boolean` · `leftAddon?`, `rightAddon?: ReactNode` | — | focus ring, invalid border `warning` + `aria-invalid`, disabled, read-only |
| `Textarea` (`textarea.tsx`) | native props · `invalid?` · `autoGrow?: boolean` · `maxLength?: number` · `showCount?: boolean` | — | Counter "123 / 5000" announced politely at ≥ 90% |
| `FormField` (`form-field.tsx`) | `label: string` · `htmlFor: string` · `hint?: string` · `error?: string` · `required?: boolean` · `children` | — | Renders label, hint and error with ids wired via `aria-describedby` |
| `Select` (`select.tsx`) | `value`, `onValueChange`, `options: {value: string; label: string; disabled?: boolean}[]`, `placeholder?`, `invalid?`, `disabled?` | — | Radix Select, type-ahead |
| `Combobox` (`combobox.tsx`) | `value: string \| null` · `onChange(v)` · `loadOptions(query): Promise<{value,label,description?}[]>` · `placeholder?` · `emptyText?` · `createOption?: { label: string; onCreate(query: string): void }` | — | Debounced 300 ms, loading row, empty row, keyboard up/down/enter |
| `ChipMultiSelect` (`chip-multi-select.tsx`) | `options: {value,label}[]` · `value: string[]` · `onChange(v)` · `max?: number` · `label: string` | — | Toggle chips with a check icon; when `max` is reached, the rest are disabled with a reason |
| `TagInput` (`tag-input.tsx`) | `value: string[]` · `onChange` · `suggest?(q): Promise<string[]>` · `max?: number` | — | Enter/comma adds; Backspace removes the last; normalizes to lowercase-hyphen |
| `Checkbox`, `RadioGroup`, `Switch` (`checkbox.tsx`, `radio-group.tsx`, `switch.tsx`) | Radix props + `label: string` · `description?: string` | RadioGroup `orientation?: 'vertical' \| 'horizontal'`, `options: {value,label,description?}[]` | disabled, focus ring |
| `RatingInput` (`rating-input.tsx`) | `value: number \| null` · `onChange` · `label: string` · `labels?: [string,string,string,string,string]` (default Very poor…Very good) · `allowSkip?: boolean` | — | Radio group of 1–5 with text labels |

### 8.2 Display, feedback and overlays

| Component (file) | Props | Variants | States and behavior |
|---|---|---|---|
| `Card` (`card.tsx`) + `CardHeader`, `CardBody`, `CardFooter` | `as?: 'div' \| 'article' \| 'section'` · `interactive?: boolean` · `padding?: 'sm' \| 'md'` | — | interactive = hover `surface-hover`; never a clickable wrapper (links inside) |
| `Badge` (`badge.tsx`) | `tone: 'neutral' \| 'accent' \| 'success' \| 'warning' \| 'info' \| 'anon' \| 'danger'` · `icon?` · `outline?: boolean` · `children` | — | Text is always present |
| `Chip` (`chip.tsx`) | `selected?: boolean` · `onClick?` · `onRemove?` · `href?` · `icon?` | — | Selected adds a check icon + `aria-pressed` |
| `Avatar` (`avatar.tsx`) | `username?: string` · `src?: string \| null` · `size?: 'xs' \| 'sm' \| 'md' \| 'lg'` (24/32/40/64) · `anonymous?: boolean` · `decorative?: boolean` | image, monogram (first 2 letters, palette §4.1), anonymous (mask icon on `anon`) | Image error falls back to the monogram |
| `AuthorBadge` (`author-badge.tsx`) | `author: AuthorRef` · `showHeadline?: boolean` · `size?: 'sm' \| 'md'` · `linkToProfile?: boolean` (default true) | member: Avatar + username (link to `/u/…`); anonymous: mask Avatar + label + `Badge tone="anon"` "Anonymous" + "OP" badge if `isOp`; deleted: "[deleted]" | Never links anonymous authors |
| `RelativeTime` (`relative-time.tsx`) | `iso: string` · `addSuffix?: boolean` | — | `<time dateTime>` with a full-date tooltip |
| `Modal` (`modal.tsx`) | `open` · `onOpenChange` · `title: string` · `description?: string` · `size?: 'sm' \| 'md' \| 'lg'` · `footer?: ReactNode` · `children` | — | Focus trap, Esc closes, returns focus, `shadow-dialog` |
| `ConfirmDialog` (`confirm-dialog.tsx`) | `open`, `onOpenChange`, `title`, `description`, `confirmLabel`, `cancelLabel?`, `destructive?: boolean`, `onConfirm(): Promise<void> \| void`, `typedConfirmation?: string` | destructive uses the danger button | Confirm shows loading; `typedConfirmation` (e.g. "DELETE") must be typed before enabling |
| `DropdownMenu` (`dropdown-menu.tsx`) | `trigger: ReactNode` · `items: Array<{ label: string; icon?: ReactNode; onSelect(): void; destructive?: boolean; disabled?: boolean } \| 'separator'>` · `align?: 'start' \| 'end'` | — | Radix; arrow keys |
| `Tooltip` (`tooltip.tsx`) | `content: string` · `children` · `side?` | — | Shows on hover and focus |
| `Popover` (`popover.tsx`) | `trigger` · `children` · `align?` | — | Radix |
| `Tabs` (`tabs.tsx`) | `value` · `onValueChange` · `tabs: {value: string; label: string; count?: number; disabled?: boolean}[]` · `children` (TabsContent by value) | `variant?: 'underline' \| 'pill'` | Arrow-key navigation |
| `Toast` (`toast.tsx`) + `Toaster` + `useToast()` | `toast({ title: string; description?: string; tone?: 'success' \| 'info' \| 'warning' \| 'error'; action?: { label: string; onClick(): void } })` | tone colors | `role="status"`, 5 s auto-dismiss (paused on hover/focus) |
| `Spinner` (`spinner.tsx`) | `size?: 'sm' \| 'md' \| 'lg'` · `label?: string` (default "Loading") | — | `role="status"` with visually hidden label |
| `Skeleton` (`skeleton.tsx`) | `variant?: 'line' \| 'block' \| 'circle'` · `width?`, `height?` · `lines?: number` | — | `aria-hidden`; parent sets `aria-busy` |
| `EmptyState` (`empty-state.tsx`) | `icon?: ReactNode` · `title: string` · `description?: string` · `action?: { label: string; href?: string; onClick?(): void }` | — | Icon `aria-hidden` |
| `ErrorState` (`error-state.tsx`) | `title?: string` (default "Something broke on our side. It's not you (this time).") · `error?: unknown` (shows `requestId` if an `ApiError`) · `onRetry?(): void` | — | `role="alert"` |
| `Banner` (`banner.tsx`) | `tone: 'info' \| 'warning' \| 'danger' \| 'success'` · `title?` · `children` · `action?` · `dismissible?` | — | Used for held notices, restrictions, feature-disabled notices |
| `Stepper` (`stepper.tsx`) | `steps: {id: string; title: string}[]` · `current: string` | — | `aria-current="step"`; mobile shows "Step 2 of 4" |
| `LoadMore` (`load-more.tsx`) | `hasMore: boolean` · `loading: boolean` · `onLoadMore(): void` · `auto?: boolean` | — | Auto mode uses an IntersectionObserver plus a visible button fallback |
| `VisuallyHidden` (`visually-hidden.tsx`) | `children` | — | sr-only |
| `PageHeader` (`page-header.tsx`) | `title: string` · `description?: string` · `actions?: ReactNode` · `breadcrumb?: {label: string; href?: string}[]` | — | Renders the page `h1` (focus target on navigation) |
| `DataTable` (`data-table.tsx`) | `columns: {key: string; header: string; render(row): ReactNode; width?: string}[]` · `rows: T[]` · `getRowId(row)` · `onRowClick?` · `loading?` · `empty?: ReactNode` | — | Semantic `<table>`; keyboard row focus; on mobile becomes stacked cards |

### 8.3 Cross-feature components (shared-contract, each with one owner)

Other teammates import these **only** from the feature's `index.ts`. Before the owner merges, use a local placeholder at the same path (§1).

| Component | Import path | Owner | Props | Used by |
|---|---|---|---|---|
| `PostCard` | `@/features/posts` | Shoaib | `post: PostSummary` · `variant?: 'default' \| 'compact'` · `showCommunity?: boolean` · `onHidden?(postId: string): void` (called after block/mute) | Shoaib (feed, lessons, search, landing), Tuli (profile, My Damage, confessions), Abnt (community feed) |
| `PostCardSkeleton` | `@/features/posts` | Shoaib | `count?: number` | same as above |
| `NotificationBell` | `@/features/notifications` | Shoaib | none (reads the unread count itself; renders nothing for visitors) | Arko (TopNav) |
| `SafetyCheckPanel` | `@/features/safety` | Tuli | `context: SafetyContext` · `fields: Record<string, string>` · `identityMode?: IdentityMode` · `acknowledged: string[]` · `onAcknowledgedChange(rules: string[]): void` · `onResult?(r: SafetyCheckResult): void` · `onJumpTo?(f: SafetyFinding): void` · `autoRun?: boolean` | Shoaib (composer, comments), Tuli (confessions, profile), Abnt (review, job report, company response, community post rules) |
| `useSafetyCheck` | `@/features/safety` | Tuli | `(input: SafetyCheckInput) => { result, run(), isRunning, error }` | same |
| `ReportDialog` | `@/features/safety` | Tuli | `open` · `onOpenChange` · `target: { type: ReportTargetType; id?: string; username?: string; summary: string }` | Shoaib (posts, comments), Tuli (profiles), Abnt (reviews, job reports, responses, communities) |
| `BlockMuteMenuItems` | `@/features/safety` | Tuli | `target: { author: AuthorRef; postId?: string; jobReportId?: string; category?: CategoryRef }` · `onDone?(kind: 'blocked' \| 'muted'): void` → returns an array of `DropdownMenu` items | Shoaib (PostCard, comments), Abnt (job report cards) |
| `CompanyCard` | `@/features/companies` | Abnt | `company: CompanySummary` · `action?: ReactNode` | Shoaib (search) |
| `JobReportCard` | `@/features/job-nightmares` | Abnt | `report: JobReport` · `variant?: 'default' \| 'compact'` | Shoaib (search), Abnt |
| `CommunityCard` | `@/features/communities` | Abnt | `community: CommunitySummary` | Shoaib (search) |
| `ReactionMeta` | `@/features/posts` | Shoaib | constant `REACTIONS: Array<{ type: ReactionType; label: string; icon: LucideIcon; tooltip: string }>` | Shoaib; read-only use by anyone |
| `ContentWarningGate` | `@/features/posts` | Shoaib | `warnings: ContentWarning[]` · `children` | Shoaib, Tuli (confession preview), Abnt (community feed) |
| `MarkdownRenderer` | `@/features/posts` | Shoaib | `source: string` · `className?` | Shoaib, Tuli (confession preview), Arko (moderation item content) |

---

## 9. State management

| Kind of state | Tool | Rule |
|---|---|---|
| Server/mock data | TanStack Query v5 | One `QueryClient` in `providers.tsx`. Hooks live in each feature's `hooks/`. |
| Session (current user) | `AuthProvider` React context (Arko) | Exposes `useSession()` (shape below). The mock session is chosen with `DevRoleSwitcher` and stored in `localStorage['lo_mock_user']`. |
| Feature flags | Part of `Me.featureFlags`; `useFlag(key)` from `@/shared/config/flags` | Visitors get the defaults in `flags.ts`. |
| UI state | Local `useState`/`useReducer` | No global store library. |
| URL state | `useUrlState` (Arko, `@/shared/hooks`) | Filters, tabs and search queries live in the URL. |
| Forms | React Hook Form + Zod resolver | Schemas in each feature's `schemas.ts`, mirroring API validation. |
| Theme | Fixed dark | No theme state. |

### 9.1 Global state shape

```ts
// src/shared/auth/useSession.ts
export type SessionStatus = 'loading' | 'anonymous' | 'authenticated';
export interface SessionState {
  status: SessionStatus;
  me: Me | null;                       // null when anonymous
  isVerified: boolean;                 // emailVerified && onboardingCompleted && accountStatus === 'active'
  hasRole(role: 'moderator' | 'admin'): boolean;
  hasPermission(p: 'identity_vault.reveal'): boolean;
  isRepresentativeOf(companyId: string): boolean;
  refresh(): Promise<void>;            // refetch Me (after onboarding, settings changes)
  signOut(): Promise<void>;
  // dev/mock only:
  setMockUser(key: MockUserKey): void;
}
export type MockUserKey = 'visitor' | 'unverified' | 'member' | 'member2' | 'member3' | 'moderator' | 'moderator2' | 'admin' | 'rep' | 'restricted';
```

```ts
// src/shared/config/flags.ts
export const defaultFlags: Record<FeatureFlagKey, boolean> = {
  'confessions.enabled': true, 'communities.enabled': true, 'registration.open': true, 'beta.invite_only': false,
};
export function useFlag(key: FeatureFlagKey): boolean;
```

### 9.2 Query key conventions

`[feature, resource, params?]`, for example `['posts', 'feed', params]`, `['posts', 'detail', postId]`, `['companies', 'reviews', companyId, filters]`, `['notifications', 'unread-count']`, `['moderation', 'queue', filters]`. Each feature invalidates **only its own keys**. Cross-feature effects (e.g. blocking someone from a PostCard) call the owner's `onHidden`/`onDone` callback, which refetches the owner's own data.

---

## 10. Layout, session and guard contracts (owner Arko)

### 10.1 Root providers

```tsx
// src/app/providers.tsx ('use client')
// <QueryClientProvider> → <AuthProvider> → {children} + <Toaster/> + (dev) <DevRoleSwitcher/> + <ReactQueryDevtools/>
```

Local copy for others: the same nesting, with `AuthProvider` reading `localStorage['lo_mock_user']` and resolving to the users in §7.3.

### 10.2 Page contract

- Every `page.tsx` renders exactly one feature page component, e.g. `export default function Page() { return <MessPage/> }`.
- Each page component renders a `PageHeader` (with the page's `h1`) and content that fits the §4.4 column width. It must look right **with or without** the shell.
- Pages set metadata via `export const metadata` in `page.tsx` (title format: `"<Page> · LinkedOut"`).

### 10.3 Guards

```tsx
<RequireAuth level="member" | "verified">{children}</RequireAuth>
// anonymous → redirect to routes.login(currentPath)
// level verified + unverified → <Banner tone="warning"> with link to /verify-email
// level verified + onboarding incomplete → redirect to /onboarding
// postingRestrictedUntil set → children render, but useSession().me.postingRestrictedUntil drives a restriction Banner inside composer pages (owners handle it)

<RequireRole role="moderator" | "admin">{children}</RequireRole>
// otherwise renders the 404 page (no information leak)

<FlagGate flag="confessions.enabled" fallback={<ComingSoon/>}>{children}</FlagGate>
```

### 10.4 Shell slots

- `src/app/(app)/layout.tsx` wraps pages in `<AppShell>`. Pages may export an aside by rendering `<PageAside>…</PageAside>` (a portal into the shell's aside region, shown from `xl`). Without the shell, `PageAside` renders nothing.
- Navigation items (`src/shared/config/nav.ts`): The Mess, Trauma Network, Job Nightmares, Confessions, Company Reality, Learned the Hard Way, Notifications, My Damage, Moderation (moderators), Admin (admins), plus the Share menu actions (UI_UX A.1).

---

## 11. Naming, files and imports

| Thing | Convention | Example |
|---|---|---|
| Component files | kebab-case file, PascalCase export | `post-card.tsx` → `PostCard` |
| Page components | `<Name>Page` in `pages/` | `features/feed/pages/mess-page.tsx` → `MessPage` |
| Hooks | `use-<name>.ts` → `useName` | `use-feed.ts` → `useFeed` |
| API | `api/types.ts` (`XxxApi`), `api/mock.ts` (`mockXxxApi`), `api/index.ts` (`xxxApi`) | `companiesApi.getCompany(slug)` |
| Fixtures | `mocks/fixtures.ts`, camelCase plural exports | `export const reviews: Review[]` |
| Copy | `copy.ts`, nested object `copy.<screen>.<key>` | `copy.mess.empty.title` |
| Schemas | `schemas.ts`, `<thing>Schema` | `reviewSchema` |
| Tests | `__tests__/<file>.test.tsx` | `__tests__/post-card.test.tsx` |
| E2E | `tests/e2e/<area>/<journey>.spec.ts` | `tests/e2e/content/journey-a.spec.ts` |
| CSS | Tailwind utilities with tokens only | `bg-surface text-text-secondary` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_TAGS` |
| Booleans | `is/has/can/should` prefix | `isLongForm` |
| Event handlers | `on<Event>` props, `handle<Event>` functions | `onLoadMore`, `handleSubmit` |

**Import rules**
1. Absolute imports via `@/`. No `../../` across feature boundaries.
2. A feature may import from: `@/shared/**`, `@/components/ui/**`, its own folder, and **other features' `index.ts` only**. It must never import another feature's `components/`, `api/`, `mocks/` or `hooks/`.
3. `app/**/page.tsx` files import only from their owner's feature folder.
4. Types are imported with `import type`.
5. Enforce with ESLint `no-restricted-imports` patterns `@/features/*/(components|api|mocks|hooks|pages)/*` (Arko adds the rule; teammates follow it from day 1).

---

## 12. Git rules

| Rule | Detail |
|---|---|
| Branches | One long-lived branch per person per module set: `feature/arko-platform`, `feature/shoaib-content`, `feature/tuli-member`, `feature/abnt-workplace`. Optional short sub-branches (`feature/<name>-<module>-<task>`) merge into your own branch. |
| Commits | Conventional Commits with scope = feature folder: `feat(feed): add category filter chips`, `test(safety): cover blocking findings`, `fix(companies): theme summary rounding`. Footer `Refs: FE-S-07, US-026`. |
| What you may commit | Only paths listed as yours in §3 and in your personal file. |
| Off-limits for everyone except the owner | `package.json`, lockfile, all config files, `src/app/layout.tsx`, `providers.tsx`, `globals.css`, `middleware.ts`, `src/shared/**`, `src/styles/**`, `src/components/layout/**` (Arko) · `src/components/ui/**`, `components.json` (Tuli) · any other person's `src/features/<f>/**`, route folders and E2E folders · this contract (all four approve changes). |
| Need a dependency? | Don't add it yourself. Write it in `docs/team/README.md` → "Dependency requests" (via PR to your own section); Arko adds it in the integration step. Meanwhile, implement without it. |
| Local copies | Shared-contract files you don't own go in `.git/info/exclude`. `git status` must never show them. |
| Pull requests | Each person opens **one PR per finished vertical slice** into their own long-lived branch (self-review against the DoD), and finally one integration PR per person into `main` (§ README "Merge plan"). Every PR to `main` needs 1 approval from another teammate, green lint/typecheck/tests, and screenshots (mobile + desktop). |
| Rebasing | Rebase your branch on `main` only during the integration week. Before that, `main` only contains the scaffold and docs. |

---

## 13. Definition of Done (every task)

1. **Matches the docs:** the UI follows [UI_UX.md](../UI_UX.md) for the screen and the linked US acceptance criteria are demonstrable with mocks.
2. **All states:** loading (skeletons, no layout shift), empty, error (with retry), success, plus the relevant special states (held, restricted, feature disabled, forbidden). Each is reachable with `?mock=empty|error|slow|forbidden` or a mock user.
3. **Responsive:** checked at 360 px, 768 px, 1280 px; no horizontal scroll at 320 px; touch targets ≥ 44 px on mobile.
4. **Accessible:** keyboard operable, visible focus ring, labelled controls, correct roles and ARIA, color contrast via tokens only, `prefers-reduced-motion` respected, 0 axe serious/critical violations on the page.
5. **Typed and lint-clean:** `pnpm typecheck` and `pnpm lint` pass with zero warnings; no `any`; no restricted imports.
6. **Tested:** the component and hook tests listed in your file pass (`pnpm test`).
7. **No console errors or warnings** in the browser or tests (including React key and act warnings).
8. **Copy:** strings come from the feature's `copy.ts` and follow the tone rules (UI_UX C.7: witty, never cruel; no "untraceable" claims; no manipulative notifications).
9. **Mock fidelity:** data comes through `<feature>Api` with §5 shapes only; the error codes used match API_SPEC.
10. **Privacy UI rules:** never show anonymous-author identity, never show profile counts of reactions or failures, never show private fields.
11. Your personal task list is ticked and your branch builds (`pnpm build`).
