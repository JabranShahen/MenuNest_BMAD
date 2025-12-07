# MenuNest Web Style Guide

This document standardizes the UI tokens and usage guidance for the Angular web app. Use these variables and patterns for any new or refactored UI.

## Color Palette (CSS Variables)
Define in `src/styles.scss` (or a dedicated `_tokens.scss`) and import everywhere.

```scss
:root {
  /* Brand primaries */
  --color-primary: #00aef0;          /* Protoss Pylon */
  --color-primary-strong: #0097dc;   /* Vanadyl Blue */
  --color-accent: #36c852;           /* Skirret Green */
  --color-accent-strong: #1bb73f;    /* Download Progress */

  /* Secondary / highlight */
  --color-highlight: #f3b532;        /* Rise-N-Shine */
  --color-highlight-strong: #d8a120; /* Nanohanacha Gold */
  --color-secondary: #a58dff;        /* Periwinkle */
  --color-secondary-strong: #8a79e8; /* Matt Purple */

  /* Neutrals */
  --color-surface: #f5f6f9;          /* Lynx White */
  --color-surface-strong: #e6e8ec;   /* Muted surface */
  --color-border: #d7d9de;
  --color-text: #1f2430;             /* Neutral ink */
  --color-text-muted: #5a667a;

  /* Info / structural blues */
  --color-info: #3f75a6;             /* Seabreeze */
  --color-info-strong: #1d3c83;      /* Mazarine Blue */
  --color-info-dark: #0c1e4f;        /* Blue Navy */

  /* Critical */
  --color-danger: #e23d1d;           /* Nasturcian Flower */
  --color-danger-strong: #c53118;

  /* Overlays */
  --color-overlay: rgba(15, 23, 42, 0.5); /* Modal scrim */
}
```

### Usage
- Primary actions: `--color-primary`; hover/focus with `--color-primary-strong`.
- Success/positive accents: `--color-accent`; hover `--color-accent-strong`.
- Warnings/highlights: `--color-highlight`; darken on hover.
- Destructive: `--color-danger`; hover `--color-danger-strong`.
- Surfaces: cards/panels on `--color-surface`; alternate rows on `--color-surface-strong`.
- Text: default `--color-text`; secondary text `--color-text-muted`.

## Typography
Set in `styles.scss` or a typography partial.
- Base font: `Inter, "Segoe UI", system-ui, sans-serif`.
- Base size: 16px; line-height: 1.5.
- Headings: semibold (600) with tight spacing; avoid all-caps except buttons.
  - H1: 28–32px / 1.2
  - H2: 22–24px / 1.3
  - H3: 18–20px / 1.35
- Body: 16px regular; captions/helper: 13–14px.

## Spacing & Layout
- Base unit: 8px grid.
- Container max width: 1200–1280px; gutter 24px.
- Card padding: 16–20px; table cell padding: 12–14px.
- Section spacing: 24–32px between major blocks.

## Radius, Shadows, Borders
- Radius: 8px for cards/buttons/inputs; 12px for modals.
- Shadows: `0 8px 24px rgba(0, 0, 0, 0.08)` for cards; `0 12px 32px rgba(0, 0, 0, 0.12)` for modals.
- Borders: 1px solid `--color-border` on inputs, tables, and neutral dividers.

## Buttons
```scss
.btn {
  border-radius: 8px;
  font-weight: 600;
  padding: 10px 14px;
  border: 1px solid transparent;
  transition: transform 120ms ease, box-shadow 120ms ease;
}
.btn-primary {
  background: var(--color-primary);
  color: #fff;
}
.btn-primary:hover { background: var(--color-primary-strong); }
.btn-secondary {
  background: #fff;
  color: var(--color-text);
  border-color: var(--color-border);
}
.btn-ghost {
  background: transparent;
  color: var(--color-text);
}
.btn-danger {
  background: var(--color-danger);
  color: #fff;
}
.btn:focus-visible { outline: 2px solid var(--color-info); outline-offset: 2px; }
```

## Forms
- Inputs/selects: 1px border `--color-border`, 8px radius, 12px padding.
- Focus: 2px outline `--color-primary`.
- Labels: 14px semibold; helper/error text 12–13px.
- Required: append `*` in `--color-danger`.
- Errors: text `--color-danger`, border `--color-danger`.
- Success: optional subtle border `--color-accent`.

## Tables / Lists
- Header: semibold, background `--color-surface-strong`.
- Row hover: `--color-surface-strong`.
- Selected row: background `--color-info-dark`, text `#fff` (or `--color-info` + dark text if contrast suffices).
- Dense padding: 12px (rows), 14px (headers).
- Empty state: card with icon, short copy, and primary CTA.

## Cards / Panels
- Background: `--color-surface`.
- Border: `--color-border`.
- Shadow: light (`0 8px 24px rgba(0,0,0,0.08)`).
- Header: H3 + muted subtext; actions on the right with secondary/ghost buttons.

## Alerts / Toasts
- Success: background `rgba(54, 200, 82, 0.12)`, border `--color-accent`.
- Info: background `rgba(63, 117, 166, 0.12)`, border `--color-info`.
- Warning: background `rgba(243, 181, 50, 0.16)`, border `--color-highlight`.
- Error: background `rgba(226, 61, 29, 0.14)`, border `--color-danger`.
- Use icons with consistent sizing (16–20px) and bold titles.

## Modals
- Scrim: `--color-overlay`.
- Panel: `--color-surface`, radius 12px, padding 20–24px, shadow `0 12px 32px rgba(0,0,0,0.12)`.
- Primary action right-aligned; destructive actions in `--color-danger`.
- Close affordance with aria-label and focus outline.

## Icons & Affordances
- Icon size: 16–20px; consistent stroke weight.
- Icon buttons: 36–40px square, radius 8px; hover background `--color-surface-strong`.
- Always provide `aria-label` for icon-only buttons.

## States & Feedback
- Loading: spinners or skeletons for lists/tables.
- Success: snack bars/toasts with concise copy.
- Errors: inline for forms; toasts for operations; avoid console-only feedback.
- Disabled: reduce opacity and disable pointer events; maintain contrast for text.

## Accessibility
- Contrast: ensure text/background meets WCAG AA (adjust shades if needed).
- Focus: visible outlines on all interactive elements.
- Labels: associate form inputs; provide aria-labels on icon buttons.
- Keyboard: ensure tab order flows logically; modals trap focus and return on close.

## Theming Examples
- Primary page background: `--color-surface`.
- Sidebar/nav: `--color-info-strong` with white text; active item `--color-primary` or `--color-accent`.
- Primary buttons: `--color-primary`; Destructive: `--color-danger`; Secondary: white with border `--color-border`.
- Chips/pills: background `--color-surface-strong`, text `--color-text`; selected: background `--color-primary`, text white.

## Implementation Notes
- Centralize tokens in one SCSS partial and import into all component styles.
- Avoid hardcoded hex values in components; use CSS variables.
- Apply consistent spacing/radii/shadows per the tokens above.
- Audit existing screens (branch/menu/table management) for: button hierarchy, selected row styling, empty states, feedback toasts, and accessibility labels, then align to this guide.
