# Component Inventory

## Mobile (Flutter)
- Shared UI/Services: Provided via `shared_package` (entities, services, utility, assets).
- Admin app:
  - Providers: `auth_provider.dart` (Firebase Auth state + API hook).
  - Screens: `screens/staff_management/...` (admin flows).
- Kitchen app:
  - Screens: `screens/` (kitchen/KDS flows).
- Serve app:
  - Screens: `screens/serve_screen_1/...` (waitstaff flows).
- Table app:
  - Screens: `screens/menu_skin_1/...` (customer ordering/menu UI).
- Shared package assets: `shared_package/assets/logos/Logo.png`.

## Backend (.NET 8)
- SignalR: `MenuNest.Services/SignalR/EntitySignalRHub.cs` (groups/join/keepalive).
- DI/Bootstrap: `MenuNest.BootStrapper` (Autofac configuration).
- Services/Abstractions/UseCases: layered projects under `MenuNest.Services*`, `MenuNest.Abstractions`, `MenuNest.UseCases`, `MenuNest.Usecase.Implementation`.
- API host: `MenuNestAPI` (controllers not inspected here), Swagger via Swashbuckle, JWT via Firebase.

## Web
- Angular SPA components not enumerated (source tree large); key stack: Angular 20 + Material + AngularFire/FirebaseUI.
- Landing pages: static HTML variants under `MenuNestLandingPage` (multiple index-*.html).

## Real-time / Integration Hooks
- SignalR hub `/EntitySignalRHub` exposed by backend; shared_package includes SignalR client dependency.
- ApiService (shared_package) wraps HTTP with Firebase ID token bearer to `https://menunestapi.azurewebsites.net/api`.
