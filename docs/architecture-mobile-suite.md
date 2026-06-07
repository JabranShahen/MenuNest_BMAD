# Architecture: Mobile Suite

## Purpose

`MenuNestApp\\menunest` contains the operational Flutter app suite:

- `admin_app`
- `table_app`
- `kitchen_app`
- `serve_app`
- `shared_package`

The suite supports staff and guest workflows while reusing shared entities, services, and realtime infrastructure.

## Architectural Style

- Multi-app Flutter workspace
- Shared domain and integration package
- Thin app shells with app-specific screens and branding
- Firebase initialization per app

## Shared Package

`shared_package` is the key architectural asset. It provides:

- Shared entity models such as account, branch, order, menu, payment, table, and ticket
- API access helpers
- Blob storage and local storage services
- Table access service
- Entity services and entity managers
- SignalR-backed realtime managers
- Shared widgets and utilities

## App Responsibilities

### Admin App

- Firebase init plus `Provider`
- Login and auth wrapper
- Dashboard, branch management, table management, staff management, and order monitoring screens

### Table App

- Guest/table ordering flow
- Branch and table selection
- Session completion
- Specialized ordering skins and menu/order presentation

### Kitchen App

- Kitchen-specific work queue and branch selection
- Separate bootstrap and entry screens

### Serve App

- Service-side ticket management
- Kitchen/service/payment card views
- Operational screen for service staff

## Integration Boundaries

- Firebase Core/Auth for identity
- Shared package REST client against `https://menunestapi.azurewebsites.net/api`
- Shared package SignalR clients against `https://menunestapi.azurewebsites.net/EntitySignalRHub`

## Risks

1. API and SignalR URLs are embedded in the shared package.
2. Domain models are mirrored from backend entities, so versioning discipline matters.
3. Shared package coupling means breaking changes can affect all app shells at once.
