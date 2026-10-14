---
name: stack-conventions
description: Baseline conventions for .NET (latest LTS) + ASP.NET Core + EF Core + MariaDB + Angular (latest stable major) projects, including the version policy — layers, persistence, Identity, validation, frontend patterns, anti-patterns, XML docs, tests and commands. Load before designing, implementing or reviewing code in this stack.
---

# Stack Conventions: .NET + Angular

Baseline for every project on this stack. **`project-description.md`, `docs/Architecture.md` and `docs/ADR/` override anything here** — check them first.

## Stack
- Frontend: Angular (latest stable major), TypeScript, Signals, standalone components, Syncfusion (`@syncfusion/ej2-angular-*`), SCSS.
- Backend: .NET (latest LTS), ASP.NET Core, Entity Framework Core, FluentValidation, ASP.NET Core Identity.
- Database: MariaDB via `Microting.EntityFrameworkCore.MySql` (not Oracle's `MySql.EntityFrameworkCore`).
- Also: AutoMapper 14.x, Hangfire (MySQL storage), Serilog, Sentry.
- Hosting: local-first; cloud-ready (Azure, Azure DevOps YAML pipelines) later.

## Version Policy
- **.NET: LTS releases only** (the even-numbered majors). Never target an STS (odd-numbered) release, preview or RC.
- **Angular: the latest stable major** (every Angular major receives an LTS period). Never a `next`/RC version.
- **New project:** start on the current .NET LTS and the latest stable Angular major. Check the official support pages before scaffolding — don't rely on memory.
- **Existing project:** use exactly the versions the repository pins — `global.json` and `<TargetFramework>` in the `.csproj` files for .NET, `@angular/core` in `package.json` for Angular. Read them before writing code; never assume a version.
- **Never upgrade a framework major without an explicit request.** An upgrade is its own milestone (planned with `plan-milestones`), never a side effect of a feature.
- **Companion packages move together:** EF Core, `Microting.EntityFrameworkCore.MySql`, ASP.NET Core packages and the .NET SDK share one major; Angular packages, Angular CLI, Syncfusion `ej2-angular-*` and TypeScript must be the versions supported by that Angular major. Mismatches are a review BLOCKER.
- **Version notes:** after detecting the versions, read the matching notes in this skill's `versions/` folder (e.g. `versions/dotnet-10.md`, `versions/angular-22.md`). They list the "use this, not that" rules for that version. If no file matches the project's version, say so and check the official docs instead of guessing.

## Backend Architecture
- Layered Modular Monolith, Domain-Driven Design where useful, event-driven where useful.
- **No CQRS/MediatR unless the user explicitly asks for it** — straight-line services in Application.
- Dependency direction: `API → Application → Infrastructure → Domain` (straight line, no inversion).
  - **API** — controllers, auth/JWT plumbing, webhook endpoints, hosted adapters (email, push), DI composition root.
  - **Application** — use-case services (names may mirror API endpoints), all validation (FluentValidation), business orchestration.
  - **Infrastructure** — everything database: DbContexts, EF configurations, migrations, seeders, repository interfaces + implementations, file storage, external gateway clients.
  - **Domain** — entities, enums, DTOs, viewmodels, request models, shared abstractions. No dependencies (base Identity types only).
- Application consumes `I*Repository` from Infrastructure, and may use `AppDbContext` directly when a repository adds no value.
- **Controllers stay thin** — no use-case logic in action bodies; it lives in Application services. (Older codebases on this stack put logic in controllers; don't copy that into new code.)
- Project naming: `<Product>.Api`, `<Product>.Application`, `<Product>.Infra`, `<Product>.Domain` (and `<Product>.Shared` for resources when localized); tests in `tests/<Product>.ApiUnitTest`. Actual names are in `project-description.md`.

### Folder layout
| Project | Folders |
|---|---|
| Domain | `Entities/<Module>/`, `Enums/`, `Requests/<Module>/`, `ViewModels/<Module>/`, `DTOs/`, `Helpers/` |
| Infra | `Context/`, `EntityMapping/<Module>/` (`<Entity>Map`), `Interfaces/<Module>/`, `Repositories/<Module>/`, `Migrations/`, `ServiceConfiguration.cs` (`AddInfra()`), `GlobalUsing.cs` |
| Application | `<Module>/` (service + interface + validators), `Mapping/` (AutoMapper profile), `ServiceConfiguration.cs` (`AddApplication()`) |
| Api | `Controllers/BaseController.cs`, `Controllers/v1/<Module>/`, `Errors/`, `Options/`, `Services/` (infrastructure adapters and Hangfire jobs), `Program.cs` (composition root) |

## Persistence Conventions
- Always EF Core. Two contexts on the same connection: `AppDbContext` (business) and `ApplicationDbContext : IdentityDbContext` (Identity). Migrations always pass `--context`.
- **Base `Entity`** on every business table: `Id` (`Guid.CreateVersion7()`, DB default `uuid()`), `CreatedBy`/`CreatedOn`, `UpdatedBy`/`UpdatedOn`, `RemovedBy`/`RemovedOn` (UTC). **No `Active` flag.** `*By` columns hold the Identity user id (max 95).
- **Audit columns are stamped only by the repository** (`AddAndSaveAsync(entity, userId)`, `UpdateAndSaveAsync(entity, userId)`), and `AppDbContext` protects them from being overwritten by the wrong operation on save.
- **Soft delete:** a record is deleted when `RemovedBy` is set (with `RemovedOn`), via the repository's `SoftDeleteAsync`. There is no `Active` flag — never add one to the base entity or use one for deletion. `BaseEntityMap<T>` adds the global query filter (`RemovedBy == null && RemovedOn == null`). Never hard-delete business data. If a feature needs "enabled/disabled" (e.g. a listing temporarily hidden), that is a separate, explicitly named business field (`IsPublished`, `Status`), not deletion.
- **Global conventions in `AppDbContext.OnModelCreating`**, applied *after* `ApplyConfigurationsFromAssembly`: all FKs `Restrict`; strings without explicit length/type default to 250 non-unicode; Guids `char(36)` + `ascii_general_ci`; table name = class name.
- **Mappings:** one `<Entity>Map : BaseEntityMap<Entity>` per entity, calling `base.Configure(entity)` first. Set explicit lengths for real text fields; never set `OnDelete(Cascade)`.
- **Repositories:** generic `IRepository<T>`/`Repository<T>` base; one `I<Entity>Repository : IRepository<Entity>` + implementation per entity, registered in `ServiceConfiguration.AddInfra()`. Every read or load-for-change method takes the owner/tenant id and filters in the query. Read-only queries use `AsNoTracking()`.
- Enum columns: `HasConversion<string>()` by default — check the entity's mapping before assuming, since that decides whether new enum values are safe anywhere or append-only.
- Dates are stored in UTC; convert to the project's timezone only for display and date-based rules.
- Money: `decimal` (`decimal(18,2)` columns), never `double`/`float`, explicit rounding rules.
- **Entities:** plain properties, except state with rules — a status workflow uses `private set` and changes only through methods that enforce the allowed transitions (throwing `DomainException`).
- **Requests vs view models:** input `*Request` classes contain only client-settable fields (never `Id`, owner ids, `Status`, audit fields); output `*ViewModel` classes don't inherit `Entity`.

## Identity Conventions
- Always ASP.NET Core Identity for authentication/authorization — never hand-roll hashing, credential storage, or confirmation/reset tokens. External logins (Google/Apple…) go through Identity's external login support.
- **Identity tables are stock.** Never subclass `IdentityUser`, add/rename columns, indexes or tables in `AspNet*`, or change the Identity key type.
- Identity has its own `ApplicationDbContext`; business data uses `AppDbContext`; same database, separate migrations.
- Profile data lives in a business `User` entity sharing the same id as `AspNetUsers.Id` (`User.Id == Guid.Parse(identityUser.Id)`, no separate link column). One email per account (Identity `RequireUniqueEmail` + unique index on `User.Email`).
- Refresh tokens live in a business `RefreshToken` table, not in `AspNetUserTokens`.

## Backend Code Profile
- Controller-first API. Every controller inherits `BaseController` (`[Authorize]` with JWT bearer + claim accessors `UserId`, `UserIdString`, …), so endpoints are authenticated by default; `[AllowAnonymous]` requires a rate limit.
- **Routes:** `[Route("api/v1/[controller]")]` (plus `[Route("{culture:culture}/api/v1/[controller]")]` in localized projects) with **named verb segments**: `GetX`, `GetById/{id:guid}`, `Create`, `Update/{id:guid}`, `<Action>/{id:guid}`, `Delete/{id:guid}`.
- OpenAPI with Swashbuckle (security scheme for JWT), enabled in Development only; `[ProducesResponseType]` on every action.
- **Mapping:** AutoMapper, **pinned to 14.x** (15+ is commercially licensed). Profiles in Application; tests use the real profile.
- **Errors:** `ValidationException` → 400 and `DomainException` → 409 as ProblemDetails via one `IExceptionHandler`; not-found/not-yours → 404 from the controller.
- **MediatR is not used** (no CQRS by default, and 13+ is commercially licensed) — don't add or register it.
- Modern C#: primary constructors for simple DI classes when it improves readability, nullability, pattern matching, string interpolation, async/await throughout (`async Task`, never `async void`).
- **Validation**: FluentValidation validators in Application, one per request type, validating only body fields — not route data, headers or context.
- **Ownership checks**: every user-scoped endpoint filters by the authenticated user's own id/owner id in the query itself, not after loading.
- **Server-side money**: totals, fees, discounts and commissions are calculated in Application — never accepted from the request body.
- **External gateways** (payments etc.) are reached through an Application-level abstraction; webhook-driven status changes are signature-verified, idempotent (dedupe by provider event id) and recorded in a status history.
- **Background work** (push, email, reminders, payouts) runs as **Hangfire** jobs with MariaDB storage: one interface per job (`I<Name>Job`), enqueued or scheduled **after** the save succeeds; job ids that must be cancelled later are stored on the entity. A scheduling failure is logged and never fails the request. The dashboard is protected outside Development.

### XML Documentation
- Every `public` or `internal` method or class you create or modify must have an XML `<summary>`. If one exists, preserve it exactly — never remove or shorten it.
- Private methods need a `<summary>` only when the logic is non-obvious.
- Properties do not require XML summaries.
- Preserve existing `//` comments in files you touch.
- Add an inline `//` comment only on genuinely non-obvious lines (hidden constraint, subtle invariant, workaround).

## Frontend Architecture
- Angular (version per the policy above), TypeScript, standalone components only — no NgModules.
- Mobile-first and responsive.
- State in component `signal()`s, or in an `@Injectable({ providedIn: 'root' })` service exposing signals/methods when shared. No NgRx, no bare RxJS `BehaviorSubject` state stores.
- DI via `inject()`, not constructor injection.
- All HTTP calls go through a single `core/services/api.service.ts` (`ApiService`, wrapping `HttpClient` with typed `get`/`post`/`put`/`patch`). Create it once if missing; never inject `HttpClient` into a feature service or component.
- Feature folders by area and domain: `features/<area>/<domain>/` (areas listed in `project-description.md`, e.g. `public|client|admin`), each with a `*.service.ts`, a `*.model.ts` (plain interfaces/types, not classes), and `pages/<page-name>/` components.
- Reactive Forms (`FormBuilder`) for anything with validation; plain `ngModel` is fine for a simple free-text field like a search box.
- UI components from Syncfusion; SCSS using the design tokens under `src/sass` — no hardcoded hex colours.
- UI copy language, currency and date formats come from `project-description.md`; code identifiers are English.
- Cross-cutting UX (toasts, confirm dialogs) through shared `ToastService` / `ConfirmService`, never `alert()`/`confirm()`.
- Route guards protect role-specific routes; feature routes are lazy-loaded.
- Display amounts returned by the API; the frontend may preview but never decides the amount charged.

## Anti-Patterns

| Anti-pattern | Why it's forbidden |
|---|---|
| `HttpClient` injected in a component or feature service | Bypasses `ApiService` and centralised error handling |
| NgRx or a bare `BehaviorSubject` state store | Use signals |
| Hardcoded hex colours in SCSS | Use the design tokens |
| `console.log` left in shipped code | Remove before delivery |
| CQRS/MediatR without an explicit ask | "No CQRS by default" rule |
| Bypassing FluentValidation for body validation | Validation lives in Application |
| Trusting prices/totals computed in the frontend | Totals are authoritative only on the server |
| Marking something Paid/Confirmed from a client-side redirect | Must come from a verified webhook |
| `double`/`float` for money | Rounding errors in financial records |
| Changing Identity tables or subclassing `IdentityUser` | Identity is stock and separate |

## Tests
- Backend: xUnit, NSubstitute, FluentAssertions (see `create-unit-tests`).
- **FluentAssertions licensing:** version 8+ is commercially licensed (paid for commercial use); 7.x remains Apache 2.0. Keep the major the repository already pins (7.x unless the project records a license), or use the API-compatible fork `AwesomeAssertions` if the project chose it. Never upgrade FluentAssertions to 8+ as a side effect.
- Frontend: Vitest via `npm test` (`ng test` under the hood — use `--watch=false` for a single run, not `--run`).
- E2E: Playwright via `npm run e2e` — needs a live backend + database; not part of the standard build gate.

## Related skills
- `stack-pitfalls` — the mistakes that actually happen in this stack (used by the senior reviewer and QA; useful for the developer too).

## Commands
```bash
# Backend
dotnet build
dotnet test

# Frontend (npm only)
npm install
npm start
npm run build
npm test -- --watch=false
npm run e2e
```
Check `package.json` and `project-description.md` for the scripts and paths that actually exist before assuming one.
