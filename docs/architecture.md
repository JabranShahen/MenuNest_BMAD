---
stepsCompleted: [1, 2, 3]
lastStep: 3
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
- Complexity level: unknown until FR/NFR clarified (likely medium-high given multi-channel + real-time).
- Estimated architectural components: multi-tenant API, authz, real-time notification layer, offline/cache strategy, storage, web/mobile frontends.

### Technical Constraints & Dependencies
- Auth via Firebase; backend validates JWT against Firebase project ID.
- Real-time via SignalR (`/EntitySignalRHub`); client side not yet implemented.
- API base URL hardcoded in shared_package (`https://menunestapi.azurewebsites.net/api`) â€” must be parameterized.
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

### Selected Starter: “Standardize Existing Stack”
**Rationale:**
- Brownfield; introducing a new framework adds risk.
- Existing stack already maps to required surfaces (mobile, API, web).
- Focus should be on hardening: env configuration, CORS tightening, SignalR client wiring, logging/observability, and tests.

**Initialization/Hardening Commands:**
- Flutter: `flutter create --template app` (for new modules), reuse shared_package; add env config pattern (flavors) and CI format/analyze.
- .NET API: `dotnet new webapi` (for new services/tests) with Autofac/SignalR/JWT wiring copied from current API; add `dotnet new xunit` for tests.
- Angular: `ng new` (for new micro-frontends if needed); in current app add stricter configs (strict TS, ESLint, env files).
