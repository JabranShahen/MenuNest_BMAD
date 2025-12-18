# UX Story: Menu Management Layout Frame + Scroll Containment (Web)

Objective: Add a global page frame and ensure each list panel (Branches, Menus, Menu Categories/Items) scrolls independently when content exceeds available height, keeping context and primary actions visible.

Scope: `MenuNestWebsite/src/app/menu-management/*` (Menu Management screen layout + styling), using the shared UI kit/tokens and the Interaction Guide patterns.

## Problem
- The screen currently grows unbounded as lists expand, creating excessive page scroll and making it easy to lose context (selected Branch/Menu) and primary actions.
- Dense rows + many actions create scanning friction and “action hunting,” especially in long lists.

## Desired UX
1) **Global frame**
   - Page content sits inside a centered container with consistent gutters (not full-width edge-to-edge).
   - The main work area uses viewport height so the overall page does not expand indefinitely.

2) **Panelized columns with independent scroll**
   - Each column is a panel with a **header** (title + primary action + optional search/filter) and a **scrollable body**.
   - When content overflows, only the panel body scrolls (not the entire page).
   - Panel headers remain visible while scrolling within the panel.

3) **Primary actions stay discoverable**
   - “Add Menu” (and similar primary actions) live in the panel header (not buried at the bottom of long lists).

4) **Responsive wrapping**
   - Desktop: 3 columns.
   - Medium widths: wrap to 2 columns.
   - Mobile: stack to 1 column.
   - Scroll containment remains intact at all breakpoints; no horizontal overflow.

## Tasks
1) Add a global container (max width + centered) and consistent page gutters.
2) Convert the 3-column layout into fixed-height panels with scrollable bodies.
3) Make each panel header sticky (within the panel) so titles/actions stay visible while scrolling.
4) Move primary CTAs into the appropriate panel headers (e.g., “Add Menu”).
5) Ensure hover/selected row states match shared patterns; keep selection context visible.
6) Add/verify empty and loading states (shared empty-state and skeletons) within each panel body.
7) Validate accessibility: aria-labels for icon buttons, keyboard focus outlines, predictable tab order, and usable keyboard scrolling.

## Acceptance Criteria
- Menu Management content is framed inside a centered container (consistent gutters, no edge-to-edge stretch).
- The overall page does not grow unbounded with long data; content overflow is handled inside panels.
- Each panel (Branches, Menus, Menu Categories/Items) has its own vertical scroll when its content exceeds available height.
- Panel headers remain visible while scrolling inside their panel (title + primary actions are always accessible).
- Primary CTAs (e.g., “Add Menu”) are visible without scrolling to the bottom of a long list.
- No horizontal scrolling at desktop/1440/1080/mobile breakpoints; the layout wraps (3→2→1 columns) without control overflow.
- Accessibility: icon-only actions have `aria-label`s, focus indicators are visible, keyboard navigation works, and contrast meets WCAG AA per the Style Guide.

## References
- `MenuNest_BMAD/docs/website/ux/style-guide.md`
- `MenuNest_BMAD/docs/website/ux/interaction-guide.md`
- `MenuNest_BMAD/docs/website/ux/ui-kit-story.md`

## Dev Notes (Implementation Hints)
- Nested scroll layouts often require `min-height: 0` on grid/flex children; ensure each panel body can shrink and scroll.
- Prefer `100dvh`-based sizing for mobile address-bar correctness when using viewport height.
- Consider virtual scrolling for very large lists as a follow-up optimization.

