# Story: MenuNest OPS — Marketing Prospects Dashboard

Status: review

---

## Story

As a MenuNest team member,
I want to view, search, and update restaurant marketing prospects in a web dashboard,
so that I can manage the outreach pipeline for restaurants extracted by the automated FHRS pipeline tool.

---

## Acceptance Criteria

1. **Backend — OCI config in appsettings**
   - `appsettings.json` contains an `OracleNoSql` placeholder section with empty string values (safe to commit)
   - `appsettings.Development.json` contains the real OCI credential values
   - The server connects to Oracle NoSQL using inline IAM credentials (no file path reference)
   - The `Oracle.NoSQL.SDK` NuGet package is added to `MenuNestAPI.csproj`

2. **Backend — GET /api/marketing/prospects**
   - Returns paginated list of prospects from `MenuNestMarketingProspects` table
   - Supports query params: `page` (default 1), `pageSize` (default 50), `tier` (int or omit for all), `search` (business name substring)
   - Response is a clean JSON object: `{ items: [...], totalCount, page, pageSize }`
   - Endpoint is protected by `[Authorize]` — Firebase JWT required

3. **Backend — GET /api/marketing/prospects/{id}**
   - Returns single prospect by primary key `id`
   - Returns 404 if not found
   - Endpoint is protected by `[Authorize]`

4. **Backend — PATCH /api/marketing/prospects/{id}/stage**
   - Accepts body `{ "stage": "contacted" }` and updates `pipeline_stage` in the `data` JSON blob
   - Valid stage values: `new`, `contacted`, `demo_sent`, `converted`, `rejected`
   - Returns 400 for invalid stage value
   - Returns 404 if prospect not found
   - Endpoint is protected by `[Authorize]`

5. **Backend — CORS**
   - `http://localhost:4201` is added to `AllowedOrigins` in `appsettings.json`

6. **Frontend — Angular app scaffolded**
   - New Angular 21 project exists at `MenuNest_OPS/menunest-ops/`
   - `ng serve` runs on port 4201
   - Angular Material ^21.2.0 and `@angular/fire ^21.0.0-rc.0` are installed
   - Firebase config for `menunest_ops` app is set in `environment.ts`
   - `environment.prod.ts` has placeholder API URL

7. **Frontend — Auth**
   - Login page at `/login` with email/password form using Firebase `signInWithEmailAndPassword`
   - All other routes are guarded — unauthenticated users redirected to `/login`
   - HTTP interceptor attaches Firebase ID token as `Authorization: Bearer <token>` on every API request
   - Token is refreshed automatically on expiry (Firebase SDK handles this)

8. **Frontend — Shell layout**
   - Fixed left nav sidebar showing: Dashboard (disabled placeholder), Marketing (active link)
   - Right content area renders the active route via `<router-outlet>`
   - Visual style matches MenuNestWebsite: primary colour `#162770`, Roboto font, Angular Material theme

9. **Frontend — Prospects list**
   - Route `/marketing/prospects` shows the prospects list
   - Filter bar with: text search (business name), tier select (All / Tier 1 Fast-Track / Tier 2 Standard), cuisine select
   - `mat-table` with columns: Business Name, City, Cuisine, Tier (chip), Website (link), Contact Email, Pipeline Stage, Agent Reason
   - Tier 1 chip is green (`#4caf50`), Tier 2 chip is blue (`#2196f3`)
   - `mat-paginator` with page size 50, shows total count from API
   - Clicking a row opens the detail panel

10. **Frontend — Prospect detail panel**
    - Appears alongside (or below) the table on row click
    - Shows: full address, website + menu URL as `<a>` links, contact email, agent confidence (%), agent reason
    - Pipeline stage `mat-select` with options: New, Contacted, Demo Sent, Converted, Rejected
    - Save button calls `PATCH /api/marketing/prospects/{id}/stage` and shows snackbar on success/failure
    - Panel closes when another row is clicked or X is pressed

---

## Tasks / Subtasks

### Backend

- [x] **Task 1 — Add Oracle.NoSQL.SDK NuGet package** (AC: 1)
  - [x] Add `<PackageReference Include="Oracle.NoSQL.SDK" Version="5.*" />` to `MenuNestAPI.csproj`
  - [x] Verify build succeeds

- [x] **Task 2 — OCI credentials in appsettings** (AC: 1)
  - [x] Add `OracleNoSql` placeholder section to `appsettings.json` (empty strings)
  - [x] Add real values to `appsettings.Development.json` (see Dev Notes for exact values)
  - [x] Verify `appsettings.Development.json` is in `.gitignore`

- [x] **Task 3 — MarketingProspectDto** (AC: 2, 3)
  - [x] Create `MenuNestAPI/Models/MarketingProspectDto.cs` with all prospect fields as camelCase properties

- [x] **Task 4 — IMarketingProspectService interface + implementation** (AC: 2, 3, 4)
  - [x] Create interface in `MenuNest.Services.Abstractions` or directly in API project
  - [x] Implement `MarketingProspectService` that creates `NoSQLClient` from `IConfiguration`
  - [x] `GetProspectsAsync(page, pageSize, tier?, search?)` — query Oracle, map to DTOs
  - [x] `GetByIdAsync(id)` — single record lookup
  - [x] `UpdateStageAsync(id, stage)` — read record, update `data.pipeline_stage`, upsert back
  - [x] Register in Autofac (see Dev Notes for registration pattern)

- [x] **Task 5 — MarketingProspectsController** (AC: 2, 3, 4)
  - [x] Create `MenuNestAPI/Controllers/MarketingProspectsController.cs`
  - [x] `[Authorize]` on controller class
  - [x] `GET /api/marketing/prospects` with query param binding
  - [x] `GET /api/marketing/prospects/{id}`
  - [x] `PATCH /api/marketing/prospects/{id}/stage`

- [x] **Task 6 — CORS** (AC: 5)
  - [x] Add `http://localhost:4201` to `AllowedOrigins` array in `appsettings.json`

### Frontend

- [x] **Task 7 — Scaffold Angular project** (AC: 6)
  - [x] `ng new menunest-ops --routing --style=scss` in `MenuNest_OPS/`
  - [x] Install: `@angular/material`, `@angular/cdk`, `@angular/fire`
  - [x] Set `"port": 4201` in `angular.json` serve options
  - [x] Set up Angular Material theme (see Dev Notes)
  - [x] Add Roboto font to `index.html`

- [x] **Task 8 — Environment files** (AC: 6)
  - [x] Populate `environment.ts` with Firebase config + `apiUrl: 'http://localhost:5000'`
  - [x] Populate `environment.prod.ts` with same Firebase config + `apiUrl: ''` (placeholder)

- [x] **Task 9 — Firebase auth setup** (AC: 7)
  - [x] Initialise `@angular/fire` in `app.config.ts` (or `app.module.ts`)
  - [x] Create `AuthService` wrapping `signInWithEmailAndPassword`, `signOut`, `authState$`
  - [x] Create `AuthGuard` — redirects to `/login` if not authenticated
  - [x] Create `LoginComponent` at `/login` — email/password form, shows error on failure
  - [x] Create `AuthInterceptor` — gets ID token from Firebase, sets `Authorization: Bearer`

- [x] **Task 10 — Shell layout** (AC: 8)
  - [x] Create `ShellComponent` with `mat-sidenav-container`
  - [x] Left nav: `mat-nav-list` with Dashboard (disabled) and Marketing items
  - [x] Apply primary colour `#162770` to sidenav and toolbar
  - [x] Right content: `<router-outlet>`

- [x] **Task 11 — Prospects service** (AC: 9, 10)
  - [x] Create `ProspectsService` with `getProspects(params)`, `getById(id)`, `updateStage(id, stage)`
  - [x] Typed response interfaces matching backend DTOs

- [x] **Task 12 — Prospects list component** (AC: 9)
  - [x] Route `/marketing/prospects` → `ProspectsListComponent`
  - [x] Filter bar: search input, tier select, cuisine select
  - [x] `mat-table` with defined columns
  - [x] Tier chip styling (green/blue)
  - [x] `mat-paginator` wired to API pagination
  - [x] Row click emits selected prospect

- [x] **Task 13 — Prospect detail panel** (AC: 10)
  - [x] `ProspectDetailComponent` rendered conditionally alongside table
  - [x] Displays all prospect fields
  - [x] Stage `mat-select` + Save button
  - [x] On save: call `updateStage`, show `MatSnackBar` success/error
  - [x] Close button clears selection

---

## Dev Notes

### Backend — Exact File Locations

```
MenuNestServer/MenuNestServer/
  MenuNestAPI/
    Controllers/
      MarketingProspectsController.cs       ← NEW
    Models/
      MarketingProspectDto.cs               ← NEW
      ProspectsPagedResponse.cs             ← NEW
      UpdateStageRequest.cs                 ← NEW
    MenuNestAPI.csproj                      ← ADD Oracle.NoSQL.SDK
    appsettings.json                        ← ADD OracleNoSql placeholders + CORS origin
    appsettings.Development.json            ← ADD real OCI values
  MenuNest.Services/ (or keep in API project)
    MarketingProspectService.cs             ← NEW
    IMarketingProspectService.cs            ← NEW
```

### Backend — appsettings.json additions

Add this section (empty placeholders, safe to commit):
```json
"OracleNoSql": {
  "UserId": "",
  "TenantId": "",
  "Fingerprint": "",
  "Region": "uk-london-1",
  "PrivateKeyPem": ""
}
```

Add to existing `Cors.AllowedOrigins` array:
```json
"http://localhost:4201"
```

### Backend — appsettings.Development.json additions

```json
"OracleNoSql": {
  "UserId": "ocid1.user.oc1..aaaaaaaaahji4meh7nte667bidw63mnph6mdchqmxwxr437ivzs6tsyqcgna",
  "TenantId": "ocid1.tenancy.oc1..aaaaaaaazlhrwrs335jgiz57cdmxphuch3a6mnrfjjlyxev73ve3gp5wznha",
  "Fingerprint": "d0:5b:01:b5:17:3c:a6:a0:a4:cb:47:38:04:95:a2:6e",
  "Region": "uk-london-1",
  "PrivateKeyPem": "-----BEGIN PRIVATE KEY-----\nMIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQDBhqvhP40i3gIW\nV6jrxslOUUUFl29vdPzq9RedHSFDkJfxz/nEIBa7oUt3S+ShzsN7CUPsNPQnwgRN\n+N6G3FJllkEZOiLJoxNSuNDNDz12L+mwi9A3ym3ZtF7V/Ceb2pQbsLSeH6fn7/FS\nenX97zDF6iZ7sp9Co2xyJukVsuz1En1LDPhKegkLnrSpio7LXK5pezVqCUVOwg+7\nPFjPsC2dPob7BWXbcvHkGGTejEMXXNtnrqE4o5AE3KeQTnyPODrzuIDgg1i51gHU\nppHX7weDJmazVaQiFHg18pxP07Gx6UVrrbWvl3lhs7GqkdG7Aps3w8jTM965gBG5\nCIg9N/ULAgMBAAECggEABXFXVyvq476mWguCwt13ND6BOEbfNEga2Yl6wwlFMg8Y\nfuKwWG2Qm3FOBopkto+dRtIRY3dvyx3KnN4cIA3akWSC6zCL4ZY2XmlikfR/mtnT\nyDUpGNCW0KchKFfW+Kv9O17NDZ9qoPwfZ4hDK5j1Cwk/+32gELHjmb2cvjv8gb3u\nnOOLGA6SV0UDlkLINlZgVfzqEEgXvbPoceDI3Q0i/zX0Du3f6qJEiRrisb1gtITP\nA6pp1KKGJRSPoZI+i/MyQ68xljMS5J+toWM/EhCH+ChBlombPJF+O3WDwZlSX+Bl\npD7lE46LCWYDDOplHup8T19dsTv1Jbp9waofWHPD3QKBgQD5NfmNMMJd2yaFCN1H\nzna93Tz18Q00OqnrnttZBPuftqoAecBki1hYqh+TGGecm2Ea2UpPeclwFjJvTGcL\nUMNMxsf4vZpg699hyn9RWJ2In0zA9jxAvJiUQg5OQ0l9+WSHTAzElo69wlIzNZfN\nsOHCvhe2caBR9hWQ5Z9+vwYOTwKBgQDGzFgx9LP6qEKj40shkC4jm/j68uLsmgLe\nw6o1LAuESsYqNGz1NHw3Nb8pPCruZevAfYbPqsGSUWlIJmJAlPPNEQc53N0Crhne\nRumLLX9HYHNjp67SC6K3EwP3IixE0rs1b0EzbNamSaeqFDM1+WCKzvM+oJbOnkok\nw/bAT1mahQKBgBd9F91P6DHycun9EOYwto5kqNdBdg4jLVrQ6Tm1t4WxMMrErvaL\nD7OjrUAu/60KFBf0vQVKpErVPMGywM+XOCEnZzexnzhdYvuTm3ZuVMLIyPIzAzDS\n1cq7gx+rReUCuY/rAhURX7jQ9PBwr7MqZcz2H8QJZ6Px/sxeaC8JECgPAoGAYP7C\n6Vzjk6EVIrF7rtySJn2rdYWcgqSCUf5Vxau/0sRI+76oitsY4DcxFgVtTPQdmsWk\nSR6fY6ylGbbgqXIDokJ0rB6/Fterd3BR8r44I7NDmZPvEDztHzX/8UyTHOFUxjWK\nMnUgJfI6BBnnAqayHAftVtkzu4wv0NBsTFhq96ECgYBi3bsute9sZuM/N+G7iP+D\nWpW54fjLtczT1Jimvot8aK9cZze/xEM915tXsl23m0tz8z+zdbVkEhrGcj5Zz069\nJzeeWjOajqqY2whhmHB4IKwF/Sl6MeLHTjF5W4BDX4eA1cgUMffwetJHAyMdhNy6\nXN8UQcXaBUmjxTH7URo4Ew==\n-----END PRIVATE KEY-----"
}
```

### Backend — IAMAuthorizationProvider construction (no file path)

```csharp
// In MarketingProspectService constructor, inject IConfiguration:
var credentials = new IAMCredentials
{
    UserId      = config["OracleNoSql:UserId"]!,
    TenantId    = config["OracleNoSql:TenantId"]!,
    Fingerprint = config["OracleNoSql:Fingerprint"]!,
    PrivateKeyPEM = config["OracleNoSql:PrivateKeyPem"]!
};

_client = new NoSQLClient(new NoSQLConfig
{
    Region = Oracle.NoSQL.SDK.Region.UK_LONDON_1,
    AuthorizationProvider = new IAMAuthorizationProvider(credentials)
});
```

**IMPORTANT:** `PrivateKeyPEM` is the property name in Oracle.NoSQL.SDK — not `PrivateKey` or `PrivateKeyPem`. Verify against the installed SDK version.

### Backend — Oracle NoSQL query pattern

The existing `OracleNoSqlService.cs` in `MenuNest_dev_ops` is the reference implementation. Key patterns:

```csharp
// Table name
private const string TableName = "MenuNestMarketingProspects";

// Paginated query — Oracle NoSQL does not have native OFFSET/LIMIT pagination
// Use GetQueryAsyncEnumerable and collect rows, then slice in memory for now
// (with 252k records, consider cursor-based pagination in future)

// Reading the data JSON blob — all prospect fields are nested under "data"
// row["fhrs_id"] and row["postcode"] are top-level STRING columns
// row["data"] is a MAP (JSON) containing all other fields

// SQL query example:
$"SELECT * FROM {TableName} WHERE data.prospect_tier = {tier}"
$"SELECT * FROM {TableName} WHERE CONTAINS(data.business_name, '{search}')"

// Escaping: always use EscapeSql (replace ' with '') on any user input
```

Reference file: `MenuNest_dev_ops/MenuNestDevOps/OracleNoSqlService.cs`

### Backend — MapFromRow pattern

All field values are read using:
```csharp
private static string? ReadString(MapValue row, string key) =>
    row.TryGetValue(key, out var value) && value.DbType == DbType.String
        ? value.AsString : null;

private static MapValue? ReadMap(MapValue row, string key) =>
    row.TryGetValue(key, out var value) && value.DbType == DbType.Map
        ? value.AsMapValue : null;
```

The `data` field is a `MapValue` — always read via `ReadMap(row, "data")` first, then read individual fields from that map.

### Backend — Autofac registration pattern

Look at `MenuNest.BootStrapper/AutoFacBootstrapper.cs` for existing registration patterns. Add:
```csharp
builder.RegisterType<MarketingProspectService>()
       .As<IMarketingProspectService>()
       .InstancePerLifetimeScope();
```

The `IConfiguration` is already available in the Autofac container via `builder.RegisterInstance(configuration)` or via the standard `IConfiguration` binding — check existing bootstrapper to confirm pattern.

### Backend — Controller structure (follow existing pattern)

Look at any existing controller e.g. `MenuController.cs` for the class-level pattern:
```csharp
[ApiController]
[Authorize]
[Route("api/marketing")]
public class MarketingProspectsController : ControllerBase
{
    private readonly IMarketingProspectService _service;
    public MarketingProspectsController(IMarketingProspectService service)
        => _service = service;
}
```

### Backend — UpdateStage implementation

Oracle NoSQL has no partial update for JSON fields. To update `pipeline_stage`:
1. Query the existing record by `id`
2. Deserialise the `data` map
3. Set `data["pipeline_stage"] = newStage`
4. Call `_client.PutAsync(TableName, row)` to upsert the entire row

Use the same `MapToRow` pattern from the reference `OracleNoSqlService.cs`.

### Frontend — Project location

```
MenuNest_OPS/
  menunest-ops/          ← Angular CLI project root
    src/
      app/
        core/
          auth/
            auth.service.ts
            auth.guard.ts
            auth.interceptor.ts
          services/
            prospects.service.ts
        features/
          marketing/
            prospects-list/
            prospect-detail/
        shell/
          shell.component.ts   ← left nav + router-outlet
        login/
          login.component.ts
      environments/
        environment.ts
        environment.prod.ts
    angular.json
    package.json
```

### Frontend — environment.ts

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:5000',
  firebase: {
    apiKey: 'AIzaSyAs_tD-guQNdWVCjrmDVx_K_QGln7IgIvA',
    authDomain: 'menu-nest.firebaseapp.com',
    projectId: 'menu-nest',
    storageBucket: 'menu-nest.firebasestorage.app',
    messagingSenderId: '561518239623',
    appId: '1:561518239623:web:955c38b2e8c815570a7d22'
  }
};
```

### Frontend — Angular Material theme

Use a custom theme in `styles.scss` matching MenuNestWebsite:
```scss
@use '@angular/material' as mat;

$primary: mat.define-palette(mat.$indigo-palette, 900);  // closest to #162770
$accent:  mat.define-palette(mat.$blue-palette, 600);
$theme:   mat.define-light-theme((
  color: (primary: $primary, accent: $accent)
));
@include mat.all-component-themes($theme);

body { font-family: Roboto, sans-serif; }
```

In `index.html`:
```html
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

### Frontend — angular.json port

In `angular.json` under `projects.menunest-ops.architect.serve.options`:
```json
"port": 4201
```

### Frontend — Firebase auth interceptor

```typescript
// auth.interceptor.ts
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private auth: Auth) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return from(getIdToken(this.auth.currentUser!)).pipe(
      switchMap(token => {
        const authReq = req.clone({
          setHeaders: { Authorization: `Bearer ${token}` }
        });
        return next.handle(authReq);
      })
    );
  }
}
```

Register in `app.config.ts`:
```typescript
provideHttpClient(withInterceptors([authInterceptor]))
// or use class-based: HTTP_INTERCEPTORS provider
```

### Frontend — Prospects API interface

```typescript
export interface MarketingProspect {
  id: string;
  fhrsId: string;
  businessName: string;
  addressLine1: string;
  addressLine2: string;
  city: string;
  county: string;
  postcode: string;
  website: string | null;
  menuUrl: string | null;
  contactEmail: string | null;
  prospectTier: number;        // 1 or 2
  cuisineType: string;
  agentConfidence: number;     // 0.0 - 1.0
  agentReason: string | null;
  pipelineStage: string;       // new | contacted | demo_sent | converted | rejected
  createdAt: string;
  updatedAt: string;
}

export interface ProspectsPagedResponse {
  items: MarketingProspect[];
  totalCount: number;
  page: number;
  pageSize: number;
}
```

### Frontend — Shell left nav structure

```html
<mat-sidenav-container>
  <mat-sidenav mode="side" opened>
    <div class="nav-header">MenuNest OPS</div>
    <mat-nav-list>
      <a mat-list-item disabled>Dashboard</a>
      <a mat-list-item routerLink="/marketing/prospects" routerLinkActive="active">
        Marketing
      </a>
    </mat-nav-list>
  </mat-sidenav>
  <mat-sidenav-content>
    <router-outlet></router-outlet>
  </mat-sidenav-content>
</mat-sidenav-container>
```

### Frontend — Pipeline stage display values

| Value | Display label |
|---|---|
| `new` | New |
| `contacted` | Contacted |
| `demo_sent` | Demo Sent |
| `converted` | Converted |
| `rejected` | Rejected |

### Key Constraints / Gotchas

1. **Oracle NoSQL SDK version** — check the version already used in `MenuNest_dev_ops/MenuNestDevOps/MenuNestDevOps.csproj` and use the same version in `MenuNestAPI.csproj` to avoid conflicts.

2. **`PrivateKeyPEM` property name** — this is the exact property name on `IAMCredentials`. Do not use `PrivateKey`, `PrivateKeyFile`, or `PrivateKeyPem` (case-sensitive).

3. **Oracle NoSQL does not support native pagination with OFFSET** — for the initial implementation, collect all matching rows and slice in memory. With 252k total records but filtered sets typically much smaller, this is acceptable. Add a comment noting this should be revisited with cursor-based pagination.

4. **`appsettings.Development.json` is NOT in `.gitignore` by default** in ASP.NET projects — verify it's excluded before committing. Check `.gitignore` in `MenuNestServer/MenuNestServer/` and add if missing.

5. **Angular 21 uses standalone components by default** — `ng new` with Angular 21 generates standalone components. Do NOT use `NgModule` pattern unless following the existing MenuNestWebsite pattern exactly. Check MenuNestWebsite's `app.module.ts` — if it uses modules, match that; if it uses standalone, match that.

6. **`@angular/fire` v21 uses modular API** — use `provideFirebaseApp(() => initializeApp(config))`, `provideAuth(() => getAuth())`. Do NOT use AngularFireAuth (legacy).

7. **Autofac and IConfiguration** — look at how existing services in `MenuNest.Services` receive `IConfiguration`. If no existing pattern, register it explicitly in `AutoFacBootstrapper.cs`.

8. **MenuNestServer port** — check `Properties/launchSettings.json` in `MenuNestAPI` for the local dev port. Default is likely `5000` (HTTP) or `5001` (HTTPS). Use `http://localhost:5000` in Angular environment.

---

## Project Structure Notes

- **Backend project root**: `MenuNestServer/MenuNestServer/MenuNestAPI/`
- **Frontend project root**: `MenuNest_OPS/menunest-ops/` (to be created)
- **Reference OracleNoSqlService**: `MenuNest_dev_ops/MenuNestDevOps/OracleNoSqlService.cs` — copy mapping patterns directly
- **Reference Angular app**: `MenuNestWebsite/src/app/` — follow component, service, guard patterns exactly

---

## References

- Reference Oracle service: `MenuNest_dev_ops/MenuNestDevOps/OracleNoSqlService.cs`
- Reference MarketingProspect model: `MenuNest_dev_ops/MenuNestDevOps/MarketingProspect.cs`
- Reference Angular app structure: `MenuNestWebsite/src/app/`
- Reference appsettings pattern: `MenuNestAPI/appsettings.json` (Firebase section)
- Reference Autofac DI: `MenuNest.BootStrapper/AutoFacBootstrapper.cs`
- Reference controller pattern: `MenuNestAPI/Controllers/MenuController.cs`

---

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Completion Notes List

- All 13 tasks implemented. 51 backend tests pass (0 failures). Angular build clean.
- Oracle.NoSQL.SDK 5.2.2 added to MenuNestAPI.csproj (matches version in dev_ops project).
- appsettings.Development.json was NOT in .gitignore — added it. This was a pre-existing gap.
- IMarketingProspectService and MarketingProspectService kept in API project (not MenuNest.Services) to avoid adding Oracle dependency to Services project. Registered via AutofacContainerBuilder in Program.cs.
- Fixed pre-existing compile error in Program.cs: missing `;` after AddJwtBearer chain (line ~190).
- Angular Material v21 removed mat.define-palette API; switched to prebuilt indigo-pink theme.
- Angular 21 required TypeScript >=5.9.0; upgraded from 5.7.3 in package.json.
- MapFromRow parameter changed from RecordValue → MapValue so unit tests can construct test rows via FieldValue.FromJsonString() (which returns MapValue, not RecordValue; RecordValue IS-A MapValue).
- IsValidStage and MapFromRow made public static to allow unit testing without InternalsVisibleTo.
- Controller delegates to MarketingProspectService.IsValidStage() — no duplicated ValidStages HashSet.

### File List

**Backend (MenuNestServer/MenuNestServer/)**
- `MenuNestAPI/MenuNestAPI.csproj` — added Oracle.NoSQL.SDK 5.2.2
- `MenuNestAPI/appsettings.json` — added OracleNoSql placeholder section + CORS origin 4201
- `MenuNestAPI/appsettings.Development.json` — added OracleNoSql real values
- `.gitignore` — added appsettings.Development.json exclusion
- `MenuNestAPI/Program.cs` — added using MenuNest.Services; registered MarketingProspectService; fixed missing semicolon bug
- `MenuNestAPI/Models/MarketingProspectDto.cs` — NEW
- `MenuNestAPI/Models/ProspectsPagedResponse.cs` — NEW
- `MenuNestAPI/Models/UpdateStageRequest.cs` — NEW
- `MenuNestAPI/Services/IMarketingProspectService.cs` — NEW
- `MenuNestAPI/Services/MarketingProspectService.cs` — NEW
- `MenuNestAPI/Controllers/MarketingProspectsController.cs` — NEW
- `MenuNestAPI.Tests/MenuNestAPI.Tests.csproj` — added Oracle.NoSQL.SDK 5.2.2
- `MenuNestAPI.Tests/MarketingProspectsControllerTests.cs` — NEW (17 tests)
- `MenuNestAPI.Tests/MarketingProspectServiceTests.cs` — NEW (21 tests)

**Frontend (MenuNest_OPS/menunest-ops/)**
- `package.json` — Angular 21 packages, @angular/fire, @angular/material, TypeScript 5.9
- `angular.json` — port 4201; prebuilt indigo-pink theme added to styles
- `src/index.html` — Roboto + Material Icons Google Fonts links
- `src/styles.scss` — global body/html styles
- `src/environments/environment.ts` — NEW
- `src/environments/environment.prod.ts` — NEW
- `src/app/app.config.ts` — Firebase + HTTP interceptor providers
- `src/app/app.routes.ts` — routing with auth guard and lazy-loaded components
- `src/app/app.component.ts` — simplified to router-outlet only
- `src/app/core/auth/auth.service.ts` — NEW
- `src/app/core/auth/auth.guard.ts` — NEW
- `src/app/core/auth/auth.interceptor.ts` — NEW
- `src/app/core/services/prospects.service.ts` — NEW
- `src/app/login/login.component.ts` — NEW
- `src/app/shell/shell.component.ts` — NEW
- `src/app/features/marketing/prospects-list/prospects-list.component.ts` — NEW
- `src/app/features/marketing/prospect-detail/prospect-detail.component.ts` — NEW
