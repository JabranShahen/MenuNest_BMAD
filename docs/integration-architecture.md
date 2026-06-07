# Integration Architecture

## System View

MenuNest is organized around a shared backend API and a set of client surfaces.

```text
Landing Page -----> Website Dashboard ----\
                                           \
Mobile Apps -------------------------------> Backend API -----> Cosmos DB
                                            |        |
                                            |        `-----> Azure Blob Storage
                                            |
                                            `-----> Azure SignalR
                                            |
                                            `-----> Firebase Admin / Auth
```

## Client-to-Backend Integration

### Website to Backend

- Uses HTTPS API calls through Angular services.
- Authentication is Firebase-based on the client, with bearer tokens sent to the API.
- Operational dashboards depend on backend data for branches, menus, tables, staff, and overview metrics.

### Mobile Suite to Backend

- Uses shared REST services in `shared_package`.
- Uses SignalR clients in the shared package for real-time updates.
- Table access and guest bootstrap flow depend on backend-generated tokens and table access records.

### Landing to Website

- Landing page routes users to hosted app/dashboard URLs.
- It also references an auth status page in the website/public asset surface.

## Auth Integration

- Website and mobile apps authenticate with Firebase.
- Backend validates Firebase ID tokens.
- Backend also uses Firebase Admin and Firebase REST APIs for account/user flows and guest token issuance.

## Data and Event Flow

1. A client authenticates via Firebase.
2. The client sends authenticated requests to the backend API.
3. The backend persists state in Cosmos DB and may generate blob URLs or guest access artifacts.
4. Entity decorators and services publish updates through SignalR.
5. Mobile clients consume real-time events via the shared package.

## Configuration Coupling

Key integration settings are currently embedded in source and config:

- API base URL
- SignalR hub URL
- Static site/dashboard URLs
- Firebase API key and service account material
- Azure connection strings

That coupling is the main operational risk in the current architecture.
