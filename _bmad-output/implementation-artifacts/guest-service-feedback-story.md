# Story: Guest Service Feedback Capture and Account-Level Feedback Management

**Status:** ready-for-dev

## Story

As a dine-in guest using the MenuNest table app,
I want to quickly rate the service and optionally leave a short comment when my session ends,
so that I can share how the experience went without interrupting ordering.

As a restaurant account holder using the MenuNest dashboard,
I want to review guest feedback in a manageable, filterable screen,
so that I can identify service issues by branch, table, and time period and follow them through review.

## Problem Statement

MenuNest currently has no first-party feedback loop between the guest table experience and the operator dashboard.

That creates three gaps:

1. **No guest voice at the end of service**. The table app can complete a session, but there is no structured opportunity for a guest to rate the service while the experience is still fresh.
2. **No account-level visibility into service quality**. The dashboard exposes operational data such as orders, tables, menus, and staff, but it has no surface for service sentiment, complaint themes, or low-rating review.
3. **No actionable workflow after feedback is submitted**. Even if feedback were collected ad hoc, the account holder would need branch context, table context, timestamps, and a triage status to manage it meaningfully.

The feature should therefore create a single end-to-end path:

- collect feedback from the guest at the end of a completed table session,
- store it with account, branch, table, session, and optional order context,
- expose it in the dashboard as a dedicated, manageable feedback screen for the account holder.

## Product Decision Summary

This story locks the following v1 decisions:

- Feedback is **session-level**, not per-item and not public-review style.
- The **primary UI trigger** is the `SessionCompleteScreen` when the session ends with reason `completed`.
- Feedback is **optional** and **non-blocking**. Guests can skip it and continue to table release/start-new-session.
- A guest may submit **at most one feedback entry per guest session**.
- The dashboard gets a dedicated **Feedback** management page with summary cards, filters, a review table, and review-status updates.
- Feedback is **internal-only** for restaurant operations. It is not shown publicly and is not syndicated outside the account dashboard.

## Acceptance Criteria

1. When a table session reaches `SessionCompleteScreen` with `SessionEndReason.completed`, the guest sees a feedback card before auto-release containing:
   - overall experience rating (1-5)
   - service rating (1-5)
   - optional issue tags
   - optional free-text comment
   - `Submit feedback` and `Skip` actions
2. When the session ends with `SessionEndReason.cancelled`, no feedback prompt is shown.
3. Submitting feedback does not block the existing table release flow; after submission or skip, the current session-complete countdown and `Start New Session` behavior still work.
4. A guest cannot submit more than one feedback entry for the same `GuestTableSession`.
5. Guest-submitted feedback is stored with account, branch, table, guest session, and created timestamp. If an active/completed order is available, its `OrderId` is also stored.
6. Guest submission is server-trusted: `AccountId`, `BranchId`, `TableId`, and `GuestTableSessionId` are derived from authenticated guest/session context on the backend rather than accepted blindly from the client body.
7. The backend exposes an authenticated dashboard read endpoint that returns feedback for the current account with optional filters for branch, status, rating band, tag, and date range.
8. The dashboard has a dedicated route and navigation entry for feedback management, and the page is reachable from the main dashboard shell.
9. The feedback management page shows feedback in a manageable way:
   - summary metrics at the top
   - branch filter
   - status filter
   - rating filter
   - date range filter
   - sortable/filterable list of individual feedback entries
   - review detail view for full comment and metadata
10. Each feedback row clearly shows at minimum:
   - submission date/time
   - branch
   - table
   - overall rating
   - service rating
   - review status
   - comment preview
11. The account holder can mark feedback status as `New`, `Reviewed`, or `Resolved` from the dashboard without editing the original guest ratings or comment.
12. Low-rated feedback is visually easy to spot in the management page through rating display and status/risk styling.
13. The feedback page only returns feedback belonging to the authenticated user’s account context; guest users cannot access account feedback management endpoints.
14. Automated tests cover guest submission flow, duplicate-per-session protection, backend account scoping, dashboard filters, and feedback status updates.

## Non-Goals

- Public reviews, testimonials, or external review-platform integration.
- In-session interruption prompts while the guest is still browsing or ordering.
- Requiring the guest to identify themselves personally beyond the existing guest session.
- Rich moderation tooling, staff assignment workflows, or notification pipelines.
- Editing guest-authored feedback after submission.
- Analytics beyond the lightweight summary cards required for manageable review.

## Tasks / Subtasks

- [ ] Add backend feedback entity and persistence wiring (AC: 5, 6, 7, 11, 13)
  - [ ] Create a new backend entity, recommended name: `CustomerFeedback` or `GuestFeedback`, under `MenuNest.Abstractions/Entities`.
  - [ ] Include at minimum: `AccountId`, `BranchId`, `TableId`, `GuestTableSessionId`, `OrderId?`, `OverallRating`, `ServiceRating`, `IssueTags`, `Comment`, `Status`, `CreatedAt`, `ReviewedAt?`, `ReviewedByUserId?`.
  - [ ] Register the entity through the existing generic entity-manager/persistence pattern so it can be inserted, queried, and updated consistently with the rest of the solution.

- [ ] Add backend feedback API surface (AC: 4, 5, 6, 7, 11, 13)
  - [ ] Create `FeedbackController` under `MenuNestAPI/Controllers`.
  - [ ] Add `POST /api/feedback` for guest submission.
  - [ ] Add `GET /api/feedback/account/{accountId}` for dashboard reads with query filters.
  - [ ] Add `GET /api/feedback/{id}` for detail reads.
  - [ ] Add `PUT /api/feedback/{id}/status` for review-status updates.
  - [ ] Enforce one feedback per `GuestTableSessionId`.
  - [ ] Enforce account scoping and guest/dashboard authorization rules.

- [ ] Add shared/mobile feedback model and service (AC: 1, 3, 4)
  - [ ] Add a Dart entity/model for feedback in `shared_package/lib/entities`.
  - [ ] Add a shared-package service for `submitFeedback(...)`.
  - [ ] Reuse the existing authenticated API client pattern; do not add a new networking stack.

- [ ] Add guest feedback UI to session completion flow (AC: 1, 2, 3)
  - [ ] Extend `SessionCompleteScreen` so the completed state renders a compact feedback card above the countdown/release CTA.
  - [ ] Keep the cancelled state unchanged.
  - [ ] Rating UX should be quick-tap and mobile-friendly.
  - [ ] Comment remains optional and constrained to a reasonable max length.
  - [ ] Skip must be explicit and easy.

- [ ] Add duplicate-submission guard in the guest flow (AC: 4)
  - [ ] Disable repeated submit taps while the request is in flight.
  - [ ] If the backend responds that feedback already exists for the session, show a non-blocking confirmation/error state and do not create another entry.

- [ ] Add dashboard feedback management page (AC: 8, 9, 10, 11, 12)
  - [ ] Create a new dashboard route: `/dashboard/feedback`.
  - [ ] Add a `Feedback` nav item to the dashboard shell.
  - [ ] Create a dedicated component using existing management-panel and shared UI patterns.
  - [ ] Add top summary cards: total feedback count, average overall rating, low-rating count, unresolved count.
  - [ ] Add filter controls for branch, status, rating, and date range.
  - [ ] Add a table/list with comment previews and status display.
  - [ ] Add a detail panel/modal/drawer showing full feedback content and metadata.
  - [ ] Add status update controls for `New`, `Reviewed`, `Resolved`.

- [ ] Add website feedback entity and service layer (AC: 7, 8, 9, 11)
  - [ ] Add a TypeScript entity/interface for feedback.
  - [ ] Add a website service to query feedback and update review status.
  - [ ] Follow the existing cache + invalidation pattern where reasonable.

- [ ] Add tests (AC: 4, 7, 11, 13, 14)
  - [ ] Backend controller tests for guest submission, duplicate rejection, and account scoping.
  - [ ] Flutter widget/service tests for completed-session feedback prompt behavior and submission/skip handling.
  - [ ] Angular component/service tests for route visibility, filters, list rendering, and review-status mutation.

## Implementation Notes

### Guest UI Placement

The correct v1 placement is the existing `SessionCompleteScreen` in the table app.

Why:

- It appears after the guest has experienced the service.
- It does not interrupt menu browsing, order review, or checkout.
- The screen already has post-session attention and a 60-second countdown window, which is ideal for a short optional action.

The feedback card should sit between the gratitude copy and the countdown/release controls, not in a blocking modal. Keep it embedded in the screen so the guest can skip without friction.

### Recommended Guest Form Structure

Use a two-step visual hierarchy within one card:

1. `Overall experience` 1-5 stars
2. `Service` 1-5 stars
3. Optional issue tags:
   - `Slow service`
   - `Order issue`
   - `Staff friendliness`
   - `Cleanliness`
   - `Payment issue`
4. Optional comment textarea with concise helper text

Do not ask for name, phone number, or email in v1.

### Review Status Model

Use a lightweight operational workflow:

- `New`: freshly submitted, not yet reviewed
- `Reviewed`: seen and acknowledged by account side
- `Resolved`: action taken or no further follow-up needed

The guest-authored fields are immutable after submission. Dashboard users only change the review status and review metadata.

### Authorization Rules

For `POST /api/feedback`:

- Allowed for guest sessions authenticated through the existing table guest token flow.
- Backend must derive `AccountId`, `BranchId`, and `TableId` from trusted guest/session context.
- Reject submission if the session is not active or feedback already exists for that session.

For dashboard read/update endpoints:

- Must require authenticated non-guest users.
- Must only return/update feedback belonging to the authenticated account context.
- If role-based restriction is implemented in this story, prefer `Owner`, `Admin`, and `Manager`. If the repo’s current auth plumbing does not cleanly enforce role restrictions yet, account scoping remains the minimum required protection.

### Manageable Dashboard UX

The page should behave like an operations inbox, not a raw dump.

Recommended structure:

1. Summary strip at top:
   - `Total Feedback`
   - `Average Rating`
   - `1-2 Star Feedback`
   - `Unresolved Feedback`
2. Filter row:
   - branch selector
   - status selector
   - rating selector
   - date range
3. Main list/table:
   - newest first by default
   - rows with branch/table/date/ratings/status/comment preview
4. Detail panel:
   - full comment
   - issue tags
   - linked metadata: branch/table/session/order
   - status change actions

This is the minimum shape required to make the feature genuinely manageable for an account holder.

### Route Placement

Add a first-class dashboard route rather than hiding feedback under `Overview` or `Account`.

Recommended route:

- `/dashboard/feedback`

Recommended nav item label:

- `Feedback`

This aligns with the existing dashboard shell, where each major operational area already has its own route and navigation entry.

### API Shape Recommendation

Recommended request model for guest submission:

```json
{
  "overallRating": 5,
  "serviceRating": 4,
  "issueTags": ["Slow service"],
  "comment": "Food was great, but drinks took a while."
}
```

Recommended read query shape:

- `GET /api/feedback/account/{accountId}?branchId=...&status=New&minRating=1&maxRating=2&from=2026-04-01&to=2026-04-30`

Recommended status update model:

```json
{
  "status": "Reviewed"
}
```

Keep the guest submission body intentionally narrow. Do not let the client author authoritative account, branch, or session linkage.

## File Targets

Primary backend targets:

- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/FeedbackController.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Models/` if dedicated DTOs are added
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/`
- dependency wiring in adjacent bootstrapper/manager projects if required by the entity-manager pattern

Primary table app / shared package targets:

- `MenuNestApp/menunest/shared_package/lib/entities/`
- `MenuNestApp/menunest/shared_package/lib/services/`
- `MenuNestApp/menunest/table_app/lib/screens/session_complete_screen.dart`
- `MenuNestApp/menunest/table_app/test/`
- `MenuNestApp/menunest/shared_package/test/`

Primary dashboard targets:

- `MenuNestWebsite/src/app/app-routing.module.ts`
- `MenuNestWebsite/src/app/dashboard/dashboard.component.ts`
- `MenuNestWebsite/src/app/services/` for feedback service
- `MenuNestWebsite/src/app/entities/` for feedback interface/model
- new `MenuNestWebsite/src/app/feedback-management/` feature folder
- relevant `*.spec.ts` files for route, service, and component coverage

## Technical Requirements

- Preserve the current table-app happy path: feedback must not block table release or session reset.
- Use existing auth and API plumbing on both Flutter and Angular sides. Do not introduce parallel auth flows.
- The backend must reject duplicate feedback for the same `GuestTableSessionId`.
- Ratings are required integers in the 1-5 range.
- Comment is optional and should be length-limited server-side and client-side.
- Issue tags must come from a bounded allow-list in v1.
- The dashboard list defaults to newest-first ordering.
- Low ratings must be visually distinguishable in the dashboard UI.
- Dashboard filters must operate without mutating the original feedback records.
- Use existing shared UI primitives in the website app where possible: management panels, count/status patterns, table wrapper, empty states, skeletons, and buttons.
- Do not alter existing order/session completion semantics beyond rendering and submitting feedback.

## Testing Requirements

- Backend:
  - guest can submit feedback once for a valid session
  - duplicate submission for same session is rejected
  - guest cannot spoof another table/account/session
  - account dashboard read endpoint only returns same-account feedback
  - status update changes only review fields, not guest-authored ratings/comment
- Flutter:
  - completed session shows feedback card
  - cancelled session does not show feedback card
  - skip preserves existing release flow
  - submit disables repeat taps and shows success/failure state correctly
- Angular:
  - feedback route appears in shell navigation
  - feedback page loads account-scoped data
  - filters update the rendered result set
  - status updates refresh row/detail state correctly
  - low-rated rows render the intended warning/critical treatment

## Risks / Edge Cases

- **Countdown collision**: the existing 60-second auto-release timer may expire while the guest is typing a comment. The implementation must decide whether feedback interaction pauses the timer or whether submission must fit inside the same countdown window. Recommendation: pause countdown while a submit request is in flight, but not while the card is merely visible.
- **Duplicate taps / bad connectivity**: mobile retry behavior can accidentally double-submit. Backend uniqueness by `GuestTableSessionId` is mandatory.
- **Sparse comments**: many guests will provide ratings without comments. The dashboard must still make unrated-comment rows useful via branch/table/date/status context.
- **Cancelled sessions**: do not ask for feedback when the session was cancelled by staff or system flow.
- **Role ambiguity**: current dashboard routing appears account-context driven more than role-guard driven. If strict role restriction cannot be added cleanly in this story, document that limitation and keep account-scoped backend enforcement intact.
- **Large feedback volume later**: if adoption grows, the page may eventually need pagination. V1 can ship without full pagination if the filtered list remains performant, but sorting and filtering must be cleanly structured.

## Definition of Done

- Guests can submit optional service feedback from the completed session screen.
- No feedback prompt appears for cancelled sessions.
- Only one feedback entry can exist per guest session.
- Feedback records are persisted with account/branch/table/session linkage and optional order linkage.
- Account-side users have a dedicated dashboard feedback page with summaries, filters, list view, detail view, and review-status actions.
- Feedback management is account-scoped and not exposed to guest users.
- Automated tests pass across backend, Flutter, and Angular coverage added by this story.

## Dev Notes

### Relevant Existing Code Patterns

**Guest session completion surface**
- `MenuNestApp/menunest/table_app/lib/screens/session_complete_screen.dart`
- This is already the end-of-session confirmation/release surface and is the primary insertion point for guest feedback UI.

**Guest ordering flow**
- `MenuNestApp/menunest/table_app/lib/screens/table_ordering/table_ordering_screen.dart`
- This screen already routes to `SessionCompleteScreen` based on order/session status changes.

**Guest session identity**
- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/GuestTableSession.cs`
- The backend already tracks guest session identity, account, branch, table, and expiry.

**Dashboard route shell**
- `MenuNestWebsite/src/app/app-routing.module.ts`
- `MenuNestWebsite/src/app/dashboard/dashboard.component.ts`
- The dashboard already uses first-class routes and left-nav items per operational area, so feedback should follow the same pattern.

**Manageable table/list pattern**
- `MenuNestWebsite/src/app/table-management/table-management.component.html`
- `MenuNestWebsite/src/app/shared/ui/ui-table/ui-table.component.ts`
- Existing management screens already combine panels, filters/scoping, and tabular rows in a way that should be reused.

**Dashboard service/cache pattern**
- `MenuNestWebsite/src/app/services/dashboard.service.ts`
- `MenuNestWebsite/src/app/services/table-service.service.ts`
- New feedback services should follow the same request-cache and invalidation style where appropriate.

**Order/account context**
- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/Order.cs`
- Existing order linkage makes optional `OrderId` capture straightforward when available at session end.

### Suggested Source References

- `docs/architecture-mobile-suite.md`
- `docs/architecture-website.md`
- `docs/architecture-backend.md`
- `docs/api-contracts-backend.md`
- `docs/integration-architecture.md`

## Open Questions

- Should the session countdown pause while the guest is actively typing, or only during submit? Recommendation for v1: pause only during submit to avoid expanding scope.
- Should managers as well as account owners/admins be allowed to review feedback? Recommendation for v1: permit `Owner`, `Admin`, and `Manager` if role enforcement is straightforward; otherwise rely on account scoping first.

## Dev Agent Record

### Agent Model Used

gpt-5

### Completion Notes

- Standalone story created because no sprint-status or planning story inventory was present under `_bmad-output/planning-artifacts`.
- Story grounded in the existing table session completion flow, dashboard route shell, and current backend guest/account context patterns.

### File List

- `_bmad-output/implementation-artifacts/guest-service-feedback-story.md`
