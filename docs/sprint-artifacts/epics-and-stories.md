# Epics and Stories - MenuNest

## Epic: Web Back-Office UX Alignment (Branch Management First) — Status: Done
**Goal:** Align the Angular branch management experience to the shared style/interaction guides for faster, more reliable branch admin flows.  
**Scope:** `MenuNestWebsite/src/app/branch-management/*` plus shared list/table styles if reused.  
**Success Measures:** Stable layouts (no button drift), clearer feedback, accessible interactions, consistent theming.

### Story: Branch Management UX Alignment — Status: Done
- **Source:** `MenuNestWebsite/src/styles/branch-ux-story.md` (UX-authored quick wins and acceptance criteria).
- **Outcome:** Branch list/add/edit/delete follow the style guide (palette tokens, spacing, typography) and interaction guide (sticky actions, empty/loading states, feedback, accessibility, responsive behavior).
- **Acceptance Criteria:** Use the criteria already defined in `src/styles/branch-ux-story.md` (CTA sticky, empty/loading states present, snack bars on CRUD, inline validation, danger confirms, hover/selected-row styles, responsive cards, aria labels/focus, modals trap focus).
- **Notes:** Hardcoded colors replaced with CSS variables from `src/styles/style-guide.md`; interaction behaviors follow `src/styles/interaction-guide.md`. Table is contained with scroll + sticky header; selection uses light info highlight.

## Epic: Environment & Config Hardening
**Goal:** Remove hardcoded endpoints and unsafe defaults so environments can be deployed safely.  
**Scope:** API appsettings, Flutter shared_package + apps, Angular env files, CORS.

### Stories
- Parameterize API base/SignalR URLs in `shared_package` and Flutter apps (flavors/env injection) instead of hardcoded Azure URL; document per-env configs.
- Add Angular `environment.*.ts` values for API/SignalR endpoints and enable strict + ESLint if not already.
- Tighten CORS per env in API; add config sanity checks/CI guard for allowlists.
- Add feature-flag scaffolding for risky changes (SignalR-only mode, menu publish, payment capture).

## Epic: Real-time Reliability (SignalR-first)
**Goal:** Deliver live order/ticket/table updates with graceful degradation.  
**Scope:** SignalR hub, shared_package client, Flutter and Angular consumers.

### Stories
- Implement SignalR client in `shared_package` with reconnect/backoff, keepalive, and group join (account/branch).
- Wire events for orders/tickets/table status into Serve/Kitchen/Table apps; add polling fallback when disconnected.
- Add minimal client telemetry/logging for connect/disconnect/errors to aid triage.

## Epic: Core Order → Kitchen → Payment Flow
**Goal:** Ensure the main operational flow is reliable and consistent across surfaces.  
**Scope:** API contracts, Flutter apps, Angular back-office where applicable.

### Stories
- Validate order/ticket/payment DTOs vs API (Swagger) and align shared_package entities; add idempotency for order/payment creation.
- Add server-side validation for totals (amounts/tips/discounts) and status transitions; surface meaningful errors to clients.
- Define and exercise a happy-path and error-path test for create order → send to kitchen → mark ready → capture payment.

## Epic: Role-based Access & AuthZ
**Goal:** Enforce least-privilege access across all clients and API.  
**Scope:** Firebase custom claims/roles, ASP.NET policies, Angular/Flutter guards.

### Stories
- Map roles (owner/manager/waitstaff/kitchen/customer) to claims and API policies; gate endpoints accordingly.
- Add Angular route guards + UI gating for roles; verify branch management is manager/owner only.
- Add Flutter-side guards/navigation for admin/serve/kitchen/table apps per role expectations.

## Epic: Observability & Quality Baseline
**Goal:** Improve debuggability and basic reliability before scale.  
**Scope:** API logging/metrics, client error hooks, health/smoke tests.

### Stories
- Add structured logging with correlation IDs in API; log auth failures, SignalR connect/disconnect, and business events (order/ticket/payment).
- Add minimal client error reporting hooks (Flutter/Angular) for crashes/network failures (no PII); surface user-facing error toasts.
- Add liveness/readiness endpoints and a CI smoke test (API up, CORS check, basic GET/health).
