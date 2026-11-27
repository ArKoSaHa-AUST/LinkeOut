# Shoaib — Content: The Mess, Posts, Comments, Reactions, Search & Notifications

## 1. Role summary

Shoaib owns the **core content experience**:
- the landing page and The Mess feed (Latest + Following);
- the post composer (story, advice, lesson, poll, images);
- post detail with comments, reactions, saves, corrections and pinned clarifications;
- Learned the Hard Way, search, and notifications (bell, center, settings).

Shoaib also builds the cross-feature content components other teammates import from `@/features/posts` and `@/features/notifications`: `PostCard`, `PostCardSkeleton`, `ContentWarningGate`, `MarkdownRenderer`, `REACTIONS`, and `NotificationBell`. Their props are fixed in the contract, so nobody waits for them. Everything runs on Shoaib's own mock services.

## 2. Owned folders and files

All paths are under `apps/web/`.

| Path | Contents |
|---|---|
| `src/features/posts/**` | PostCard, ContentWarningGate, MarkdownRenderer, ReactionBar, SaveButton, PostMenu, Composer (+ lesson, poll, media), PostDetail, Corrections, PollCard; posts mock API |
| `src/features/comments/**` | CommentThread, CommentItem, CommentForm, PinnedClarification; comments mock API |
| `src/features/feed/**` | MessPage, LandingPage, FeedFilters, FeedList, FeedModeTabs, WinsAside, FollowButton; feed mock API (feed, categories, tags, follows) |
| `src/features/lessons/**` | LessonsPage, LessonCard |
| `src/features/search/**` | SearchPage, SearchResults; search mock API |
| `src/features/notifications/**` | NotificationBell, NotificationCenterPage, NotificationSettingsPage; notifications mock API |
| `src/app/(marketing)/page.tsx` | Landing route |
| `src/app/(app)/mess/**`, `src/app/(app)/post/**`, `src/app/(app)/posts/**`, `src/app/(app)/lessons/**`, `src/app/(app)/search/**`, `src/app/(app)/notifications/**`, `src/app/(app)/settings/notifications/**` | Route files |
| `tests/e2e/content/**` | E2E specs |

## 3. Pages and screens

### 3.1 S-01 Landing — `/`
- **Purpose:** explain LinkedOut and convert visitors. **US:** US-001 (preview). **FR:** FR-24.
- **Components:** `LandingHero` ("Your career. Your chaos.", sub-line, CTAs "Join LinkedOut" / "Read the Mess"), `PrivacyStrip` (3 cards: Pseudonym · Anonymous to members · What we never promise), `LatestPreview` (5 `PostCard variant="compact"`), `IsIsntColumns` (plan §2 lists).
- **UI details:** `display-xl` headline; preview links each card to `/posts/[id]`; members are redirected to `/mess` (middleware, Arko).
- **States:** loading → 5 compact skeletons; empty → "The Mess is suspiciously clean. Be the first to share."; error → the preview section is hidden and the static content stays.
- **Responsive:** hero stacks on mobile; privacy cards 1 → 3 columns from `md`.
- **Mock data:** `feedApi.getFeed({ limit: 5 })` → `Page<PostSummary>`.
- **Actions:** Join → `/signup`; Read the Mess → `/mess`; card → post detail.

### 3.2 S-07 The Mess — `/mess`
- **Purpose:** central discovery feed. **US:** US-001, US-026, US-027, US-057. **FR:** FR-24, FR-25, FR-69, FR-70.
- **Components:** `FeedModeTabs` (Latest | Following; Following only for members), `FeedFilters` (category multi-chips, type select, language select, each category chip has a "Hide this category" menu item), `FeedList` (PostCards + `LoadMore auto`), `WinsAside` (rendered in `PageAside`: latest "Workplace Wins and Recovery" posts, guidelines snippet, "Feeling heavy? Mute topics" link to `/settings/blocked`), `FollowSuggestions` (empty Following), `FollowButton` on category chips (Should).
- **PostCard (exported):** AuthorBadge, relative time, category chip, type badge (Advice/Lesson/Confession/Poll), title link, 3-line excerpt or `ContentWarningGate`, tags, `ReactionBar` (compact), comment count, `SaveButton`, `PostMenu` (Report → `ReportDialog`, Block/Mute → `BlockMuteMenuItems`, Mute category, Copy link). Members-only posts show a lock overlay with "Sign in to read" for visitors.
- **States:** loading 5 skeletons; empty (filters) "Nothing here yet. Either this category is thriving or everyone's in a meeting." + "Clear filters"; empty Following → FollowSuggestions; error → ErrorState with retry (already-loaded pages stay); hidden category → toast "Hidden. Find it again in Settings → Blocked & muted."
- **Responsive:** filters collapse into a "Filters" button that opens a bottom sheet on mobile; aside appears from `xl`.
- **Mock data:** `feedApi.getFeed(params: FeedParams): Page<PostSummary>` over 40 fixture posts (contract §7.3), filtering by category, type and language; `feedApi.listCategories(): Category[]`; `feedApi.hideCategory(id) → Mute`; `feedApi.listFollows(): Follow[]`, `follow(...)`, `unfollow(id)`.
- **Actions:** change filter → URL query updates and the list refetches; react → optimistic count update; save → toggle + toast; Report/Block/Mute → contract components, then `onHidden` removes the card; scroll → next page.

### 3.3 S-08 Create post — `/post/new?type=story|advice|lesson|poll`
- **Purpose:** compose stories, advice requests, lessons, polls. **US:** US-022, US-023, US-024, US-037, US-054, US-056. **FR:** FR-20, FR-22, FR-23, FR-35, FR-67, FR-68.
- **Components:** `ComposerPage` inside `RequireAuth level="verified"`; `PostTypeSwitcher`; `TitleField` (counter /150); `MarkdownEditor` (toolbar bold/italic/list/quote/link, Write/Preview tabs, highlight ranges for findings); `LessonFields` (6 sections; "What I learned" required; up to 5 https links); `PollFields` (2–6 options, duration 1–14 days); `CategorySelect`; `TagInput` (autocomplete, max 5); `DesiredResponseRadio`; `ContentWarningSelect`; `VisibilitySelect` (Public / Members only); `LanguageSelect`; `ImagePicker` (≤ 4 images, JPEG/PNG/WebP/GIF ≤ 5 MB, alt text required, "Checking image…" state); `SafetyCheckPanel` (from `@/features/safety`); `DiscardDialog`; posting-restriction `Banner`.
- **UI details:** Publish runs the safety check first. Medium findings need acknowledgement checkboxes in the panel; blocking findings disable Publish.
- **States:** submitting (form read-only, button loading); `SAFETY_CHECK_BLOCKED` / `SAFETY_CONFIRMATION_REQUIRED` → panel shows findings; `POSTING_RESTRICTED` → banner with end date; success → `/posts/[id]` + toast "Posted. The Mess thanks you."; held → post page with the moderation notice; dirty + leave → DiscardDialog; draft text autosaved to `localStorage` every 5 s.
- **Responsive:** single column; sticky Publish bar on mobile; side-by-side Write/Preview from `lg`.
- **Mock data:** `postsApi.createPost(input: CreatePostInput): PostDetail` (validates the limits in FR-20/FR-35/FR-67; text containing "John Smith" → `moderationNotice.status = 'held'`); `feedApi.suggestTags(q): Tag[]`; `postsApi.uploadImage(file, altText) → MediaItem` (pending → clean after 1.5 s; files named `eicar*` → infected).
- **Actions:** type switch keeps shared fields; add image → validation and upload progress; publish → create → navigate.

### 3.4 S-11 Post detail & comments — `/posts/[postId]`
- **Purpose:** read and discuss. **US:** US-002, US-025, US-028, US-029, US-030, US-031, US-032, US-036, US-055. **FR:** FR-20, FR-21, FR-26–FR-30, FR-34, FR-67.
- **Components:** `PostDetailView` (breadcrumb, header with AuthorBadge, edited marker, badges "Asking for advice"/type, `ModerationNoticeBanner` for authors, `ContentWarningGate`, `MarkdownRenderer` body with long-form "Read more", `LessonSectionsView`, `ConfessionMeta` row, `CorrectionsBlock`, `PollCard`, `ImageGallery`), `ReactionBar` (full), `SaveButton`, Share (copy link), `PostMenu` (author: Edit → composer in edit mode at `/post/new?edit=<id>`, Delete → ConfirmDialog, Add correction → `CorrectionDialog`; others: Report, Block), `PinnedClarification`, `CommentThread` (sort Oldest/Newest; depth ≤ 3 with indentation, "Replying to @x" beyond; "[deleted]"; "Hidden: you blocked this member"; OP badge), `CommentForm` (sticky on mobile, CW toggle, safety check on submit), comment menu (Edit, Delete, Pin as clarification for the post author's own comments, Report, Block).
- **States:** loading header/body skeleton then comments skeleton; not found/held/removed (non-author) → 404 component; members-only for visitors → title + "Sign in to read"; empty comments → "No replies yet. Be the person you needed when this happened to you."; `BLOCKED` on comment → "You can't comment on this post"; 429 → "You're going a bit fast"; correction limit → message.
- **Responsive:** body `max-w-[68ch]`; comment indentation shrinks to 12 px per level on mobile.
- **Mock data:** `postsApi.getPost(id): PostDetail` (posts 1–40, including 21–26 lessons, 31–35 confessions, one poll post 40, one held post 39 authored by `rejected_but_rising`); `commentsApi.list(postId, {sort, cursor}): Page<Comment>`, `listReplies(commentId)`, `create`, `update`, `remove`, `pin(postId, commentId)`, `unpin(postId)`; `postsApi.react`, `unreact`, `save`, `unsave`, `addCorrection(postId, body) → Correction`, `deletePost`, `votePoll(postId, optionId) → Poll`.
- **Actions:** react (toggle/change; arrow-key picker); save; comment/reply; edit/delete own; pin; add correction; vote; delete post → `/mess` + toast.

### 3.5 S-14 Learned the Hard Way — `/lessons`
- **Purpose:** browse lessons. **US:** US-038. **FR:** FR-35.
- **Components:** intro banner "Mistakes, converted into advice", category chips, `LessonCard` (title, "What I learned" preview, category, author, reactions), "Write a lesson" CTA → `/post/new?type=lesson`.
- **States:** loading; empty "No lessons here yet. Someone has definitely learned something, though."; error.
- **Mock data:** `feedApi.getLessons({ category, cursor }) → Page<PostSummary>` with `lessonPreview`.

### 3.6 S-21 Search — `/search?q=&type=`
- **Purpose:** find content. **US:** US-004, US-044. **FR:** FR-43.
- **Components:** `SearchInput` (≥ 2 chars hint), `SearchTypeTabs` (All, Posts, Lessons, Confessions, Job reports, Companies, Members, Communities), category filter, `SearchAllResults` (groups with "See all"), typed lists using `PostCard`, `JobReportCard` (from `@/features/job-nightmares`), `CompanyCard` (from `@/features/companies`), `CommunityCard` (from `@/features/communities`), `MemberResult`; highlights rendered only as `<mark>` (styled with `mark-bg` + underline).
- **States:** idle (no query) → tips; too short → inline hint, no request; loading; empty "No results for '{q}'. Try fewer words or a different spelling."; error.
- **Responsive:** tabs scroll horizontally on mobile.
- **Mock data:** `searchApi.searchAll(q, category?) → SearchAllResult` (top 5 per group), `searchApi.search(q, type, cursor) → Page<…>`; fixtures include job reports, companies (acme-widgets, globex-analytics), members (discoverable only) and communities, using canonical IDs.
- **Actions:** submit → URL update; tab → type change; result click → that owner's route via `routes.*`.

### 3.7 S-36 Notification center — `/notifications` (+ `NotificationBell`)
- **Purpose:** see replies and decisions. **US:** US-052. **FR:** FR-64.
- **Components:** `NotificationBell` (exported; bell icon with unread count badge, polls `getUnreadCount` every 60 s while the tab is visible, links to `/notifications`, label "Notifications, 4 unread"), `NotificationCenterPage` (filter All/Unread, `NotificationRow` with icon by type, text, time, unread dot; "Mark all as read"; link to settings).
- **States:** loading; empty "All quiet. Enjoy it."; error; visitors → bell not rendered.
- **Mock data:** `notificationsApi.list({ unreadOnly, cursor }) → Page<AppNotification>` (12 items covering all 9 types, including a batched "12 people reacted to your post"), `getUnreadCount() → { count }`, `markRead(id)`, `markAllRead() → { updated }`.
- **Actions:** click row → mark read + navigate to `link`; mark all → count 0.

### 3.8 S-37 Settings → Notifications — `/settings/notifications`
- **Purpose:** control notifications. **US:** US-053. **FR:** FR-65, FR-66.
- **Components:** `PreferencesMatrix` (rows = 9 types, columns In-app / Email switches; "Moderation decision" email locked on with an explanation), `QuietHoursFields` (start, end, time zone select), Save.
- **States:** loading; error; saving; success toast "Saved. We'll keep it down."; validation `required_notification`.
- **Responsive:** matrix becomes stacked cards per type on mobile.
- **Mock data:** `notificationsApi.getPreferences(): NotificationPreferences`, `updatePreferences(p)`.

## 4. Task checklist (build order)

### Phase A — Content foundations
- [ ] **FE-S-01** · 3h · `posts` and `feed` fixtures: 40 posts per contract §7.3 (all types, warnings, languages, anonymous authors, communities), categories, tags. **AC:** every object type-checks against `PostSummary`/`PostDetail`.
- [ ] **FE-S-02** · 3h · `feedApi` + `postsApi` mocks (getFeed with filters, getPost, getLessons, categories, tags, react/save, follows, hideCategory). **AC:** scenarios `empty`/`error` honored; reactions keep counts consistent.
- [ ] **FE-S-03** · 2h · `MarkdownRenderer` (react-markdown + rehype-sanitize allow-list, safe links) + `ContentWarningGate`. **AC:** script/`javascript:` payloads render inert; gate hides content from screen readers until revealed.
- [ ] **FE-S-04** · 4h · `PostCard` (default, compact, locked, anonymous author, community chip) + `PostCardSkeleton` + exports in `index.ts`. **AC:** matches UI_UX S-07 card; title is the only card link.
- [ ] **FE-S-05** · 4h · `ReactionBar` + picker (6 reactions, counts, optimistic toggle/change, arrow keys, disabled for visitors/unverified with a sign-in prompt). **AC:** US-031 criteria 1–5.
- [ ] **FE-S-06** · 2h · `SaveButton` and `PostMenu` (Report/Block/Mute/Copy link wiring to safety components). **AC:** `onHidden` removes the card.

### Phase B — Feed and landing
- [ ] **FE-S-07** · 3h · `FeedFilters` with URL state (category multi, type, language, hide category). **AC:** US-026, US-027 criteria.
- [ ] **FE-S-08** · 3h · `FeedList` with infinite scroll, LoadMore fallback, loading/empty/error states. **AC:** no duplicates across pages.
- [ ] **FE-S-09** · 3h · `MessPage` with mode tabs, `WinsAside`, mobile filter sheet. **AC:** US-001 criteria 1–5.
- [ ] **FE-S-10** · 3h · Following mode, `FollowButton`, follow suggestions (Should). **AC:** US-057 criteria 1–2.
- [ ] **FE-S-11** · 3h · Landing page S-01. **AC:** sections per UI_UX S-01; preview states.

### Phase C — Composer
- [ ] **FE-S-12** · 4h · Composer base: type switcher, title, MarkdownEditor with preview, category, tags, desired response, warnings, visibility, language; Zod schema. **AC:** US-022, US-023, US-024 criteria.
- [ ] **FE-S-13** · 3h · Safety integration (`SafetyCheckPanel`, acknowledgements, jump-to-range highlighting), restriction banner, autosave, DiscardDialog. **AC:** US-022 criterion 3; 422 handling.
- [ ] **FE-S-14** · 2h · Lesson mode fields. **AC:** US-037 criteria 1–3.
- [ ] **FE-S-15** · 2h · Poll mode fields (Should). **AC:** US-054 criteria 1–2.
- [ ] **FE-S-16** · 3h · `ImagePicker` with type/size validation, alt text, scan states (Should). **AC:** US-056 criteria 1–4 (EXIF note shown as "We remove location data from photos").
- [ ] **FE-S-17** · 2h · Edit mode (`?edit=<id>`) prefilled from `getPost`, saves via `updatePost`. **AC:** US-025 criterion 1.

### Phase D — Post detail and comments
- [ ] **FE-S-18** · 4h · `PostDetailView`: header, notices, gate, body, lesson/confession sections, images, author menu (edit, delete with ConfirmDialog). **AC:** US-002, US-025 criteria 2–3.
- [ ] **FE-S-19** · 2h · `CorrectionsBlock` + `CorrectionDialog`. **AC:** US-036 criteria 1–3.
- [ ] **FE-S-20** · 2h · `PollCard` voting and results (Should). **AC:** US-055 criteria 1–3.
- [ ] **FE-S-21** · 3h · `comments` mock API + fixtures (nested threads, deleted, blocked-hidden, OP anonymous replies, pinned).
- [ ] **FE-S-22** · 4h · `CommentThread` (sort, nesting, placeholders, OP badge, load replies). **AC:** US-028 criterion 2; correct semantics.
- [ ] **FE-S-23** · 3h · `CommentForm` (safety check, CW toggle, BLOCKED and 429 errors) + edit/delete + pin as clarification. **AC:** US-028, US-029, US-030 criteria.

### Phase E — Lessons, search, notifications
- [ ] **FE-S-24** · 2h · Lessons page S-14. **AC:** US-038 criteria.
- [ ] **FE-S-25** · 3h · `searchApi` mock + fixtures (posts, lessons, confessions, job reports, companies, members, communities) with highlights.
- [ ] **FE-S-26** · 4h · Search page S-21 (input, tabs, grouped and typed results). **AC:** US-004, US-044 criteria; highlights render only `<mark>`.
- [ ] **FE-S-27** · 2h · `notifications` mock API + fixtures (12 notifications, preferences).
- [ ] **FE-S-28** · 2h · `NotificationBell` (polling with visibility API, accessible label). **AC:** US-052 criterion 1.
- [ ] **FE-S-29** · 3h · Notification center S-36. **AC:** US-052 criteria 1–3.
- [ ] **FE-S-30** · 3h · Notification settings S-37. **AC:** US-053 criteria 1–3.

### Phase F — Testing and accessibility
- [ ] **FE-S-31** · 3h · Component tests: PostCard, ContentWarningGate, MarkdownRenderer, ReactionBar.
- [ ] **FE-S-32** · 3h · Component tests: Composer (validation, lesson, poll, safety acknowledgement, image validation).
- [ ] **FE-S-33** · 3h · Component tests: CommentThread/CommentForm, PollCard, SearchPage, NotificationBell, PreferencesMatrix.
- [ ] **FE-S-34** · 3h · E2E (`tests/e2e/content`): browse feed with filters, publish story, react/comment/save, search; axe on S-01, S-07, S-08, S-11, S-21. **AC:** green on chromium + mobile-chrome.
- [ ] **FE-S-35** · 2h · Accessibility and responsive pass on all owned screens. **AC:** 0 axe serious/critical; 320 px OK.

## 5. Mock data and mock services

| File | Exports | Returns |
|---|---|---|
| `src/features/feed/api/mock.ts` | `mockFeedApi`: `getFeed`, `getLessons`, `listCategories`, `suggestTags`, `hideCategory`, `listFollows`, `follow`, `unfollow` | `Page<PostSummary>`, `Category[]`, `Tag[]`, `Mute`, `Follow[]`, `Follow` |
| `src/features/posts/api/mock.ts` | `mockPostsApi`: `getPost`, `createPost`, `updatePost`, `deletePost`, `addCorrection`, `react`, `unreact`, `save`, `unsave`, `votePoll`, `uploadImage`, `getMedia` | `PostDetail`, `Correction`, `{reactionType, reactionCounts}`, `Poll`, `MediaItem` |
| `src/features/comments/api/mock.ts` | `mockCommentsApi`: `list`, `listReplies`, `create`, `update`, `remove`, `pin`, `unpin` | `Page<Comment>`, `Comment`, `{ pinnedCommentId }` |
| `src/features/search/api/mock.ts` | `mockSearchApi`: `searchAll`, `search` | `SearchAllResult`, `Page<…>` |
| `src/features/notifications/api/mock.ts` | `mockNotificationsApi`: `list`, `getUnreadCount`, `markRead`, `markAllRead`, `getPreferences`, `updatePreferences` | `Page<AppNotification>`, `{count}`, `{updated}`, `NotificationPreferences` |

Fixtures: 40 posts, 60 comments (with depth 0–2, deleted, OP and pinned examples), 25 tags, 12 categories, 6 follows, 12 notifications, search fixtures (8 job reports, 3 companies, 6 members, 7 communities) built with canonical IDs and names (contract §7.3).

## 6. Testing tasks

| Test file | What to test |
|---|---|
| `posts/__tests__/post-card.test.tsx` | Author kinds (member/anonymous/deleted), badges, locked overlay, menu items, `onHidden` |
| `posts/__tests__/content-warning-gate.test.tsx` | Hidden by default (incl. from the accessibility tree), reveal moves focus |
| `posts/__tests__/markdown-renderer.test.tsx` | XSS payloads inert; links get `rel="nofollow ugc noopener noreferrer"` |
| `posts/__tests__/reaction-bar.test.tsx` | Toggle, change, remove, optimistic rollback on error, arrow-key navigation, accessible names with counts |
| `posts/__tests__/composer.test.tsx` | Title/body limits, ≤ 5 tags, lesson required field, https links, poll 2–6 options, safety blocking vs acknowledgement, restriction banner |
| `comments/__tests__/comment-thread.test.tsx` | Nesting cap, deleted/hidden placeholders, OP badge, pin only own comment |
| `search/__tests__/search-page.test.tsx` | Short-query hint, tabs in URL, highlight rendering, empty state |
| `notifications/__tests__/notification-bell.test.tsx` | Count label, polling paused when hidden, not rendered for visitors |
| `notifications/__tests__/preferences-matrix.test.tsx` | Moderation email locked; quiet hours validation |

## 7. Accessibility checklist

- [ ] PostCard: `<article>` with a heading; the title is the single link; menu is an `IconButton` labelled "More actions for {title}".
- [ ] ReactionBar: toolbar with arrow-key movement; each button `aria-pressed` and labelled "Same Here, 27 reactions"; counts update via a polite live region.
- [ ] ContentWarningGate: button "Show content: burnout, mental health"; hidden content uses `hidden`/`inert`.
- [ ] Composer: labelled fields, counters announced near limits, error summary, findings panel uses `aria-live="polite"`, "Jump to text" moves focus and selects the range.
- [ ] Comments: nested `<ul>` lists; Reply buttons labelled "Reply to @user"; the form returns focus to the new comment.
- [ ] Infinite scroll always has a visible "Load more" button; focus isn't lost when items append.
- [ ] Search highlights use `<mark>` + underline (not color only).
- [ ] Bell count is part of the accessible name; notification rows are links.
- [ ] All colors via tokens; touch targets ≥ 44 px for reactions on mobile.

## 8. Integration notes (mock → real API)

| Mock function | Real endpoint |
|---|---|
| `feedApi.getFeed` | `GET /feed` |
| `feedApi.getLessons` | `GET /lessons` |
| `feedApi.listCategories` / `suggestTags` | `GET /categories` / `GET /tags?q=` |
| `feedApi.hideCategory` | `POST /me/mutes` `{ categoryId }` |
| `feedApi.listFollows` / `follow` / `unfollow` | `GET /me/follows` / `POST /me/follows` / `DELETE /me/follows/:followId` |
| `postsApi.getPost` | `GET /posts/:postId` |
| `postsApi.createPost` / `updatePost` / `deletePost` | `POST /posts` / `PATCH /posts/:postId` / `DELETE /posts/:postId` |
| `postsApi.addCorrection` | `POST /posts/:postId/corrections` |
| `postsApi.react` / `unreact` | `POST` / `DELETE /posts/:postId/reactions` |
| `postsApi.save` / `unsave` | `PUT` / `DELETE /posts/:postId/save` |
| `postsApi.votePoll` | `POST /posts/:postId/poll/votes` |
| `postsApi.uploadImage` / `getMedia` | `POST /media/uploads` → PUT to the presigned URL → `POST /media/:mediaId/complete`; `GET /media/:mediaId` |
| `commentsApi.list` / `listReplies` | `GET /posts/:postId/comments` / `GET /comments/:commentId/replies` |
| `commentsApi.create` / `update` / `remove` | `POST /posts/:postId/comments` / `PATCH` / `DELETE /comments/:commentId` |
| `commentsApi.pin` / `unpin` | `PUT` / `DELETE /posts/:postId/pinned-comment` |
| `searchApi.searchAll` / `search` | `GET /search?q=&type=all` / `GET /search?q=&type=…&cursor=` |
| `notificationsApi.list` / `getUnreadCount` / `markRead` / `markAllRead` | `GET /notifications` / `GET /notifications/unread-count` / `POST /notifications/:id/read` / `POST /notifications/read-all` |
| `notificationsApi.getPreferences` / `updatePreferences` | `GET` / `PATCH /me/notification-preferences` |

## 9. Independence check

This work depends on **no other teammate's work**. Until the owners merge, local placeholders (same paths and props, excluded from git) are used for:
- `@/shared/**` and app scaffold (Arko): copied from contract §2.3, §5–§10;
- `@/components/ui/*` (Tuli): props from contract §8;
- `SafetyCheckPanel`, `ReportDialog`, `BlockMuteMenuItems` from `@/features/safety` (Tuli): placeholders that render a simple list/dialog with the contract props;
- `CompanyCard`, `JobReportCard`, `CommunityCard` (Abnt): placeholders that render the name and a link.

No task waits on anyone.

## 10. Total estimated hours

**100 hours** across 35 tasks (Content foundations 18 h · Feed and landing 15 h · Composer 16 h · Post detail and comments 18 h · Lessons, search, notifications 19 h · Testing and accessibility 14 h).
