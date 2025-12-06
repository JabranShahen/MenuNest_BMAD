# Project Parts Metadata

| Part            | Type     | Path                                                           | Key Tech / Notes                                                       |
|-----------------|----------|----------------------------------------------------------------|------------------------------------------------------------------------|
| admin_app       | mobile   | MenuNestApp/menunest/admin_app                                 | Flutter 3.x, Firebase Auth, provider, shared_package services          |
| kitchen_app     | mobile   | MenuNestApp/menunest/kitchen_app                               | Flutter 3.x, Firebase Core, shared_package                             |
| serve_app       | mobile   | MenuNestApp/menunest/serve_app                                 | Flutter 3.x, Firebase Auth, shared_package                             |
| table_app       | mobile   | MenuNestApp/menunest/table_app                                 | Flutter 3.x, Firebase Auth, HTTP, shared_preferences, shared_package   |
| shared_package  | library  | MenuNestApp/menunest/shared_package                            | Dart library; entities/services; Firebase Auth, HTTP, SignalR client   |
| server          | backend  | MenuNestServer                                                 | ASP.NET Core (.NET 8), Autofac, SignalR, Firebase JWT, Azure Blob      |
| website         | web      | MenuNestWebsite                                                | Angular 20, AngularFire, Material, FirebaseUI                          |
| landing_page    | web      | MenuNestLandingPage                                            | Static HTML variants                                                   |

## Build/Run Commands (per part)
- Flutter apps: `flutter pub get`; `flutter run`; `flutter build apk/ios`.
- Shared package: `flutter pub get`; `flutter test` (if/when added).
- Backend: `cd MenuNestServer/MenuNestAPI && dotnet run` (or `dotnet publish`).
- Angular SPA: `cd MenuNestWebsite && npm install && npm run start` (or `npm run build`).
- Landing pages: serve static files (e.g., `npx serve .`).

## Config Notes
- Firebase project IDs: set in backend `appsettings*.json` and client Firebase configs.
- API base URL: currently hardcoded in `shared_package/lib/services/api_service.dart`; parameterize per environment.
- CORS: backend currently allows any origin; tighten for production.
