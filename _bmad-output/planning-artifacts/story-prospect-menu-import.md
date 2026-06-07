# Story: Prospect Menu Import & Automated Pre-Onboarding

**Status:** Review  
**Created:** 2026-04-27  
**Author:** Mary (Business Analyst, BMad)  
**Sprint:** TBD  
**Story Points:** TBD  

---

## 1. Story Title

**Prospect Menu Import â€” Automated Demo Restaurant Creation from Public Website Menus**

---

## 2. User Story

> As an **internal MenuNest operator / growth automation system**, I want to submit one or more restaurant website URLs, have the system automatically extract the public menu data, validate its quality, and â€” when it meets the defined standard â€” create a prospect/demo restaurant instance inside MenuNest using existing account, branch, menu, table, and QR code creation logic, so that we can deliver a personalised pre-onboarded demo experience to prospects before they ever sign up.

---

## 3. Business Context

MenuNest's go-to-market strategy includes a product-led outbound motion where prospects receive a live, personalised demo of their own restaurant already set up inside MenuNest. This dramatically reduces the friction of onboarding and increases the probability that a restaurant owner converts to a paying customer.

The system must be fully automated â€” no human review step in the core flow. Automated validation acts as the gatekeeper. Only menu data of sufficient quality should result in a demo restaurant being created.

This is a **purely internal capability**. Public restaurant website data is used only where it is publicly accessible. No authenticated scraping, no paywalled content, no personal data collection.

---

## 4. Functional Requirements

### FR-1: URL Submission
- An internal operator must be able to submit one or more restaurant website URLs for processing.
- URLs may be submitted via the `MenuNestDevOps` desktop tool (new "Prospect Importer" tab) and/or via a protected internal API endpoint.
- Batch processing of multiple URLs in a single submission must be supported.

### FR-2: Menu Source Discovery
- For each URL, the system must attempt to locate a menu source on the restaurant's website.
- Discovery strategies (in priority order):
  1. Find a linked menu page (e.g. `/menu`, `/our-menu`, `/food`)
  2. Find a linked PDF menu
  3. Extract food/menu content from the homepage or main content area
- If no menu source is found, the job must fail with reason: `NO_MENU_SOURCE_FOUND`.

### FR-3: Content Extraction
- The system must extract from the identified menu source:
  - Restaurant name (from page title, header, or meta tags)
  - Menu categories (section headings in the menu)
  - Menu items per category (name, description if present, price if present)
  - Currency (if detectable)
- Extraction must avoid capturing navigation elements, cookie banners, footer text, contact page content, or unrelated website copy.

### FR-4: Data Transformation
- Extracted data must be transformed into MenuNest-compatible structured data matching the existing entity shapes:
  - `Account.AccountName` â† restaurant name
  - `Branch.Name` â† restaurant name + " â€” Main Branch"
  - `Branch.Currency` â† detected currency, or left blank if not found
  - `Menu.Name` â† "Main Menu"
  - `Menu.MenuCategories[]` â† extracted categories, each with `MenuItems[]`
  - `MenuItem.Name`, `MenuItem.Description`, `MenuItem.Price`, `MenuItem.IsActive = true`, `MenuItem.DisplayOrder`
  - `MenuCategory.Name`, `MenuCategory.IsActive = true`, `MenuCategory.DisplayOrder`
- The `MenuCategory.CategoryId` field should be set to a newly generated GUID (it is nullable in the existing model but is good practice to populate for consistency).

### FR-5: Automated Validation & Quality Scoring
- See Section 9 for full scoring rules.
- Validation is fully automated â€” no human review step.
- Score â‰¥ 80: proceed to demo creation.
- Score 60â€“79: store `ProspectImportJob` with extracted data; do NOT create demo restaurant.
- Score < 60: record failure; discard extracted data.
- Failed validation must produce both machine-readable codes and human-readable reasons.

### FR-6: Prospect/Demo Restaurant Creation
- When validation passes (score â‰¥ 80), the system must create the following entities using existing API endpoints or entity managers:
  1. `Account` (with new prospect fields â€” see Section 6)
  2. `Branch` (one branch, using existing `POST /api/branch` or `IEntityManager<Branch>`)
  3. `Menu` with full `MenuCategories` and `MenuItems` (using existing `POST /api/menu` or `IEntityManager<Menu>` â€” single-payload nested insert already supported)
  4. Five default `Table` records (e.g. "Table 1" through "Table 5") via `IEntityManager<Table>`
  5. One `TableQrAccess` record per table via existing `POST /api/tableaccess/{tableId}/generate` or `IEntityManager<TableQrAccess>`
- No `EMenuUser` (real staff account) is created for a prospect restaurant. The account has no real owner until claimed.
- A `PreviewToken` (GUID) must be generated and stored on the `Account` record for future preview link generation.

### FR-7: Duplicate Detection
- Before processing a URL, the system must check whether a `ProspectImportJob` already exists for that URL.
- If a completed job (status `created` or `extracted`) exists, skip re-processing and return the existing result.
- If a failed job exists, allow re-processing (retry is permitted).

### FR-8: Job Tracking
- Every submitted URL must produce a `ProspectImportJob` record regardless of outcome.
- Each job must record its full lifecycle state and all diagnostic data.

### FR-9: Result Reporting
- After processing, the system must return a result per URL containing:
  - Processing status
  - Quality score
  - Demo account ID (if created)
  - Failure reasons (if any)
  - Extracted restaurant name (if found)
  - Source menu URL (if found)

### FR-10: Isolation from Production Data
- Prospect/demo accounts must be clearly distinguishable from real customer accounts at all times.
- No existing production account, branch, menu, or order data must be touched.

---

## 5. Non-Functional Requirements

### NFR-1: Performance
- Each URL should be processed within 30 seconds under normal network conditions.
- Batch processing should not block the DevOps tool UI â€” use async processing with progress feedback.

### NFR-2: Resilience
- A processing failure for one URL (network error, timeout, unexpected HTML structure) must not crash or abort the rest of the batch.
- Each URL is processed independently; exceptions are caught and recorded as job failures.

### NFR-3: Security
- The internal API endpoint must be protected by an internal API key (not exposed publicly).
- The API key must be stored in environment configuration / Azure Key Vault â€” NOT hardcoded in source or `appsettings.json`.
- Only publicly accessible website content may be fetched. No authenticated scraping.
- No PII from extracted website content (e.g. staff names in contact pages) should be stored in extracted data.

### NFR-4: Observability
- All processing steps must produce structured log entries at appropriate levels (Info for lifecycle events, Warning for quality failures, Error for exceptions).
- The `ProspectImportJob` entity must store enough raw diagnostic data to reproduce and debug any extraction failure.

### NFR-5: Data Retention
- Unclaimed prospect/demo accounts should have an expiry timestamp (`ProspectExpiresAtUtc`).
- A background cleanup process (out of scope for this story; noted as follow-up) should eventually purge expired unclaimed accounts.

---

## 6. Data Model Impact

### 6a. Extend `Account` entity

**File:** [MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/Account.cs](MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/Account.cs)

Add the following fields to `Account : AbstractEntity`:

```csharp
// Prospect/demo lifecycle
public string AccountStatus { get; set; } = "active";
// Values: "active" | "prospect" | "claimed"

public bool IsProspect { get; set; } = false;

// Sourcing metadata (null for real accounts)
public string? SourceWebsiteUrl { get; set; }
public string? SourceMenuUrl { get; set; }
public string? ProspectCreatedBy { get; set; }
// e.g. "system:prospect-importer"

public DateTime? ProspectImportedAtUtc { get; set; }
public DateTime? ProspectExpiresAtUtc { get; set; }

// Preview token for future preview link generation
public string? PreviewToken { get; set; }
```

> **Rationale:** `Account` is the tenant root. All prospect flags live here so any query across the account tree can quickly filter by `IsProspect`. Cosmos DB's schemaless nature means existing documents are unaffected â€” new fields default to null/false on read of old documents.

> **Drift warning:** The Flutter `shared_package` mirrors backend entities. If any mobile app reads `Account` objects, the Dart model must be updated in parallel. For this story, prospect accounts are never accessed by the mobile apps â€” but the contract must be noted.

### 6b. New entity: `ProspectImportJob`

**File (new):** [MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/ProspectImportJob.cs](MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/ProspectImportJob.cs)

```csharp
using HosannaTech.Abstractions.Entities;

namespace MenuNest.Abstractions.Entities;

public class ProspectImportJob : AbstractEntity
{
    public string SourceWebsiteUrl { get; set; } = string.Empty;
    public string Status { get; set; } = "queued";
    // Values: "queued" | "processing" | "extracted" | "validated" | "created" | "failed"

    public int QualityScore { get; set; }
    public List<string> FailureReasons { get; set; } = new();
    public List<string> FailureCodes { get; set; } = new();

    public string? DetectedRestaurantName { get; set; }
    public string? DetectedMenuUrl { get; set; }
    public string? DetectedCurrency { get; set; }

    public string? CreatedAccountId { get; set; }

    public DateTime CreatedAtUtc { get; set; } = DateTime.UtcNow;
    public DateTime? ProcessedAtUtc { get; set; }

    public string? RawExtractionResultJson { get; set; }
    // Stores the full MenuExtractionResult JSON for debugging
}
```

**Registration:** Add to `AutoFacBootstrapper.RegisterTypes()`:
```csharp
builder.RegisterType<ProspectImportJob>();
```

### 6c. New value type: `MenuExtractionResult`

This is a **non-persisted DTO** used within the extraction pipeline. It does NOT extend `AbstractEntity` and is NOT stored in Cosmos DB directly (its JSON is stored in `ProspectImportJob.RawExtractionResultJson`).

```csharp
public class MenuExtractionResult
{
    public bool Success { get; set; }
    public string SourceWebsiteUrl { get; set; }
    public string? MenuSourceUrl { get; set; }
    public string? RestaurantName { get; set; }
    public string? Currency { get; set; }
    public List<ExtractedCategory> Categories { get; set; } = new();
    public int QualityScore { get; set; }
    public string ValidationStatus { get; set; } // "pass" | "partial" | "fail"
    public List<string> ValidationFailureCodes { get; set; } = new();
    public List<string> ValidationFailureReasons { get; set; } = new();
    public Dictionary<string, object> DiagnosticMeta { get; set; } = new();
}

public class ExtractedCategory
{
    public string Name { get; set; }
    public List<ExtractedMenuItem> Items { get; set; } = new();
}

public class ExtractedMenuItem
{
    public string Name { get; set; }
    public string? Description { get; set; }
    public decimal? Price { get; set; }
}
```

---

## 7. API / UI / Desktop App Impact

### 7a. New Internal API Endpoint

**New Controller (new file):**
[MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/ProspectImportController.cs](MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/ProspectImportController.cs)

```
POST /api/internal/prospect/import
```

- Accepts: `{ "urls": ["https://restaurant.com", ...] }`
- Protected by: `[Authorize(Policy = "InternalApiKey")]`
- Returns: array of `ProspectImportJobResult` per URL

```
GET /api/internal/prospect/jobs
GET /api/internal/prospect/jobs/{id}
```

- Read endpoints for job status and diagnostics.

**New Authentication Policy: Internal API Key**

The existing backend uses Firebase JWT bearer authentication. All existing controllers use `[Authorize]` which maps to the Firebase policy. A new, separate authentication scheme must be added:

- Add an `InternalApiKeyAuthenticationHandler` that reads `X-MenuNest-Internal-Key` header.
- Register in `Program.cs` alongside the existing Firebase JWT scheme.
- Apply via a new `[Authorize(Policy = "InternalApiKey")]` attribute â€” does NOT replace Firebase auth on existing endpoints.
- The key value is read from `appsettings.json â†’ InternalApi:Key` (to be moved to Azure Key Vault in production).

> **Important:** This does NOT change or relax authentication on any existing endpoint. Only the new `/api/internal/*` routes use the internal API key policy.

### 7b. New Use Case: `CreateProspectAccountUseCase`

**New file:** [MenuNestServer/MenuNestServer/MenuNest.Usecase.Implementation/Prospect/CreateProspectAccountUseCase.cs](MenuNestServer/MenuNestServer/MenuNest.Usecase.Implementation/Prospect/CreateProspectAccountUseCase.cs)

Mirrors the pattern of the existing `CreateAccountUseCase` ([MenuNestServer/MenuNestServer/MenuNest.Usecase.Implementation/Auth/CreateAccountUseCase.cs](MenuNestServer/MenuNestServer/MenuNest.Usecase.Implementation/Auth/CreateAccountUseCase.cs)):

```csharp
public class CreateProspectAccountUseCase
{
    private readonly IEntityManager<Account> _accountManager;
    private readonly IEntityManager<Branch> _branchManager;
    private readonly IEntityManager<Menu> _menuManager;
    private readonly IEntityManager<Table> _tableManager;
    private readonly IEntityManager<TableQrAccess> _tableAccessManager;
    private readonly IEntityManager<ProspectImportJob> _jobManager;

    public async Task<CreateProspectResult> ExecuteAsync(
        MenuExtractionResult extraction,
        string sourceWebsiteUrl)
    { ... }
}
```

This use case:
1. Creates `Account` with `IsProspect = true`, `AccountStatus = "prospect"`, `SourceWebsiteUrl`, `PreviewToken = Guid.NewGuid().ToString("N")`, `ProspectImportedAtUtc = DateTime.UtcNow`, `ProspectExpiresAtUtc = DateTime.UtcNow.AddDays(90)`
2. Creates `Branch` with `AccountId` linked; `Name = "{restaurantName} â€” Main Branch"`
3. Creates `Menu` with full `MenuCategories` and `MenuItems` populated from extraction â€” uses the same single-payload insert path already validated in `MenuController`
4. Creates 5 `Table` records (`Table 1` through `Table 5`) with `BranchId` linked
5. For each table, creates a `TableQrAccess` with `PublicCode = Guid.NewGuid().ToString("N")`, `IsActive = true`
6. Updates `ProspectImportJob.Status = "created"`, `ProspectImportJob.CreatedAccountId`

**Register in `AutoFacBootstrapper.RegisterTypes()`:**
```csharp
builder.RegisterType<CreateProspectAccountUseCase>().AsSelf();
```

### 7c. MenuNestDevOps Desktop Tool â€” New "Prospect Importer" Tab

**Existing tool:** [MenuNest_dev_ops/MenuNestDevOps/Form1.cs](MenuNest_dev_ops/MenuNestDevOps/Form1.cs) â€” currently a WinForms .NET 8 app managing Flutter app releases.

**Extension:** Add a new tab page ("Prospect Importer") to the existing `Form1` (or a new `Form2` if tab complexity grows). Following the established `AppSectionControls` record pattern:

UI elements:
- `TextBox` â€” multi-line URL input (one URL per line) or file-browse for a `.txt` list
- `Button` â€” "Start Import"  
- `DataGridView` â€” results grid (URL, status, score, restaurant name, failure reasons, account ID)
- `Label` â€” status / progress indicator
- `Button` â€” "Export Results" (CSV)

Behaviour:
- On "Start Import": calls `POST https://menunestapi.azurewebsites.net/api/internal/prospect/import` with the `X-MenuNest-Internal-Key` header.
- API key is read from a local config file or environment variable on the DevOps machine â€” never hardcoded.
- Results grid polls `GET /api/internal/prospect/jobs/{id}` or is returned inline from the import response.
- Errors per URL do not prevent other URLs from processing.

**New service file:**
[MenuNest_dev_ops/MenuNestDevOps/ProspectImportService.cs](MenuNest_dev_ops/MenuNestDevOps/ProspectImportService.cs)

Follows same pattern as `RolloutBuildService` and `GooglePlayReleaseService` â€” a plain C# service class with async methods, no static state.

---

## 8. Integration Points with Existing Logic

| Existing component | How it is reused |
|---|---|
| `IEntityManager<Account>` | Used directly in `CreateProspectAccountUseCase` â€” same generic manager pattern |
| `IEntityManager<Branch>` | Used directly in `CreateProspectAccountUseCase` |
| `IEntityManager<Menu>` | Used directly â€” accepts full nested `MenuCategories + MenuItems` payload, same as `MenuController.Create` |
| `IEntityManager<Table>` | Used directly |
| `IEntityManager<TableQrAccess>` | Used directly |
| `AutoFacBootstrapper` | Register `ProspectImportJob`, `CreateProspectAccountUseCase`, and the new `ProspectImportOrchestrator` |
| `EntityDataService<>` with `projectName = "MenuNest"` | Cosmos DB persistence; new entities go into the same database automatically |
| `MenuController` image validation | `MenuItem.ImagePaths` defaults to empty list â€” no images imported from extraction, so image limit validation passes without issue |
| `BranchController` guest guard pattern | Not applicable for internal endpoints â€” internal API key policy bypasses guest context |

---

## 9. Validation & Scoring Rules

### Scoring Rubric (0â€“100)

| Rule | Points | Failure Code |
|---|---|---|
| Restaurant name was identified | 15 | `NO_RESTAURANT_NAME` |
| Menu source URL was found | 15 | `NO_MENU_SOURCE_FOUND` |
| At least 2 non-empty categories extracted | 15 | `INSUFFICIENT_CATEGORIES` |
| At least 8 menu items extracted across all categories | 15 | `INSUFFICIENT_ITEMS` |
| â‰¥ 70% of items have a parseable price > 0 | 20 | `INSUFFICIENT_PRICES` |
| â‰¥ 90% of items have a non-empty, non-junk name | 10 | `POOR_ITEM_NAMES` |
| No empty categories (each category has â‰¥ 1 item) | 5 | `EMPTY_CATEGORY` |
| Extracted content passes junk filter | 5 | `JUNK_CONTENT_DETECTED` |
| **Total** | **100** | |

### Thresholds

| Score | Decision |
|---|---|
| â‰¥ 80 | **PASS** â€” create prospect/demo restaurant |
| 60â€“79 | **PARTIAL** â€” store `ProspectImportJob` with extracted data, `Status = "extracted"`, no demo created |
| < 60 | **FAIL** â€” store `ProspectImportJob` with `Status = "failed"`, discard extracted data |

### Junk Filter Rules

Content is flagged as junk if the extracted categories or items appear to be primarily composed of:
- Navigation link text (e.g. "Home", "About", "Contact", "Book a Table")
- Cookie consent text
- Social media handles or URLs
- Phone numbers / addresses only
- Categories with identical names to well-known non-food sections

Implementation: a configurable `JunkContentFilter` class with a keyword blocklist and heuristics (e.g. if > 30% of "item names" match navigation keywords â†’ junk flag raised â†’ `JUNK_CONTENT_DETECTED`).

### Validation Notes (Grounded in Existing Model)

- The `MenuCategory` entity has no minimum items constraint in the current model. The 2-category minimum is enforced by this validator only.
- The `MenuItem.Price` field is `decimal` (not nullable) â€” extracted items with no price are imported with `Price = 0`. The validator checks the ratio of items with `Price > 0` against total items to compute the price coverage score.
- The `MenuItem.Name` field has no max-length constraint in the current model â€” junk names are identified by content heuristics, not length.

---

## 10. Error Handling & Retry Behaviour

### Per-URL Error Handling

| Error scenario | Behaviour |
|---|---|
| Network timeout or DNS failure for source URL | Record job as `failed`; reason: `FETCH_TIMEOUT`; exception message in `DiagnosticMeta` |
| URL returns non-200 HTTP status | Record job as `failed`; reason: `HTTP_ERROR_{code}` |
| URL content is not parseable as HTML or PDF | Record job as `failed`; reason: `UNPARSEABLE_CONTENT` |
| Extraction produces zero categories | Score 0; record as `failed`; reason: `NO_MENU_SOURCE_FOUND` |
| Validation fails (score < 60) | Record job as `failed` with all failure codes and reasons |
| Cosmos DB write failure during account creation | Roll back: delete any partially created entities; record job as `failed`; reason: `PERSISTENCE_ERROR` |
| Duplicate URL (already has a `created` or `extracted` job) | Return existing job result; do not reprocess; log as `DUPLICATE_SKIPPED` |

### Retry

- Operator may re-submit any URL for reprocessing.
- Only `failed` jobs are re-runnable. Jobs with status `created` are considered final.
- On retry, a NEW `ProspectImportJob` record is created; the old failed job is not overwritten (preserves audit trail).

### Batch Isolation

- Each URL is processed in isolation within the batch.
- If URL #3 of a batch of 10 throws an unhandled exception, URLs #4â€“10 still process.
- The batch result always returns a result entry for every submitted URL.

---

## 11. Acceptance Criteria (Given / When / Then)

### AC-1: Successful full flow

**Given** a valid restaurant website URL with a discoverable menu containing â‰¥ 2 categories, â‰¥ 8 items, and â‰¥ 70% priced items  
**When** the internal operator submits the URL via the DevOps tool or internal API  
**Then** the system creates a `ProspectImportJob` with `Status = "created"`, quality score â‰¥ 80, creates an `Account` with `IsProspect = true`, a linked `Branch`, a `Menu` with the extracted categories and items, 5 `Table` records, and 5 `TableQrAccess` records, and returns all created entity IDs.

### AC-2: Validation failure â€” insufficient items

**Given** a restaurant website URL where extraction finds < 8 menu items  
**When** the system processes the URL  
**Then** the `ProspectImportJob.Status = "failed"`, `FailureCodes` contains `INSUFFICIENT_ITEMS`, no `Account`, `Branch`, `Menu`, `Table`, or `TableQrAccess` records are created.

### AC-3: Validation partial â€” score 60â€“79

**Given** a restaurant website URL where extraction finds â‰¥ 2 categories and â‰¥ 8 items, but only 55% of items have prices  
**When** the system processes the URL  
**Then** the `ProspectImportJob.Status = "extracted"`, quality score is between 60 and 79, extracted data is stored, `FailureCodes` contains `INSUFFICIENT_PRICES`, no `Account` or downstream entities are created.

### AC-4: No menu source found

**Given** a URL that is a valid restaurant website but has no discoverable menu page, PDF, or food content  
**When** the system processes the URL  
**Then** `ProspectImportJob.Status = "failed"`, `FailureCodes` contains `NO_MENU_SOURCE_FOUND`, quality score = 0.

### AC-5: Duplicate URL handling

**Given** a URL that was already successfully processed (`Status = "created"`)  
**When** the same URL is submitted again  
**Then** the system returns the existing job result without creating a new job or any new entities, and logs a `DUPLICATE_SKIPPED` diagnostic.

### AC-6: Network failure isolation

**Given** a batch of 3 URLs where URL #2 times out  
**When** the batch is processed  
**Then** URL #1 and URL #3 process normally, URL #2 records a `failed` job with reason `FETCH_TIMEOUT`, and the batch result contains entries for all 3 URLs.

### AC-7: Prospect account is distinguishable

**Given** a prospect/demo restaurant was created  
**When** a developer or internal user queries the Cosmos DB `Account` container  
**Then** `Account.IsProspect = true`, `Account.AccountStatus = "prospect"`, `Account.SourceWebsiteUrl` is populated, and `Account.PreviewToken` is populated.

### AC-8: Production data is untouched

**Given** any number of prospect import runs  
**When** the system is inspected  
**Then** no existing production `Account`, `Branch`, `Menu`, `Order`, or `EMenuUser` records have been modified.

### AC-9: Internal API is not publicly accessible

**Given** a request to `POST /api/internal/prospect/import` with no `X-MenuNest-Internal-Key` header  
**When** the request is processed  
**Then** the API returns `401 Unauthorized`.

### AC-10: Junk content is rejected

**Given** a URL where the "menu" content is primarily navigation text, cookie banners, or contact page content  
**When** the junk filter runs  
**Then** `JUNK_CONTENT_DETECTED` is added to `FailureCodes` and the quality score is reduced by 5 points; if the total score falls below 60, the job fails.

---

## 12. Out of Scope (This Story)

The following items are explicitly deferred to follow-up stories:

- **Preview link generation** â€” `PreviewToken` is created and stored in this story; the endpoint that resolves a token to a live demo view is a follow-up.
- **Personalised outreach email** â€” Email composition and sending (e.g. "We built your menu in MenuNest â€” take a look") is a follow-up.
- **Prospect claim flow** â€” The mechanism for a prospect to claim their demo restaurant and convert it to a live account is a follow-up.
- **Screenshot/image generation** â€” Capturing a screenshot of the live demo for use in outreach emails is a follow-up.
- **Expired account cleanup** â€” Background purging of unclaimed accounts past `ProspectExpiresAtUtc` is a follow-up.
- **Menu image import** â€” Scraping and importing item images from the source website is a follow-up (too fragile for v1).
- **PDF menu parsing** â€” While PDF discovery is in scope for this story, robust PDF text extraction may be descoped to a follow-up if the implementation effort is too high. In that case, only HTML menu pages are processed in v1, and a PDF source is recorded in `DetectedMenuUrl` for future use.
- **Real-time progress streaming** â€” Streaming extraction progress via SignalR or WebSocket is a follow-up; polling is sufficient for v1.
- **Multi-branch extraction** â€” For chains with multiple locations, only the first/primary branch is imported in v1.
- **Flutter model sync** â€” `Account` entity changes need to propagate to the Flutter `shared_package` Dart models; that sync work is tracked separately and is low risk since prospect accounts are never accessed by mobile apps.

---

## 13. Implementation Tasks

### Backend (`MenuNestServer`)

- [x] **T1** â€” Extend `Account` entity with prospect fields (`AccountStatus`, `IsProspect`, `SourceWebsiteUrl`, `SourceMenuUrl`, `ProspectCreatedBy`, `ProspectImportedAtUtc`, `ProspectExpiresAtUtc`, `PreviewToken`)
- [x] **T2** â€” Create `ProspectImportJob` entity in `MenuNest.Abstractions/Entities/`
- [x] **T3** â€” Create `MenuExtractionResult`, `ExtractedCategory`, `ExtractedMenuItem` DTOs
- [x] **T4** â€” Register `ProspectImportJob` in `AutoFacBootstrapper.RegisterTypes()`
- [x] **T5** â€” Implement `InternalApiKeyAuthenticationHandler` and register new `InternalApiKey` auth policy in `Program.cs`
- [x] **T6** â€” Create `CreateProspectAccountUseCase` in `MenuNest.Usecase.Implementation/Prospect/`
- [x] **T7** â€” Register `CreateProspectAccountUseCase` in `AutoFacBootstrapper.RegisterTypes()`
- [x] **T8** â€” Create `ProspectImportController` with `POST /api/internal/prospect/import`, `GET /api/internal/prospect/jobs`, `GET /api/internal/prospect/jobs/{id}`
- [x] **T9** â€” Implement `MenuExtractionOrchestrator` service (calls extractor, transforms, validates, creates demo) in `MenuNest.Services/Prospect/`
- [x] **T10** â€” Write unit tests for `MenuExtractionValidator` scoring logic
- [x] **T11** â€” Write integration tests for `CreateProspectAccountUseCase`

### Extraction Component (`MenuNest.MenuExtractor` â€” new project or in `MenuNest.Services`)

- [x] **T12** â€” Implement `WebMenuExtractor` â€” HTTP fetch + HTML parse (using `HtmlAgilityPack` or `AngleSharp`)
- [x] **T13** â€” Implement menu source discovery logic (linked menu pages, PDF links, content heuristics)
- [x] **T14** â€” Implement content extraction â€” category heading detection, item/price pattern matching
- [x] **T15** â€” Implement `JunkContentFilter`
- [x] **T16** â€” Implement `MenuExtractionValidator` with scoring rubric
- [x] **T17** â€” Ensure extraction component is isolated from core domain logic (no Cosmos DB dependency, no entity managers)

### DevOps Desktop Tool (`MenuNest_dev_ops`)

- [x] **T18** â€” Add "Prospect Importer" tab to `Form1` Designer (or new `ProspectImportForm`)
- [x] **T19** â€” Implement `ProspectImportService` to call the backend internal API
- [x] **T20** â€” Implement results `DataGridView` binding with status, score, restaurant name, failure reasons
- [x] **T21** â€” Add "Export Results to CSV" button
- [x] **T22** â€” Read internal API key from local environment config (not hardcoded)

---

## Dev Agent Record

### Debug Log

- 2026-04-27: Added red entity/DTO tests; initial failure confirmed missing `MenuNest.Abstractions.Prospect` and prospect fields.
- 2026-04-27: Added red internal API key handler tests; initial failure confirmed missing `MenuNestAPI.Auth`.
- 2026-04-27: Added red prospect account creation test; initial failure confirmed missing `MenuNest.UseCases.Prospect`.
- 2026-04-27: Added red validator, extractor, and orchestrator tests before implementing service classes.
- 2026-04-27: DevOps build initially failed due local config JSON options scope; fixed by adding config-local serializer options.

### Completion Notes

- Implemented prospect account lifecycle fields on `Account` and new `ProspectImportJob` persistence entity.
- Implemented extraction DTOs, `HtmlAgilityPack`-based `WebMenuExtractor`, menu link/PDF discovery, heading/list item parsing, price/currency extraction, junk filtering, and validation scoring.
- Implemented internal API key authentication scheme and protected `/api/internal/prospect/*` endpoints without changing existing Firebase bearer auth for current endpoints.
- Implemented prospect import orchestration with duplicate skip, per-URL exception isolation, `ProspectImportJob` tracking, partial extraction storage, failed extraction discard, and pass-through demo account creation.
- Implemented `CreateProspectAccountUseCase` with prospect `Account`, linked `Branch`, nested `Menu`, 5 `Table` records, 5 `TableQrAccess` records, job update, and rollback on persistence failure.
- Implemented MenuNest DevOps "Prospect Importer" tab at runtime in `Form1`, backend service client, result grid binding, CSV export, and internal API key lookup from `MENUNEST_INTERNAL_API_KEY` or `prospect-importer.local.json`.
- Decision: score caps were added for `INSUFFICIENT_PRICES`, `INSUFFICIENT_ITEMS`, and `JUNK_CONTENT_DETECTED` to satisfy the explicit acceptance criteria where the point table alone would otherwise allow a pass.

### File List

- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/Account.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/ProspectImportJob.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Prospect/MenuExtractionResult.cs`
- `MenuNestServer/MenuNestServer/MenuNest.BootStrapper/AutoFacBootstrapper.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Services/MenuNest.Services.csproj`
- `MenuNestServer/MenuNestServer/MenuNest.Services/Prospect/IMenuExtractor.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Services/Prospect/JunkContentFilter.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Services/Prospect/MenuExtractionValidator.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Services/Prospect/ProspectImportOrchestrator.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Services/Prospect/WebMenuExtractor.cs`
- `MenuNestServer/MenuNestServer/MenuNest.Usecase.Implementation/Prospect/CreateProspectAccountUseCase.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Auth/InternalApiKeyAuthenticationHandler.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/ProspectImportController.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Program.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/CreateProspectAccountUseCaseTests.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/InternalApiKeyAuthenticationHandlerTests.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/MenuExtractionValidatorTests.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/ProspectEntityContractTests.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/ProspectImportOrchestratorTests.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/WebMenuExtractorTests.cs`
- `MenuNest_dev_ops/MenuNestDevOps/Form1.cs`
- `MenuNest_dev_ops/MenuNestDevOps/ProspectImportService.cs`
- `_bmad-output/planning-artifacts/story-prospect-menu-import.md`

### Validation

- `dotnet test MenuNestAPI.Tests\MenuNestAPI.Tests.csproj` -> 36 passed.
- `dotnet build MenuNest.sln` -> succeeded.
- `dotnet build MenuNestDevOps.csproj` -> succeeded.

## Change Log

- 2026-04-27: Implemented prospect menu import backend, extraction pipeline, DevOps importer UI, and automated tests. Status moved to Review.

---

## 14. Risks & Open Questions

### Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Restaurant websites use JavaScript-rendered menus (React/Vue SPAs) that return empty HTML without JS execution | High | High | In v1, document this limitation. Accept that JS-rendered menus will fail extraction and surface `NO_MENU_SOURCE_FOUND`. In v2, evaluate headless browser (Playwright) for JS rendering. |
| R2 | Menu PDF parsing is technically complex and fragile | Medium | Medium | Descope PDF parsing from v1 if needed. Record the PDF URL in `DetectedMenuUrl` for manual or v2 processing. |
| R3 | Cosmos DB `Account` container schema drift â€” adding new fields to `Account` could cause old document reads to fail if Cosmos SDK is strict about unknown fields | Low | Medium | Verify `HosannaTech.EntityManager` and `HosannaTech.DataService` deserialisation uses `JsonIgnore` or ignores missing properties. Test read of old `Account` documents after model change. |
| R4 | Rate limiting / bot detection on restaurant websites | Medium | Medium | Respect `robots.txt`. Add a configurable delay between URL fetches in a batch. Accept that some sites will block extraction â€” these fail gracefully. |
| R5 | `MenuItem.Price` is `decimal` (non-nullable) in the current model â€” items without prices must be stored as `Price = 0`, which could look like free items | Low | Low | Document `Price = 0` as "price not found". Consider `decimal?` in a future model refactor (out of scope here). |
| R6 | Flutter `shared_package` Dart models mirror `Account` â€” adding fields without updating Dart models could cause deserialization errors on mobile | Low | Low | Prospect accounts are never accessed by mobile apps. However, schedule a Dart model sync in the next sprint as a precaution. |

### Open Questions

| # | Question | Owner |
|---|---|---|
| Q1 | Should `AccountStatus` be a `string` or a C# `enum`? Cosmos DB's schemaless nature favours strings; enums are safer in code. Recommended: string constant class (not enum) to stay Cosmos-friendly. | Backend dev |
| Q2 | What should the expiry period for unclaimed prospect accounts be? 90 days suggested â€” confirm with business. | Product / Jabran |
| Q3 | Should the internal API key be per-environment (dev/prod) or a single shared key? Recommended: per-environment. | DevOps / Jabran |
| Q4 | Is there a preference for the HTML parsing library? `HtmlAgilityPack` is mature and widely used in .NET; `AngleSharp` is more modern. Either works â€” pick one and commit. | Backend dev |
| Q5 | Should the `ProspectImportJob` container in Cosmos be separate from other entities, or use the same generic `EntityDataService<>` which uses `projectName = "MenuNest"`? Recommended: same container (same Cosmos database) for simplicity, since all entities currently share one database. | Backend dev |
| Q6 | For the DevOps tool, should the API key be stored in a local `.env` file, Windows Credential Manager, or a config file? Recommended: `appsettings.json` local to the DevOps tool (not committed; gitignored). | DevOps / Jabran |

---

## 15. Follow-Up Stories

| # | Story | Depends on |
|---|---|---|
| S2 | **Preview Link Generation** â€” Implement `GET /api/preview/{token}` that resolves a `PreviewToken` to a read-only live view of the demo restaurant (menu, tables, QR codes). | This story |
| S3 | **Personalised Outreach Email** â€” Generate and send a personalised email to a target contact at the restaurant with their preview link. | S2 |
| S4 | **Prospect Claim Flow** â€” Allow a restaurant owner to click "Claim this restaurant" in the preview, create a real Firebase account, and convert the prospect account to `AccountStatus = "active"`. | S2 |
| S5 | **Screenshot Generation** â€” Render a screenshot of the live demo menu page for use in outreach email content (headless browser). | S2 |
| S6 | **Expired Account Cleanup** â€” Background job (Azure Function or hosted service) to purge unclaimed prospect accounts past `ProspectExpiresAtUtc`. | This story |
| S7 | **JS-Rendered Menu Support** â€” Extend `WebMenuExtractor` to use a headless browser (Playwright) for websites that require JavaScript execution to render menu content. | This story |
| S8 | **PDF Menu Extraction** â€” Implement robust PDF text extraction and menu structure parsing for restaurants that publish their menus as PDF files. | This story |
| S9 | **Prospect Dashboard (Internal)** â€” An internal view in the DevOps tool or a dedicated internal web page showing all prospect jobs, their status, quality scores, and one-click access to the created demo restaurant. | This story |

---

*Story produced by Mary (BMad Strategic Analyst) on 2026-04-27, based on full codebase inspection of MenuNest monorepo.*
