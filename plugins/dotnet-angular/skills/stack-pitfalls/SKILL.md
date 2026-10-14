---
name: stack-pitfalls
description: Checklist of the mistakes that actually happen in .NET + EF Core + ASP.NET Core Identity + MariaDB + Angular code — what the senior reviewer and QA look for, and what the developer should avoid. Load before reviewing or validating code in this stack.
---

# Stack Pitfalls: .NET + Angular

Each item: what goes wrong → what to look for. Classify a hit as BLOCKER when it breaks security, money, ownership, data integrity or Identity rules; otherwise MUST FIX.

## EF Core / data access
- **Ownership checked after loading** → `FirstOrDefaultAsync(x => x.Id == id)` followed by `if (x.PartnerId != userPartnerId)`. The owner filter must be in the query.
- **Filtering in memory** → `.ToList()` / `.AsEnumerable()` before `.Where()`, `.Count()`, paging.
- **N+1 queries** → navigation properties read inside a loop; missing `Include` or projection.
- **Cartesian explosion** → several collection `Include`s in one query; use `AsSplitQuery()` or a projection.
- **Tracking on read-only queries** → missing `AsNoTracking()` (or a `Select` projection) on list/detail reads.
- **Entities returned from controllers** → domain entities serialized directly instead of DTOs (over-posting, leaked fields, cycles).
- **Missing pagination** on list endpoints, or `Skip/Take` without a stable `OrderBy`.
- **Cascade delete** sneaking in → `OnDelete(Cascade)` in a `*Map` (if the global Restrict loop runs before `ApplyConfigurationsFromAssembly`, the mapping wins); hard `Remove()`/`RemoveAndSaveAsync` on business data.
- **Wrong soft-delete marker** → an `Active`/`IsDeleted` flag added for deletion, or a query/report checking such a flag; deleted means `RemovedBy` is set (with `RemovedOn`), through `SoftDeleteAsync`.
- **Soft delete bypassed** → `IgnoreQueryFilters()` without a reason, or a raw SQL/report query that doesn't exclude rows with `RemovedBy` set.
- **Audit columns set by hand** in services/controllers instead of through the repository's `AddAndSaveAsync`/`UpdateAndSaveAsync`.
- **Over-posting** → a view model that inherits `Entity`, or AutoMapper mapping a request that contains `Id`, owner/tenant ids, `Status` or audit fields onto an entity.
- **Non-unicode text** → the 250/non-unicode string default applied to user-entered text (names, descriptions with accents); set explicit lengths and check the column charset stores the project's languages.
- **Tenant-only check on user-owned data** → a record found by tenant/church is not necessarily the caller's; when only the owner (e.g. the assigned member) may act, check that id too.
- **Filtering in memory after a broad load** → loading all candidates with `Include` then checking overlaps/conditions in C#; push the condition into the query.
- **Migration in the wrong context**, or a business migration touching `AspNet*` tables.
- **Enum stored as int** where the convention is `HasConversion<string>()`, or a new enum value inserted mid-list on an int-stored enum.
- **Money as `double`/`float`**, missing `decimal(18,2)` column type, rounding done implicitly.
- **Concurrency** → availability/stock checks done with read-then-write and no unique constraint or concurrency token.
- **`DateTime.Now` / local time** stored instead of UTC.
- **`SaveChangesAsync` inside loops**, or several `SaveChanges` where one unit of work was needed.

## ASP.NET Core / Identity / security
- **Identity tables modified** → `IdentityUser` subclass, extra columns, changed key type.
- **Profile data in Identity** instead of the business `User`; `User.Id` not equal to the Identity id.
- **Hand-rolled auth** → custom hashing, custom reset/confirmation tokens, JWT validation shortcuts (`ValidateLifetime = false`, `ValidateIssuer`/`ValidateAudience` = false, signing key in source).
- **Secrets committed** → connection strings, JWT keys, gateway/push keys in `appsettings*.json` or pipeline YAML; use user secrets locally and the pipeline's secret variables/Key Vault.
- **Claim self-grant / registration into another tenant** → endpoints that let a user set their own permissions, roles or tenant/church id from the request body.
- **Anonymous endpoints without rate limiting**, or that disclose tokens (password-reset tokens must only be emailed, never returned).
- **Refresh tokens** not rotated, not revoked on reuse, or stored in `AspNetUserTokens`.
- **Missing `[Authorize]` / role or policy** on a new controller or action; `[AllowAnonymous]` on something that changes data.
- **Amounts trusted from the request body** (price, total, fee, discount, commission).
- **Webhooks** without signature verification or idempotency (no unique provider event id).
- **Secrets** in `appsettings.json` committed, or logged.
- **Personal data logged** (documents, bank data, tokens, full request bodies).
- **Validation in controllers** instead of FluentValidation in Application; validators reading route/headers.
- **Business logic in controllers.**
- **Exceptions for control flow** leaking 500s for expected cases (not found, forbidden, validation).

## C# / async
- `async void`, `.Result`, `.Wait()`, fire-and-forget `Task.Run` instead of a Hangfire job.
- Missing `CancellationToken` propagation on async repository/service methods that already accept one elsewhere.
- Removed or shortened XML `<summary>` blocks; public/internal members created without one.

## Angular
- **`HttpClient` injected** outside `ApiService`.
- **Effects writing signals** to derive state → should be `computed()` / `linkedSignal()`.
- **Leaking subscriptions** → `subscribe()` without `takeUntilDestroyed()` or the `async`/signal equivalent.
- **Old syntax by habit** → `*ngIf`, `*ngFor`, `@Input()`, constructor injection, `standalone: true`, NgModules.
- **`@for` without a meaningful `track`** (tracking by index on mutable lists).
- **Charged totals computed in the frontend.**
- **Missing loading / empty / error states.**
- **Hardcoded colours**, inline styles, `console.log` left behind.
- **Route without guard** for a role-specific area; feature not lazy-loaded.
- **`innerHTML` / `bypassSecurityTrust*`** with user content.
- **Unhandled API errors** → no error state, no toast via `ToastService`.

## Tests
- Tests that pass without asserting the behaviour (no assertion, or asserting the mock's own setup).
- Missing negative cases: validation failure, forbidden/other owner, invalid status transition, duplicate webhook.
- Time-dependent tests using the real clock.
- Shared mutable state between tests; order-dependent tests.
- `Received()` checks missing for important side effects (or `DidNotReceive()` missing for the failure path).

## Versions
- AutoMapper upgraded to 15+ or MediatR (13+) added — both commercially licensed.
- FluentAssertions bumped to 8+ (commercial license) without a recorded decision.
- A framework major or companion package changed without an upgrade milestone.
- EF Core / provider / ASP.NET Core majors out of sync; Syncfusion or TypeScript outside the Angular major's supported range.
