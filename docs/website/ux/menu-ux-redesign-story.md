# Story: Menus Screen Redesign (Fresh Components)

**Goal**  
Rebuild the Menus page with a clean component set that matches Branch/Categories patterns and scales across desktop (3 cols), 1440/1080 (wrap to 2 cols), and mobile (stack).

**Scope**  
New components (do not patch legacy): MenuHeader, MenuList, CategoryList, ItemList, using shared tokens and UI primitives where sensible. Legacy menu styles remain untouched. Use existing menu code only as a behavioral reference (data flow/intent), not as a structural template.

**What to build**
- Layout: responsive shell with consistent gutters; auto-fit grid (desktop 3 cols; wrap to 2 on 1440/1080; stack on mobile). Max width ~1400px, centered.
- Menus panel: table with Name + Actions only; row hover/selected states; uniform padding/row height; action column using shared icon buttons/toggles.
- Categories panel: table with Name/Items/Actions; row hover/selected states; compact action column; counts via shared badge.
- Items: render as compact nested list under each category row; small images/icons, tight padding; drag/drop preserved; no separate long cards below the table.
- Header: shared management header (eyebrow/title/subtext) with consistent typography and spacing.
- Empty/loading: shared empty state and skeletons for branches/menus/categories/items.
- Styling: white cards with border/shadow; shared tokens for colors/typography/spacing; WCAG AA contrast; focus states on interactive elements.

**Acceptance Criteria**
1) New components live in a dedicated folder; legacy menu styles/components are not modified.  
2) Layout wraps gracefully at 1920/1440/1366/mobile: no overflow, columns wrap instead of shrinking controls/text.  
3) Menus table shows only Name + Actions; Categories table shows Name/Items/Actions; rows have clear dividers, hover, and selected states.  
4) Item lists are nested under each category, compact, with consistent padding, small images/icons, and contained height.  
5) Actions/toggles use shared icon button sizes/styles; no ad-hoc icons.  
6) Shared empty states and skeletons present for each list.  
7) Tokens/typography match Branch/Categories scales; row heights and paddings are consistent.  
8) Accessibility: focus states on all interactive elements; WCAG AA contrast maintained.

**Definition of Done**
- New Menu components rendered on the Menus page with the above structure and responsiveness verified at desktop, 1440, 1080, and mobile widths.  
- No regressions to Branch/Categories screens; legacy menu code left intact.  
- Build passes and linting as applicable.
