# Architecture: Landing

## Purpose

`MenuNestLandingPage` is the public marketing surface for MenuNest. It introduces the product, communicates the restaurant operations narrative, and routes users into the hosted application experience.

## Style

- Static HTML page
- Bundled CSS and vendor JS assets
- No framework-level component system
- Hosted as a static web app

## Main Entry Points

- `index.html`
- `assets/css/style.css`
- `assets/js/script.js`
- `staticwebapp.config.json`

## Structural Notes

- The page is monolithic and content-heavy, with branding, value proposition, product explanation, and CTA sections in a single HTML document.
- CTA behavior is driven by embedded configuration values in the page.
- There is a small smoke test in `tests/landing-page-smoke.ps1` validating key strings and hosted URL wiring.

## External Dependencies

- Hosted dashboard URL
- Hosted auth status page
- Static asset bundle and images

## Risks

1. Runtime environment URLs are embedded directly in markup/script configuration.
2. The page has low modularity, so design or content changes are more likely to create accidental regressions.
3. The site appears to depend on external dashboard/auth origins that may diverge from backend configuration.
