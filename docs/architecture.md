---
stepsCompleted: [1, 2, 3, 4]
lastStep: 4
workflowType: 'architecture'
project_name: 'MenuNest_BMAD'
user_name: 'Jabran'
date: '2025-12-06T01:54:56Z'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
- Multi-app workflow: admin (staff/config), kitchen (KDS), serve (waitstaff), table (customer ordering).
- Shared domain: orders, tickets, menu, payments, tables/branches/accounts.
- Web SPA + static landing: marketing and/or customer entry point.
- API/services: REST over HTTPS with Firebase bearer auth; SignalR hub for real-time updates.

**Non-Functional Requirements:**
- Authentication: Firebase ID tokens end-to-end.
- Real-time: SignalR hub available; client wiring TBD.
- Environment: API base URL currently hardcoded; needs env config.
- (Awaiting explicit targets for perf/latency, uptime, scale, security/compliance.)

**Scale & Complexity:**
- Primary domain: restaurant operations across multiple roles and apps.
- Complexity level: medium-high given multi-channel + real-time.
- Estimated architectural components: multi-tenant API, authz, real-time notification layer, offline/cache strategy, storage, web/mobile frontends.

### Technical Constraints & Dependencies
- Auth via Firebase; backend validates JWT against Firebase project ID.
- Real-time via SignalR (`/EntitySignalRHub`); client side not yet implemented.
- API base URL hardcoded in shared_package (`https://menunestapi.azurewebsites.net/api`) — must be parameterized.
- CORS currently wide-open in backend (tighten for prod).
- Azure Blob referenced; usage not detailed.

### Cross-Cutting Concerns Identified
- AuthZ/roles across admin/serve/kitchen/table/web.
- Configurable API endpoints per environment.
- Real-time updates for orders/tickets/status.
- Data consistency for orders/tickets/payments across apps.
- Observability (logging/metrics) and error handling not defined.
- Offline/resilience for mobile apps (pending requirements).

## Starter Template Evaluation

### Primary Technology Domain
Multi-part (Flutter mobile, ASP.NET Core API, Angular SPA, static landing).

### Starter Options Considered
- Flutter: continue with existing workspace; add a common module template (shared_package) and per-app template conventions (folder structure, providers, env config).
- ASP.NET Core: keep current API; add standardized solution template (API + tests + infra configs) to enforce DI, auth, SignalR, CORS/env profiles.
- Angular: keep current Angular 20 app; add a starter profile with strict TS, lint, and env separation.

### Selected Starter: "Standardize Existing Stack"
**Rationale:**
- Brownfield; introducing a new framework adds risk.
- Existing stack already maps to required surfaces (mobile, API, web).
- Focus should be on hardening: env configuration, CORS tightening, SignalR client wiring, logging/observability, and tests.

**Initialization/Hardening Commands:**
- Flutter: `flutter create --template app` (for new modules), reuse shared_package; add env config pattern (flavors) and CI format/analyze.
- .NET API: `dotnet new webapi` (for new services/tests) with Autofac/SignalR/JWT wiring copied from current API; add `dotnet new xunit` for tests.
- Angular: `ng new` (for new micro-frontends if needed); in current app add stricter configs (strict TS, ESLint, env files).

## Target Architecture Summary (aligned to PRD)
- Multi-surface clients (Flutter mobile apps for admin/serve/kitchen/table; Angular SPA back-office) consume a single ASP.NET Core API and SignalR hub.
- Firebase Auth is the identity provider; JWT validation happens in API; clients enforce role-aware navigation/guards.
- Shared domain models live in `shared_package`; API DTOs align to these models and Swagger/OpenAPI.
- Real-time events (orders, tickets, table status) flow over SignalR with reconnect/backoff; polling fallback for resilience.
- Environment-specific configuration (API base, SignalR URL, CORS allowlists, feature flags) is externalized per env.
- Observability baseline: API structured logging + minimal metrics; client error logging hooks; later add tracing.

## Application Topology & Responsibilities
- Flutter apps: role-specific UX; use shared_package for HTTP, entities, and SignalR client wiring; local cache for short offline gaps.
- Angular SPA: branch/back-office CRUD (branches, menus, tables, staff, reports); AngularFire for auth; Material design per style guide.
- API (.NET 8): REST endpoints for branches, menus, orders, tickets, payments, users; SignalR hub `/EntitySignalRHub`; Autofac DI; Azure Blob (TBD usage).
- Landing: static marketing pages; minimal coupling except links/deep links.

## Architecture Decisions
1) **Stay on existing stack, harden it**: No new frameworks; focus on env config, authz, real-time, and tests (per PRD goals R1).  
2) **Environment-driven configuration**: API base/SignalR URLs, CORS allowlists, feature flags come from env files/appsettings; no hardcoded URLs in clients.  
3) **Role-based access model**: Roles (owner/manager/waitstaff/kitchen/customer) enforced both server-side (policies/claims) and client-side (guards/navigation).  
4) **Real-time first, poll fallback**: SignalR events are primary; clients implement reconnect/backoff + offline queue where safe; HTTP polling as backup for orders/tickets/tables.  
5) **Consistency via shared contracts**: Maintain OpenAPI/Swagger as source of truth; align shared_package entities with API DTOs; add contract tests.  
6) **Security posture**: Tighten CORS per env; validate Firebase JWT; least-privilege policies; secure secrets via environment or KeyVault-like store.  
7) **Observability baseline**: Structured logs (correlation IDs), error metrics, and minimal client error capture to unblock triage before scaling.

## Integration & API Strategy
- REST endpoints remain primary; finalize OpenAPI and generate client stubs if needed.  
- SignalR groups: join per-account and per-branch; emit events for order/ticket/table status changes.  
- Versioning: add URL or header-based versioning to API once contracts stabilize.  
- Idempotency: apply for payment/order creation endpoints to prevent duplicates.  
- CORS: locked to known origins per environment; allow credentials only where required.

## Auth & Security
- Identity: Firebase Auth (ID tokens). Backend validates against ProjectId; cache JWKS with TTL.  
- Roles/claims: map Firebase custom claims to app roles; enforce via ASP.NET policies and Angular/Flutter guards.  
- Data access: scope queries by account/branch; no cross-tenant leakage.  
- Secrets: keep in env vars/appsettings.* excluded from repo; rotate regularly.  
- Transport: HTTPS enforced; HSTS in production.

## Configuration & Environments
- API: appsettings.{Environment}.json for Firebase, Azure Blob, CORS, FeatureFlags, SignalR settings.  
- Flutter: introduce flavors or runtime config for API base/SignalR URLs (dev/stage/prod) instead of hardcoded Azure URL.  
- Angular: use `environment.ts` variants for API/SignalR/CORS expectations; enable strict mode + ESLint.  
- Feature flags: wrap risky changes (SignalR-only mode, menu publish flow, payment capture) for safe rollout.

## Real-time & Offline
- SignalR client: implement in shared_package with reconnect/backoff, keepalive, and group join (account/branch).  
- Event surface: orders, tickets, table status, menu availability changes.  
- Fallback: poll critical lists on interval when disconnected; surface banner for degraded state.  
- Offline: queue safe mutations (e.g., order submissions) briefly; reconcile on reconnect with server truth.

## Data & Storage
- Domain entities per `shared_package/lib/entities` (Account, Branch, Table, Menu, Order, Ticket, Payment, User).  
- Validate totals (amounts/tips/discounts) and status transitions server-side.  
- Azure Blob: clarify usage (assets/menu images/logs); restrict SAS and access per env.  
- Migrations/versioning: align DTOs with schema; include migration scripts in API repo when schema changes.

## Observability & Ops
- Logging: structured logging in API with correlation IDs; capture auth failures, SignalR connect/disconnect, and business events (order/ticket/payment).  
- Metrics: request latency, error rates, SignalR reconnects, queue depths (tickets).  
- Client telemetry: minimal error reporting hooks in Flutter/Angular for crashes/network failures; avoid PII.  
- Health: liveness/readiness endpoints; basic smoke tests in CI; CORS tests in CI for env drift.

## Quality Attributes Alignment (from PRD)
- Reliability: aim 99.5% API uptime; SignalR reconnect <5s with jitter.  
- Performance: P95 <500ms for CRUD; initial menu load <2s (cache + pagination).  
- Security: tightened CORS; JWT validation; least-privilege roles.  
- Compatibility: Flutter stable, Angular 20, .NET 8; recent mobile OS support.  
- Observability: logging/metrics + client error hooks in R1.

## Risks & Mitigations
- Env drift from hardcoded URLs → enforce env configs and PR checks for config.  
- Stale state if SignalR missing → mandatory reconnect + polling fallback; retries.  
- Authz gaps → explicit role matrix and guarded routes/endpoints; test matrix.  
- CORS misconfig → per-env allowlists and automated checks.  
- Payment/total inconsistencies → server-side validation and idempotency keys.

## Open Questions
- Payment provider scope beyond recording payments? Card-present vs external POS?  
- Depth of offline support expected for mobile flows?  
- Do we need split/merge checks in R1 (server + client)?  
- Reporting KPIs to prioritize (sales, voids, prep times)?  
- Azure Blob usage confirmation (assets vs other).
