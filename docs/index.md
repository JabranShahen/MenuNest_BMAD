# Project Documentation Index

## Project Overview
- **Type:** Multi-part (mobile + backend + web)
- **Primary Language:** Dart/Flutter, .NET 8, TypeScript
- **Architecture:** Layered API with SignalR; shared Dart package across Flutter apps; Angular SPA; static landing pages.

## Quick Reference
- **Parts:**
  - admin_app — Flutter (Firebase Auth, provider) — `MenuNestApp/menunest/admin_app`
  - kitchen_app — Flutter (KDS) — `MenuNestApp/menunest/kitchen_app`
  - serve_app — Flutter (waitstaff) — `MenuNestApp/menunest/serve_app`
  - table_app — Flutter (table-side) — `MenuNestApp/menunest/table_app`
  - shared_package — Dart library (entities/services/SignalR client) — `MenuNestApp/menunest/shared_package`
  - server — ASP.NET Core API/SignalR — `MenuNestServer`
  - website — Angular 20 SPA — `MenuNestWebsite`
  - landing_page — static HTML — `MenuNestLandingPage`

## Generated Documentation
- [Project Overview](./project-overview.md)
- [Architecture](./architecture.md)
- [Source Tree Analysis](./source-tree-analysis.md)
- [Component Inventory](./component-inventory.md)
- [Development & Deployment Guide](./development-guide.md)
- [Integration Architecture](./integration-architecture.md)
- [API Contracts](./api-contracts.md)
- [Data Models](./data-models.md)
- [Project Parts Metadata](./project-parts.md)

## Existing Documentation
- `MenuNestApp/README.md`
- `MenuNestApp/menunest/admin_app/README.md`
- `MenuNestApp/menunest/kitchen_app/README.md`
- `MenuNestApp/menunest/serve_app/README.md`
- `MenuNestApp/menunest/table_app/README.md`
- `MenuNestWebsite/README.md`
- `Designs/MenuNestDesigns-Entity Relation.drawio.xml` (ERD)

## Getting Started
1) Choose your area:
   - Mobile apps: open corresponding Flutter app folder, run `flutter pub get`, then `flutter run`.
   - Backend: `cd MenuNestServer/MenuNestAPI && dotnet run` (configure appsettings for Firebase/Blob).
   - Angular SPA: `cd MenuNestWebsite && npm install && npm run start`.
2) Configure environments:
   - Firebase project IDs in backend `appsettings*.json`.
   - ApiService base URL in `shared_package/lib/services/api_service.dart` should be env-driven.
   - Tighten CORS in backend for production.
3) Review ERD: `Designs/MenuNestDesigns-Entity Relation.drawio.xml`.

## Incomplete Items
- None noted in this pass. If gaps are found, add markers and re-run generation.
