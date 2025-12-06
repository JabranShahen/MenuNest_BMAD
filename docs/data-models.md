# Data Models

Source of truth: `shared_package/lib/entities` (Dart) used across all Flutter apps. JSON serialization via json_serializable/build_runner.

## Core Entities (observed)
- Account, AccountConfig
- Branch
- Table (restaurant_table)
- Menu, Category
- Order, OrderItem, OrderHistory, OrderItemHistory
- Ticket, KitchenTicket, KitchenTicketRail
- Payment
- EMenuUser (user profile)
- Enums: ticket_status, etc.

## Serialization
- Annotations: `@JsonSerializable(explicitToJson: true)` on entities; generated `*.g.dart`.
- Date handling: e.g., `DateTimeNowFallbackConverter` in Order uses `DateTime.now().toUtc()` when missing.
- Numeric fields: prices/totals stored as double and minor units (ints) in some models.

## Relationships (from entities + ERD)
- Account → Branch → Table/Menu
- Menu → Categories → Items (implied)
- Order → OrderItems → OrderItemHistory; OrderHistory; Payments
- Tickets/KitchenTickets linked to Orders and Rails
- Users linked to Accounts/Branches; permissions implied in Abstractions/Services.

## Actions Needed
- Align DTOs with backend contracts exported from Swagger/OpenAPI.
- Confirm nullable vs required fields per endpoint.
- Add validation rules (currency codes, status enums, totals consistency).
- Ensure build_runner is run after model changes (table_app dev deps include build_runner/json_serializable).
