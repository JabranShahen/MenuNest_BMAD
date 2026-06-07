# Architecture: Backend

## Purpose

`MenuNestServer\\MenuNestServer` is the backend system of record for MenuNest. It exposes the API, validates Firebase tokens, manages domain entities, issues guest access and upload URLs, and publishes real-time updates.

## Architectural Style

- Layered ASP.NET Core solution
- Dependency injection through Autofac
- Generic data and entity manager abstractions
- Cosmos DB as the active persistence implementation
- SignalR for real-time entity notifications

## Startup Pipeline

`MenuNestAPI/Program.cs` configures:

- Controllers with camelCase JSON serialization
- CORS from `Cors:AllowedOrigins`
- SignalR with `/EntitySignalRHub`
- Firebase Admin initialization
- JWT bearer authentication using Firebase JWKS
- Autofac container registration

## Solution Layers

- `MenuNestAPI`: HTTP entry point and controllers
- `MenuNest.Abstractions`: domain entities
- `MenuNest.UseCases` and `MenuNest.Usecase.Implementation`: use case contracts and implementations
- `MenuNest.Services` and `MenuNest.Services.Abstractions`: runtime services
- `MenuNest.EntityManager`: decorators and entity orchestration
- `MenuNest.BootStrapper`: dependency registration
- `HosannaTech.*`: generic persistence and entity manager framework

## Key Capabilities

- Account, branch, category, menu, table, order, ticket, and user management
- Guest table access and token redemption
- Blob SAS URL generation
- Branding asset path validation
- Dashboard aggregation endpoints
- Real-time hub notifications

## Auth Model

- Default fallback policy requires authenticated users.
- Firebase ID tokens are validated by issuer, audience, lifetime, and signing key.
- Select endpoints are anonymous, notably login and guest table access redemption.

## Persistence Model

- Cosmos DB is accessed through `HosannaTech.implementations`.
- A singleton `DatabaseInstance` holds Cosmos client/database references.
- Domain entities derive from shared abstractions and are managed via generic entity managers.

## Real-Time Model

- `EntitySignalRHub` exposes a hub endpoint.
- `SignalREntityNotifier<T, THub>` integrates entity change notifications with SignalR.
- The shared Flutter package includes SignalR clients that subscribe to entity changes.

## Test Surface

- `MenuNestAPI.Tests` currently contains targeted controller tests around blob validation and account branding behavior.

## Risks

1. Sensitive credentials are committed in `appsettings.json` and infrastructure code.
2. Cosmos DB configuration still references `dukanz` names, suggesting inherited technical debt and incomplete rebranding.
3. Authentication and integration concerns are highly centralized in `Program.cs`, which increases startup complexity.
