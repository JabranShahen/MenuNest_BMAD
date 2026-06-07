# MenuNest Documentation Index

## Project Overview

- **Type:** Monorepo with 4 parts
- **Primary Languages:** TypeScript, C#, Dart, HTML/CSS/JS
- **Architecture:** Multi-client platform around a shared backend API and realtime hub

## Quick Reference

### Landing

- **Type:** Web
- **Root:** `MenuNestLandingPage`
- **Role:** Marketing site and hosted app handoff

### Website

- **Type:** Web
- **Root:** `MenuNestWebsite`
- **Role:** Restaurant operator dashboard

### Backend

- **Type:** Backend
- **Root:** `MenuNestServer\\MenuNestServer`
- **Role:** Core API, auth validation, persistence, realtime, guest access

### Mobile Suite

- **Type:** Mobile
- **Root:** `MenuNestApp\\menunest`
- **Role:** Admin, table, kitchen, and service apps

## Generated Documentation

- [Project Overview](./project-overview.md)
- [Source Tree Analysis](./source-tree-analysis.md)
- [Architecture - Landing](./architecture-landing.md)
- [Architecture - Website](./architecture-website.md)
- [Architecture - Backend](./architecture-backend.md)
- [Architecture - Mobile Suite](./architecture-mobile-suite.md)
- [Component Inventory - Website](./component-inventory-website.md)
- [Component Inventory - Mobile Suite](./component-inventory-mobile-suite.md)
- [Development Guide - Landing](./development-guide-landing.md)
- [Development Guide - Website](./development-guide-website.md)
- [Development Guide - Backend](./development-guide-backend.md)
- [Development Guide - Mobile Suite](./development-guide-mobile-suite.md)
- [API Contracts - Backend](./api-contracts-backend.md)
- [Data Models - Backend](./data-models-backend.md)
- [Integration Architecture](./integration-architecture.md)
- [Deployment Guide](./deployment-guide.md)
- [Project Parts Metadata](./project-parts.json)
- [Project Scan Report](./project-scan-report.json)

## Existing Documentation and Signals

- `../MenuNestWebsite/README.md`
- `../MenuNestApp/README.md`
- Flutter app `README.md` files under `MenuNestApp/menunest/*`
- `../MenuNestLandingPage/tests/landing-page-smoke.ps1`
- Azure service dependency files under `../MenuNestServer/MenuNestServer/MenuNestAPI/Properties/ServiceDependencies`

## Getting Started

For brownfield planning or implementation:

1. Start with [Project Overview](./project-overview.md).
2. Read the relevant architecture file for the part you are touching.
3. Use [Integration Architecture](./integration-architecture.md) before making cross-part changes.
4. For backend contract work, consult [API Contracts - Backend](./api-contracts-backend.md) and [Data Models - Backend](./data-models-backend.md).
5. Use the per-part development guide for setup and execution commands.
