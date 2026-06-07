# Data Models: Backend

## Overview

The backend domain model is centered on restaurant operations and is defined primarily under `MenuNest.Abstractions/Entities`. Equivalent or near-equivalent models also exist in the Flutter shared package.

## Core Entities

### Account

Represents the restaurant account or tenant-level business context. Includes branding metadata such as `LogoBlobPath`.

### Branch

Represents an operating branch within an account. Branches scope menus, tables, tickets, and staff activity.

### Category

Represents menu categorization under an account or menu grouping.

### Menu

Primary menu container.

Important child entities:

- `MenuCategory`
- `MenuItem`

Fields indicate menu naming, descriptions, branch ownership, image references, display ordering, and activation state.

### Table

Represents a restaurant table.

Observed fields:

- `BranchId`
- `Name`
- `SeatingCapacity`
- `IsOccupied`

### TableQrAccess

Represents QR-based access or token material used to connect guests with a table workflow.

### GuestTableSession

Tracks guest access session state and includes Firebase UID linkage.

### Order

Represents a live or historical order.

Observed fields:

- `TableId`
- `BranchId`
- `Status`
- `Subtotal`
- `Tax`
- `Discount`
- `TotalPrice`
- `Currency`
- `TotalAmountMinor`
- `AmountPaidMinor`
- `OutstandingAmountMinor`

Nested collections:

- `Items`
- `History`
- `Payments`

### OrderItem

Nested item inside an order.

Observed fields:

- `MenuItemId`
- `Name`
- `Quantity`
- `Status`
- `ItemPrice`
- `TotalPrice`
- `IsTicketGenerated`
- `TicketId`

### Payment

Payment state for an order. Comments indicate states such as pending, authorized, captured, failed, refunded, and partially refunded.

### Ticket / KitchenTicket / KitchenTicketRail

Operational kitchen and service workflow entities linking orders to execution pipelines.

### EMenuUser

Represents application users or staff profiles tied to restaurant operations.

## Relationships

- An **Account** has many **Branches**.
- A **Branch** has many **Menus**, **Tables**, **Orders**, **KitchenTickets**, and staff.
- A **Menu** has many **MenuCategories**.
- A **MenuCategory** has many **MenuItems**.
- A **Table** can have current or historical **Orders** and **GuestTableSessions**.
- An **Order** has many **OrderItems**, **Payments**, and history records.
- **OrderItems** can map to tickets or ticket rails.

## Persistence Notes

- Cosmos DB is the concrete persistence layer.
- The project uses generic abstractions and entity manager patterns rather than a visible EF Core relational model.
- No formal migration framework was found in the scanned backend surface.

## Drift Watch

The Flutter shared package mirrors many backend entities. Any field additions, renames, enum changes, or lifecycle changes should be treated as cross-surface contract work.
