# MenuNest Project Overview

## Executive Summary

MenuNest is a restaurant operations platform split across four product surfaces in one repository:

- A static landing page for marketing and acquisition.
- An Angular dashboard for account, branch, menu, table, staff, and settings management.
- An ASP.NET Core API for business entities, authentication, guest table access, file upload URLs, and real-time updates.
- A Flutter app suite covering admin, table ordering, kitchen operations, and service workflows.

The product architecture is effectively a monorepo with a shared domain centered on restaurant accounts, branches, menus, tables, orders, kitchen tickets, staff, and guest sessions.

## Repository Classification

- **Type:** Monorepo
- **Primary languages:** TypeScript, C#, Dart, HTML/CSS/JS
- **Main runtime platforms:** Browser, ASP.NET Core, Flutter mobile/desktop targets
- **Primary architecture pattern:** Frontend clients plus shared backend API and event hub

## Tech Stack Summary

| Category | Technology | Notes |
|---|---|---|
| Marketing site | Static HTML/CSS/JS | `MenuNestLandingPage` |
| Dashboard web app | Angular 21 | AngularFire compat, Material, CDK, ngx-toastr, ngx-echarts |
| Backend API | ASP.NET Core | Autofac, Firebase Admin, Azure Blob Storage, SignalR |
| Data layer | Azure Cosmos DB | Wrapped through `HosannaTech` data abstractions |
| Mobile apps | Flutter | Four app shells plus shared package |
| Authentication | Firebase Auth | Used by website, mobile apps, and backend validation |
| Real-time updates | Azure SignalR | `/EntitySignalRHub` on backend and SignalR client in shared mobile package |
| Hosting footprint | Azure Static Web Apps, Azure App Service | Static sites and API endpoints reference Azure-hosted URLs |

## Product Parts

### 1. Landing

- **Path:** `MenuNestLandingPage`
- **Purpose:** Public-facing marketing site
- **Primary concerns:** Messaging, CTA routing, auth handoff, static asset delivery

### 2. Website

- **Path:** `MenuNestWebsite`
- **Purpose:** Restaurant operator dashboard
- **Primary concerns:** Account onboarding, operational CRUD, reporting, and management UI

### 3. Backend

- **Path:** `MenuNestServer\\MenuNestServer`
- **Purpose:** Domain API, guest access, persistence, and event distribution
- **Primary concerns:** Auth, entity orchestration, storage integration, real-time state propagation

### 4. Mobile Suite

- **Path:** `MenuNestApp\\menunest`
- **Purpose:** Operational apps for staff and guests
- **Primary concerns:** Branch selection, table ordering, kitchen execution, service workflows

## Key Domain Concepts

- **Account:** Restaurant business/account container, including branding.
- **Branch:** Restaurant branch associated with an account.
- **Menu / Category / MenuItem:** Content structure for ordering.
- **Table / TableQrAccess / GuestTableSession:** Dine-in access and session lifecycle.
- **Order / Payment / Ticket / KitchenTicketRail:** Operational order flow from guest request to kitchen/service completion.
- **EMenuUser:** Staff and account-linked users.

## High-Value Existing Docs and Signals

- `MenuNestWebsite/README.md`
- `MenuNestApp/README.md`
- Flutter app-level `README.md` files
- `MenuNestLandingPage/tests/landing-page-smoke.ps1`
- Azure service dependency files under `MenuNestServer/MenuNestServer/MenuNestAPI/Properties/ServiceDependencies`

## Immediate Risks

1. Sensitive secrets are committed in repository configuration, including Firebase service account JSON, Azure Blob connection strings, Azure SignalR connection strings, and Cosmos DB keys.
2. Hosted URLs differ across parts and appear environment-specific, increasing configuration drift risk.
3. Shared business entities exist in both backend C# and Flutter shared package, creating ongoing schema synchronization risk.

## Documentation Map

- [Architecture - Landing](./architecture-landing.md)
- [Architecture - Website](./architecture-website.md)
- [Architecture - Backend](./architecture-backend.md)
- [Architecture - Mobile Suite](./architecture-mobile-suite.md)
- [API Contracts - Backend](./api-contracts-backend.md)
- [Data Models - Backend](./data-models-backend.md)
- [Component Inventory - Website](./component-inventory-website.md)
- [Component Inventory - Mobile Suite](./component-inventory-mobile-suite.md)
- [Integration Architecture](./integration-architecture.md)
- [Deployment Guide](./deployment-guide.md)
- [Source Tree Analysis](./source-tree-analysis.md)
