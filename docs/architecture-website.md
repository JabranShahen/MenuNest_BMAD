# Architecture: Website

## Purpose

`MenuNestWebsite` is the operator-facing management dashboard. It handles onboarding and ongoing restaurant administration across branches, menus, categories, tables, staff, branding, and settings.

## Architectural Style

- Angular 21 application using `NgModule`
- Route-driven feature segmentation
- Service-oriented client integration layer
- Firebase Auth on the client
- Backend CRUD over HTTPS

## Application Bootstrap

- `src/main.ts` is the browser entry point.
- `src/app/app.module.ts` wires Firebase, Angular Material/CDK, toastr, charts, and the route module.
- Firebase configuration is embedded in the app module rather than injected via runtime config.

## Route Map

Primary routes from `app-routing.module.ts`:

- `/sign-up`
- `/login`
- `/sign-in`
- `/email-verification`
- `/reset-password`
- `/delete-account`
- `/dashboard`

Authenticated dashboard child routes include:

- `/dashboard/branches`
- `/dashboard/menus`
- `/dashboard/categories`
- `/dashboard/tables`
- `/dashboard/staff`
- `/dashboard/account`
- `/dashboard/ui-reference`
- `/dashboard/settings`

## Major Modules

- `services/`: API access and data orchestration
- `shared/`: reusable UI elements, charts, notifications, and layout helpers
- `dashboard/`: authenticated shell and overview
- `branch-management/`, `menu-management/`, `category-management/`, `table-management/`, `staff-management/`, `account-management/`: domain-specific workflows
- `use-cases/`: thin application workflows such as account creation

## Auth Flow

- Uses AngularFire compat modules.
- `AuthService` owns client-side Firebase login, persistence, verification, reset email, and ID token retrieval.
- `AuthGuard` protects the dashboard route tree.
- `AuthInterceptor` likely attaches bearer tokens to backend requests.

## Integration Boundaries

- Backend API root is configured to `https://menunestapi.azurewebsites.net/api` in environment and service files.
- Public assets include `auth-status.html`, which the landing page references for auth coordination.

## Testing Surface

- Jasmine/Karma unit tests exist for guards, routing, reset password, data preload, and several services/components.

## Risks

1. Firebase configuration and API URLs are hard-coded in source.
2. The app has a large app module and many declarations, which makes future modularization harder.
3. Generated `dist` output is committed, which can create drift between source and deployed artifacts.
