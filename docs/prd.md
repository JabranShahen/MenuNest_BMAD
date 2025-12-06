---
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - docs/analysis/product-brief-MenuNest_BMAD-2025-12-06.md
  - docs/index.md
workflowType: 'prd'
lastStep: 4
project_name: 'MenuNest_BMAD'
user_name: 'Jabran'
date: '2025-12-06'
---

# Product Requirements Document - MenuNest_BMAD

**Author:** Jabran  
**Date:** 2025-12-06

## 1) Summary
MenuNest is a brownfield, multi-surface restaurant platform (Flutter mobile apps for admin/serve/kitchen/table, Angular SPA for back-office, .NET 8 API, and static marketing). The PRD clarifies product intent, core flows, and quality bars so we can align the apps and harden the platform.

## 2) Goals and Non-Goals
- Goals:
  - Provide a consistent order lifecycle across channels (table ordering, waitstaff entry, kitchen display, payments, status updates).
  - Centralize branch/account configuration (menus, tables, staff roles, hours) with environment-ready API endpoints.
  - Deliver reliable real-time updates (orders, tickets, table status) via SignalR across all apps.
  - Ensure secure, role-aware access using Firebase Auth with clearer authorization rules.
  - Establish a web back-office experience for branch managers (Angular SPA) that matches the design system.
- Non-Goals (current scope):
  - Third-party delivery/aggregator integrations.
  - Advanced loyalty/CRM, marketing automation.
  - Complex offline order sync beyond basic retry/cache.

## 3) Target Users and Personas
- Owner/Manager: sets up accounts/branches, menus, staff, pricing, hours, oversees performance.
- Branch Manager: day-to-day control of menus/tables/staff at a branch, resolves exceptions.
- Waitstaff (Serve app): seats guests, places/modifies orders, takes payments, monitors status.
- Kitchen Staff (Kitchen app): receives tickets, updates prep status, signals ready.
- Diners (Table app / web ordering): browse menu, place orders, view status, pay.
- Back-office Admin (Web SPA): branch configuration, menu mgmt, staff access, reporting.

## 4) Problem Statement / Business Objectives
MenuNest must unify disparate app behaviors into a reliable, real-time restaurant workflow while reducing configuration friction. Success is measured by faster table turns, fewer order errors, and predictable operations across branches.

## 5) Scope
- In-scope Features:
  - Account/branch setup: branch info, hours, tables, menus/categories/items, pricing/tax.
  - Role-based access via Firebase (owner/manager/staff/kitchen/customer) with guardrails in mobile/web.
  - Order lifecycle: create/modify/cancel orders, course handling, send to kitchen, track tickets and rails, payment capture.
  - Kitchen display: ticket queue, status changes, rails, ready notifications.
  - Real-time updates: SignalR-driven state for orders/tickets/table status; fall back to polling.
  - Web back-office: branch dashboard, menu editor, table layout, staff access control, basic reporting.
  - Multi-environment readiness: configurable API base URLs, CORS hardening, feature flags per env.
- Out-of-scope (for now):
  - Delivery integrations, loyalty/points, marketing campaigns, reservations, advanced analytics.

## 6) Use Cases / User Stories (high level)
- Manager sets up a new branch with tables, hours, and publishes a menu to all channels.
- Waitstaff seats a table, starts an order, adds items/modifiers, sends to kitchen, takes payment.
- Kitchen receives tickets, updates status (ack/prep/ready), and notifies front-of-house in real time.
- Diner scans/opens the menu, places an order, sees live status, and pays from their device.
- Manager updates pricing/menu availability and sees it reflected across apps without redeploys.
- Admin reviews daily sales and kitchen throughput for a branch.

## 7) Functional Requirements (draft)
- FR1: Auth & Roles — Firebase-based auth for all clients; enforce role-aware routes/UI (owner/manager/waitstaff/kitchen/customer).
- FR2: Branch & Table Mgmt — CRUD branches, hours, tables; assign tables to servers; manage table status (open/seated/paid).
- FR3: Menu & Catalog — CRUD menus/categories/items/modifiers; schedule availability; support item notes/options.
- FR4: Ordering — Create/modify/cancel orders; split/merge checks; handle notes; capture tips/discounts.
- FR5: Kitchen Tickets — Generate tickets from orders; queue by rails; update statuses with audit trail.
- FR6: Real-time Sync — SignalR events for orders/tickets/table status; resilient reconnect; graceful polling fallback.
- FR7: Payments — Capture payments and tips; reflect payment status on orders; basic receipt data (amounts, timestamps).
- FR8: Web Back-office — Branch dashboard, menu editor, staff access settings, and simple reports in Angular SPA.
- FR9: Configurability — Environment-based API/SignalR endpoints (dev/stage/prod) and CORS whitelists; feature flags for risky changes.
- FR10: Observability — Basic logging of API calls/events and client error hooks (crashes/network) for triage.

## 8) Non-Functional Requirements (draft)
- Reliability: Target 99.5% uptime for API; SignalR reconnect within 5s with jittered retries.
- Performance: P95 API latency < 500 ms for standard CRUD; initial menu load < 2s on broadband.
- Security: Firebase JWT validation; least-privilege role enforcement; tighten CORS per env; secure secrets in appsettings/env vars.
- Offline/Resilience: Client retries with backoff; queue writes when offline for short interruptions; clear user messaging on failures.
- Compatibility: Flutter apps on current stable; Angular 20; API .NET 8; support recent mobile OS versions per store baselines.
- Observability: Centralized API logs; client error logging; minimal analytics events for flows (auth, order create, payment).

## 9) User Flows (representative)
- Table/Diner flow: open menu → browse → add items/modifiers → submit order → view status → pay → receipt confirmation.
- Waitstaff flow: select table → start order → add items → send to kitchen → monitor status → collect payment → close table.
- Kitchen flow: receive tickets in queue/rail → acknowledge → set preparing → set ready → notify FOH/diners.
- Manager flow: create branch → define hours/tables → publish menu → invite staff → monitor sales/tickets.

## 10) Data & Integrations
- Identity: Firebase Auth across clients; backend validates Firebase project ID (JWKS).
- API: REST over HTTPS (/api) with environment-specific base URLs; Swagger/OpenAPI to be finalized.
- Real-time: SignalR hub `/EntitySignalRHub`; join per account/branch; keep-alive/heartbeat required.
- Storage: Azure Blob (usage TBD); ensure access per env.
- Shared models: Entities in `shared_package/lib/entities` align with API DTOs (orders, tickets, payments, menus, branches, tables, users).

## 11) Release & Milestones (proposed)
- R1: Platform hardening — env configs, CORS tightening, menu/branch CRUD stable, basic order→kitchen→payment flow, SignalR wiring MVP, observability baseline.
- R2: Experience polish — responsive web back-office per style/interaction guide; accessibility uplift; reliable waiter/table/kitchen UX; payment/tip flows tightened.
- R3: Growth/ops — reporting dashboards, performance tuning, feature flags, and rollout safety for menus/pricing changes.

## 12) Success Metrics (initial)
- Operational: Order-to-kitchen delivery time P95 < 2s; ticket-to-ready median < kitchen target; failure rate of order submissions < 0.5%.
- Engagement: % of orders placed via table app vs waiter; repeat table-app usage per branch.
- Reliability: SignalR reconnect success > 99%; API 5xx rate < 0.5%.
- Quality: Crash-free sessions > 99.5% across mobile apps; accessibility checks pass for web.

## 13) Risks & Mitigations
- Risk: Hardcoded API base URL leads to env drift. Mitigation: env config + feature flags before release.
- Risk: Incomplete SignalR client wiring causes stale state. Mitigation: implement reconnect/backoff + polling fallback.
- Risk: Role enforcement gaps expose admin actions. Mitigation: explicit role matrix and guarded routes/components.
- Risk: CORS currently wide open. Mitigation: locked per-env allowlists.
- Risk: Payment accuracy and receipt integrity. Mitigation: totals validation and audit fields in orders/payments.

## 14) Open Questions
- Which payment providers are in scope (if any) beyond recording payments? Card present vs external POS?
- Are reservations or waitlist flows needed, or only walk-in/table flows?
- Do we need split/merge checks in R1, or can it wait for R2?
- Required reporting KPIs for managers (sales, voids, prep times)?
- Offline requirements for mobile (depth of caching, duration tolerated)?
