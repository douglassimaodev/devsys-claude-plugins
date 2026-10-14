# Backend — Tests (xUnit + NSubstitute + FluentAssertions 7.x)

Project: `tests/<Product>.ApiUnitTest/` with folders mirroring the code (`Services/<Module>/`, `Controllers/<Module>/`, `Repositories/`). One class per tested class, named `<Class>Tests`. Test names: `MethodName_StateUnderTest_ExpectedBehavior`.
Mock repositories with NSubstitute; use the **real** AutoMapper profile so mapping mistakes fail the tests. FluentAssertions stays on 7.x (8+ is commercially licensed).

## Service tests — `Services/Catalog/ProductServiceTests.cs`

```csharp
namespace App.ApiUnitTest.Services.Catalog;

public class ProductServiceTests
{
    private readonly Guid _userId = Guid.CreateVersion7();
    private readonly IProductRepository _products = Substitute.For<IProductRepository>();
    private readonly ProductService _sut;

    public ProductServiceTests()
    {
        var mapper = new MapperConfiguration(cfg => cfg.AddProfile<AppMappingProfile>()).CreateMapper();
        _sut = new ProductService(_products, mapper, new ProductRequestValidator());
    }

    [Fact]
    public async Task CreateAsync_ValidRequest_SavesDraftOwnedByCaller()
    {
        _products.AddAndSaveAsync(Arg.Any<Product>(), Arg.Any<string>()).Returns(1);

        var result = await _sut.CreateAsync(new ProductRequest { Name = "Chair", Price = 10.50m }, _userId);

        result.Status.Should().Be(ProductStatus.Draft);
        await _products.Received(1).AddAndSaveAsync(
            Arg.Is<Product>(p => p.OwnerId == _userId && p.Name == "Chair" && p.Price == 10.50m),
            _userId.ToString());
    }

    [Theory]
    [InlineData("", 10)]
    [InlineData("Chair", -1)]
    public async Task CreateAsync_InvalidRequest_ThrowsValidationExceptionAndSavesNothing(string name, decimal price)
    {
        var act = () => _sut.CreateAsync(new ProductRequest { Name = name, Price = price }, _userId);

        await act.Should().ThrowAsync<ValidationException>();
        await _products.DidNotReceiveWithAnyArgs().AddAndSaveAsync(default, default);
    }

    [Fact]
    public async Task PublishAsync_ProductOfAnotherOwner_ReturnsNullAndSavesNothing()
    {
        _products.GetAsync(Arg.Any<Guid>(), _userId).Returns((Product)null);

        var result = await _sut.PublishAsync(Guid.CreateVersion7(), _userId);

        result.Should().BeNull();
        await _products.DidNotReceiveWithAnyArgs().UpdateAndSaveAsync(default, default);
    }

    [Fact]
    public async Task PublishAsync_DraftProduct_PublishesAndSaves()
    {
        var product = new Product { OwnerId = _userId, Name = "Chair", Price = 10m };
        _products.GetAsync(product.Id, _userId).Returns(product);
        _products.UpdateAndSaveAsync(product, Arg.Any<string>()).Returns(1);

        var result = await _sut.PublishAsync(product.Id, _userId);

        result.Status.Should().Be(ProductStatus.Published);
        await _products.Received(1).UpdateAndSaveAsync(product, _userId.ToString());
    }

    [Fact]
    public async Task PublishAsync_AlreadyPublished_ThrowsDomainExceptionAndSavesNothing()
    {
        var product = new Product { OwnerId = _userId, Name = "Chair", Price = 10m };
        product.Publish();
        _products.GetAsync(product.Id, _userId).Returns(product);

        var act = () => _sut.PublishAsync(product.Id, _userId);

        await act.Should().ThrowAsync<DomainException>();
        await _products.DidNotReceiveWithAnyArgs().UpdateAndSaveAsync(default, default);
    }
}
```

## Controller tests — `Controllers/Catalog/ProductsControllerTests.cs`

Controllers are thin, so their tests check wiring: the caller's id comes from the token, and results map to the right status codes.

```csharp
namespace App.ApiUnitTest.Controllers.Catalog;

public class ProductsControllerTests
{
    private readonly Guid _userId = Guid.CreateVersion7();
    private readonly IProductService _products = Substitute.For<IProductService>();
    private readonly ProductsController _controller;

    public ProductsControllerTests()
    {
        _controller = new ProductsController(_products)
        {
            ControllerContext = new ControllerContext
            {
                HttpContext = new DefaultHttpContext
                {
                    User = new ClaimsPrincipal(new ClaimsIdentity(
                    [
                        new Claim(nameof(ClaimTypes.NameIdentifier), _userId.ToString())
                    ]))
                }
            }
        };
    }

    [Fact]
    public async Task Publish_ProductNotFoundOrNotOwned_ReturnsNotFound()
    {
        _products.PublishAsync(Arg.Any<Guid>(), _userId).Returns((ProductViewModel)null);

        var result = await _controller.Publish(Guid.CreateVersion7());

        result.Result.Should().BeOfType<NotFoundResult>();
    }

    [Fact]
    public async Task Create_ValidRequest_PassesCallerIdFromTokenAndReturnsCreated()
    {
        var request = new ProductRequest { Name = "Chair", Price = 10m };
        _products.CreateAsync(request, _userId).Returns(new ProductViewModel { Id = Guid.CreateVersion7(), Name = "Chair" });

        var result = await _controller.Create(request);

        result.Result.Should().BeOfType<CreatedAtActionResult>();
        await _products.Received(1).CreateAsync(request, _userId);
    }
}
```

Time-dependent rules in services: inject `TimeProvider` and use `FakeTimeProvider` (`Microsoft.Extensions.TimeProvider.Testing`) in tests — never the real clock in an assertion.
