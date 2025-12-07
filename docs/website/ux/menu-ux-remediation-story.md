# Story: Menus Screen UX Remediation

**Goal**  
Bring the Menus screen in line with the established Branch/Categories patterns and the shared UI kit so users get a clean hierarchy, predictable spacing, and accessible, readable controls.

**Context (current screen snapshot)**  
Three columns (Branches, Menus, Menu Categories) on a mid-gray background with oversized toggles/plus icons, cramped padding, and inconsistent styling vs Branch/Categories.

**Issues Observed**
- Visual hierarchy: Mid-gray panels and lack of card borders make the page feel flat and inconsistent with Branch/Categories (which use white cards with subtle shadow/border).
- Layout & spacing: Tight horizontal padding inside columns; rows touch edges; uneven vertical rhythm; content left-heavy with large empty areas elsewhere.
- List presentation: Rows lack clear dividers/hover/selected states; branch/menu titles don’t align on a grid; empty-state messages are small and misaligned.
- Controls: Toggles and plus icons are oversized; action icons differ from shared icon-button style; primary CTAs don’t use the shared button.
- Typography & color: Inconsistent token usage for text colors/backgrounds; titles lack the shared heading style; muted text on gray reduces contrast.
- Empty/feedback: Empty states are minimal and not using the shared empty-state component; no inline feedback/affordance for loading/error states.
- Responsiveness: Columns don’t scale gracefully; controls risk overflow on smaller widths.

**Acceptance Criteria**
1) Apply shared shell  
   - Use shared management header (eyebrow/title/subtext) and panel/card from the UI kit; background white, with subtle border + shadow; no large gray canvases.  
2) Grids and spacing  
   - Table/list areas use the shared table shell: clear row dividers, consistent padding (12–16px), hover and selected states, fixed action column width.  
   - Columns align on a consistent gutter; content not flush to edges; avoids large unused gray areas.  
3) Controls and icons  
   - Use shared primary/secondary/icon buttons for Add Menu and per-row actions.  
   - Status toggles and “add category” affordance sized to match the shared kit (no oversized icons); icons use the shared tokenized colors.  
4) Typography and tokens  
   - Headings, labels, and body text use the same sizes/weights/colors as Branch/Categories; backgrounds/text follow the design tokens (no ad-hoc grays/blues).  
5) Empty/feedback states  
   - Each column shows the shared empty-state component with descriptive text and CTA guidance when empty; loading states use the shared skeleton if applicable.  
6) Responsiveness  
   - Layout adapts on medium/narrow viewports: columns stack or resize without control overflow; padding remains consistent.  
7) Accessibility  
   - Interactive elements have focus states and aria labels matching the shared kit; color contrast meets WCAG AA for text/icons.

**Definition of Done**
- Menus page visually matches Branch/Categories patterns (white cards, borders, consistent spacing, shared buttons/icon buttons/toggles).  
- No oversized toggles/plus icons; action controls match shared sizing.  
- Empty, loading, and selected/hover states present and consistent.  
- Uses only shared tokens/components for header/panel/table/buttons/empty states; no leftover one-off styles for buttons/toggles/icons.
