# Epics and Stories - MenuNest

## Epic: Web Back-Office UX Alignment (Branch Management First) — Status: Done
**Goal:** Align the Angular branch management experience to the shared style/interaction guides for faster, more reliable branch admin flows.  
**Scope:** `MenuNestWebsite/src/app/branch-management/*` plus shared list/table styles if reused.  
**Success Measures:** Stable layouts (no button drift), clearer feedback, accessible interactions, consistent theming.

### Story: Branch Management UX Alignment - Status: Done
- **Source:** `MenuNest_BMAD/docs/website/ux/branch-ux-story.md` (UX-authored quick wins and acceptance criteria).
- **Outcome:** Branch list/add/edit/delete follow the style guide (palette tokens, spacing, typography) and interaction guide (sticky actions, empty/loading states, feedback, accessibility, responsive behavior).
- **Acceptance Criteria:** Use the criteria already defined in `MenuNest_BMAD/docs/website/ux/branch-ux-story.md` (CTA sticky, empty/loading states present, snack bars on CRUD, inline validation, danger confirms, hover/selected-row styles, responsive cards, aria labels/focus, modals trap focus).
- **Notes:** Hardcoded colors replaced with CSS variables from `src/styles/style-guide.md`; interaction behaviors follow `src/styles/interaction-guide.md`. Table is contained with scroll + sticky header; selection uses light info highlight.

### Story: Branch Management UX Remediation - Status: Done
- **Source:** `MenuNestWebsite/src/styles/branch-ux-remediation.md` (remediation story and audit matrix).
- **Outcome:** Branch list/add/edit fully aligns to style and interaction guides; misalignments fixed and tracked in the audit matrix.
- **Acceptance Criteria:** See remediation story; covers sticky CTA, empty/loading states, snack bars on CRUD, inline validation/required markers, danger confirms, hover/selected styles, responsive cards, aria/focus, tokenized theming, sticky headers with contained scroll, and completed audit matrix.
- **Notes:** UI-only scope unless delete semantics require backend support; ensure CSS tokens live in `styles.scss`.

## Epic: Menu Management UX Alignment - Status: In Progress
**Goal:** Bring Category Management UX in line with the Style and Interaction Guides, keeping parity with Branch Management patterns for clarity, safety, and responsiveness.  
**Scope:** `MenuNestWebsite/src/app/category-management/*` plus any shared category list/chip components.  
**Success Measures:** Clear create/edit/disable/delete flows with predictable feedback, accessible interactions, and responsive layouts that reuse shared tokens/styles.

### Story: Category Management UX Alignment - Status: Done
- **Source:** `MenuNest_BMAD/docs/website/ux/category-ux-story.md` (UX story and acceptance criteria).
- **Outcome:** Categories list/add/edit/disable/delete follow the same sticky header CTA, empty/loading states, snackbars, inline validation, and theming tokens used in Branch Management; delete is blocked when linked items exist and users are guided to disable/reassign instead; status toggle provides the safe alternative to delete.
- **Acceptance Criteria:** See story file; includes sticky CTA, empty/loading states, snack bars on CRUD, inline required + uniqueness validation, delete blocked with linked items and guided messaging, danger confirm on allowed deletes, hover/selected row styles, responsive cards, aria/focus, tokenized theming, Name-sort (no drag-and-drop reorder), and backend behavior to expose `itemsCount` plus a blocked delete status (e.g., 409) when items exist.
- **Notes:** Role gating: owner/manager for add/edit/disable/delete; no reorder for now. Backend needs items count exposure and delete guardrails; owner: Amelia (Dev).

## Epic: Shared UI Kit Alignment - Status: In Progress
**Goal:** Standardize shared UI primitives (header/panel/table/inputs/buttons/labels) across Branch and Category screens to reduce duplication and ensure visual parity.
**Scope:** `MenuNestWebsite/src/app/shared/ui-*` (new shared components), refactors in Branch and Category screens to consume shared kit.
**Success Measures:** Branch and Category screens use shared header/panel, table shell, buttons, inputs, status pills, badges, empty/skeleton, and action icon styles; no duplicate table/button/input styling in feature SCSS.

### Story: Extract Shared UI Components - Status: Done
- **Source:** `MenuNest_BMAD/docs/website/ux/ui-kit-story.md`.
- **Outcome:** Create shared UI kit components (buttons: primary/secondary/ghost/danger/icon; inputs: text/textarea with label/helper/error; table shell with actions column; status pills and count badges; toast helper; empty-state and skeleton blocks; modal shell; panel/card). Refactor Branch and Category screens to consume the shared kit and drop local duplicates.
- **Acceptance Criteria:** Branch and Category screens render with shared buttons/inputs/table shell/empty/skeleton/status pills/badges/icon buttons; feature SCSS no longer defines duplicate table/button/input styling; visual parity matches Branch spec.
- **Notes:** Keep existing tokens in `variables.scss`; ensure standalone components are imported (not declared) and CommonModule is included where needed. Owner: Amelia (Dev).

### Story: Menus Screen UI Alignment - Status: Done
- **Source:** `MenuNest_BMAD/docs/website/ux/menu-ux-story.md`.
- **Outcome:** Menus screen adopts shared UI kit (header/panel, buttons, empty-state, table/list shell) and matches Branch/Categories visual spec; background/layout normalized (no mid-gray panels), consistent spacing/typography.
- **Acceptance Criteria:** Menus page uses shared header/panel; panels/cards on white with border/shadow; buttons use `app-ui-button`; empty states use shared component; lists/tables have visible dividers; no inline ad-hoc styling remains. Visual parity with Branch/Categories for padding, hover, and action column.
- **Notes:** Keep tokens from `variables.scss`; ensure standalone components are imported properly. Owner: Amelia (Dev).

### Story: Menus Screen UX Remediation - Status: Done
- **Source:** `MenuNest_BMAD/docs/website/ux/menu-ux-remediation-story.md`.
- **Outcome:** Fix current Menus layout to match Branch/Categories patterns: white cards with borders/shadow, balanced gutters, shared table shell with dividers/hover/selected states, right-sized toggles/icons, and shared empty/loading states.
- **Acceptance Criteria:** Use shared header/panel/table/buttons/icon buttons/toggles and tokens; remove oversized toggles/plus icons; apply consistent padding/typography/colors; show shared empty-state and loading/skeleton where applicable; responsive layout without overflow; accessible focus/aria and WCAG AA contrast.
- **Notes:** Completed.

### Story: Menus Screen Redesign (Fresh Components) - Status: In Progress
- **Source:** `MenuNest_BMAD/docs/website/ux/menu-ux-redesign-story.md`.
- **Outcome:** Rebuild the Menus page with a fresh component set (MenuHeader, MenuList, CategoryList, ItemList) using shared tokens and clean layout. Desktop shows three columns; 1440/1080 wraps to two; mobile stacks. Menus table shows Name + Actions only; Categories table shows Name/Items/Actions; items render as compact nested lists per category.
- **Acceptance Criteria:** New components live in a dedicated folder; legacy menu styles/components untouched. Consistent gutters, row heights, paddings, and typography matching Branch/Categories; shared icon buttons/toggles sized uniformly; shared empty/skeleton states for branches/menus/categories/items; hover/selected states on rows; responsive wrapping at 1920/1440/1366/mobile without overflow; WCAG AA focus/contrast.
- **Notes:** Build from scratch rather than patching legacy; reuse shared tokens and UI primitives where sensible.

### Story: Menu Management Layout Frame & Scroll Containment - Status: In Progress
- **Source:** `MenuNest_BMAD/docs/website/ux/menu-ux-frame-scroll-story.md`.
- **Outcome:** Frame the Menu Management screen in a global container and give each column panel its own scrollable body (with sticky panel headers) so long lists don’t create endless page scroll.
- **Acceptance Criteria:** See story file; includes global framing, per-panel scroll behavior, sticky headers, CTA placement, responsive wrap, and accessibility requirements.
- **Notes:** Can be implemented as a quick remediation now or folded into the Menus Screen Redesign work.
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

