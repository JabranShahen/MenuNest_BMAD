# Project Overview

## Summary
MenuNest is a multi-part restaurant platform comprising multiple Flutter apps (admin, kitchen/KDS, serve, table-side), a shared Dart package, a .NET 8 backend with SignalR and Firebase JWT auth, an Angular SPA, and static landing pages. An ERD is available at `Designs/MenuNestDesigns-Entity Relation.drawio.xml`.

## Parts at a Glance
- admin_app (Flutter): Admin console with Firebase Auth, provider state, shared_package services.
- kitchen_app (Flutter): KDS-style app consuming shared_package.
- serve_app (Flutter): Waitstaff app with Firebase Auth, shared_package.
- table_app (Flutter): Customer ordering/menu app; uses shared_package, HTTP, shared_preferences.
- shared_package (Dart library): Shared entities/services (HTTP + Firebase Auth bearer; SignalR client), utilities, logo assets.
- server (.NET 8): ASP.NET Core API with Autofac DI, Firebase JWT bearer auth, SignalR hub (`/EntitySignalRHub`), Azure Blob, Swagger.
- website (Angular 20): AngularFire + Material SPA; Firebase UI for auth.
- landing_page (static): Marketing/landing HTML variants.

## Key Technologies
- Mobile: Flutter 3.x, Firebase Core/Auth, provider (admin), HTTP/shared_preferences/uuid (table_app), shared_package.
- Shared: Firebase Auth token-based ApiService; SignalR client dependency.
- Backend: ASP.NET Core, SignalR, Autofac, JWT (Firebase), Azure Storage Blobs, Swagger.
- Web: Angular 20, AngularFire, FirebaseUI, Angular Material.
- Auth: Firebase across clients; backend validates Firebase ID tokens.

## Data & Integrations
- ERD: `Designs/MenuNestDesigns-Entity Relation.drawio.xml` (entities: Account, Branch, Table, Menu, Order, Ticket, Payment, User, etc.).
- API base: `https://menunestapi.azurewebsites.net/api` (hardcoded in shared_package ApiService; parameterize for envs).
- Real-time: SignalR hub `/EntitySignalRHub`; clients can join groups, keep alive; Flutter SignalR client available in shared_package.

## Deployment Notes
- Backend: Configure `MenuNestServer/MenuNestAPI/appsettings*.json` (Firebase:ProjectId, Azure Blob, CORS). CORS currently allows any origin—tighten for prod.
- Flutter apps: Platform Firebase configs present (google-services.json, GoogleService-Info.plist); build per app.
- Angular: `npm run build`; host on static hosting (Firebase hosting likely).
- Landing pages: static hosting of `MenuNestLandingPage/`.

## Risks / Follow-ups
- ApiService base URL hardcoded; externalize per environment.
- SignalR client not wired into app flows—define subscription strategy for live updates.
- AuthProvider (admin_app) uses placeholder profile load—implement real profile/account binding.
- Tests missing across stacks—add smoke/contract tests (Flutter, .NET, Angular).
- CORS policy wide open; restrict for production.
