# Development & Deployment Guide

## Prerequisites
- Flutter SDK 3.6.x (Dart >=3.6) for mobile apps and shared package.
- .NET SDK 8.0 for backend.
- Node.js 18+ for Angular SPA.
- Firebase CLI (optional) for managing Firebase projects.
- Android/iOS tooling for Flutter builds (Xcode, Android Studio/SDKs).

## Setup
- Global:
  - Install Flutter SDK and run `flutter doctor`.
  - Install .NET 8 SDK.
  - Install Node.js (18+).
- Per part (from repo root `MenuNest/`):
  - Mobile apps & shared package (runs in Flutter workspace):
    - `cd MenuNestApp/menunest/<app>` (admin_app, kitchen_app, serve_app, table_app).
    - `flutter pub get`.
    - For table_app (uses build_runner/json_serializable): `dart run build_runner build --delete-conflicting-outputs` when models change.
  - Shared package (library):
    - `cd MenuNestApp/menunest/shared_package`
    - `flutter pub get`.
  - Backend:
    - `cd MenuNestServer`
    - `dotnet restore`
  - Web SPA:
    - `cd MenuNestWebsite`
    - `npm install`
  - Landing pages:
    - Static HTML; no install required.

## Run
- Mobile apps (example admin_app):
  - `cd MenuNestApp/menunest/admin_app`
  - `flutter run` (select device/emulator).
  - Other apps: switch to `kitchen_app`, `serve_app`, `table_app` similarly.
- Shared package tests/build:
  - `cd MenuNestApp/menunest/shared_package`
  - `flutter test` (if tests added).
- Backend API:
  - `cd MenuNestServer/MenuNestAPI`
  - `dotnet run` (uses appsettings.* for Firebase JWT, Azure Blob).
- Angular SPA:
  - `cd MenuNestWebsite`
  - `npm run start` (dev server).
- Landing pages:
  - Serve via any static server (e.g., `npx serve .` inside `MenuNestLandingPage`).

## Build
- Mobile:
  - `flutter build apk` or `flutter build ios` per app (admin_app, kitchen_app, serve_app, table_app).
- Backend:
  - `cd MenuNestServer/MenuNestAPI`
  - `dotnet publish -c Release`
- Angular SPA:
  - `cd MenuNestWebsite`
  - `npm run build`
- Landing pages:
  - Static files ready; bundle/minify optional via your static site workflow.

## Tests
- Flutter apps/shared package: `flutter test` (no tests present yet).
- Backend: add and run `dotnet test` if/when test projects exist (none present now).
- Angular: `npm test` (Karma/Jasmine scaffolded).

## Configuration & Environment
- Backend:
  - `MenuNestServer/MenuNestAPI/appsettings.json` & `appsettings.Development.json` hold Firebase ProjectId, Azure Blob, JWT settings, and SignalR hub mapping (`/EntitySignalRHub`).
  - Auth: JWT Bearer validated against Firebase project; JWKS fetched from Google; Autofac DI configured in `Program.cs`.
- Mobile apps:
  - Firebase Core/Auth initialized; credentials typically in platform-specific configs (google-services.json, GoogleService-Info.plist) already present per app.
  - Shared package includes SignalR client dependency (`signalr_netcore`).
- Web SPA:
  - AngularFire/FirebaseUI; check `src/environments` (not inspected here) and `firebase.json` for hosting config.
- Landing pages:
  - Pure static; form handler lives under `MenuNestLandingPage/form`.

## CI/CD & Infra
- No Docker, Kubernetes, or IaC files detected.
- No CI pipelines found in repo roots.
- Deploy recommendations:
  - Backend: containerize or publish with `dotnet publish`; configure Firebase project settings and Azure Blob connection strings.
  - Angular: `npm run build` and host via static hosting/CDN; align with Firebase hosting if used.
  - Flutter: use platform-specific stores or sideload for field ops; ensure Firebase configs per app.

## Contribution Notes
- No CONTRIBUTING.md found. Recommended defaults:
  - Run `flutter format`/`dart format` before commits in Flutter code; `dotnet format` for backend; `npm run lint` (if added) for Angular.
  - Prefer feature branches and PRs with screenshots/gifs for UI changes.
