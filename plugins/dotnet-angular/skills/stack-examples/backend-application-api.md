# Backend — Application (`<Product>.Application`) and API (`<Product>.Api`)

Controllers stay thin: auth, routing, status codes. Use-case logic lives in Application services — never in controller actions.

## Validator — `Application/Catalog/ProductRequestValidator.cs`

Validates body fields only — never route values, headers or the current user. Messages come from the project's resource file when the project is localized.

```csharp
namespace App.Application.Catalog;

/// <summary>Validates the body of a create-product request.</summary>
public class ProductRequestValidator : AbstractValidator<ProductRequest>
{
    public ProductRequestValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(200);
        RuleFor(x => x.Price).GreaterThanOrEqualTo(0).PrecisionScale(18, 2, ignoreTrailingZeros: true);
    }
}
```

## Mapping profile — `Application/Mapping/AppMappingProfile.cs`

AutoMapper is pinned to **14.x** (the last MIT-licensed major — 15+ is commercially licensed).

```csharp
namespace App.Application.Mapping;

/// <summary>AutoMapper profile for request → entity and entity → view model maps.</summary>
public class AppMappingProfile : Profile
{
    public AppMappingProfile()
    {
        CreateMap<ProductRequest, Product>(); // Requests carry only client-settable fields
        CreateMap<Product, ProductViewModel>();
    }
}
```

## Application service — `Application/Catalog/ProductService.cs`

The current user's id comes in as a parameter from the controller (taken from the token), never from the body.

```csharp
namespace App.Application.Catalog;

/// <summary>Use cases for the current user's products.</summary>
public interface IProductService
{
    Task<PagedResult<ProductViewModel>> GetMyPageAsync(Guid userId, int page, int pageSize);
    Task<ProductViewModel> GetMineAsync(Guid id, Guid userId);
    Task<ProductViewModel> CreateAsync(ProductRequest request, Guid userId);
    Task<ProductViewModel> PublishAsync(Guid id, Guid userId);
    Task<bool> DeleteAsync(Guid id, Guid userId);
}

/// <summary>Default implementation of <see cref="IProductService"/>.</summary>
public class ProductService(
    IProductRepository products,
    IMapper mapper,
    IValidator<ProductRequest> validator) : IProductService
{
    private const int MaxPageSize = 100;

    /// <summary>One page of the user's products.</summary>
    public async Task<PagedResult<ProductViewModel>> GetMyPageAsync(Guid userId, int page, int pageSize)
    {
        page = Math.Max(page, 1);
        pageSize = Math.Clamp(pageSize, 1, MaxPageSize);

        var result = await products.GetPageByOwnerAsync(userId, page, pageSize);

        return new PagedResult<ProductViewModel>
        {
            Items = mapper.Map<List<ProductViewModel>>(result.Items),
            Page = result.Page,
            PageSize = result.PageSize,
            TotalCount = result.TotalCount
        };
    }

    /// <summary>One of the user's products, or null when it doesn't exist or isn't theirs.</summary>
    public async Task<ProductViewModel> GetMineAsync(Guid id, Guid userId)
    {
        var product = await products.GetAsync(id, userId);
        return product == null ? null : mapper.Map<ProductViewModel>(product);
    }

    /// <summary>Validates the request and creates a draft product owned by the user.</summary>
    public async Task<ProductViewModel> CreateAsync(ProductRequest request, Guid userId)
    {
        await validator.ValidateAndThrowAsync(request);

        var product = mapper.Map<Product>(request);
        product.OwnerId = userId; // Owner comes from the token, never from the body

        await products.AddAndSaveAsync(product, userId.ToString());
        return mapper.Map<ProductViewModel>(product);
    }

    /// <summary>Publishes one of the user's products. Null when it doesn't exist or isn't theirs.</summary>
    public async Task<ProductViewModel> PublishAsync(Guid id, Guid userId)
    {
        var product = await products.GetAsync(id, userId);
        if (product == null)
            return null; // Same answer for "missing" and "not yours": no information leak

        product.Publish(); // Throws DomainException on an invalid transition
        await products.UpdateAndSaveAsync(product, userId.ToString());
        return mapper.Map<ProductViewModel>(product);
    }

    /// <summary>Soft-deletes one of the user's products.</summary>
    public async Task<bool> DeleteAsync(Guid id, Guid userId)
    {
        if (await products.GetAsync(id, userId) == null)
            return false;

        return await products.SoftDeleteAsync(id, userId.ToString()) > 0;
    }
}
```

Registration — `Application/ServiceConfiguration.cs`:

```csharp
public static IServiceCollection AddApplication(this IServiceCollection services)
{
    services.AddAutoMapper(typeof(AppMappingProfile));
    services.AddValidatorsFromAssemblyContaining<ProductRequestValidator>();
    services.AddScoped<IProductService, ProductService>();
    return services;
}
```

## Base controller — `Api/Controllers/BaseController.cs` (exists once)

Every controller inherits it, so every endpoint requires a valid JWT by default. Opting out with `[AllowAnonymous]` requires a rate limit on that endpoint.

```csharp
namespace App.Api.Controllers;

/// <summary>JWT-protected base controller with accessors for the caller's claims.</summary>
[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
public class BaseController : ControllerBase
{
    /// <summary>The caller's user id (the Identity user id).</summary>
    protected Guid UserId => Guid.Parse(User.FindFirst(nameof(ClaimTypes.NameIdentifier)).Value); // Claim name as issued by the project's token service

    /// <summary>The caller's user id as a string, for audit columns.</summary>
    protected string UserIdString => UserId.ToString();

    /// <summary>The caller's email.</summary>
    protected string Email => User.FindFirst(ClaimTypes.Email)?.Value;
}
```

## Controller — `Api/Controllers/v1/Catalog/ProductsController.cs`

Routes use **named verb segments**: `GetX`, `GetXById/{id:guid}`, `Create`, `Update/{id:guid}`, `<Action>/{id:guid}`, `Delete/{id:guid}`. Localized projects add the culture-prefixed route as well.

```csharp
namespace App.Api.Controllers.v1.Catalog;

/// <summary>Endpoints for the caller's products.</summary>
[Route("{culture:culture}/api/v1/[controller]")] // Only in localized projects
[Route("api/v1/[controller]")]
[ApiController]
public class ProductsController(IProductService products) : BaseController
{
    /// <summary>One page of the caller's products.</summary>
    [HttpGet("GetMyProducts")]
    [ProducesResponseType(typeof(PagedResult<ProductViewModel>), StatusCodes.Status200OK)]
    public async Task<ActionResult<PagedResult<ProductViewModel>>> GetMyProducts([FromQuery] int page = 1, [FromQuery] int pageSize = 20) =>
        Ok(await products.GetMyPageAsync(UserId, page, pageSize));

    /// <summary>One of the caller's products.</summary>
    [HttpGet("GetById/{id:guid}")]
    [ProducesResponseType(typeof(ProductViewModel), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ProductViewModel>> GetById(Guid id) =>
        await products.GetMineAsync(id, UserId) is { } product ? Ok(product) : NotFound();

    /// <summary>Creates a draft product.</summary>
    [HttpPost("Create")]
    [ProducesResponseType(typeof(ProductViewModel), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<ProductViewModel>> Create([FromBody] ProductRequest request)
    {
        var created = await products.CreateAsync(request, UserId);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    /// <summary>Publishes a draft product.</summary>
    [HttpPut("Publish/{id:guid}")]
    [ProducesResponseType(typeof(ProductViewModel), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status409Conflict)]
    public async Task<ActionResult<ProductViewModel>> Publish(Guid id) =>
        await products.PublishAsync(id, UserId) is { } product ? Ok(product) : NotFound();

    /// <summary>Soft-deletes a product.</summary>
    [HttpDelete("Delete/{id:guid}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult> Delete(Guid id) =>
        await products.DeleteAsync(id, UserId) ? NoContent() : NotFound();
}
```

## Exception handling — `Api/Errors/AppExceptionHandler.cs`

Expected failures become ProblemDetails; everything else stays a 500 (logged, without personal data).

```csharp
namespace App.Api.Errors;

/// <summary>Maps validation and domain exceptions to ProblemDetails responses.</summary>
public class AppExceptionHandler(IProblemDetailsService problemDetails) : IExceptionHandler
{
    /// <summary>Handles ValidationException (400) and DomainException (409); other exceptions fall through.</summary>
    public async ValueTask<bool> TryHandleAsync(HttpContext httpContext, Exception exception, CancellationToken cancellationToken)
    {
        ProblemDetails details = exception switch
        {
            ValidationException validation => new ProblemDetails
            {
                Status = StatusCodes.Status400BadRequest,
                Title = "Validation failed",
                Extensions =
                {
                    ["errors"] = validation.Errors
                        .GroupBy(e => e.PropertyName)
                        .ToDictionary(g => g.Key, g => g.Select(e => e.ErrorMessage).ToArray())
                }
            },
            DomainException domain => new ProblemDetails
            {
                Status = StatusCodes.Status409Conflict,
                Title = domain.Message
            },
            _ => null
        };

        if (details == null)
            return false;

        httpContext.Response.StatusCode = details.Status.Value;
        return await problemDetails.TryWriteAsync(new ProblemDetailsContext
        {
            HttpContext = httpContext,
            ProblemDetails = details,
            Exception = exception
        });
    }
}
```

`Program.cs`:

```csharp
builder.Services.AddProblemDetails();
builder.Services.AddExceptionHandler<AppExceptionHandler>();
builder.Services.AddApplication();
// ...
app.UseExceptionHandler();
```

## Background jobs (Hangfire)

Side effects that shouldn't block the request (push, email, reminders) are Hangfire jobs behind an interface, enqueued **after** the save succeeds. A scheduling failure is logged and never fails the request.

```csharp
public interface IProductPublishedNotificationJob
{
    Task NotifyFollowersAsync(Guid productId);
}

// In the service, after UpdateAndSaveAsync succeeded:
backgroundJobs.Enqueue<IProductPublishedNotificationJob>(x => x.NotifyFollowersAsync(product.Id));
```
