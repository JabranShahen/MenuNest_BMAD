# Component Inventory: Mobile Suite

## Shared Package Assets

### Shared Entities

- `Account`
- `AccountConfig`
- `Branch`
- `Category`
- `EMenuUser`
- `Order`
- `Payment`
- `RestaurantTable`
- `TableAccessBootstrap`
- `Ticket`
- `KitchenTicket`
- `KitchenTicketRail`

### Shared Services

- `ApiService`
- `BlobStorageService`
- `LocalStorageService`
- `TableAccessService`
- `ServiceContainer`

### Shared Entity Services

- `AccountService`
- `BranchService`
- `EMenuUserService`
- `KitchenTicketRailService`
- `KitchenTicketService`
- `MenuService`
- `OrderService`
- `TableService`

### Shared Entity Managers

- `MenuManager`
- `OrderManager`
- `TicketRailManager`

## Admin App Screens

- `LoginScreen`
- `DashboardScreen`
- `BranchManagementScreen`
- `TableManagementScreen`
- `StaffManagementScreen`
- `OrderMonitoringScreen`

## Table App Screens

- `TableAppEntryScreen`
- `TableAppBootstrapScreen`
- `BranchSelectionScreen`
- `TableSelectionScreen`
- `StartOrderScreen`
- `SessionCompleteScreen`
- `TableOrderingScreen`
- `MenuView`
- `OrderView`

## Kitchen App Screens

- `KitchenAppEntryScreen`
- `KitchenAppBootstrapScreen`
- `BranchSelectionScreen`
- `KitchenScreen1`
- `StartOrderScreen`

## Serve App Screens

- `ServeAppEntryScreen`
- `ServeAppBootstrapScreen`
- `OrderManagementScreen`
- `ServeScreen1`
- `KitchenTicketCard`
- `PaymentTicketCard`
- `ServiceTicketCard`

## Notes

- The mobile suite is centered around operational tasks rather than general navigation chrome.
- Shared entities and managers are the most important maintenance boundary in the Flutter codebase.
