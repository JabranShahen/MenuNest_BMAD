# UX Story: Menus Screen Alignment

Objective: Align the Menus screen with the shared UI kit and Branch/Categories UX patterns for consistent layout, spacing, and affordances.

Scope: `MenuNestWebsite/src/app/menu-management/*` (menus, categories list, item cards), using shared components for header/panel/buttons/table/empty states, and normalizing imagery/sizing.

Quick Wins / Tasks
1) Layout
   - Set grid columns to sensible ratios (branches ~20–25%, menus ~30%, categories ~45–50%).
   - Use shared panels with consistent padding/border/shadow on white background.

2) Lists / Tables
   - Branch list uses shared table shell with visible dividers and fixed layout.
   - Menus list adopts shared list/table styling and shared buttons for actions.
   - Categories/items area constrained in width; item rows use consistent padding and no oversized icons.

3) Buttons & Icons
   - Use shared buttons (primary/ghost) and icon buttons; normalize icon sizes (20x20) and spacing.
   - Keep toggles/plus/add/edit/delete icons at consistent sizes; avoid oversized imagery.

4) Imagery
   - Constrain category/item images to reasonable sizes (e.g., 40x40) with object-fit.
   - Remove or resize any oversized toggle/plus graphics; use standard icon buttons instead.

5) Empty & Loading
   - Use shared empty-state component for menus and categories when lists are empty.
   - Use shared skeleton if loading states are shown.

6) Typography & Spacing
   - Align section headings and body text with Branch/Categories (font weight, size, and padding).
   - Apply consistent padding within panels and rows; ensure scroll areas have breathing room.

Acceptance Criteria
- Menus screen uses shared header/panel; background and padding match Branch/Categories.
- Branch list uses shared table shell with visible borders; menus list styled consistently without overflow.
- Category and item cards are constrained (no oversized toggles/plus icons); images max ~40px; icons 20px.
- Empty states for menus and categories use shared empty-state component; loading uses shared skeleton (if used).
- All buttons use shared UI kit; no ad-hoc button styles remain in Menus SCSS.
- Layout proportions keep content readable without huge blank areas or blown-up visuals.

Assumptions / Notes
- Reuse tokens from `variables.scss`.
- Leverage existing shared UI kit components; avoid inline styles for sizing.
- Keep ARIA: icon buttons have aria-label, focus-visible states remain intact.
