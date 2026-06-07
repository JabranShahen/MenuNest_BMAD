# Story: Multi-Image Support for Menu Items

**ID:** MENU-ITEM-MULTI-IMAGE  
**Status:** In Progress  
**Author:** Mary (Business Analyst)  
**Date:** 2026-04-25  
**Surfaces Affected:** Backend · Website (Angular) · Mobile Table App (Flutter) · Shared Package (Flutter)

---

## User Stories

**As a restaurant operator,**  
I want to upload up to 3 images for each menu item on the add/edit screens,  
so that I can showcase my dishes from multiple angles and drive guest appetite.

**As a restaurant guest (table app user),**  
I want to swipe through multiple images and tap to zoom on a menu item detail,  
so that I can get a clear and appealing view of what I'm ordering.

---

## Background

MenuNest currently supports a single image per menu item. Images are stored as blob paths in Azure Blob Storage via SAS URL generation (`BlobController`). The operator uploads via the Angular website (`AddMenuItemModalComponent` / `EditMenuItemModalComponent`), and the table app displays the image in `MenuView`.

This story extends that to a maximum of 3 images per item. The first image remains the primary thumbnail everywhere it currently appears. Images remain entirely optional. The multi-image gallery is a table app (guest-facing) feature only — kitchen, serve, and admin apps are out of scope for this story.

---

## Acceptance Criteria

### Surface 1 — Backend (ASP.NET Core)

- [ ] `MenuItem` entity updated to replace the single image field with `List<string> ImagePaths` (max 3 items).
- [ ] Existing `MenuItem` records that carry a legacy single image field are treated as having one item in `ImagePaths` at index 0 — no data migration script required given Cosmos DB's schema-less nature, but deserialization must handle both old and new shapes gracefully.
- [ ] Use case / service layer enforces the 3-image maximum and rejects payloads that exceed it with a `400 Bad Request`.
- [ ] `POST /api/menu` and `PUT /api/menu/{id}` accept and persist the updated entity shape.
- [ ] `GET /api/menu/{id}` and `GET /api/menu/branch/{branchId}` return `imagePaths` as an ordered array.
- [ ] `BlobController` SAS generation requires no changes — callers request one SAS per image path as they already do today.
- [ ] No changes to `KitchenTicketController`, `OrderController`, or any non-menu controller.

---

### Surface 2 — Angular Website (Operator Upload)

- [ ] `AddMenuItemModalComponent` replaces the single image upload control with a multi-image upload panel supporting up to 3 slots.
  - Slot 1 is labelled **Primary Image**. Slots 2 and 3 are labelled **Additional Image**.
  - All slots are optional.
  - Each filled slot shows: image thumbnail preview · remove (×) button.
  - Each empty slot shows: an upload button that triggers file selection.
  - Slots 2 and 3 are always visible (not revealed progressively) so the operator understands the 3-image limit upfront.
  - The "3 images maximum" limit is communicated via helper text beneath the panel.
- [ ] `EditMenuItemModalComponent` applies the same panel and pre-populates slots from the existing `imagePaths` array on load.
- [ ] Each upload follows the existing SAS flow via `BlobStorageService`: request upload SAS → upload to blob → store returned path.
- [ ] Removing an image from a slot (×) deletes its path from the local array only; no blob deletion is performed (consistent with current single-image behaviour).
- [ ] The order of images in the array reflects the visual order of slots (slot 1 = index 0, always primary).
- [ ] Slot ordering is fixed — no drag-to-reorder in this story.
- [ ] `MenuService` updated to send `imagePaths: string[]` (not a singular `imagePath`) in create and update payloads.
- [ ] Existing Jasmine/Karma unit tests for `MenuService` and the modal components updated to reflect the new shape.

---

### Surface 3 — Flutter Shared Package

- [ ] `MenuItem` Dart entity updated: singular image field replaced with `List<String> imagePaths` (defaults to empty list, not null).
- [ ] JSON serialisation mapping updated — `imagePaths` maps to the `imagePaths` JSON key.
- [ ] Backward-compat deserialisation: if the API returns the legacy single-image field (old document), map it to `imagePaths[0]` so existing data renders correctly without a backend migration.
- [ ] No changes to `MenuService` business logic beyond the updated entity shape — API responses map through automatically.
- [ ] All existing entity service and manager references to the image field updated to use `imagePaths`.
- [ ] `build_runner` run after entity change to regenerate any JSON serialisation code if the project uses `json_serializable`.

---

### Surface 4 — Flutter Table App (Guest Gallery)

**Menu List View (`MenuView`)**
- [ ] Thumbnail display unchanged in behaviour — uses `imagePaths.isNotEmpty ? imagePaths[0] : null` as the primary image. If `imagePaths` is empty, the existing no-image placeholder is shown.

**Menu Item Detail View**
- [ ] Tapping a menu item opens (or enhances the existing) product detail screen.
- [ ] If the item has **1 image**: display a single static image (no carousel chrome, no dots).
- [ ] If the item has **2 or 3 images**: display a swipeable horizontal image carousel (`PageView`).
  - A dot indicator below the carousel shows current position and total count.
  - Swipe left/right navigates between images.
- [ ] If the item has **0 images**: the image section is hidden; content below fills the space.
- [ ] Tapping any image opens a full-screen tap-to-zoom lightbox (`InteractiveViewer` or `photo_view` package).
  - Pinch-to-zoom and double-tap-to-zoom are both supported.
  - Swiping left/right in the lightbox navigates to adjacent images (if multiple).
  - Tapping outside the image or pressing back dismisses the lightbox.
- [ ] The detail view handles network image loading states: loading spinner while fetching, error placeholder if load fails.
- [ ] Only the table app receives these changes — `admin_app`, `kitchen_app`, and `serve_app` are untouched.

---

## Implementation Guidance by Surface

### Backend

**Files to change:**
- `MenuNest.Abstractions/Entities/MenuItem.cs` — swap image field type
- `MenuNest.Usecase.Implementation` (or equivalent) — add max-3 validation
- `MenuNest.Services` / `MenuNest.EntityManager` if image field is referenced there

**Backward-compat strategy:**  
Cosmos DB stores JSON documents. Old documents have the singular field; new documents have `imagePaths`. Use a custom Cosmos serializer or a computed property to coalesce: if `imagePaths` is null/absent and the legacy field is present, return `[legacyField]`. This avoids any data migration job.

**Validation pattern:**
```csharp
if (menuItem.ImagePaths?.Count > 3)
    return BadRequest("A menu item may have a maximum of 3 images.");
```

---

### Angular Website

**Files to change:**
- `src/app/menu-management/add-menu-item-modal/` — upload UI
- `src/app/menu-management/edit-menu-item-modal/` — upload UI + pre-population
- `src/app/services/menu.service.ts` — payload shape

**Upload slot component pattern:**

Each slot can be extracted into a small reusable `ImageUploadSlotComponent` (or inline template block) that accepts:
- `imageUrl: string | null` (current value)
- `(upload): EventEmitter` — triggers SAS flow
- `(remove): EventEmitter` — clears the slot

The parent modal maintains `imagePaths: string[]` (max length 3) and passes each index to a slot.

---

### Flutter Shared Package

**Files to change:**
- `shared_package/lib/entities/menu_item.dart` (or equivalent entity file)

**Entity field change:**
```dart
// Before
final String? imagePath;

// After
final List<String> imagePaths;
```

**Backward-compat deserialisation:**
```dart
factory MenuItem.fromJson(Map<String, dynamic> json) {
  final paths = json['imagePaths'] as List?;
  final legacyPath = json['imagePath'] as String?;
  return MenuItem(
    imagePaths: paths != null
        ? List<String>.from(paths)
        : (legacyPath != null ? [legacyPath] : []),
    // ... other fields
  );
}
```

---

### Flutter Table App

**Files to change / create:**
- `table_app/lib/screens/menu_item_detail_screen.dart` — create or enhance
- `table_app/lib/screens/menu_view.dart` — update image reference to `imagePaths[0]`

**Recommended packages (add to `shared_package/pubspec.yaml` or table app):**
- `photo_view: ^0.15.x` — lightbox with pinch-to-zoom and swipe navigation

**Carousel pattern:**
```dart
PageView.builder(
  itemCount: item.imagePaths.length,
  itemBuilder: (context, index) => GestureDetector(
    onTap: () => _openLightbox(context, item.imagePaths, index),
    child: CachedNetworkImage(imageUrl: item.imagePaths[index]),
  ),
)
```

**Lightbox pattern:**
```dart
PhotoViewGallery.builder(
  itemCount: imagePaths.length,
  builder: (context, index) => PhotoViewGalleryPageOptions(
    imageProvider: NetworkImage(imagePaths[index]),
    minScale: PhotoViewComputedScale.contained,
    maxScale: PhotoViewComputedScale.covered * 2,
  ),
  pageController: PageController(initialPage: initialIndex),
)
```

---

## Data Contract

**Updated MenuItem JSON shape (API response and request body):**

```json
{
  "id": "...",
  "name": "Grilled Chicken",
  "description": "...",
  "price": 12.99,
  "imagePaths": [
    "accounts/abc/menus/item1-front.jpg",
    "accounts/abc/menus/item1-side.jpg"
  ],
  "categoryId": "...",
  "isAvailable": true
}
```

- `imagePaths` is always an array (never null in responses — return `[]` if no images).
- Array length is 0–3 inclusive.
- Index 0 is always the primary/thumbnail image.

---

## Out of Scope (This Story)

- Drag-to-reorder images in the operator UI
- Blob deletion when an image is removed from a slot
- Image display in `admin_app`, `kitchen_app`, `serve_app`
- Image compression or resizing server-side
- Video support
- Alt text / accessibility metadata per image
- Reporting or analytics on image engagement

---

## Dependencies & Risk

| Risk | Impact | Mitigation |
|------|--------|------------|
| `MenuItem` is mirrored in C# and Dart — both must ship together | High | Coordinate backend + shared package deployment; do not deploy backend alone |
| Legacy Cosmos documents with singular image field | Medium | Backward-compat deserialisation on both C# and Dart sides (see above) |
| `photo_view` package version conflict with existing Flutter deps | Low | Check pubspec before adding; `InteractiveViewer` is a zero-dep fallback |
| `dist/` committed to repo may show stale Angular build | Low | Rebuild Angular and commit updated `dist` after website changes |

---

---

## Tasks / Subtasks

### Task 1 — Backend: MenuItem entity + validation
- [x] 1.1 Update `MenuItem.cs` — replace `Image: string` with `ImagePaths: List<string>` + backward-compat computed getter preserving `Image` for old Cosmos documents
- [x] 1.2 Update `MenuController.cs` — add max-3 image validation in `Create` and `UpdateMenu` before manager calls
- [x] 1.3 Add `MenuControllerImageValidationTests.cs` — xUnit tests covering create/update with >3, ≤3, and 0 images

### Task 2 — Flutter Shared Package: MenuItem entity
- [x] 2.1 Update `shared_package/lib/entities/menu.dart` — replace `dynamic image` with `List<String> imagePaths`; backward-compat `fromJson` coalesces from legacy `image` key; `toJson` writes `imagePaths`
- [x] 2.2 Add `shared_package/test/entities/menu_entity_test.dart` — unit tests for fromJson/toJson including legacy backward-compat path

### Task 3 — Angular Website: Multi-image upload
- [x] 3.1 Update `MenuNestWebsite/src/app/entities/menu.ts` — replace `image: any` with `imagePaths: string[]`; update `MenuItem` constructor
- [x] 3.2 Update `add-menu-item-modal.component.ts` and `.html` — 3-slot upload panel; slots always visible; slot 1 = Primary Image, 2–3 = Additional Image; helper text; SAS flow per slot
- [x] 3.3 Update `edit-menu-item-modal.component.ts` and `.html` — same 3-slot panel; pre-populate from `editableItem.imagePaths` on load; preview/remove per slot
- [x] 3.4 Add `add-menu-item-modal.component.spec.ts` and update `edit-menu-item-modal.component.spec.ts` — reflect new `imagePaths` shape

### Task 4 — Flutter Table App: Gallery + Lightbox
- [x] 4.1 Add `photo_view: ^0.15.0` to `table_app/pubspec.yaml`
- [x] 4.2 Update `table_ordering_screen.dart` — `_loadImages` resolves all paths in `imagePaths`; `_imageForOrderItem` returns `imagePaths.firstOrNull`
- [x] 4.3 Update `menu_view.dart` — `MenuItemCard` thumbnail uses `imagePaths.isNotEmpty ? imagePaths[0] : null`; tap on card calls `_onItemTapped` which navigates to detail screen
- [x] 4.4 Create `table_app/lib/screens/menu_item_detail_screen.dart` — 0 images → hidden section; 1 image → static image; 2–3 images → `PageView` carousel with dot indicator; tap → `PhotoViewGallery` lightbox with swipe + pinch/double-tap zoom; loading spinner and error placeholder
- [x] 4.5 Add `table_app/test/menu_view_multi_image_test.dart` — widget tests for thumbnail logic (0/1/3 images)
- [x] 4.6 Add `table_app/test/menu_item_detail_screen_test.dart` — widget tests for 0/1/multi image sections and carousel visibility

---

## Dev Agent Record

### Implementation Plan
- Surface 1 (Backend): Computed `ImagePaths` getter on `MenuItem` handles old/new Cosmos docs transparently. Controller validates max-3 before async calls. Tests use `InMemoryManager<Menu>` pattern matching `FeedbackControllerTests`.
- Surface 2 (Shared Package): Manual JSON serialization. `fromJson` checks `imagePaths` first then falls back to legacy `image` string. `toJson` always writes `imagePaths`.
- Surface 3 (Angular): `MenuItem.imagePaths: string[]` replaces `image: any`. 3-slot template inline in modals. Edit modal loads preview via `downloadFileByPath` for each path in `imagePaths`.
- Surface 4 (Table App): `photo_view` for lightbox. `MenuItemCard` becomes tappable, navigating to `MenuItemDetailScreen`. `_loadImages` resolves all paths in `imagePaths` list.

### Completion Notes
- Completed backend `MenuItem.ImagePaths` support with legacy `Image` fallback, controller max-3 validation, and normalization before persistence.
- Completed shared Flutter `MenuItem` contract update and backward-compatible JSON mapping. `imagePaths` is authoritative when present, including an explicit empty list.
- Completed Angular add/edit 3-slot image upload panels with previews, remove behavior, fixed slot ordering, and `imagePaths` payloads.
- Completed table app primary-thumbnail resolution, all-path download resolution, detail carousel, dot indicator, and `PhotoViewGallery` lightbox.
- Story remains **In Progress** because broad regression suites still have failures outside the multi-image implementation: full `table_app` fails existing `table_ordering_widgets_test.dart` presentation assertions, and full Angular Karma fails 7 cross-spec assertions. Targeted story tests pass.

### Debug Log
- 2026-04-25: Corrected partially implemented Dart/C# fallback behavior so explicit `imagePaths: []` is not overridden by a legacy `image` value.
- 2026-04-25: `npm test` without `ChromeHeadless` timed out waiting on Karma; reran with `npx ng test --watch=false --browsers=ChromeHeadless`.
- 2026-04-25: Full `table_app` regression still fails in `test/table_ordering_widgets_test.dart` on `BottomReviewCard` overflow and a line-total text assertion. Targeted multi-image table tests pass.
- 2026-04-25: Full Angular Karma still reports 7 failures outside changed add/edit modal specs. Targeted add/edit modal specs pass.
- 2026-04-26: Corrected menu item image path convention so new website uploads store `menu-items/{itemId}/image-{slot}.{ext}` instead of raw user filenames. Website and table app path resolution preserve legacy filename/account-prefixed paths.

---

## File List

- `MenuNestServer/MenuNestServer/MenuNest.Abstractions/Entities/Menu.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI/Controllers/MenuController.cs`
- `MenuNestServer/MenuNestServer/MenuNestAPI.Tests/MenuControllerImageValidationTests.cs`
- `MenuNestApp/menunest/shared_package/lib/entities/menu.dart`
- `MenuNestApp/menunest/shared_package/lib/services/entity_services/menu_service.dart`
- `MenuNestApp/menunest/shared_package/test/entities/menu_entity_test.dart`
- `MenuNestApp/menunest/shared_package/test/services/entity_managers/order_manager_test.dart`
- `MenuNestApp/menunest/table_app/lib/screens/menu_skin_1/menu_view.dart`
- `MenuNestApp/menunest/table_app/lib/screens/table_ordering/table_ordering_screen.dart`
- `MenuNestApp/menunest/table_app/lib/screens/menu_item_detail_screen.dart`
- `MenuNestApp/menunest/table_app/pubspec.yaml`
- `MenuNestApp/menunest/table_app/pubspec.lock`
- `MenuNestApp/menunest/table_app/test/menu_view_multi_image_test.dart`
- `MenuNestApp/menunest/table_app/test/menu_item_detail_screen_test.dart`
- `MenuNestApp/menunest/table_app/test/table_ordering_widgets_test.dart`
- `MenuNestWebsite/src/app/entities/menu.ts`
- `MenuNestWebsite/src/app/menu-management/add-menu-item-model/add-menu-item-modal.component.ts`
- `MenuNestWebsite/src/app/menu-management/add-menu-item-model/add-menu-item-modal.component.html`
- `MenuNestWebsite/src/app/menu-management/add-menu-item-model/add-menu-item-modal.component.scss`
- `MenuNestWebsite/src/app/menu-management/add-menu-item-model/add-menu-item-modal.component.spec.ts`
- `MenuNestWebsite/src/app/menu-management/edit-menu-item-modal/edit-menu-item-modal.component.ts`
- `MenuNestWebsite/src/app/menu-management/edit-menu-item-modal/edit-menu-item-modal.component.html`
- `MenuNestWebsite/src/app/menu-management/edit-menu-item-modal/edit-menu-item-modal.component.scss`
- `MenuNestWebsite/src/app/menu-management/edit-menu-item-modal/edit-menu-item-modal.component.spec.ts`
- `MenuNestWebsite/src/app/menu-management/menu-management.component.ts`
- `MenuNestWebsite/src/app/menu-management/menu-management.component.spec.ts`
- `MenuNestWebsite/dist/menunestwebsite/browser/index.html`
- `MenuNestWebsite/dist/menunestwebsite/browser/main-B6XNDHL2.js`
- `MenuNestWebsite/dist/menunestwebsite/browser/main-MUXK3K7T.js`
- `MenuNestWebsite/dist/menunestwebsite/browser/main-VYG4V74F.js` (deleted by rebuild)
- `MenuNestApp/menunest/table_app/test/table_ordering_blob_path_test.dart`

---

## Change Log

- 2026-04-25: Continued interrupted implementation, completed multi-image support across backend, Angular website, Flutter shared package, and table app.
- 2026-04-25: Added/updated backend xUnit, Angular Karma, shared package Flutter, and table app Flutter tests for `imagePaths` behavior.
- 2026-04-25: Rebuilt Angular production bundle.
- 2026-04-26: Fixed website/mobile image path mismatch by moving new uploads to item-owned blob folders and verifying legacy path resolution.

---

## Definition of Done

- [x] Backend entity, validation, and serialisation changes implemented and unit-tested
- [x] Angular add/edit modals support 3-slot image upload with working SAS flow
- [x] Flutter shared package entity updated with backward-compat deserialisation
- [x] Table app menu list uses `imagePaths[0]` as primary thumbnail
- [x] Table app product detail shows swipeable carousel for 2–3 images, single image for 1, no image section for 0
- [x] Lightbox opens on image tap with pinch/double-tap zoom and swipe navigation
- [x] All existing tests updated to reflect new entity shape
- [ ] Manual smoke test: create item with 0, 1, 2, and 3 images; verify all surfaces render correctly
- [ ] Backend and shared package deployed together in the same release
