# Development Guide: Landing

## Purpose

This part is a static marketing site. Most changes will be copy, layout, branding, CTA, or hosted URL updates.

## Working Files

- `MenuNestLandingPage/index.html`
- `MenuNestLandingPage/assets/css`
- `MenuNestLandingPage/assets/js`
- `MenuNestLandingPage/images`
- `MenuNestLandingPage/tests/landing-page-smoke.ps1`

## Typical Tasks

- Update brand positioning or product copy
- Adjust CTA behavior
- Replace images or brand assets
- Modify static theme styles
- Validate hosted URLs referenced by the landing page

## Validation

- Open `index.html` in a browser or static host
- Run `MenuNestLandingPage/tests/landing-page-smoke.ps1`
- Confirm outbound URLs still match dashboard and auth status hosting

## Cautions

- Keep hosted URLs consistent with dashboard and backend deployment state.
- This page is not componentized, so make small changes carefully.
