# Story: Unified SignalR Resilience Across Table, Kitchen, and Serve Apps

Status: done

## Story

As an operations platform team,
I want a single shared realtime connection strategy for the table, kitchen, and serve apps,
so that all three apps stay connected to SignalR during active use and automatically recover from disconnects without requiring manual reload.

## Problem Statement

The current realtime behavior is inconsistent across the Flutter app suite:

- `OrderManager` and `TicketRailManager` each own their own SignalR connection and reconnect behavior.
- `TicketRailManager` is the live-update path for kitchen and serve workflows, but its reconnect behavior is timer-based and screen-driven rather than centrally managed.
- The kitchen and serve screens manually reload state, which acts as a recovery path when SignalR silently stops delivering updates.
- The backend hub timeout is relatively aggressive for long-running mobile and tablet sessions.

Observed user symptom:

- After the apps have been running for some time, a new order submitted from the table app does not appear on the kitchen app until the operator presses the reload button.

This strongly indicates a gap in realtime continuity on the client side and a missing uniform recovery pattern across the three apps.

## Acceptance Criteria

1. A single shared realtime connection service exists in `shared_package` and is the only owner of the SignalR `HubConnection` lifecycle for the Flutter app suite.
2. `OrderManager` and `TicketRailManager` no longer implement independent raw SignalR connection management; they consume the shared realtime connection service instead.
3. While the app is in the foreground and authenticated, the shared realtime connection service maintains the SignalR connection in the background and exposes connection state changes to consuming managers.
4. If the SignalR connection drops because of network loss, token expiry, idle disconnect, or app resume, the shared realtime connection service automatically attempts reconnection with bounded backoff and without requiring user action.
5. After reconnection, the shared realtime connection service automatically re-joins all previously registered groups for the current app context.
6. After reconnection and group rejoin, consuming managers automatically perform a lightweight REST reconciliation refresh so that events missed during the disconnect window are recovered.
7. Table app order updates continue to reach the backend through existing REST calls, and kitchen/serve app ticket views recover automatically even if SignalR was temporarily unavailable.
8. Kitchen, serve, and table apps all integrate the same lifecycle behavior for foreground/resume handling through shared-package code rather than three separate ad hoc implementations.
9. Existing manual reload actions remain available as a fallback UX, but normal realtime recovery no longer depends on them.
10. Shared logging or debug tracing is added for connect, disconnect, reconnect, rejoin, and post-reconnect refresh so future field diagnosis can identify where failure occurred.
11. Automated tests cover reconnect and recovery behavior at the shared service / manager level, including at minimum:
    - reconnect after connection close
    - rejoin of previously registered groups
    - post-reconnect refresh callback execution
    - manager behavior when the connection is temporarily unavailable
12. Backend SignalR timeout settings are reviewed and adjusted if needed to better tolerate long-running mobile and tablet sessions, without weakening auth or group security rules.

## Non-Goals

- Replacing REST order creation or ticket retrieval flows.
- Removing manual reload controls from the UI.
- Re-architecting backend entity notification semantics beyond what is required for reliable reconnect and recovery.
- Introducing separate SignalR behavior per app shell.

## Tasks / Subtasks

- [x] Design and add a shared realtime foundation in `shared_package` (AC: 1, 3, 4, 5, 6, 8, 10)
  - [x] Add a new shared service, tentatively `shared_package/lib/services/realtime/signalr_connection_service.dart`, that owns `HubConnection` creation, start, stop, reconnect, token refresh, and state notifications.
  - [x] Add a shared group registry so registered groups survive reconnect and can be re-joined automatically.
  - [x] Add a post-reconnect callback or subscription mechanism so managers can run REST reconciliation after reconnect.
  - [x] Centralize hub URL and SignalR-specific configuration in the new service rather than duplicating it in managers.

- [x] Refactor `OrderManager` to consume the shared realtime connection service (AC: 2, 3, 4, 5, 6, 7, 8, 10)
  - [x] Remove direct `HubConnection` ownership from `OrderManager`.
  - [x] Register the correct groups for authenticated operator or table-guest scenarios through the shared service.
  - [x] Preserve existing `EntityChanged` handling, but route event registration through the shared service.
  - [x] Trigger order/ticket reconciliation after reconnect so stale table app state self-heals.

- [x] Refactor `TicketRailManager` to consume the shared realtime connection service (AC: 2, 3, 4, 5, 6, 7, 8, 10)
  - [x] Remove direct `HubConnection` ownership and internal reconnect timers from `TicketRailManager`.
  - [x] Register `branch-kitchen-{branchId}` through the shared service.
  - [x] Re-run `_refreshTickets(branchId)` automatically after reconnect and group rejoin.
  - [x] Ensure kitchen and serve views update from restored realtime flow without manual reload.

- [x] Add a shared lifecycle bridge for app foreground/resume handling (AC: 3, 4, 8, 10)
  - [x] Add shared-package support for app lifecycle observation so foreground/resume can trigger `ensureConnected()` and reconciliation.
  - [x] Wire table, kitchen, and serve app startup paths to activate the lifecycle bridge through their existing service registration/bootstrap patterns.
  - [x] Avoid screen-specific reconnect logic where the behavior belongs in shared services.

- [x] Review and harden backend hub tolerance settings (AC: 12)
  - [x] Review `KeepAliveInterval` and `ClientTimeoutInterval` in `MenuNestAPI/Program.cs`.
  - [x] Adjust values if they are too aggressive for tablets/phones that may briefly pause network activity.
  - [x] Preserve existing authentication, authorization, and group security constraints in `EntitySignalRHub`.

- [x] Add tests and validation (AC: 11)
  - [x] Add or extend shared-package tests around realtime reconnect behavior.
  - [x] Add targeted tests for manager-level recovery behavior where feasible.
  - [x] Validate that the kitchen symptom is covered by a regression scenario: connection loss followed by successful ticket refresh/recovery without manual reload.

## Implementation Notes

### Recommended Design

Use a single shared abstraction rather than duplicating reconnect logic in multiple managers.

Recommended structure:

- `shared_package/lib/services/realtime/signalr_connection_service.dart`
- `shared_package/lib/services/realtime/realtime_group_registry.dart`
- `shared_package/lib/services/realtime/realtime_lifecycle_bridge.dart`

Suggested responsibilities:

- `SignalRConnectionService`
  - build and own the single `HubConnection`
  - expose connection state stream / notifier
  - handle `onclose`, `onreconnecting`, `onreconnected`
  - apply bounded retry/backoff
  - refresh auth token through Firebase on reconnect
  - register and dispatch hub event handlers

- `RealtimeGroupRegistry`
  - track desired groups for the current authenticated/app context
  - re-join groups after reconnect
  - support add/remove semantics for branch/table context changes

- `RealtimeLifecycleBridge`
  - react to app foreground/resume
  - call `ensureConnected()`
  - trigger reconciliation callback if the app returns from an idle/suspended period

### Uniform Recovery Pattern

All three apps must follow the same runtime pattern:

1. App authenticates and becomes active.
2. Shared realtime service ensures the hub connection is established.
3. Manager registers required groups with the shared service.
4. If disconnect occurs, reconnect runs automatically in the background.
5. When reconnected, the shared service re-joins registered groups.
6. Manager executes a post-reconnect REST refresh to recover missed events.
7. UI updates via existing manager notifications without user reload.

### Why This Must Be Shared

Current drift:

- `OrderManager` has one reconnect model.
- `TicketRailManager` has another reconnect model.
- Kitchen and serve screens rely on load/reload patterns to recover visibility.

That is the structural source of the problem. A patch in one app would leave the same class of failure in the others.

## Technical Requirements

- Preserve current SignalR hub endpoint: `/EntitySignalRHub`.
- Preserve current group semantics:
  - `table-{branchId}-{tableId}`
  - `branch-{branchId}`
  - `branch-kitchen-{branchId}`
- Preserve guest restrictions enforced by `EntitySignalRHub`.
- Preserve the table app’s existing REST-based order submission flow.
- Do not break `EntityChanged` event processing already used by managers.
- Do not create parallel hub connections per screen; the shared service should be application-scoped within each app process.
- Keep the manual reload UI as a fallback and diagnostic path.

## File Targets

Primary expected touch points:

- `MenuNestApp/menunest/shared_package/lib/services/entity_managers/order_manager.dart`
- `MenuNestApp/menunest/shared_package/lib/services/entity_managers/ticket_rail_manager.dart`
- `MenuNestApp/menunest/shared_package/lib/services/service_container.dart`
- `MenuNestApp/menunest/table_app/lib/service_registrar.dart`
- `MenuNestApp/menunest/kitchen_app/lib/service_registrar.dart`
- `MenuNestApp/menunest/serve_app/lib/service_registrar.dart`
- New shared realtime service files under `MenuNestApp/menunest/shared_package/lib/services/realtime/`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Program.cs`
- Shared-package tests under `MenuNestApp/menunest/shared_package/test/`

Secondary review targets:

- `MenuNestApp/menunest/table_app/lib/services/table_app_bootstrapper.dart`
- `MenuNestApp/menunest/kitchen_app/lib/screens/kitchen_screen_1/kitchen_screen_1.dart`
- `MenuNestApp/menunest/serve_app/lib/screens/serve_screen_1/serve_screen_1.dart`

These may need only minimal changes if lifecycle handling is correctly centralized.

## Testing Requirements

Minimum validation matrix:

- Table app connected, kitchen app connected, serve app connected: realtime updates still flow normally.
- Kitchen app loses network temporarily: app reconnects and receives new kitchen tickets after recovery without manual reload.
- Serve app loses network temporarily: app reconnects and receives service/payment/kitchen ticket updates after recovery without manual reload.
- Table app resumes after idle/sleep: order updates and table-specific group subscriptions recover automatically.
- Auth token refresh / expiry scenario: reconnect path still acquires a valid token and restores subscriptions.
- Group-context change scenario: branch change or table change updates the registered group set correctly.
- Backend still enforces guest group restrictions after reconnect.

Preferred automated tests:

- Unit/service tests with mocked hub lifecycle behavior.
- Manager tests that verify post-reconnect refresh execution.
- Existing UI/manual smoke verification across the three Flutter shells after implementation.

## Risks / Edge Cases

- Multiple reconnect triggers firing concurrently and creating duplicate starts.
- Event handler duplication after reconnect if subscriptions are not de-registered or normalized.
- Group membership drift if old groups are not removed when branch/table context changes.
- Reconnect loops causing repeated REST refresh storms.
- App lifecycle hooks being attached at screen level instead of app/service level.
- Token refresh behavior forcing unnecessary disconnect churn if implemented too aggressively.

## Definition of Done

- Shared realtime connection foundation exists and is used by both `OrderManager` and `TicketRailManager`.
- Kitchen, serve, and table apps all rely on the same reconnect/rejoin/reconcile policy.
- Manual reload is no longer required for routine recovery from temporary SignalR disconnects.
- Relevant automated tests are added and passing.
- Backend timeout settings have been reviewed and updated if needed.
- Debug logging clearly shows connect, disconnect, reconnect, rejoin, and reconcile stages.

## Dev Notes

### Existing Repo Signals

- `OrderManager` currently owns a raw `HubConnection` and custom reconnect logic.
- `TicketRailManager` currently owns a separate raw `HubConnection` and a different reconnect path.
- Kitchen and serve screens both call `attachSelectedBranch()` on load and expose manual reload actions.
- Backend SignalR is hosted in `EntitySignalRHub` and uses group-based delivery through notifier decorators.

### Relevant Source References

- Realtime and architecture context:
  - [Source: docs/integration-architecture.md#Client-to-Backend Integration]
  - [Source: docs/architecture-mobile-suite.md#Integration Boundaries]
  - [Source: docs/architecture-backend.md#Real-Time Model]

- Existing client implementation:
  - [Source: MenuNestApp/menunest/shared_package/lib/services/entity_managers/order_manager.dart]
  - [Source: MenuNestApp/menunest/shared_package/lib/services/entity_managers/ticket_rail_manager.dart]
  - [Source: MenuNestApp/menunest/kitchen_app/lib/screens/kitchen_screen_1/kitchen_screen_1.dart]
  - [Source: MenuNestApp/menunest/serve_app/lib/screens/serve_screen_1/serve_screen_1.dart]
  - [Source: MenuNestApp/menunest/table_app/lib/screens/table_ordering/table_ordering_screen.dart]

- Existing backend implementation:
  - [Source: MenuNestServer/MenuNestServer/MenuNestAPI/Program.cs]
  - [Source: MenuNestServer/MenuNestServer/MenuNest.Services/SignalR/EntitySignalRHub.cs]
  - [Source: MenuNestServer/MenuNestServer/MenuNest.EntityManager/KitchenTicketNotifierDecorator.cs]
  - [Source: MenuNestServer/MenuNestServer/MenuNest.EntityManager/OrderNotifierDecorator.cs]
  - [Source: MenuNestServer/MenuNestServer/MenuNest.EntityManager/OrderToKitchenTicketDecorator.cs]

### Project Structure Notes

- This work should stay inside the existing Flutter shared package rather than introducing per-app realtime implementations.
- Use the existing service registration pattern through each app’s `service_registrar.dart`.
- Preserve current shared-package ownership of domain integration logic.

## Open Questions

- Should the shared realtime connection service be a single app-process singleton with event multiplexing, or should there be one scoped instance per manager but with shared lifecycle primitives? Recommendation: one app-process singleton.
- Should backend timeout tuning be conservative first, or changed only after client refactor lands? Recommendation: implement client unification first, then tune timeout values with field verification if needed.

## Dev Agent Record

### Agent Model Used

gpt-5

### Debug Log References

- `flutter test test\services\entity_managers\order_manager_test.dart test\services\entity_managers\ticket_rail_manager_test.dart test\services\realtime\signalr_connection_service_test.dart`
- `dotnet test MenuNestAPI.Tests\MenuNestAPI.Tests.csproj`

### Completion Notes List

- Standalone story created because no sprint plan or existing BMAD story inventory was present in `_bmad-output`.
- Added a shared realtime foundation with centralized SignalR connection ownership, owner-scoped group registration, reconnect recovery callbacks, and lifecycle resume handling in the Flutter shared package.
- Refactored `OrderManager` and `TicketRailManager` to consume the shared realtime service instead of managing independent hub connections and reconnect logic.
- Increased backend SignalR tolerance from `15s/30s` keepalive-timeout settings to `20s/90s` to better tolerate mobile and tablet idle periods without weakening hub authorization rules.
- Added shared-package reconnect tests plus a ticket-rail recovery regression test covering disconnect, reconnect, group rejoin, and post-reconnect refresh.

### File List

- `_bmad-output/implementation-artifacts/unified-signalr-resilience-story.md`
- `MenuNestApp/menunest/shared_package/lib/services/realtime/realtime_group_registry.dart`
- `MenuNestApp/menunest/shared_package/lib/services/realtime/signalr_connection_service.dart`
- `MenuNestApp/menunest/shared_package/lib/services/realtime/realtime_lifecycle_bridge.dart`
- `MenuNestApp/menunest/shared_package/lib/services/entity_managers/order_manager.dart`
- `MenuNestApp/menunest/shared_package/lib/services/entity_managers/ticket_rail_manager.dart`
- `MenuNestApp/menunest/table_app/lib/service_registrar.dart`
- `MenuNestApp/menunest/kitchen_app/lib/service_registrar.dart`
- `MenuNestApp/menunest/serve_app/lib/service_registrar.dart`
- `MenuNestApp/menunest/shared_package/test/services/realtime/test_realtime_fakes.dart`
- `MenuNestApp/menunest/shared_package/test/services/realtime/signalr_connection_service_test.dart`
- `MenuNestApp/menunest/shared_package/test/services/entity_managers/order_manager_test.dart`
- `MenuNestApp/menunest/shared_package/test/services/entity_managers/ticket_rail_manager_test.dart`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Program.cs`
