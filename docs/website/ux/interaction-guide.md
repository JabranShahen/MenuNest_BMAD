# Interaction & Layout Patterns (Web)

Standard behaviors for lists/tables, actions, responsive layouts, and feedback. Apply alongside the style tokens.

## Action Placement
- Primary actions belong in a fixed bar (top of list/table). Do not push actions down as content grows.
- Secondary inline actions (edit/view) stay near the item; destructive actions separated (kebab or secondary column) with confirm.
- On long forms, keep primary/secondary buttons in a sticky footer within the form container.

## Lists/Tables
- Header: sticky when scrolling; includes primary action(s).
- Row states: hover (surface-strong), selected (info-dark or primary with white text), focus outline.
- Empty state: replace the table body with a card (icon, message, primary CTA).
- Loading: skeleton rows; avoid layout jump.
- Pagination vs infinite: use pagination for admin data; keep page size consistent.

## Responsive Behavior
- Breakpoints: at narrow widths, switch tables to card lists with inline actions; keep primary CTA at top (sticky if needed).
- Reduce columns on small screens; show key fields only; move extra info into expandable/collapsible details.

## Forms & Modals
- Use modals for short, focused edits; side panel or full page for longer forms.
- Keep actions right-aligned; destructive in danger color.
- Validation: inline messages under fields; disable submit until valid; required markers on labels.
- Focus: trap in modals; restore focus on close.

## Feedback & States
- Success/error: snack bars/toasts for operations; inline errors for forms; no console-only feedback.
- Loading: spinners or skeletons; disable actions while loading; show progress where possible.
- Destructive flows: confirm modal with explicit labels (Delete branch), danger styling.

## Selection & Context
- When selection is required for secondary actions, keep those actions disabled until a row/card is selected; show helper text.
- Keep selected state visible even when scrolling (sticky header can show selected count).

## Scroll/Overflow
- Sticky headers for tables and action bars.
- Avoid horizontal scroll; if unavoidable, keep first column sticky and show shadow hints.

## Accessibility
- Aria labels on icon-only buttons.
- Visible focus outline on all interactive elements.
- Ensure keyboard access to modals, menus, and sticky actions.

## Pattern Examples (Branch Management)
- Header bar: Page title + primary “Add branch” (primary color), search/filter on the right. Header stays in place.
- Table/list: Hover highlight; click opens detail/edit. Delete in a kebab menu or secondary column; confirm modal.
- Empty state: Card with icon/text + “Add branch” CTA.
- Feedback: Snack bars for create/update/delete; inline field errors in add/edit modals.
- Responsive: At small widths, show branch cards (name, location, currency) with inline actions; sticky top CTA.

## Implementation Checklist
- Add sticky action header for list/table views; move primary CTAs there.
- Add empty state component and skeleton loaders.
- Add snack bars for CRUD success/error.
- Standardize selected-row class across branch/menu/table screens.
- Apply aria-labels to icon buttons and ensure focus styles.***
