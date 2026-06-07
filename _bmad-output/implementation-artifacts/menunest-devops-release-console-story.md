# Story: MenuNest DevOps Release Console for Table, Kitchen, and Serve Apps

**Status:** dev-complete

## Story

As a MenuNest operator and maintainer,
I want a local Windows DevOps console inside `MenuNest_dev_ops` that can independently version, build, and publish the Table, Kitchen, and Serve Flutter apps to Google Play production,
so that each app can be released quickly from one self-contained workspace without manual command sequences or cross-app confusion.

## Problem Statement

MenuNest currently has three separate production Android apps that require similar release steps:

- read the app version from `pubspec.yaml`
- optionally bump patch + build or build number only
- build the Android App Bundle
- publish the bundle to Google Play production

Those tasks are repetitive and error-prone when done manually. Dukanz already solved this with a small Windows desktop utility, and MenuNest needs the same release pattern, but adapted for three apps instead of one.

The key UX constraint is structural:

- the app must have **three distinct release sections**
- one section for `table_app`
- one section for `kitchen_app`
- one section for `serve_app`
- each section acts on its own app only
- there must **not** be a single button that releases all three apps together

The tool should be self-contained under `MenuNest_dev_ops`, use the existing Google Play service-account JSON stored under `MenuNest_dev_ops/google_json`, and operate against the real MenuNest Play apps already present in Google Play Console.

## Scope

### In Scope

- Create a new Windows desktop DevOps app in `MenuNest_dev_ops`
- Support only:
  - `table_app`
  - `kitchen_app`
  - `serve_app`
- Mirror Dukanz DevOps behavior:
  - refresh app version/status
  - bump patch + build
  - bump build
  - create production release
  - open output folder
- Use one shared Google Play service-account JSON:
  - `MenuNest_dev_ops/google_json/menu-nest-de1680309afc.json`
- Publish to the Google Play `production` track
- Generate release notes automatically
- Keep actions isolated per app section

### Out of Scope

- `admin_app`
- iOS release automation
- internal / closed / open test tracks
- batch release of multiple apps in one click
- Firebase reconfiguration
- CI/CD or GitHub Actions rollout automation

## Acceptance Criteria

1. A new Windows desktop app exists under `MenuNest_dev_ops` and can be run locally to manage MenuNest mobile releases.
2. The UI contains three clearly separated sections, one each for:
   - MenuNest Table
   - MenuNest Kitchen
   - MenuNest Serve
3. Each section displays, for its app only:
   - target `pubspec.yaml`
   - current version text
   - current build number
   - output folder
   - Google Play JSON path
   - status text
4. Each section provides its own action buttons equivalent to Dukanz DevOps:
   - `Refresh`
   - `Bump Patch + Build`
   - `Bump Build`
   - `Create Production Release`
   - `Open Output Folder`
5. Clicking an action in one section must affect only that app and must not mutate or build the other two apps.
6. `Create Production Release` for each section performs the same core workflow as Dukanz DevOps:
   - bump patch + build number
   - build `flutter build appbundle --release`
   - upload the generated `.aab`
   - publish to the Google Play `production` track
7. The tool reads version information directly from each app's `pubspec.yaml` and does not duplicate versions in separate config files.
8. The Google Play JSON path resolves to:
   - `MenuNest_dev_ops/google_json/menu-nest-de1680309afc.json`
9. The tool uses the correct package name for each app:
   - `com.menunesttable`
   - `com.menunestkitchen`
   - `com.menunestserve`
10. The tool uses the correct app roots for each app:
    - `MenuNestApp/menunest/table_app`
    - `MenuNestApp/menunest/kitchen_app`
    - `MenuNestApp/menunest/serve_app`
11. The tool locates each app's bundle output under its own Flutter build directory and can open that folder in Explorer.
12. Automatic release notes are generated per app release and include at minimum:
    - app display name
    - released version
    - generation timestamp
13. If the JSON file is missing, the `pubspec.yaml` version is malformed, the Flutter build fails, or Play upload fails, the affected app section surfaces a clear error without crashing the whole application.
14. The repository excludes the service-account JSON from git tracking through ignore rules that cover `MenuNest_dev_ops/google_json/*`.
15. Manual local validation confirms that all three sections can:
    - refresh state
    - bump versions correctly
    - build their own app bundle
    - publish successfully to the correct Play app

## App Configuration Baseline

### Table App

- App label: `MenuNest Table`
- Root: `MenuNestApp/menunest/table_app`
- Version source: `MenuNestApp/menunest/table_app/pubspec.yaml`
- Package name: `com.menunesttable`
- Key properties: `MenuNestApp/menunest/table_app/android/key.properties`
- Bundle output:
  `MenuNestApp/menunest/table_app/build/app/outputs/bundle/release/app-release.aab`

### Kitchen App

- App label: `MenuNest Kitchen`
- Root: `MenuNestApp/menunest/kitchen_app`
- Version source: `MenuNestApp/menunest/kitchen_app/pubspec.yaml`
- Package name: `com.menunestkitchen`
- Key properties: `MenuNestApp/menunest/kitchen_app/android/key.properties`
- Bundle output:
  `MenuNestApp/menunest/kitchen_app/build/app/outputs/bundle/release/app-release.aab`

### Serve App

- App label: `MenuNest Serve`
- Root: `MenuNestApp/menunest/serve_app`
- Version source: `MenuNestApp/menunest/serve_app/pubspec.yaml`
- Package name: `com.menunestserve`
- Key properties: `MenuNestApp/menunest/serve_app/android/key.properties`
- Bundle output:
  `MenuNestApp/menunest/serve_app/build/app/outputs/bundle/release/app-release.aab`

## Tasks / Subtasks

- [x] Create the MenuNest DevOps desktop project shell (AC: 1, 14)
  - [x] Create a new .NET Windows Forms project under `MenuNest_dev_ops`
  - [x] Add package references required for Google Play publishing, matching the Dukanz pattern
  - [x] Add repo-local conventions for locating the MenuNest root from the DevOps app runtime directory
  - [x] Add or update git ignore rules so `MenuNest_dev_ops/google_json/*` is not tracked

- [x] Implement shared release infrastructure for multiple app definitions (AC: 7, 8, 9, 10, 11, 13)
  - [x] Create a typed app-definition model that describes one release target:
    - display name
    - app root
    - pubspec path
    - local properties path
    - bundle output path
    - package name
    - Play track
  - [x] Create shared repo path resolution utilities similar to Dukanz DevOps, but parameterized per app
  - [x] Create shared version parsing and mutation logic for Flutter `version: x.y.z+n`
  - [x] Create shared Flutter build execution logic that works per app definition
  - [x] Create shared Google Play publish logic that takes app definition + bundle path

- [x] Build the Table app release section (AC: 2, 3, 4, 5, 6, 9, 10, 11, 12, 13)
  - [x] Add a dedicated UI group/panel for `MenuNest Table`
  - [x] Display Table app version, build number, output folder, JSON path, and status
  - [x] Wire `Refresh` to read only `table_app/pubspec.yaml`
  - [x] Wire `Bump Patch + Build` to mutate only the Table app version
  - [x] Wire `Bump Build` to mutate only the Table app version
  - [x] Wire `Create Production Release` to bump patch + build, build the Table `.aab`, and publish only `com.menunesttable`
  - [x] Wire `Open Output Folder` to the Table app bundle directory

- [x] Build the Kitchen app release section (AC: 2, 3, 4, 5, 6, 9, 10, 11, 12, 13)
  - [x] Add a dedicated UI group/panel for `MenuNest Kitchen`
  - [x] Display Kitchen app version, build number, output folder, JSON path, and status
  - [x] Wire `Refresh` to read only `kitchen_app/pubspec.yaml`
  - [x] Wire `Bump Patch + Build` to mutate only the Kitchen app version
  - [x] Wire `Bump Build` to mutate only the Kitchen app version
  - [x] Wire `Create Production Release` to bump patch + build, build the Kitchen `.aab`, and publish only `com.menunestkitchen`
  - [x] Wire `Open Output Folder` to the Kitchen app bundle directory

- [x] Build the Serve app release section (AC: 2, 3, 4, 5, 6, 9, 10, 11, 12, 13)
  - [x] Add a dedicated UI group/panel for `MenuNest Serve`
  - [x] Display Serve app version, build number, output folder, JSON path, and status
  - [x] Wire `Refresh` to read only `serve_app/pubspec.yaml`
  - [x] Wire `Bump Patch + Build` to mutate only the Serve app version
  - [x] Wire `Bump Build` to mutate only the Serve app version
  - [x] Wire `Create Production Release` to bump patch + build, build the Serve `.aab`, and publish only `com.menunestserve`
  - [x] Wire `Open Output Folder` to the Serve app bundle directory

- [x] Add production publishing parity with Dukanz DevOps (AC: 6, 8, 9, 12, 13)
  - [x] Use the MenuNest service-account JSON from `MenuNest_dev_ops/google_json/menu-nest-de1680309afc.json`
  - [x] Scope credentials with `AndroidPublisherService.Scope.Androidpublisher`
  - [x] Create an edit, upload the `.aab`, update the `production` track, and commit the edit
  - [x] Generate app-specific release notes with app name and version
  - [x] Surface publishing failures clearly in the section that triggered them

- [x] Add validation and manual release guardrails (AC: 13, 15)
  - [x] Validate JSON file existence before enabling publish
  - [x] Validate bundle file existence after build
  - [x] Validate version parsing before any mutation
  - [x] Disable controls during long-running build/publish for the active section
  - [x] Preserve responsiveness of the overall app while one section is busy

- [ ] Validate the completed tool locally (AC: 15)
  - [ ] Run the app locally from `MenuNest_dev_ops`
  - [ ] Verify refresh and version bump behavior for Table, Kitchen, and Serve
  - [ ] Verify each app builds independently
  - [ ] Verify each app publishes to its correct Play app in production

## UI Requirements

The UI should stay close to Dukanz DevOps, but expanded to three independent app panels rather than one app screen.

### Required Structure

- Window title: `MenuNest DevOps`
- Short subtitle explaining this is a local Windows utility for repeatable MenuNest release tasks
- Three app sections:
  - `MenuNest Table`
  - `MenuNest Kitchen`
  - `MenuNest Serve`

### Required Per-App Fields

Each app section must show:

- `Target file`
- `Current version`
- `Build`
- `Status`
- `Output folder`
- `Play API JSON`

### Required Per-App Actions

Each app section must expose:

- `Create Production Release`
- `Open Output Folder`
- `Refresh`
- `Bump Patch + Build`
- `Bump Build`

### UX Constraint

There must be no combined `Release All` or equivalent cross-app publish button in v1.

## Technical Requirements

- Use Windows Forms on .NET 8, matching the Dukanz DevOps desktop approach.
- Use a shared service layer rather than duplicating all logic three times.
- Do not hard-code app-specific logic directly into button event handlers beyond selecting the app definition.
- Resolve the MenuNest repo root dynamically from the running application location.
- Keep `pubspec.yaml` as the source of truth for app versioning.
- Implement the same version mutation semantics as Dukanz:
  - `Patch` increments patch and build
  - `Build` increments build only
- Production release action must always perform patch + build increment before building.
- Use the Flutter SDK path from each app's `android/local.properties` when available, otherwise fall back to `flutter` on PATH.
- Keep publishing app-specific:
  - Table section publishes only `com.menunesttable`
  - Kitchen section publishes only `com.menunestkitchen`
  - Serve section publishes only `com.menunestserve`
- Keep track fixed to `production` in v1 to preserve Dukanz behavior.

## File Targets

Expected primary files:

- `MenuNest_dev_ops/MenuNestDevOps/MenuNestDevOps.csproj`
- `MenuNest_dev_ops/MenuNestDevOps/Program.cs`
- `MenuNest_dev_ops/MenuNestDevOps/Form1.cs`
- `MenuNest_dev_ops/MenuNestDevOps/Form1.Designer.cs`
- `MenuNest_dev_ops/MenuNestDevOps/RepoLayout.cs` or equivalent shared path/config file
- `MenuNest_dev_ops/MenuNestDevOps/VersionBumpService.cs`
- `MenuNest_dev_ops/MenuNestDevOps/RolloutBuildService.cs`
- `MenuNest_dev_ops/MenuNestDevOps/GooglePlayReleaseService.cs`
- `MenuNest_dev_ops/README.md`
- `.gitignore` or `MenuNest_dev_ops/.gitignore`

Context/reference files the implementation should rely on:

- `MenuNestApp/menunest/table_app/pubspec.yaml`
- `MenuNestApp/menunest/kitchen_app/pubspec.yaml`
- `MenuNestApp/menunest/serve_app/pubspec.yaml`
- `MenuNestApp/menunest/table_app/android/local.properties`
- `MenuNestApp/menunest/kitchen_app/android/local.properties`
- `MenuNestApp/menunest/serve_app/android/local.properties`
- `MenuNest_dev_ops/google_json/menu-nest-de1680309afc.json`

## Testing Requirements

### Minimum Manual Validation

- Table section refreshes and reports the current version correctly.
- Kitchen section refreshes and reports the current version correctly.
- Serve section refreshes and reports the current version correctly.
- Table bump buttons change only `table_app/pubspec.yaml`.
- Kitchen bump buttons change only `kitchen_app/pubspec.yaml`.
- Serve bump buttons change only `serve_app/pubspec.yaml`.
- Table production release uploads only to the `MenuNest Table` Play app.
- Kitchen production release uploads only to the `MenuNest Kitchen` Play app.
- Serve production release uploads only to the `MenuNest Serve` Play app.
- Output folder buttons open the correct app-specific bundle directories.
- Missing JSON or failed builds show clear errors without crashing the full app.

### Automated Validation Where Practical

- Unit-test version parsing and mutation.
- Unit-test app definition resolution.
- Unit-test release note generation.
- Unit-test repo-root and file-path resolution.

## Risks / Edge Cases

- Accidentally publishing the wrong app because package name and file paths are mismatched.
- Updating the wrong `pubspec.yaml` if app section wiring is not isolated.
- Build failures if `local.properties` is missing or Flutter SDK path is invalid.
- Play upload failures if the JSON file is moved or Play permissions change.
- Secrets leakage if `google_json` is not git-ignored.
- UI confusion if the three sections do not clearly communicate which app they target.

## Definition of Done

- A working `MenuNest_dev_ops` Windows desktop app exists in the repo.
- It mirrors Dukanz DevOps behavior, adapted to three separate MenuNest app sections.
- Each app section works independently and never triggers actions on the other two apps.
- Production publishing works for Table, Kitchen, and Serve using the shared MenuNest JSON.
- Repo ignore rules protect the JSON secret from source control.
- Manual validation confirms successful versioning, build, and publish flows for all three apps.

## Dev Notes

### Confirmed Inputs

- JSON filename:
  `menu-nest-de1680309afc.json`
- JSON location:
  `C:\Users\Jabra\Documents\GitHub\MenuNest\MenuNest_dev_ops\google_json\menu-nest-de1680309afc.json`
- Play target:
  `production`
- Release model:
  one selected app at a time through separate UI sections
- V1 app scope:
  `table_app`, `kitchen_app`, `serve_app`

### Relevant Existing Pattern

The existing Dukanz DevOps project is the behavioral template. MenuNest should preserve these specific patterns:

- Windows Forms desktop app
- version read directly from `pubspec.yaml`
- `Bump Patch + Build`
- `Bump Build`
- `Create Production Release`
- release-note generation during Play upload
- opening the bundle output folder in Explorer

### Important Non-Blocking Follow-Up

The MenuNest Firebase `google-services.json` files previously appeared to share the same `mobilesdk_app_id`, which should be reviewed separately. That does not block this DevOps story, because Play publishing for Android bundles depends on package identity and Play permissions rather than Firebase app registration quality.

## Dev Agent Record

### Implementation Summary

Created `MenuNest_dev_ops/MenuNestDevOps/` — a .NET 8 Windows Forms app mirroring Dukanz DevOps, adapted for three independent app sections.

**Key design decisions:**
- `AppDefinition` sealed record carries all per-app config (paths, package name, display name). All services accept it as a parameter — no per-app branching logic in handlers.
- `AppSectionControls` record in `Form1.cs` groups all UI references for one section; `ToggleBusyState`, `RefreshVersionView`, `ExecuteVersionUpdate`, and `ExecuteProductionRelease` are parameterized by it — each button handler is a one-liner delegation to the shared method.
- `RepoLayout.FindRepoRoot()` walks up from `AppContext.BaseDirectory` looking for `TableApp.PubspecRelativePath`, the same sentinel-walk pattern from Dukanz.
- Service account JSON path is resolved at `GooglePlayReleaseService` construction time (no throw if missing) and validated only at publish time — allows the UI to show the path even when the JSON is absent.
- `Form1.Designer.cs` uses a shared `BuildAppSection()` static helper to avoid repeating 30+ property assignments three times.
- `VersionBumpService.ParsePubspecVersion` and `ApplyVersionBump` are internal static methods (no file I/O) to enable unit testing.
- `GooglePlayReleaseService.BuildReleaseNotes` is internal static for the same reason.

### Tests Created

`MenuNestDevOps.Tests/` — xUnit, 30 tests, 100% pass.

- `VersionBumpServiceTests` — 9 tests covering parse, patch bump, build bump, content preservation, single-line replacement
- `AppDefinitionTests` — 14 tests covering package names, display names, app roots, path distinctness, bundle file naming, track constant
- `ReleaseNotesTests` — 7 tests covering per-app name inclusion, version text, timestamp, and cross-app note distinctness

### File List

- `MenuNest_dev_ops/.gitignore`
- `MenuNest_dev_ops/README.md`
- `MenuNest_dev_ops/MenuNestDevOps.sln`
- `MenuNest_dev_ops/MenuNestDevOps/MenuNestDevOps.csproj`
- `MenuNest_dev_ops/MenuNestDevOps/Program.cs`
- `MenuNest_dev_ops/MenuNestDevOps/RepoLayout.cs`
- `MenuNest_dev_ops/MenuNestDevOps/VersionBumpService.cs`
- `MenuNest_dev_ops/MenuNestDevOps/RolloutBuildService.cs`
- `MenuNest_dev_ops/MenuNestDevOps/GooglePlayReleaseService.cs`
- `MenuNest_dev_ops/MenuNestDevOps/Form1.cs`
- `MenuNest_dev_ops/MenuNestDevOps/Form1.Designer.cs`
- `MenuNest_dev_ops/MenuNestDevOps.Tests/MenuNestDevOps.Tests.csproj`
- `MenuNest_dev_ops/MenuNestDevOps.Tests/VersionBumpServiceTests.cs`
- `MenuNest_dev_ops/MenuNestDevOps.Tests/AppDefinitionTests.cs`
- `MenuNest_dev_ops/MenuNestDevOps.Tests/ReleaseNotesTests.cs`
