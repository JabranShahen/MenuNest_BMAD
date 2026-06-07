# API Contracts: Backend

## Overview

The backend exposes a REST-style API under `/api/*` plus a SignalR hub at `/EntitySignalRHub`.

## Controllers and Endpoint Families

### `AccountController`

- `POST /api/account`
- `GET /api/account/{id}`
- `GET /api/account`
- `PUT /api/account/{id}`
- `DELETE /api/account/{id}`
- `GET /api/account/{id}/branding/logo`
- `PUT /api/account/{id}/branding/logo`
- `DELETE /api/account/{id}/branding/logo`

### `AuthController`

- `POST /api/auth/login`

Anonymous login endpoint that exchanges email/password against Firebase REST APIs.

### `BlobController`

- `GET /api/blob/generate-upload-sas`
- `GET /api/blob/generate-download-sas`

Used for SAS generation with blob path validation and sanitization.

### `BranchController`

- `POST /api/branch`
- `GET /api/branch/{id}`
- `DELETE /api/branch/{id}`
- `GET /api/branch/account/{accountId}`
- `PUT /api/branch/{id}`

### `CategoryController`

- `POST /api/category`
- `GET /api/category/{id}`
- `GET /api/category/account/{accountId}`
- `GET /api/category`
- `PUT /api/category/{id}`
- `DELETE /api/category/{id}`

### `CreateAccountController`

- `POST /api/createaccount`
- `GET /api/createaccount/userexists/{email}`

### `DashboardController`

- `GET /api/dashboard/account/{accountId}`

### `EMenuUserController`

- `GET /api/emenuser/{id}`
- `GET /api/emenuser`
- `PUT /api/emenuser/{id}`
- `DELETE /api/emenuser/{id}`
- `GET /api/emenuser/accountid`
- `POST /api/emenuser/staff`
- `GET /api/emenuser/staff/account/{accountId}`
- `PUT /api/emenuser/staff/{id}/deactivate`
- `PUT /api/emenuser/staff/{id}/activate`

### `KitchenTicketController`

- `POST /api/kitchenticket`
- `GET /api/kitchenticket/{id}`
- `PUT /api/kitchenticket/{id}`
- `DELETE /api/kitchenticket/{id}`
- `GET /api/kitchenticket/branch/{branchId}`
- `GET /api/kitchenticket/order/{orderId}`

### `KitchenTicketRailController`

- `GET /api/kitchenticketrail/branch/{branchId}`

### `MenuController`

- `POST /api/menu`
- `GET /api/menu/{id}`
- `DELETE /api/menu/{id}`
- `GET /api/menu/branch/{branchId}`
- `PUT /api/menu/{id}`

### `OrderController`

- `POST /api/order`
- `GET /api/order/{id}`
- `PUT /api/order/{id}`
- `GET /api/order/table/{tableId}/current`
- `GET /api/order/table/{tableId}/by-status`
- `GET /api/order/branch/{branchId}`

### `TableAccessController`

- `POST /api/tableaccess/redeem` (anonymous)
- `GET /api/tableaccess/branch/{branchId}`
- `GET /api/tableaccess/table/{tableId}`
- `POST /api/tableaccess/{tableId}/generate`
- `POST /api/tableaccess/{tableId}/rotate`

### `TableController`

- `POST /api/table`
- `GET /api/table/{id}`
- `GET /api/table/branch/{branchId}`
- `GET /api/table`
- `PUT /api/table/{id}`
- `DELETE /api/table/{id}`

## Auth Expectations

- Most endpoints require Firebase bearer authentication.
- Explicit anonymous paths include login and guest table access redemption.
- SignalR can receive token via query string for hub connections.

## Real-Time Contract

- REST calls mutate state.
- Clients subscribe to `/EntitySignalRHub` for entity change notifications.

## Notable Validation Behavior

- Branding logo paths are constrained to a safe naming convention.
- Blob SAS generation validates paths and rejects traversal/control characters.
