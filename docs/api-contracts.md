# API Contracts

Current state: clients authenticate with Firebase ID tokens and call the backend at `https://menunestapi.azurewebsites.net/api` via `ApiService` (shared_package). Detailed endpoint specifications are not present in the repo; below is a starter outline and the observable contracts.

## Auth
- Scheme: JWT Bearer (Firebase ID token).
- Backend validation: issuer `https://securetoken.google.com/{FirebaseProjectId}`, audience = `{FirebaseProjectId}`.

## Known Integration Points
- **SignalR Hub:** `/EntitySignalRHub` (join/leave groups, keepalive). Intended for real-time entity updates.
- **REST Base:** `https://menunestapi.azurewebsites.net/api` (hardcoded; should be made env-specific).

## Suggested Endpoint Map (to verify/complete)
- `auth/` — login/refresh/profile (backed by Firebase token validation).
- `accounts/` — account + branch metadata.
- `menus/` — menus, categories, items.
- `orders/` — CRUD, status transitions, payments.
- `tickets/` — kitchen tickets, rails.
- `tables/` — table assignment, status.
- `users/` — staff management, roles/permissions.
- `payments/` — payment intents/records.

## Payloads & Models
- Client DTOs are defined in `shared_package/lib/entities`:
  - Order, OrderItem, OrderHistory, Ticket, Menu, Category, Table, Branch, Payment, EMenuUser, Account, AccountConfig, KitchenTicketRail, etc.
- Serialization: json_serializable; Dates often default to `DateTime.now().toUtc()` when missing.

## Headers
- `Authorization: Bearer <FirebaseIdToken>`
- `Content-Type: application/json`

## Actions Needed
- Export actual Swagger/OpenAPI from running backend (`MenuNestAPI` uses Swashbuckle) and replace this stub with concrete endpoints, methods, request/response schemas, and sample payloads.
- Parameterize API base URL per environment; document env variables/hostnames.
