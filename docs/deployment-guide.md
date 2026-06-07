# Deployment Guide

## Observed Hosting Model

- **Landing site:** Azure Static Web Apps
- **Angular dashboard:** Azure Static Web Apps style deployment with `staticwebapp.config.json`
- **Backend API:** Azure App Service at `https://menunestapi.azurewebsites.net`
- **API management metadata:** Azure API Management artifacts are present in service dependency files
- **Storage:** Azure Blob Storage
- **Realtime:** Azure SignalR
- **Data:** Azure Cosmos DB
- **Auth:** Firebase

## Relevant Files

- `MenuNestLandingPage/staticwebapp.config.json`
- `MenuNestWebsite/staticwebapp.config.json`
- `MenuNestWebsite/src/staticwebapp.config.json`
- `MenuNestWebsite/src/web.config`
- `MenuNestServer/MenuNestServer/MenuNestAPI/appsettings.json`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Properties/ServiceDependencies/**`

## Deployment Notes by Part

### Landing

- Static site with direct hosted URL references.
- Validate all linked app origins before release.

### Website

- Angular build output targets `dist/menunestwebsite`.
- Static hosting config is already included in source/build assets.
- Firebase and API environment values are compile-time source values today.

### Backend

- ASP.NET project includes Azure service dependency metadata.
- App settings include CORS, guest web base URL, blob storage, SignalR, and Firebase configuration.

### Mobile Suite

- Mobile apps contain Firebase config files for platform targets.
- Backend/API URLs are compiled into Dart services.

## Critical Operational Risks

1. Secrets are checked into source control.
2. Environment-specific URLs are hard-coded in multiple places.
3. There is no obvious central environment management layer across website, landing page, backend, and mobile suite.

## Recommended Hardening

1. Move all secrets to environment-specific secret stores or CI/CD variables.
2. Introduce environment configuration for API, SignalR, landing, guest, and dashboard URLs.
3. Remove or tightly control committed build artifacts such as `dist`.
4. Add explicit deployment runbooks per part once environment management is cleaned up.
