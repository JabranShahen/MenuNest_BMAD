# Component Inventory: Website

## Management Features

- `DashboardComponent`
- `DashboardOverviewComponent`
- `BranchManagementComponent`
- `MenuManagementComponent`
- `CategoryManagementComponent`
- `TableManagementComponent`
- `StaffManagementComponent`
- `AccountManagementComponent`
- `SettingsComponent`

These components appear to be the primary workflow owners for restaurant operations.

## Auth and Access Components

- `LoginComponent`
- `SignInComponent`
- `CreateNewAccountComponent`
- `EmailVerificationComponent`
- `ResetPasswordComponent`
- `DeleteRequestComponent`

## Modal and Editor Components

- `AddBranchModelComponent`
- `EditBranchModelComponent`
- `AddMenuModalComponent`
- `EditMenuModalComponent`
- `AddMenuItemModalComponent`
- `EditMenuItemModalComponent`
- `AddCategoryModalComponent`
- `EditCategoryModalComponent`
- `AddTableModalComponent`
- `EditTableModalComponent`
- `AddStaffModalComponent`
- `EditStaffModalComponent`
- `ConfirmDialogComponent`

## Shared UI Layer

- `ManagementHeaderComponent`
- `ManagementPanelComponent`
- `UiButtonComponent`
- `UiIconButtonComponent`
- `UiStatusPillComponent`
- `UiCountBadgeComponent`
- `UiEmptyStateComponent`
- `UiSkeletonComponent`
- `UiTableComponent`
- `UiFormFieldComponent`
- `UiReferenceComponent`
- `MenuNestChartComponent`
- `UiAdvancedSelectComponent`

## Service Layer

Key Angular services:

- `ApiService`
- `AuthService`
- `BranchService`
- `MenuService`
- `CategoryService`
- `TableService`
- `TableAccessService`
- `DashboardService`
- `EMenuService`
- `BlobStorageService`
- `WebsiteDataPreloadService`
- `WebsiteDataInvalidationService`
- `GlobalParameterService`
- `ImageSourceService`
- `CountryService`
- `MenuNestToastService`

## Notes

- The feature surface is broad, but the shared UI layer is becoming clearer and is worth preserving as a design system nucleus.
- The presence of `ui-reference` and reusable shared components suggests the project is already moving toward consistency.
