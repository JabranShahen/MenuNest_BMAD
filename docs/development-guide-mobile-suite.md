# Development Guide: Mobile Suite

## Prerequisites

- Flutter SDK compatible with Dart `>=3.5` or `>=3.6` per app
- Android/iOS toolchains as needed
- Firebase mobile configuration where required

## Workspace Layout

- `MenuNestApp/menunest/shared_package`
- `MenuNestApp/menunest/admin_app`
- `MenuNestApp/menunest/table_app`
- `MenuNestApp/menunest/kitchen_app`
- `MenuNestApp/menunest/serve_app`

## Typical Commands

Run within each app folder as needed:

```bash
flutter pub get
flutter run
flutter test
```

For code generation in projects that use JSON serialization:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

## Development Guidance

- Put shared entity or service changes in `shared_package` first.
- Keep app-specific UI and bootstrap logic inside each app shell.
- If backend contracts change, update both C# entities/API handling and Dart shared models/services.

## Integration Notes

- All apps depend on Firebase initialization.
- Shared package points to the hosted backend API and SignalR hub.
- `ServiceRegistrar` files are the first place to inspect when dependency wiring changes.

## Cautions

- Shared package changes have wide blast radius across all Flutter apps.
- Embedded environment URLs should be reviewed before creating new deployment environments.
