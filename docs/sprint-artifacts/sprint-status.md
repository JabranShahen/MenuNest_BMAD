# Sprint Status - MenuNest

**Status Date:** 2025-12-06  
**Sprint Goal:** Start hardening the platform: environment/config readiness and real-time reliability (SignalR), while UX branch alignment is already done.  
**Sprint Status:** Started

## Summary
- PRD drafted (`docs/prd.md`) and architecture updated (`docs/architecture.md`).
- Epics and stories seeded (`docs/sprint-artifacts/epics-and-stories.md`), including UX branch alignment, env/config hardening, SignalR reliability, core order/kitchen/payment flow, authZ, and observability.
- Completed: Web Back-Office UX Alignment epic and Branch Management UX Alignment story.
- Completed: Branch Management UX Remediation (UI-only scope, see `MenuNestWebsite/src/styles/branch-ux-remediation.md`).
- Completed: Category Management UX Alignment (owner: Amelia) — blocked delete when items exist, Name-sort only, owner/manager gating.
- Completed: Shared UI Kit Alignment (shared header/panel/table/empty/buttons/pills/badges applied across screens).
- Completed: Menus Screen UI Alignment (shared kit applied to Menus page).
- Completed: Menus Screen UX Remediation (legacy patching).
- Ready for Dev: Menus Screen Redesign (fresh component set).

## Board Snapshot
- Backlog (Not started): core order/kitchen/payment flow; AuthZ; Observability.
- Ready for Dev: Menus Screen Redesign (fresh components).
- Done: Menus Screen UI Alignment.
- Done: Menus Screen UX Remediation (legacy patching).
- Done: Branch Management UX Remediation (UI-only remediation per `MenuNestWebsite/src/styles/branch-ux-remediation.md`).
- Done: Category Management UX Alignment (Amelia) — UX parity with Branch patterns.
- Done: Shared UI Kit Alignment.
- In Progress: Env/config hardening; SignalR reliability.
- Done: Web Back-Office UX Alignment (Branch Management UX Alignment story).

## Risks / Blockers
- Open questions: payment provider scope, offline depth, split/merge checks timing, reporting KPIs, Azure Blob usage.
- Environment drift risk until API base/CORS are parameterized and locked per env.

## Next Actions to Start a Sprint
- Add estimates/owners for env/config hardening and SignalR reliability stories; keep them In Progress.
- Confirm remaining scope for the sprint (if any) from backlog.
- Create a simple task board (e.g., in your tracker) mirroring `epics-and-stories.md`.
