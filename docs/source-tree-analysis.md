# Source Tree Analysis

## Top-Level Structure

```text
MenuNest/
|-- MenuNestLandingPage/   # Static marketing site and smoke test
|-- MenuNestWebsite/       # Angular management dashboard
|-- MenuNestServer/        # ASP.NET backend solution wrapper
|   `-- MenuNestServer/    # Actual .NET solution root
|-- MenuNestApp/           # Flutter app workspace
|   `-- menunest/          # App suite and shared package
|-- docs/                  # Generated project knowledge
|-- _bmad/                 # BMAD framework files
`-- _bmad-output/          # BMAD generated artifacts
```

## Landing Site

```text
MenuNestLandingPage/
|-- index.html             # Main marketing page and CTA wiring
|-- assets/
|   |-- css/               # Theme and vendor styles
|   |-- js/                # Static scripts
|   `-- fonts/             # Bundled fonts
|-- images/                # Marketing images and branding assets
|-- tests/
|   `-- landing-page-smoke.ps1
`-- staticwebapp.config.json
```

Observations:

- The landing page is a hand-authored static site, not a SPA.
- It contains hard-coded hosted URLs for app, dashboard, and auth status handoff.

## Website

```text
MenuNestWebsite/
|-- src/
|   |-- main.ts            # Angular browser entry
|   |-- app/
|   |   |-- app.module.ts  # Root module and Firebase bootstrap
|   |   |-- app-routing.module.ts
|   |   |-- services/      # API and operational services
|   |   |-- shared/        # Reusable UI and utility components
|   |   |-- dashboard/     # Dashboard shell and overview
|   |   |-- branch-management/
|   |   |-- menu-management/
|   |   |-- category-management/
|   |   |-- table-management/
|   |   |-- staff-management/
|   |   |-- account-management/
|   |   `-- settings/
|   |-- assets/
|   `-- staticwebapp.config.json
|-- public/
|   `-- assets/auth-status.html
|-- angular.json
|-- package.json
|-- firebase.json
`-- dist/                  # Built output committed to repo
```

Observations:

- The website app is a classic Angular NgModule app rather than standalone bootstrap.
- The route structure is dashboard-centric, with authenticated child routes for core restaurant operations.
- `services/` acts as the integration boundary to the backend API.

## Backend

```text
MenuNestServer/MenuNestServer/
|-- MenuNestAPI/                   # ASP.NET entry project
|   |-- Program.cs                 # Startup, auth, CORS, SignalR
|   |-- Controllers/               # Public API surface
|   |-- Models/                    # API DTOs
|   `-- appsettings*.json
|-- MenuNest.Abstractions/         # MenuNest domain entities
|   `-- Entities/
|-- MenuNest.UseCases/             # Use case interfaces
|-- MenuNest.Usecase.Implementation/
|-- MenuNest.Services/             # Service implementations
|   `-- SignalR/
|-- MenuNest.Services.Abstractions/
|-- MenuNest.EntityManager/        # Domain decorators and managers
|-- MenuNest.BootStrapper/         # Autofac registrations
|-- HosannaTech.Abstractions/      # Shared framework abstractions
|-- HosannaTech.implementations/   # Cosmos-based data service
|-- HosannaTech.EntityManager/
`-- MenuNestAPI.Tests/             # Targeted controller tests
```

Observations:

- The solution is layered: API, abstractions, services, use cases, entity managers, and infrastructure.
- Persistence sits behind a generic abstraction layer, but the concrete implementation is Cosmos DB.
- Real-time updates are implemented through SignalR notifier decorators and a hub.

## Mobile Suite

```text
MenuNestApp/menunest/
|-- shared_package/
|   `-- lib/
|       |-- entities/             # Shared domain models mirrored from backend
|       |-- services/             # API, storage, table access, local persistence
|       |-- services/entity_services/
|       |-- services/entity_managers/
|       `-- widgets/
|-- admin_app/
|   `-- lib/
|       |-- main.dart
|       |-- providers/
|       `-- screens/
|-- table_app/
|   `-- lib/
|       |-- main.dart
|       |-- services/
|       `-- screens/
|-- kitchen_app/
|   `-- lib/
|       |-- main.dart
|       `-- screens/
`-- serve_app/
    `-- lib/
        |-- main.dart
        `-- screens/
```

Observations:

- `shared_package` is the architectural center of the mobile suite.
- App shells are thin and mostly wire Firebase initialization, service registration, theming, and entry screens.
- The table app appears to be the most mature guest flow, with table ordering and session-specific screens.

## Generated and Vendor Material Worth Ignoring During Feature Work

- `MenuNestWebsite/dist`
- `MenuNestWebsite/node_modules`
- Flutter build outputs and `.dart_tool`
- Azure generated ARM dependency files unless deployment work is in scope

For most implementation work, the meaningful source roots are:

- `MenuNestLandingPage/index.html`
- `MenuNestWebsite/src/app`
- `MenuNestServer/MenuNestServer/MenuNestAPI` plus adjacent domain/service projects
- `MenuNestApp/menunest/shared_package/lib` and app-specific `lib` folders
