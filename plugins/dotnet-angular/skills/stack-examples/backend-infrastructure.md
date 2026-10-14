# Backend — Infrastructure (`<Product>.Infra`)

Folders: `Context/`, `EntityMapping/<Module>/`, `Interfaces/<Module>/`, `Repositories/<Module>/`, `Migrations/`, `ServiceConfiguration.cs`, `GlobalUsing.cs`.

Identity lives in its own `ApplicationDbContext : IdentityDbContext` (stock tables, own migrations). The business context below holds business data only.

## Business DbContext — `Context/AppDbContext.cs`

Global conventions live here so individual mappings stay short. They run **after** the mappings, so nothing in a `*Map` can override them (e.g. an accidental `OnDelete(Cascade)`).

```csharp
namespace App.Infra.Context;

/// <summary>
/// Business data context. Applies the mappings, then the global conventions, and protects audit columns on save.
/// </summary>
public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<User> Users { get; set; }
    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        builder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);

        // All FKs Restrict — applied after the mappings so none can opt back into cascade
        foreach (var relationship in builder.Model.GetEntityTypes().SelectMany(e => e.GetForeignKeys()))
            relationship.DeleteBehavior = DeleteBehavior.Restrict;

        // Strings without an explicit length or type default to 250, non-unicode
        foreach (var property in builder.Model.GetEntityTypes().SelectMany(e => e.GetProperties())
                     .Where(p => p.ClrType == typeof(string) && p.GetMaxLength() == null && p.GetColumnType() == null))
        {
            property.SetMaxLength(250);
            property.SetIsUnicode(false);
        }

        // Guids as char(36) with ASCII collation
        foreach (var property in builder.Model.GetEntityTypes().SelectMany(e => e.GetProperties())
                     .Where(p => p.ClrType == typeof(Guid)))
        {
            property.SetColumnType("char(36)");
            property.SetCollation("ascii_general_ci");
        }

        // Table name = class name (no pluralisation)
        foreach (var entityType in builder.Model.GetEntityTypes().Where(e => e.ClrType != null))
            entityType.SetTableName(entityType.ClrType.Name);
    }

    /// <summary>Saves after protecting audit columns from being overwritten by the wrong operation.</summary>
    public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        ProtectAuditColumns();
        return base.SaveChangesAsync(cancellationToken);
    }

    /// <summary>Saves after protecting audit columns from being overwritten by the wrong operation.</summary>
    public override int SaveChanges()
    {
        ProtectAuditColumns();
        return base.SaveChanges();
    }

    private void ProtectAuditColumns()
    {
        ChangeTracker.DetectChanges();

        foreach (var entry in ChangeTracker.Entries<Entity>())
        {
            if (entry.State == EntityState.Added)
            {
                entry.Property(x => x.UpdatedBy).IsModified = false;
                entry.Property(x => x.UpdatedOn).IsModified = false;
                entry.Property(x => x.RemovedBy).IsModified = false;
                entry.Property(x => x.RemovedOn).IsModified = false;
            }
            else if (entry.State == EntityState.Modified)
            {
                entry.Property(x => x.CreatedBy).IsModified = false;
                entry.Property(x => x.CreatedOn).IsModified = false;
            }
        }
    }
}
```

Migrations always name the context (there are two):

```bash
dotnet ef migrations add AddProduct --context AppDbContext --project src/App.Infra --startup-project src/App.Api
```

## Base mapping — `EntityMapping/BaseEntityMap.cs` (exists once)

```csharp
namespace App.Infra.EntityMapping;

/// <summary>Mapping shared by every entity: key, audit column sizes and the soft-delete filter (RemovedBy not null = deleted).</summary>
public class BaseEntityMap<TEntity> : IEntityTypeConfiguration<TEntity> where TEntity : Entity
{
    /// <summary>Configures the columns every entity inherits from <see cref="Entity"/>.</summary>
    public virtual void Configure(EntityTypeBuilder<TEntity> builder)
    {
        builder.HasKey(x => x.Id);
        builder.Property(x => x.Id).IsRequired().HasDefaultValueSql("(uuid())");

        builder.Property(x => x.CreatedBy).HasMaxLength(95).IsRequired();
        builder.Property(x => x.UpdatedBy).HasMaxLength(95);
        builder.Property(x => x.RemovedBy).HasMaxLength(95);

        builder.HasQueryFilter(x => x.RemovedBy == null && x.RemovedOn == null); // Soft delete: RemovedBy set = deleted
    }
}
```

## Entity mapping — `EntityMapping/Catalog/ProductMap.cs`

```csharp
namespace App.Infra.EntityMapping.Catalog;

/// <summary>EF Core mapping for <see cref="Product"/>.</summary>
public class ProductMap : BaseEntityMap<Product>
{
    /// <summary>Configures product columns, relationships and indexes.</summary>
    public override void Configure(EntityTypeBuilder<Product> entity)
    {
        base.Configure(entity);

        entity.Property(x => x.Name).HasMaxLength(200).IsRequired();
        entity.Property(x => x.Price).HasColumnType("decimal(18,2)");
        entity.Property(x => x.Status).HasConversion<string>().HasMaxLength(32);

        entity.HasOne(x => x.Owner)
              .WithMany()
              .HasForeignKey(x => x.OwnerId); // Restrict is applied globally — never set Cascade here

        entity.HasIndex(x => new { x.OwnerId, x.Status });
    }
}
```

## Generic repository — `Interfaces/IRepository.cs`, `Repositories/Repository.cs` (exists once)

Every module repository inherits it. It is the only place that stamps audit columns. Shown here trimmed to the members the examples use — the project's real base may have more; don't recreate it.

```csharp
namespace App.Infra.Interfaces;

/// <summary>Common data access for every entity, including audit stamping and soft delete.</summary>
public interface IRepository<T> where T : Entity
{
    IQueryable<T> GetQueryable();
    Task<int> AddAndSaveAsync(T entity, string userId, CancellationToken cancellationToken = default);
    Task<int> UpdateAndSaveAsync(T entity, string userId, CancellationToken cancellationToken = default);
    Task<int> SoftDeleteAsync(Guid id, string userId, CancellationToken cancellationToken = default);
}
```

```csharp
namespace App.Infra.Repositories;

/// <summary>EF Core implementation of <see cref="IRepository{T}"/>.</summary>
public class Repository<T>(AppDbContext db) : IRepository<T> where T : Entity
{
    protected readonly AppDbContext Db = db;
    protected DbSet<T> DbSet => Db.Set<T>();

    /// <summary>Queryable over the set (soft-deleted rows are filtered out by the query filter).</summary>
    public IQueryable<T> GetQueryable() => DbSet.AsQueryable();

    /// <summary>Stamps CreatedBy/CreatedOn, adds and saves.</summary>
    public virtual async Task<int> AddAndSaveAsync(T entity, string userId, CancellationToken cancellationToken = default)
    {
        entity.CreatedBy = userId;
        entity.CreatedOn = DateTime.UtcNow;
        await DbSet.AddAsync(entity, cancellationToken);
        return await Db.SaveChangesAsync(cancellationToken);
    }

    /// <summary>Stamps UpdatedBy/UpdatedOn and saves. Works for tracked and detached entities.</summary>
    public virtual Task<int> UpdateAndSaveAsync(T entity, string userId, CancellationToken cancellationToken = default)
    {
        entity.UpdatedBy = userId;
        entity.UpdatedOn = DateTime.UtcNow;

        var tracked = DbSet.Local.FirstOrDefault(e => e.Id == entity.Id);
        if (tracked != null)
            Db.Entry(tracked).CurrentValues.SetValues(entity);
        else
            DbSet.Update(entity);

        return Db.SaveChangesAsync(cancellationToken);
    }

    /// <summary>Soft-deletes by stamping RemovedBy/RemovedOn. Rows are never hard-deleted.</summary>
    public virtual async Task<int> SoftDeleteAsync(Guid id, string userId, CancellationToken cancellationToken = default)
    {
        var entity = await DbSet.FirstOrDefaultAsync(x => x.Id == id, cancellationToken);
        if (entity == null)
            return 0;

        entity.RemovedBy = userId;
        entity.RemovedOn = DateTime.UtcNow;
        return await Db.SaveChangesAsync(cancellationToken);
    }
}
```

## Module repository — `Interfaces/Catalog/IProductRepository.cs`, `Repositories/Catalog/ProductRepository.cs`

Every method that reads or loads for change takes the owner (or tenant) id and filters **in the query**.

```csharp
namespace App.Infra.Interfaces.Catalog;

/// <summary>Product data access. Every query is scoped to an owner.</summary>
public interface IProductRepository : IRepository<Product>
{
    Task<PagedResult<Product>> GetPageByOwnerAsync(Guid ownerId, int page, int pageSize);
    Task<Product> GetAsync(Guid id, Guid ownerId);
}
```

```csharp
namespace App.Infra.Repositories.Catalog;

/// <summary>EF Core implementation of <see cref="IProductRepository"/>.</summary>
public class ProductRepository(AppDbContext db) : Repository<Product>(db), IProductRepository
{
    /// <summary>One page of the owner's products, newest first, read-only.</summary>
    public async Task<PagedResult<Product>> GetPageByOwnerAsync(Guid ownerId, int page, int pageSize)
    {
        var query = DbSet.AsNoTracking().Where(x => x.OwnerId == ownerId);

        return new PagedResult<Product>
        {
            Items = await query
                .OrderByDescending(x => x.CreatedOn)
                .ThenBy(x => x.Id) // Stable order for paging
                .Skip((page - 1) * pageSize)
                .Take(pageSize)
                .ToListAsync(),
            Page = page,
            PageSize = pageSize,
            TotalCount = await query.CountAsync()
        };
    }

    /// <summary>Loads a product for change only if it belongs to the owner (tracked).</summary>
    public Task<Product> GetAsync(Guid id, Guid ownerId) =>
        DbSet.FirstOrDefaultAsync(x => x.Id == id && x.OwnerId == ownerId); // Ownership in the query, never after loading
}
```

## Registration — `ServiceConfiguration.cs`

```csharp
namespace App.Infra;

public static class ServiceConfiguration
{
    /// <summary>Registers the business DbContext and every repository.</summary>
    public static IServiceCollection AddInfra(this IServiceCollection services)
    {
        services.AddScoped<IProductRepository, ProductRepository>();
        // One line per repository, grouped by module with a comment header
        return services;
    }
}
```

`Program.cs` registers both contexts against the same connection string:

```csharp
var serverVersion = new MariaDbServerVersion(new Version(mariaDbMajor, mariaDbMinor));
builder.Services.AddDbContext<AppDbContext>(o => o.UseMySql(connectionString, serverVersion, x => x.EnableRetryOnFailure()));
builder.Services.AddDbContext<ApplicationDbContext>(o => o.UseMySql(connectionString, serverVersion, x => x.EnableRetryOnFailure()));
builder.Services.AddInfra();
```
