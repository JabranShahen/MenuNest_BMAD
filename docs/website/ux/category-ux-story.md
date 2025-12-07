# UX Story: Category Management Screen (Web)

Objective: Align the Categories screen with the Style Guide and Interaction Guide, matching the Branch Management UX patterns for consistency, clarity, and accessibility.

Scope: `MenuNestWebsite/src/app/category-management/*` (table/list, add/edit modals) plus any shared category chips/lists if reused elsewhere.

Quick Wins / Tasks
1) Information architecture and actions
   - Table/list columns: Name, Status (Active/Disabled), Items count (# linked menu items).
   - Sticky header bar with page title + primary CTA "Add category" (btn-primary using palette tokens).
   - Keep action column consistent (edit/delete in a kebab or dedicated column).

2) Empty and loading states
   - Empty state card: icon + "No categories yet" + CTA to add.
   - Skeleton rows (and/or spinner) for loading instead of a blank table.

3) Feedback and validation
   - Snack bars/toasts for create/update/delete success and errors.
   - Inline form validation: required markers, helper/error text; disable Save until valid.
   - Name uniqueness check; show inline error if duplicate.

4) Safety and status handling
   - Destructive actions use a confirm modal with danger styling.
   - Block delete when linked menu items exist; show the count and guide users to disable or reassign instead.
   - Surface the impact of delete (when allowed) and provide a status toggle (enable/disable) as the safe alternative.
   - Backend support: delete endpoint should return a clear blocked response (e.g., 409) when linked items exist; list/detail should expose `itemsCount` (or equivalent) so the UI can render counts and messaging.

5) Table/list behavior
   - Hover and selected-row styles per style guide; keep edit/delete in a consistent column.
   - Sorting: deterministic by Name (asc) with optional Status filter; no drag-and-drop reorder for now.
   - Keep action buttons stable so rows do not shift on hover.

6) Accessibility
   - aria-label on icon-only buttons; focus outline visible.
   - Keyboard order logical; modals trap focus and restore on close.
   - Ensure table headers announce sortable state if sorting is enabled.

7) Responsive
   - At narrow widths, switch to cards with inline actions; keep primary CTA sticky at top.
   - Preserve visibility of Name, Status, and Items count in the card layout.

8) Theming hookup
   - Use palette/spacing/radius/shadow tokens from `style-guide.md`; no hardcoded colors.
   - Reuse shared classes used by branch management for consistency.

Acceptance Criteria
- Primary CTA remains fixed in a header; no movement with row count or scroll.
- Empty state and loading skeletons appear appropriately.
- CRUD operations show snack bars; errors are visible and actionable.
- Forms block submit until valid; required and uniqueness errors are inline.
- Destructive actions are styled as danger and confirmed; delete is blocked when linked items exist and the user is guided to disable/reassign.
- Selected-row and hover styles match shared classes and are consistent with branch/table screens.
- Sorting/filtering works as declared; Name-sort is the default; no drag-and-drop reorder.
- Responsive view presents cards; primary action remains easy to reach.
- Icon buttons have aria-labels; focus is visible; modals are keyboard accessible and trap focus.
- Tokens from `style-guide.md`/`interaction-guide.md` are applied; no hardcoded colors remain.
- Backend behavior: list/detail expose `itemsCount`; delete returns a blocked status (e.g., 409) with message when items exist; delete succeeds only when count is zero.

Assumptions / Open Questions
- Role gating: owner/manager for add/edit/disable/delete actions.
