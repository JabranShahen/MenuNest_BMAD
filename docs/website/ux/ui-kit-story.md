# UX Story: Shared UI Kit Extraction

Objective: Create a shared UI kit for management screens (Branch/Category) to standardize buttons, inputs, tables, status pills, badges, empty/skeleton states, toasts, and modal shells, eliminating duplicated styling and improving consistency.

Scope: `MenuNestWebsite/src/app/shared/ui-*` components plus refactors in Branch and Category screens to consume the shared kit.

Quick Wins / Tasks
1) Buttons
   - Variants: primary, secondary, ghost, danger; icon-button.
   - States: hover, focus-visible outline, disabled (no lift/shadow).
   - Shared sizing: min height 40px, padding 10x14, radius 8px.

2) Inputs
   - Text and textarea with label + helper + error slots.
   - States: focus outline, error state styling.
   - Consistent spacing: label/top/bottom margins.

3) Table shell
   - Header row sticky, shared padding, dividers, hover row.
   - Action column sizing, icon-button styling reused from buttons.
   - Optional column width inputs per screen.

4) Status + badges
   - Status pill (success/muted) and count badge style.
   - Shared font size/weight and radius.

5) Empty + skeleton
   - Empty-state block: icon/title/body + optional CTA.
   - Skeleton rows: configurable count, pill/short variants.

6) Toast helper
   - Success/error/info variants matching existing palette.

7) Modal shell
   - Header/body/footer layout, close affordance, spacing, background.

8) Containers
   - Panel (shared) and optional card list style.

Acceptance Criteria
- Branch and Category screens use shared header/panel (existing), table shell, buttons, inputs, status pills, badges, empty-state, skeleton, toast helper, and modal shell.
- Feature SCSS no longer redefines table/button/input/action icon styling; uses shared kit classes/components instead.
- Visual parity matches Branch spec (padding, hover tint, shadow/border).
- Standalone components import CommonModule where directives are used; shared components are imported (not declared) in modules.
- Tokens from `variables.scss` remain the source of palette/spacing; no hardcoded colors beyond fallbacks.

Assumptions / Notes
- Keep existing tokens in `variables.scss`.
- Use standalone components for the kit; import into AppModule or feature modules as needed.
- Maintain ARIA: icon buttons have aria-label; modals focusable; inputs with label/aria-describedby; buttons focus-visible outline.
