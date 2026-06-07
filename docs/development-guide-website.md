# Development Guide: Website

## Prerequisites

- Node.js `22.x`
- npm `>=10`
- Angular CLI compatible with Angular 21

## Core Files

- `MenuNestWebsite/package.json`
- `MenuNestWebsite/angular.json`
- `MenuNestWebsite/src/app`
- `MenuNestWebsite/src/app/environments`

## Common Commands

Run from `MenuNestWebsite`:

```bash
npm install
npm start
npm run build
npm test
```

## Feature Development Workflow

1. Add or modify UI under `src/app/<feature>`.
2. Use the relevant service in `src/app/services`.
3. If a route is needed, update `app-routing.module.ts`.
4. If the feature is shared, prefer extending components under `src/app/shared`.

## Auth and API Notes

- Firebase is initialized in `app.module.ts`.
- The app expects a backend API at `https://menunestapi.azurewebsites.net/api`.
- Route protection flows through `AuthGuard`.

## Testing

- Karma/Jasmine unit tests are wired through `npm test`.
- Existing tests are present for routing, guards, services, and selected components.

## Cautions

- Avoid duplicating operational UI patterns when shared UI components already exist.
- Review environment and auth assumptions before changing hosted endpoints.
