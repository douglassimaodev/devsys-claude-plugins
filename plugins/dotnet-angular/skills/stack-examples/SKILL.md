---
name: stack-examples
description: Reference code for every core pattern of the .NET + EF Core + Identity + Angular stack — base entity, owner-scoped entity with status workflow, EF configuration, repository, service + validator, controller, error handling, xUnit/NSubstitute tests, ApiService, feature service, signal page and Vitest test. Load before implementing a pattern for the first time or when no example exists in the codebase yet.
---

# Stack Examples

Complete, minimal examples of how this stack is built. `Product` and `OwnerId` are placeholder names — map them to the project's own entities and ownership rules from `project-description.md`.

**Which example wins:**
1. Code that already exists in the repository for the same kind of thing (especially subtasks marked `Pattern-setting: yes` once done).
2. These examples.
Never mix two styles in one codebase — if the repository differs from these examples, follow the repository and mention the difference in your handoff.

| File | Contents |
|---|---|
| `backend-domain.md` | `Entity` base class (Guid v7, CreatedBy/On, UpdatedBy/On, RemovedBy/On — RemovedBy set = deleted), `DomainException`, owner-scoped entity with a status workflow, request model vs view model, `PagedResult<T>` |
| `backend-infrastructure.md` | `AppDbContext` (global conventions, audit-column protection), `BaseEntityMap<T>` + `<Entity>Map`, generic `IRepository<T>`/`Repository<T>` (audit stamping, soft delete), module repository scoped by owner, `ServiceConfiguration.AddInfra()` |
| `backend-application-api.md` | FluentValidation validator, AutoMapper 14 profile, application service, `BaseController`, controller with named verb routes, exception handler → ProblemDetails, Hangfire job pattern |
| `backend-tests.md` | Service tests and controller tests (xUnit + NSubstitute + FluentAssertions 7, real AutoMapper profile, claims setup) |
| `frontend.md` | `ApiService`, model, feature service, signal-based page with loading/empty/error states, template with built-in control flow, Vitest component test |

Read only the files relevant to the subtask.
