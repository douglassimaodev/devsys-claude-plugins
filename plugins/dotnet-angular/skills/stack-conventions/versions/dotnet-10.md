# Version notes — .NET 10 (LTS, supported until Nov 2028)

C# 14 · EF Core 10 · ASP.NET Core 10. Read after confirming the project targets `net10.0`.
These notes cover what models tend to get wrong because they default to older .NET. When unsure about an API, check the official docs (Microsoft Learn) instead of guessing, and add the finding here.

## Use this, not that

| Use | Not | Why |
|---|---|---|
| Primary constructors for simple DI classes | Field + constructor boilerplate | Project convention (C# 12+) |
| Collection expressions `[]`, `[..a, ..b]` | `new List<T>()`, `Array.Empty<T>()` | C# 12+ |
| `field` keyword in a property accessor when you need a backing field | A hand-written private backing field | C# 14 |
| Null-conditional assignment `order?.Status = x` where it reads clearly | `if (order is not null) order.Status = x` | C# 14 |
| `Guid.CreateVersion7()` | `Guid.NewGuid()` for entity ids | Project convention; sortable ids |
| Injected `TimeProvider` (`FakeTimeProvider` in tests) for time-based business rules in services | `DateTime.Now`, or `DateTime.UtcNow` inside rule logic you need to test | Testable time; UTC always (the repository stamping audit columns with `DateTime.UtcNow` is fine) |
| `async`/`await` all the way | `.Result`, `.Wait()`, `async void` | Deadlocks, lost exceptions |
| `System.Text.Json` | Newtonsoft.Json (unless the project already uses it) | Framework default |
| The OpenAPI package the project already uses | Adding a second OpenAPI generator | Templates since .NET 9 ship `Microsoft.AspNetCore.OpenApi` instead of Swashbuckle — follow the repository |
| `.slnx` if the repo already uses it | Converting the solution format unasked | .NET 10 SDK creates `.slnx` by default |

## EF Core 10
- **Named query filters** let an entity have more than one global filter and disable one by name — a fit for the soft-delete filter (`RemovedBy == null`) next to other filters. Use them only if the project/ADR adopts them; otherwise follow the existing filter style.
- **`LeftJoin` / `RightJoin` LINQ operators** exist — prefer them over the `GroupJoin` + `SelectMany` + `DefaultIfEmpty` pattern when writing a left join.
- `Microting.EntityFrameworkCore.MySql` must be the build that matches EF Core 10 — check the package version before adding or upgrading.
- Migrations: always pass the context explicitly (`--context AppDbContext` or `--context ApplicationDbContext`) — this stack has two.

## Add findings here
When the senior reviewer or QA finds a .NET-10-specific correction that keeps coming back, add it to the table above.
