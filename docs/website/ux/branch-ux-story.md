# UX Story: Branch Management Screen (Web)

Objective: Align branch management with the Style Guide and Interaction Guide, delivering quick wins for usability and consistency.

Scope: `MenuNestWebsite/src/app/branch-management/*` (table/list, add/edit modals), shared branch selection in menu/table management if reused styles are added.

Quick Wins / Tasks
1) Action placement
   - Add a sticky header bar above the table with page title + primary CTA “Add branch” (btn-primary using palette variables).
   - Ensure actions do not shift when rows grow.

2) Empty & loading states
   - Empty state card: icon + “No branches yet” + CTA.
   - Skeleton rows for loading (and/or spinner) instead of blank table.

3) Feedback & validation
   - Snack bars/toasts for create/update/delete success and errors.
   - Inline form validation: required markers, helper/error text; disable Save until valid.
   - Destructive actions use confirm modal with danger styling.

4) Table/list behavior
   - Row hover and selected-row styles per style guide (shared class). Remove default confirm prompts; use styled modals.
   - Keep edit/delete in a consistent column or kebab; reduce risk of accidental delete.
   - Add optional search/filter if the list grows (nice-to-have).

5) Accessibility
   - aria-label on icon-only buttons; ensure focus outline visible.
   - Keyboard order logical; modals trap focus and restore on close.

6) Responsive
   - At narrow widths, switch to card layout with inline actions; keep primary CTA sticky at top.

7) Theming hookup
   - Add CSS variables from `style-guide.md` to `styles.scss`; replace hardcoded colors in branch components with variables.
   - Standardize spacing/radius/shadows per guides.

Acceptance Criteria
- Primary CTA remains fixed in header; no movement with row count.
- Empty state and loading skeletons appear appropriately.
- CRUD operations show snack bars; errors are visible (not just console).
- Forms block submit until valid; inline errors shown; required indicators present.
- Destructive actions are styled as danger and confirmed via modal.
- Selected-row and hover styles match shared classes and are consistent with menu/table screens.
- Responsive view presents cards; primary action still easily accessible.
- Icon buttons have aria-labels; focus is visible; modals are keyboard accessible.

Artifacts
- Refer to `src/styles/style-guide.md` and `src/styles/interaction-guide.md` for tokens and behavior rules.
