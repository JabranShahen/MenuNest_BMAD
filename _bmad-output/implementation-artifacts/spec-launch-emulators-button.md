---
title: 'Launch Emulators Button'
type: 'feature'
created: '2026-05-06'
status: 'done'
route: 'one-shot'
---

# Launch Emulators Button

## Intent

**Problem:** The local MenuNest DevOps Windows utility did not expose a one-click way to start the Android emulators used for Kitchen, Serve, and Table app workflows.

**Approach:** Add a top-level `Launch emulators` button that runs `flutter emulators --launch` sequentially for `Kitchen_APP`, `Serve_APP`, `Table_APP`, and `Table_App_2`, then reports per-emulator launch results without blocking the UI thread.

## Suggested Review Order

**Launch service**

- Fixed AVD list and shell command construction live here.
  [`EmulatorLaunchService.cs:31`](../../MenuNest_dev_ops/MenuNestDevOps/EmulatorLaunchService.cs#L31)

- Each emulator launch continues after failures and captures result details.
  [`EmulatorLaunchService.cs:43`](../../MenuNest_dev_ops/MenuNestDevOps/EmulatorLaunchService.cs#L43)

**UI binding**

- Global action panel gains the requested button.
  [`Form1.Designer.cs:230`](../../MenuNest_dev_ops/MenuNestDevOps/Form1.Designer.cs#L230)

- Click handler disables conflicting global actions and reports launch results.
  [`Form1.cs:503`](../../MenuNest_dev_ops/MenuNestDevOps/Form1.cs#L503)

**Verification**

- Unit tests lock the requested emulator IDs and launch command shape.
  [`EmulatorLaunchServiceTests.cs:6`](../../MenuNest_dev_ops/MenuNestDevOps.Tests/EmulatorLaunchServiceTests.cs#L6)

- README documents the new global action.
  [`README.md:30`](../../MenuNest_dev_ops/README.md#L30)
