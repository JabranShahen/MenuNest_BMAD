# Development Guide: Backend

## Prerequisites

- .NET SDK compatible with the solution projects
- Access to Azure dependencies if running against live services
- Firebase project credentials if auth flows need to work end-to-end

## Core Files

- `MenuNestServer/MenuNestServer/MenuNest.sln`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Program.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI/appsettings.json`
- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities`

## Common Commands

Run from `MenuNestServer/MenuNestServer`:

```bash
dotnet restore
dotnet build MenuNest.sln
dotnet test MenuNestAPI.Tests/MenuNestAPI.Tests.csproj
dotnet run --project MenuNestAPI/MenuNestAPI.csproj
```

## Where to Make Changes

- New endpoint: `MenuNestAPI/Controllers`
- Domain entity change: `MenuNest.Abstractions/Entities`
- Use case or orchestration logic: `MenuNest.Usecase.Implementation` or `MenuNest.EntityManager`
- Dependency wiring: `MenuNest.BootStrapper`
- Persistence behavior: `HosannaTech.implementations`

## Auth and Security Notes

- Default authorization is global unless endpoints explicitly opt out.
- Firebase bearer validation is configured in startup.
- Guest table access is handled in `TableAccessController`.

## Testing

- Existing tests focus on controller validation and branding behavior.
- New API work should extend `MenuNestAPI.Tests` where practical.

## Cautions

- Secrets currently live in checked-in config and should be externalized before serious environment work.
- Changes to shared domain entities should be coordinated with the Flutter shared package.
