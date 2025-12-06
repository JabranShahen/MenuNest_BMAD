# Integration Architecture

## Identity & Auth
- **Provider:** Firebase Auth (all Flutter apps, Angular SPA).
- **Backend validation:** JWT Bearer against Firebase project ID (`Firebase:ProjectId` in `MenuNestServer/MenuNestAPI/appsettings*.json`); JWKS fetched from Google; tokens cached with TTL.
- **Clients:** Firebase ID token acquired in apps; `ApiService` attaches `Authorization: Bearer <idToken>` to API calls.

## API Interaction
- **Base URL:** `https://menunestapi.azurewebsites.net/api` (hardcoded in `shared_package/lib/services/api_service.dart`; should be made environment-specific).
- **Transport:** HTTPS with Firebase bearer tokens.
- **Client wrapper:** `ApiService` (shared_package) provides GET/POST/PUT/DELETE with query support.
- **Error handling:** Not centralized; recommend global interceptors and retry/backoff for network errors.

## Real-Time Updates
- **SignalR Hub:** `/EntitySignalRHub` (backend). Methods: `JoinGroup`, `LeaveGroup`, `KeepAlive`.
- **Client:** `signalr_netcore` dependency present in shared_package; apps can subscribe to entity/group events (implementation pending).
- **Usage recommendation:** Join per-account or per-branch groups; push order/ticket/table status updates to reduce polling.

## Data Model Sharing
- **Shared entities:** Defined in `shared_package/lib/entities` (Order, Ticket, Menu, Payment, Branch, Table, User, etc.); used by all Flutter apps.
- **Serialization:** json_serializable/build_runner (table_app dev deps); consistent DTOs for API payloads.

## Cross-Part Interactions
- Mobile apps ↔ Backend: REST + Firebase bearer; SignalR planned for live updates.
- Web SPA ↔ Backend: AngularFire Auth + REST; CORS currently wide-open (tighten in production).
- Landing pages: Static; no backend coupling.

## Environment & Config Flow
- Backend config: `appsettings*.json` (Firebase ProjectId, Azure Blob, CORS policy, JWT).
- Client config: Firebase platform files (google-services.json / GoogleService-Info.plist), API base URL (needs env injection), SignalR endpoint (same host `/EntitySignalRHub`).

## Gaps / TODOs
- Parameterize API base URL per environment (dev/stage/prod) instead of hardcoding Azure site.
- Implement SignalR client wiring in Flutter apps; define group/topic strategy and message contracts.
- Tighten CORS policy; define allowed origins per environment.
- Add error/retry strategy and logging/telemetry across clients.
