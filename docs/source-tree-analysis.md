# Source Tree Analysis

## Repository Layout
- `MenuNestApp/` — Flutter workspace with multiple apps sharing a common Dart package.
- `MenuNestServer/` — .NET 8 solution for APIs, domain services, and SignalR.
- `MenuNestWebsite/` — Angular 20 SPA.
- `MenuNestLandingPage/` — Static HTML landing variants.
- `Designs/` — Draw.io ERD: `MenuNestDesigns-Entity Relation.drawio.xml`.
- `.bmad/` — Automation tooling (excluded from product docs).

## Part Breakdown (roots)
- admin_app (mobile): `MenuNestApp/menunest/admin_app`
- kitchen_app (mobile): `MenuNestApp/menunest/kitchen_app`
- serve_app (mobile): `MenuNestApp/menunest/serve_app`
- table_app (mobile): `MenuNestApp/menunest/table_app`
- shared_package (library): `MenuNestApp/menunest/shared_package`
- server (backend): `MenuNestServer`
- website (web SPA): `MenuNestWebsite`
- landing_page (static): `MenuNestLandingPage`

## Annotated Structure
```
MenuNest/
├─ MenuNestApp/                         # Flutter workspace
│  └─ menunest/
│     ├─ admin_app/                     # Admin console
│     │  ├─ lib/
│     │  │  ├─ providers/               # AuthProvider (Firebase Auth + API hook)
│     │  │  └─ screens/staff_management # Admin UI screens
│     │  ├─ android/ | ios/ | web/ ...  # Flutter platforms
│     │  └─ pubspec.yaml                # Uses shared_package, Firebase Core/Auth, provider
│     ├─ kitchen_app/                   # KDS-style app
│     │  ├─ lib/screens/                # Kitchen views
│     │  └─ pubspec.yaml                # Uses shared_package, Firebase Core
│     ├─ serve_app/                     # Service staff app
│     │  ├─ lib/screens/serve_screen_1/ # Serving workflow UI
│     │  └─ pubspec.yaml                # Uses shared_package, Firebase Core/Auth
│     ├─ table_app/                     # Table-side ordering/menu
│     │  ├─ lib/screens/menu_skin_1/    # Menu/order flows
│     │  └─ pubspec.yaml                # Firebase Core/Auth, HTTP, shared_preferences
│     └─ shared_package/                # Shared Dart library
│        ├─ lib/
│        │  ├─ entities/                # Data models (e.g., Order, Ticket, Menu, User)
│        │  ├─ services/                # API, entity managers, SignalR support
│        │  └─ utility/                 # Size utils/config helpers
│        └─ pubspec.yaml                # Firebase Core/Auth, HTTP, SignalR client
├─ MenuNestServer/                      # Backend (.NET 8)
│  ├─ MenuNestAPI/                      # ASP.NET Core host
│  │  ├─ Program.cs                     # Autofac, Firebase JWT auth, SignalR hub mapping
│  │  ├─ appsettings*.json              # Config (Firebase project, Azure Blob, etc.)
│  │  └─ MenuNestAPI.csproj             # Swagger, Autofac, FirebaseAdmin, Azure.Storage.Blobs
│  ├─ MenuNest.BootStrapper/            # Autofac registrations
│  ├─ MenuNest.Services/                # Domain services + SignalR hubs
│  ├─ MenuNest.Services.Abstractions/   # Service contracts
│  ├─ MenuNest.Abstractions/            # Domain entities (Order, Ticket, Menu, etc.)
│  ├─ MenuNest.UseCases/                # Use case definitions
│  ├─ MenuNest.Usecase.Implementation/  # Use case implementations
│  ├─ HosannaTech.* / EntityFabric.*    # Shared infra abstractions
│  └─ MenuNest.sln                      # Solution entry
├─ MenuNestWebsite/                     # Angular SPA (Angular 20, AngularFire, Material)
│  ├─ src/                              # App source
│  ├─ public/                           # Static assets
│  └─ package.json                      # Dependencies
├─ MenuNestLandingPage/                 # Static marketing/landing HTML variants
│  ├─ assets/ images/ form/             # Assets and form handlers
│  └─ index-*.html                      # Multiple hero/layout variants
├─ Designs/                             # Design artifacts
│  └─ MenuNestDesigns-Entity Relation.drawio.xml  # ERD (entities/relationships)
└─ MenuNest_BMAD/                       # BMad automation project
   └─ docs/                             # Generated outputs (this file, future docs)
```
