# Backend — Domain (`<Product>.Domain`)

Folders: `Entities/<Module>/`, `Enums/`, `ViewModels/<Module>/`, `Requests/<Module>/`, `DTOs/`, `Helpers/`.
Nullable reference types are **off** in this stack — don't add `string?`/`Product?` annotations.

## Base entity — `Entities/Entity.cs` (exists once per solution)

Every table gets: a version-7 id and who/when audit columns. **A record is deleted when `RemovedBy` is set** (with `RemovedOn`) — there is no `Active` flag. Audit values are set by the repository (`AddAndSaveAsync(entity, userId)`), never by hand in services or controllers.

```csharp
namespace App.Domain.Entities;

/// <summary>
/// Base class for every business entity: version-7 id and audit columns (who/when, UTC).
/// Soft delete = RemovedBy/RemovedOn set; there is no Active flag.
/// </summary>
public abstract class Entity
{
    public virtual Guid Id { get; set; } = Guid.CreateVersion7();

    public string CreatedBy { get; set; }
    public DateTime CreatedOn { get; set; }      // UTC

    public string UpdatedBy { get; set; }
    public DateTime? UpdatedOn { get; set; }     // UTC

    public string RemovedBy { get; set; }
    public DateTime? RemovedOn { get; set; }     // UTC — RemovedBy not null means deleted
}
```

## Domain exception — `Helpers/DomainException.cs`

```csharp
namespace App.Domain.Helpers;

/// <summary>
/// Thrown when a business rule is violated (e.g. an invalid status transition). Mapped to HTTP 409.
/// </summary>
public class DomainException(string message) : Exception(message);
```

## Owner-scoped entity with a status workflow — `Entities/Catalog/Product.cs`

Plain properties (so EF Core and AutoMapper work without ceremony), **except** state that has rules: a status workflow has a `private set` and changes only through methods that enforce the allowed transitions.

```csharp
namespace App.Domain.Entities.Catalog;

/// <summary>
/// A product offered by an owner. Example of an owner-scoped entity with a status workflow.
/// </summary>
public class Product : Entity
{
    public Guid OwnerId { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public ProductStatus Status { get; private set; } = ProductStatus.Draft;

    public virtual User Owner { get; set; }

    /// <summary>Publishes a draft product.</summary>
    public void Publish()
    {
        if (Status != ProductStatus.Draft)
            throw new DomainException($"Only draft products can be published (current status: {Status}).");

        Status = ProductStatus.Published;
    }

    /// <summary>Archives a product that is not archived yet.</summary>
    public void Archive()
    {
        if (Status == ProductStatus.Archived)
            throw new DomainException("Product is already archived.");

        Status = ProductStatus.Archived;
    }
}
```

```csharp
namespace App.Domain.Enums;

public enum ProductStatus
{
    Draft,
    Published,
    Archived
}
```

When a status workflow is business-critical (orders, bookings, payments, payouts), also record every transition in a `<Entity>StatusHistory` entity (from, to, on, by) in the same save.

## Request model and view model — `Requests/Catalog/`, `ViewModels/Catalog/`

Input and output are **separate classes**. A request contains only what the client may set — never `Id`, `OwnerId`, `Status` or audit fields — so nothing can be over-posted. View models don't inherit `Entity`.

```csharp
namespace App.Domain.Requests.Catalog;

/// <summary>Body of the create-product endpoint. Validated by FluentValidation in Application.</summary>
public class ProductRequest
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

```csharp
namespace App.Domain.ViewModels.Catalog;

/// <summary>Product as returned by the API.</summary>
public class ProductViewModel
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public ProductStatus Status { get; set; }
    public DateTime CreatedOn { get; set; }
}
```

```csharp
namespace App.Domain.DTOs;

/// <summary>One page of results plus the total count.</summary>
public class PagedResult<T>
{
    public List<T> Items { get; set; } = [];
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
}
```
