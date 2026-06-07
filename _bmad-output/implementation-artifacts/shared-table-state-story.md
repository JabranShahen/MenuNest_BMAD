# Story: Shared Table State — Device-Agnostic Table Sessions

Status: review

## Story

As a restaurant guest at a table,
I want any device that scans the same table QR code to see the live order state for that table,
so that multiple people at the same table can browse the menu, add items, and track their order from their own device without losing context or duplicating orders.

## Problem Statement

When multiple devices scan the same table QR code today, each device boots into an independent, isolated state. A second device sees either a blank ordering screen or, in a race condition, may attempt to create a second separate order for the same table.

The backend already implements shared guest sessions correctly — `GetOrCreateGuestSessionAsync` returns the same `GuestTableSession` to every device that redeems the same QR code version, and issues a Firebase custom token tied to the same `FirebaseUid` (`"guest-table-{session.id}"`). All devices are therefore already authenticated as the same principal.

The gap is on the Flutter client:

1. **Race condition on order creation**: When two devices bootstrap simultaneously and no active order exists yet, both get `null` from `loadActiveOrder()`, both create a local order object with independent UUIDs, and both try to `createOrder()` — resulting in two separate orders for the same table.
2. **No "joining" signal on boot**: When a second device bootstraps and an active order already exists, the ordering screen loads correctly (via `loadActiveOrder()`) but shows no indication that the device is joining a session in progress, potentially confusing users.
3. **`_needsSessionStart` double-fires**: If no order exists yet, both devices call `updateTable(isOccupied: true)` redundantly (harmless because the PUT is idempotent, but worth guarding).

Real-time sync between devices is already fully implemented through the unified SignalR resilience work. Once an order exists on the server, all subscribed devices on `table-{branchId}-{tableId}` will receive `EntityChanged` events and stay in sync automatically.

## Acceptance Criteria

1. When a second device redeems the same table QR code and an active order already exists, the ordering screen loads with all currently submitted order items visible, without any user action.
2. When a second device joins a table that has an existing active order, a non-blocking, non-intrusive banner or indicator is displayed on the ordering screen confirming that the device has joined an existing table session.
3. When two devices both bootstrap with no active order and both attempt to submit their first order, only one order is created on the backend; the second device's items are merged into the existing order rather than creating a duplicate.
4. All devices subscribed to the same table group receive live `EntityChanged` updates in real time when any device submits or modifies the shared order.
5. A device that closes and reopens the table app within the session's 4-hour validity window resumes the shared session state without requiring a new QR scan, picking up the current order from the server.
6. The `updateTable(isOccupied: true)` call is made at most once per table state transition — not redundantly by every joining device.

## Non-Goals

- Implementing exclusive device locking (Option A was explicitly declined; Option B — shared state — is the target).
- Syncing draft/unsubmitted cart items between devices in real time. Each device manages its own local draft. Only submitted order items are shared.
- Re-architecting the backend session model. The backend already handles shared sessions correctly.
- Changes to the kitchen app, serve app, or admin app.
- Replacing existing REST order submission flows.

## Tasks / Subtasks

- [x] Fix race condition: guard `createOrder` against duplicate table orders (AC: 3)
  - [x] In `OrderManager._submitOrder()`, before calling `orderService.createOrder()`, call `orderService.getTableOrderByStatuses(tableId, ['Active', 'InProgress'])` to check for a server-side active order.
  - [x] If a server order is found (race condition detected), update `activeOrder` to that server order and call `orderService.updateOrder()` with the merged item list instead of `createOrder()`.
  - [x] Merge strategy: append local draft items to the server order's `items` list, preserving all existing server items unchanged.
  - [x] This guard must run ONLY when `activeOrder.id` was locally generated (i.e., the order has not yet been persisted — check that `isTicketGenerated` is false for all items and the order was never successfully created).

- [x] Add "joining existing session" UX context (AC: 2)
  - [x] In `TableAppBootstrapper.run()`, after `orderManager.loadActiveOrder()`, detect if the loaded order is non-null and contains items (`items.isNotEmpty`). Set a flag `isJoiningExistingSession = true` on `TableOrderingInitialData`.
  - [x] Add `isJoiningExistingSession` boolean field to `TableOrderingInitialData`.
  - [x] In `TableOrderingScreen.initState()`, when `isJoiningExistingSession` is true, show a temporary, auto-dismissing banner (e.g., 4 seconds) with message: "You've joined the table session — {N} item(s) already ordered."
  - [x] Banner should be non-blocking and must not interfere with the ordering flow.

- [x] Guard `_needsSessionStart` against redundant `updateTable` calls (AC: 6)
  - [x] Captured `isReturningGuest` before `_bootstrapGuestSession()` runs (reading pre-existing `guestSessionId` from localStorage). Guard condition is `_needsSessionStart(activeOrder) && !isReturningGuest`.
  - [x] `guestSessionId` persisted to localStorage under `LocalStorageService.guestSessionIdKey` during `_bootstrapGuestSession()`.

- [x] Verify and harden bootstrap resumption path (AC: 5)
  - [x] `_isGuestSessionExpired()` added — checks cached `guestTableSession.expiresAt`, clears all session keys and routes to login if expired.
  - [x] Called on the resumption path (no QR code in URL) before Firebase auth check.
  - [x] `orderManager.loadActiveOrder()` already called in the main `run()` body for both fresh-QR and resumption paths.

- [x] Verify SignalR group subscription for Device 2 (AC: 4)
  - [x] `_configureRealtimeForActiveOrder()` is called inside `loadActiveOrder()` whenever a server order is found — no change required.
  - [x] Group name `table-{branchId}-{tableId}` is set correctly in the existing implementation.
  - [x] `refreshActiveOrder` / `_mergeServerOrderWithLocalDraft()` full-replacement path verified correct when `_hasUnsavedDraftChanges` is false.

- [x] Add tests (AC: 1, 3, 4)
  - [x] `order_manager_test.dart` — 5 new `tryResolveRaceCondition` tests: returns false when ID non-empty, returns false on 404, merges on race detected, sums quantities for matching menuItemId, appends distinct local items.
  - [x] `table_app_bootstrapper_test.dart` — `isJoiningSession` (4 cases), `needsSessionStart` (4 cases), `isGuestSessionExpiredForTest` (4 cases). All 12 pass.
  - [ ] Integration smoke test (manual): Two physical devices scan the same QR code. Device 1 submits an order. Device 2 (on any screen — menu or order view) receives the order in real time without navigating away. Device 2 adds and submits additional items → both devices update in real time.

## Implementation Notes

### Backend Is Already Correct — Do Not Change Core Session Logic

`TableAccessController.GetOrCreateGuestSessionAsync()` (lines ~237–280 of `TableAccessController.cs`) already:
- Queries for existing active `GuestTableSession` records for the table with matching `TableQrAccessVersion`
- Returns the existing session if valid, creates a new one only when none exists
- Issues a Firebase custom token with `FirebaseUid = "guest-table-{session.id}"` — this means ALL devices on the same session authenticate as the same Firebase principal

All devices receive the same `guestSessionId`, the same `tableId`, `branchId`, and `accountId` claims in their Firebase token. The `GuestPrincipalContext` extracted via `User.GetGuestContext()` will be identical across devices.

**Do not modify `GetOrCreateGuestSessionAsync`, `TableAccessController`, or any backend entity.**

### Race Condition Fix — Merge Strategy

The race condition only occurs during the window between bootstrap completing (both devices have `null` as `activeOrder`) and one device successfully creating the order on the server. After the first `createOrder()` succeeds and SignalR broadcasts `EntityChanged`, the second device's `_onEntityChanged` handler will call `_syncActiveOrderFromServer()` — but by then, the second device's submit may already be in flight.

The fix is a **pre-flight server check inside `_submitOrder()`**, not a lock or a global state mechanism. Keep it simple:

```dart
// In _submitOrder(), before orderService.createOrder():
final existingOrder = await orderService.getTableOrderByStatuses(
  activeOrder!.tableId,
  ['Active', 'InProgress'],
);
if (existingOrder != null && existingOrder.id != activeOrder!.id) {
  // Race condition: another device created the order first.
  // Merge our local draft items into the server order and update.
  final merged = _mergeLocalDraftIntoServerOrder(existingOrder, activeOrder!);
  activeOrder = merged;
  await orderService.updateOrder(merged);
  return; // Skip createOrder() below
}
```

The `_mergeLocalDraftIntoServerOrder()` method should:
- Take the server order as the base (canonical state)
- Append any local draft items (those with `isTicketGenerated == false`) that don't already appear in the server order (match by `menuItemId`)
- For matching items, sum the quantities
- Recalculate `subtotal`, `tax`, `totalPrice` after merge

### "Joining" Banner

Use the same notification pattern already established in the table app. The banner should be visually light — consider a bottom snackbar-style widget. Do not use a dialog or blocking modal. Duration: 3–4 seconds auto-dismiss.

### `isJoiningExistingSession` Flag

Add this boolean to `TableOrderingInitialData` (or the equivalent data class passed from bootstrap to the ordering screen). Default to `false`. Set to `true` only when `loadActiveOrder()` returns an order that is already in `'Active'` or `'InProgress'` status — meaning it was created before this device joined.

### SignalR Group — Already Enforced Correctly

`EntitySignalRHub.JoinGroup()` already validates that guests can only join `table-{branchId}-{tableId}`. Multiple devices with the same Firebase UID (same guest session) will each have their own SignalR connection but subscribe to the same group. All `EntityChanged` events for that table's orders and tickets will be broadcast to all connected devices in the group.

After the unified-signalr-resilience story, `OrderManager` consumes `SignalRConnectionService` for group management and no longer owns a raw `HubConnection`. The group registration call in `_configureRealtimeForActiveOrder()` must be verified to fire when loading an existing server order (not just when initializing a new local order).

### LocalStorage Key for `guestSessionId`

Add `guestSessionId` as a first-class localStorage key alongside `selectedBranch`, `selectedTable`, and `guestTableSession`. Persist it immediately after `TableAccessService.redeemPublicCode()` returns. Clear it when the session expires or when the app detects `SessionCompleteScreen` routing (order completed/cancelled).

## Technical Requirements

- Flutter Dart `>=3.5` / `>=3.6` as per app pubspec constraints.
- Do not introduce any new packages. Use existing `signalr_netcore`, `firebase_auth`, `provider`, and `shared_preferences` patterns already in `shared_package`.
- Preserve current SignalR hub endpoint: `/EntitySignalRHub`.
- Preserve current guest group pattern: `table-{branchId}-{tableId}`.
- Preserve all existing `EntityChanged` event processing in `OrderManager`.
- The `_mergeServerOrderWithLocalDraft()` method already exists — extend or reuse it for the race-condition merge rather than duplicating logic.
- REST submit flow is unchanged: `POST /api/order` creates, `PUT /api/order/{id}` updates.
- Auth header pattern unchanged: Firebase ID token in `Authorization: Bearer` header via `ApiService`.

## File Targets

Primary touch points:

- `MenuNestApp/menunest/shared_package/lib/services/entity_managers/order_manager.dart`
  → Add pre-flight server check in `_submitOrder()`, add `_mergeLocalDraftIntoServerOrder()`
- `MenuNestApp/menunest/table_app/lib/services/table_app_bootstrapper.dart`
  → Set `isJoiningExistingSession` on result, persist `guestSessionId` to localStorage, guard `_needsSessionStart`
- `MenuNestApp/menunest/table_app/lib/screens/table_ordering/table_ordering_screen.dart`
  → Render "joining session" banner when flag is set
- `MenuNestApp/menunest/shared_package/lib/services/local_storage_service.dart`
  → Add `guestSessionId` key constant and get/set helpers

Secondary review (likely no changes, but verify):

- `MenuNestApp/menunest/table_app/lib/main.dart`
  → Confirm routing from `BootstrapDestination` values; ensure `StartOrderScreen` is not shown when joining existing session
- `MenuNestApp/menunest/table_app/lib/service_registrar.dart`
  → No expected changes; verify `OrderManager` and `LocalStorageService` are correctly wired
- `MenuNestApp/menunest/shared_package/lib/services/entity_managers/order_manager.dart`
  → Verify `_configureRealtimeForActiveOrder()` is called when loading an existing server order

Test files:

- `MenuNestApp/menunest/shared_package/test/services/entity_managers/order_manager_test.dart`
  → Add race condition and merge tests
- `MenuNestApp/menunest/table_app/test/` (create if not exists)
  → Add bootstrapper tests for `isJoiningExistingSession` and `_needsSessionStart` guard

## Testing Requirements

Minimum validation matrix:

- **Single device, no prior order**: Scan QR → ordering screen shows empty cart. Submit items → order created. ✓
- **Single device, prior order exists**: Scan QR → ordering screen shows existing submitted items. Can add more items. ✓
- **Two devices, no prior order, simultaneous bootstrap**: Both scan QR → only ONE order created on backend (no duplicate). Both devices show the same order after submission. ✓
- **Two devices, Device 1 ordered first**: Device 2 scans QR → sees Device 1's submitted items on load → banner shown → can add more items → real-time sync works. ✓
- **Device 1, close app, reopen within session window**: Resumes session, loads current order state from server, no new QR required. ✓
- **Device 1, session expired (>4 hours)**: App clears cached session, routes to entry/QR screen. ✓
- **SignalR disconnect/reconnect with two devices**: Both devices recover and re-sync without manual reload (inherited from unified-signalr-resilience story). ✓

Preferred automated tests:

- Unit/service tests with mocked `OrderService` and `LocalStorageService`.
- Existing `OrderManager` test patterns in `shared_package/test/services/entity_managers/order_manager_test.dart` should be followed for new test additions.
- Manual smoke test: two physical or emulated devices running `table_app` simultaneously.

## Risks / Edge Cases

- **QR rotation mid-session**: If staff rotate the QR while two devices are active, `ExpireActiveSessionsForRotatedAccessAsync` in the backend will expire the current session. Devices will eventually fail auth token refresh. Current reconnect error handling should surface this gracefully (devices are routed to entry). No additional handling needed; document this behaviour in comments.
- **Session expiry during active ordering**: The `GuestTableSession` has a 4-hour TTL. Firebase custom tokens expire independently. If a device's Firebase token expires mid-session, the `ApiService` must catch 401 responses and attempt token refresh via `Firebase.signInWithCustomToken`. Verify existing `ApiService` error handling covers this.
- **Items submitted by Device 2 conflicting with Device 1 in-flight**: The merge strategy appends by `menuItemId`. If both devices add the same item independently, quantities are summed. This is acceptable but could surprise users. Consider logging a debug note when quantities are merged.
- **`isJoiningExistingSession` flag when order has no items**: An active order may exist on the server with zero items (order was created but all items were removed). In this case, set `isJoiningExistingSession = false` to avoid showing the misleading "items already ordered" banner.
- **Concurrent `_submitOrder()` calls from the same device**: Guard against calling `_submitOrder()` while a previous submit is in flight. A simple `_isSubmitting` boolean flag (likely already present) covers this.

## Definition of Done

- A second device scanning the same table QR loads the existing active order with submitted items visible on first render.
- No duplicate orders are created when two devices bootstrap simultaneously.
- A "joining session" banner is shown on Device 2 when an in-progress order is found.
- `updateTable(isOccupied: true)` is not called redundantly when a table is already occupied.
- Session resumption (app reopen within TTL) works without a new QR scan.
- All new automated tests pass.
- Manual smoke test with two devices passes end-to-end.

## Dev Notes

### Key Existing Code Patterns

**`GetOrCreateGuestSessionAsync` (backend — read-only reference):**
- Path: `MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/TableAccessController.cs` lines ~237–280
- Already returns the same session to every device. All devices get `FirebaseUid = "guest-table-{session.id}"` in their custom token.
- `GuestTableSession.Status` is `"Active"` or `"Expired"`. Only `"Active"` sessions are returned.
- `LastSeenAt` is updated on every `redeem` call — this is how the backend knows the session is still alive.

**`GuestPrincipalContext` (backend — read-only reference):**
- Path: `MenuNestServer/MenuNestServer/MenuNest.Services/Security/GuestPrincipalExtensions.cs`
- All guest endpoints validate `guest.TableId` matches the requested `tableId` via `User.GetGuestContext()`.
- Multiple devices with the same guest session will pass this validation identically.

**`OrderManager.loadActiveOrder()` (Flutter):**
- Path: `shared_package/lib/services/entity_managers/order_manager.dart`
- Calls `orderService.getTableOrderByStatuses(tableId, ['Active', 'InProgress'])`
- Maps to `GET /api/order/table/{tableId}/by-status?statuses=Active,InProgress`
- Sets `activeOrder`, loads tickets, calls `_configureRealtimeForActiveOrder()`.
- Creates a fresh local `Order` if none found on server.

**`OrderManager._mergeServerOrderWithLocalDraft()` (Flutter):**
- Preserves local draft items (`isTicketGenerated == false`) when server sends update.
- Applies server updates to submitted items (`isTicketGenerated == true`).
- Extend this logic for the race-condition merge, do not duplicate.

**`TableAppBootstrapper.run()` (Flutter):**
- Path: `table_app/lib/services/table_app_bootstrapper.dart`
- Key method to modify: add `isJoiningExistingSession` detection after `loadActiveOrder()`.
- Key method to modify: persist `guestSessionId` to `LocalStorageService` after `redeemPublicCode()`.
- Key method to review: `_needsSessionStart(order)` — add the localStorage guard.

**`EntitySignalRHub.JoinGroup()` (backend — read-only reference):**
- Path: `MenuNestServer/MenuNestServer/MenuNest.Services/SignalR/EntitySignalRHub.cs`
- Guests validated against: `"table-{guest.RequiredBranchId}-{guest.RequiredTableId}"`
- Multiple devices on same session will each independently join the same group. This is correct and supported.

### Relevant Source References

- Session backend: [Source: MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/TableAccessController.cs#GetOrCreateGuestSessionAsync]
- Backend SignalR enforcement: [Source: MenuNestServer/MenuNestServer/MenuNest.Services/SignalR/EntitySignalRHub.cs#JoinGroup]
- Guest auth claims: [Source: MenuNestServer/MenuNestServer/MenuNest.Services/Security/GuestPrincipalExtensions.cs]
- Order controller: [Source: MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/OrderController.cs#GetTableOrderByStatuses]
- Order manager: [Source: MenuNestApp/menunest/shared_package/lib/services/entity_managers/order_manager.dart]
- Bootstrap: [Source: MenuNestApp/menunest/table_app/lib/services/table_app_bootstrapper.dart]
- Ordering screen: [Source: MenuNestApp/menunest/table_app/lib/screens/table_ordering/table_ordering_screen.dart]
- SignalR service: [Source: MenuNestApp/menunest/shared_package/lib/services/realtime/signalr_connection_service.dart]
- Integration architecture: [Source: docs/integration-architecture.md]
- Mobile architecture: [Source: docs/architecture-mobile-suite.md]

### Project Structure Notes

- All Flutter changes belong in `shared_package` (order manager logic) or `table_app` (bootstrap, screen). Do not touch kitchen_app, serve_app, or admin_app.
- Follow existing `ServiceContainer` singleton pattern for any service dependencies — do not use `Provider.of` or `context.read` inside `OrderManager`.
- `LocalStorageService` uses `SharedPreferences` under the hood. Add `guestSessionId` as a `static const String` key alongside existing keys.
- The unified-signalr-resilience story has already moved SignalR management to `SignalRConnectionService`. Do not regress by adding raw `HubConnection` logic in new code.

## Open Questions

- Does `TableOrderingInitialData` already exist as a named class, or is it an anonymous/record type? If a record, adding `isJoiningExistingSession` may require converting it to a named class. Confirm before modifying.
- Does `StartOrderScreen` have any routing condition that could incorrectly appear when Device 2 joins a session? Inspect the routing from `TableAppEntryScreen` or `TableAppBootstrapScreen` for `BootstrapDestination` handling and confirm Device 2 always routes to `TableOrderingScreen`, not `StartOrderScreen`, when an active order exists.

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- `flutter test MenuNestApp/menunest/shared_package/test/services/entity_managers/order_manager_test.dart`
- `flutter test MenuNestApp/menunest/table_app/test/`

### Completion Notes List

- Standalone story. No sprint plan or story inventory found in `_bmad-output/planning-artifacts`.
- Backend session sharing is already fully implemented — `GetOrCreateGuestSessionAsync` returns the same session to all devices scanning the same QR code within the same version window.
- Primary client work is: (1) race condition guard on order creation, (2) "joining session" UX signal, (3) `_needsSessionStart` guard, (4) session resumption verification.
- SignalR real-time sync across devices is already operational from the unified-signalr-resilience story — no SignalR changes are needed.

### File List

- `_bmad-output/implementation-artifacts/shared-table-state-story.md`
- `MenuNestApp/menunest/shared_package/lib/services/entity_managers/order_manager.dart`
- `MenuNestApp/menunest/shared_package/lib/services/local_storage_service.dart`
- `MenuNestApp/menunest/table_app/lib/services/table_app_bootstrapper.dart`
- `MenuNestApp/menunest/table_app/lib/screens/table_ordering/table_ordering_screen.dart`
- `MenuNestApp/menunest/table_app/lib/screens/table_ordering/table_ordering_initial_data.dart`
- `MenuNestApp/menunest/shared_package/test/services/entity_managers/order_manager_test.dart`
- `MenuNestApp/menunest/table_app/test/bootstrap/table_app_bootstrapper_test.dart`

### Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-04-22 | Implementation complete — all production code and automated tests written; status set to review | claude-sonnet-4-6 |
| 2026-04-23 | Bug fix: `_onEntityChanged` in `order_manager.dart` now syncs server order into Device 2 when local activeOrder has empty id and tableId matches. Added 2 new tests (`handleEntityChangedForTest`). All 10 tests pass. | claude-sonnet-4-6 |
