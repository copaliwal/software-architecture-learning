# Layered Architecture (N-Tier Architecture)

## 📖 Definition

**Layered Architecture** is one of the most common architectural styles that organizes the system into horizontal layers, where each layer has a specific responsibility and only interacts with adjacent layers. Also known as **N-Tier Architecture**.

---

## 🎯 Core Characteristics

- **Separation of Concerns** - Each layer has distinct responsibility
- **Layer Isolation** - Changes in one layer minimally impact others
- **Unidirectional Dependencies** - Upper layers depend on lower layers, not vice versa
- **Abstraction** - Each layer provides abstraction for the layer above
- **Reusability** - Lower layers can be reused by multiple upper layers

---

## 🏛 Scope

- **Application level**
- Layered architecture defines how an entire application is organized
- One of the most fundamental and widely-used architectural patterns

---

## 🏗️ Common Layers

### Traditional 3-Tier Architecture

```
┌─────────────────────────────────┐
│   Presentation Layer (UI)       │  ← User Interface
├─────────────────────────────────┤
│   Business Logic Layer (BLL)    │  ← Business Rules
├─────────────────────────────────┤
│   Data Access Layer (DAL)       │  ← Database Access
└─────────────────────────────────┘
```

### Modern 4-Layer Architecture

```
┌─────────────────────────────────┐
│   Presentation Layer            │  ← Controllers, Views, APIs
├─────────────────────────────────┤
│   Application/Service Layer     │  ← Use Cases, Orchestration
├─────────────────────────────────┤
│   Domain/Business Layer         │  ← Business Logic, Entities
├─────────────────────────────────┤
│   Infrastructure/Data Layer     │  ← Database, External Services
└─────────────────────────────────┘
```

---

## ⚙️ Layer Details

### 1. Presentation Layer

**Responsibility:** User interface and user interaction

```csharp
// ASP.NET Core Controller (Presentation Layer)
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    
    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }
    
    [HttpGet]
    public async Task<IActionResult> GetAllProducts()
    {
        var products = await _productService.GetAllProductsAsync();
        return Ok(products);
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        
        if (product == null)
            return NotFound();
        
        return Ok(product);
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateProduct([FromBody] CreateProductDto dto)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
        
        var productId = await _productService.CreateProductAsync(dto);
        return CreatedAtAction(nameof(GetProduct), new { id = productId }, null);
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateProduct(int id, [FromBody] UpdateProductDto dto)
    {
        await _productService.UpdateProductAsync(id, dto);
        return NoContent();
    }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        await _productService.DeleteProductAsync(id);
        return NoContent();
    }
}

// DTOs (Data Transfer Objects)
public class CreateProductDto
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Range(0.01, 10000)]
    public decimal Price { get; set; }
    
    public string Description { get; set; }
}

public class UpdateProductDto
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Range(0.01, 10000)]
    public decimal Price { get; set; }
    
    public string Description { get; set; }
}
```

### 2. Application/Service Layer

**Responsibility:** Orchestrates business logic, coordinates use cases

```csharp
// Service Interface
public interface IProductService
{
    Task<IEnumerable<ProductDto>> GetAllProductsAsync();
    Task<ProductDto> GetProductByIdAsync(int id);
    Task<int> CreateProductAsync(CreateProductDto dto);
    Task UpdateProductAsync(int id, UpdateProductDto dto);
    Task DeleteProductAsync(int id);
}

// Service Implementation
public class ProductService : IProductService
{
    private readonly IProductRepository _productRepository;
    private readonly ILogger<ProductService> _logger;
    private readonly IMapper _mapper;
    
    public ProductService(
        IProductRepository productRepository,
        ILogger<ProductService> logger,
        IMapper mapper)
    {
        _productRepository = productRepository;
        _logger = logger;
        _mapper = mapper;
    }
    
    public async Task<IEnumerable<ProductDto>> GetAllProductsAsync()
    {
        _logger.LogInformation("Retrieving all products");
        
        var products = await _productRepository.GetAllAsync();
        return _mapper.Map<IEnumerable<ProductDto>>(products);
    }
    
    public async Task<ProductDto> GetProductByIdAsync(int id)
    {
        _logger.LogInformation($"Retrieving product with ID: {id}");
        
        var product = await _productRepository.GetByIdAsync(id);
        return _mapper.Map<ProductDto>(product);
    }
    
    public async Task<int> CreateProductAsync(CreateProductDto dto)
    {
        _logger.LogInformation($"Creating new product: {dto.Name}");
        
        // Business validation
        if (await _productRepository.ProductExistsAsync(dto.Name))
        {
            throw new BusinessException("Product with this name already exists");
        }
        
        var product = _mapper.Map<Product>(dto);
        product.CreatedAt = DateTime.UtcNow;
        product.IsActive = true;
        
        await _productRepository.AddAsync(product);
        await _productRepository.SaveChangesAsync();
        
        _logger.LogInformation($"Product created with ID: {product.Id}");
        return product.Id;
    }
    
    public async Task UpdateProductAsync(int id, UpdateProductDto dto)
    {
        _logger.LogInformation($"Updating product ID: {id}");
        
        var product = await _productRepository.GetByIdAsync(id);
        
        if (product == null)
        {
            throw new NotFoundException($"Product with ID {id} not found");
        }
        
        _mapper.Map(dto, product);
        product.UpdatedAt = DateTime.UtcNow;
        
        await _productRepository.UpdateAsync(product);
        await _productRepository.SaveChangesAsync();
        
        _logger.LogInformation($"Product ID {id} updated successfully");
    }
    
    public async Task DeleteProductAsync(int id)
    {
        _logger.LogInformation($"Deleting product ID: {id}");
        
        var product = await _productRepository.GetByIdAsync(id);
        
        if (product == null)
        {
            throw new NotFoundException($"Product with ID {id} not found");
        }
        
        await _productRepository.DeleteAsync(product);
        await _productRepository.SaveChangesAsync();
        
        _logger.LogInformation($"Product ID {id} deleted successfully");
    }
}
```

### 3. Domain/Business Layer

**Responsibility:** Core business logic and entities

```csharp
// Domain Entity
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Description { get; set; }
    public int StockQuantity { get; set; }
    public bool IsActive { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? UpdatedAt { get; set; }
    
    // Business logic methods
    public void UpdatePrice(decimal newPrice)
    {
        if (newPrice <= 0)
            throw new ArgumentException("Price must be greater than zero");
        
        Price = newPrice;
        UpdatedAt = DateTime.UtcNow;
    }
    
    public void AdjustStock(int quantity)
    {
        if (StockQuantity + quantity < 0)
            throw new InvalidOperationException("Insufficient stock");
        
        StockQuantity += quantity;
        UpdatedAt = DateTime.UtcNow;
    }
    
    public void Deactivate()
    {
        IsActive = false;
        UpdatedAt = DateTime.UtcNow;
    }
    
    public bool IsInStock() => StockQuantity > 0;
    
    public decimal CalculateDiscountedPrice(decimal discountPercentage)
    {
        if (discountPercentage < 0 || discountPercentage > 100)
            throw new ArgumentException("Discount must be between 0 and 100");
        
        return Price * (1 - discountPercentage / 100);
    }
}

// Domain Service (for complex business logic involving multiple entities)
public class PricingService
{
    public decimal CalculateTotalPrice(IEnumerable<Product> products, Customer customer)
    {
        var subtotal = products.Sum(p => p.Price);
        
        // Apply customer tier discount
        var discount = customer.Tier switch
        {
            CustomerTier.Bronze => 0.05m,
            CustomerTier.Silver => 0.10m,
            CustomerTier.Gold => 0.15m,
            _ => 0m
        };
        
        return subtotal * (1 - discount);
    }
}
```

### 4. Infrastructure/Data Layer

**Responsibility:** Data persistence, external services, infrastructure concerns

```csharp
// Repository Interface (in Domain/Business Layer)
public interface IProductRepository
{
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product> GetByIdAsync(int id);
    Task<bool> ProductExistsAsync(string name);
    Task AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(Product product);
    Task SaveChangesAsync();
}

// Repository Implementation (in Infrastructure Layer)
public class ProductRepository : IProductRepository
{
    private readonly ApplicationDbContext _context;
    
    public ProductRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<IEnumerable<Product>> GetAllAsync()
    {
        return await _context.Products
            .Where(p => p.IsActive)
            .OrderBy(p => p.Name)
            .ToListAsync();
    }
    
    public async Task<Product> GetByIdAsync(int id)
    {
        return await _context.Products
            .FirstOrDefaultAsync(p => p.Id == id);
    }
    
    public async Task<bool> ProductExistsAsync(string name)
    {
        return await _context.Products
            .AnyAsync(p => p.Name.ToLower() == name.ToLower());
    }
    
    public async Task AddAsync(Product product)
    {
        await _context.Products.AddAsync(product);
    }
    
    public async Task UpdateAsync(Product product)
    {
        _context.Products.Update(product);
    }
    
    public async Task DeleteAsync(Product product)
    {
        _context.Products.Remove(product);
    }
    
    public async Task SaveChangesAsync()
    {
        await _context.SaveChangesAsync();
    }
}

// DbContext
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    public DbSet<Product> Products { get; set; }
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>(entity =>
        {
            entity.HasKey(e => e.Id);
            entity.Property(e => e.Name).IsRequired().HasMaxLength(100);
            entity.Property(e => e.Price).HasColumnType("decimal(18,2)");
            entity.HasIndex(e => e.Name);
        });
    }
}
```

---

## 🔧 Dependency Injection Setup

```csharp
// Program.cs or Startup.cs
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        // Add services to container
        builder.Services.AddControllers();
        
        // Infrastructure Layer
        builder.Services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
        
        // Register Repositories (Infrastructure)
        builder.Services.AddScoped<IProductRepository, ProductRepository>();
        builder.Services.AddScoped<ICustomerRepository, CustomerRepository>();
        
        // Register Services (Application Layer)
        builder.Services.AddScoped<IProductService, ProductService>();
        builder.Services.AddScoped<ICustomerService, CustomerService>();
        
        // Add AutoMapper
        builder.Services.AddAutoMapper(typeof(Program));
        
        // Add Swagger
        builder.Services.AddSwaggerGen();
        
        var app = builder.Build();
        
        if (app.Environment.IsDevelopment())
        {
            app.UseSwagger();
            app.UseSwaggerUI();
        }
        
        app.UseHttpsRedirection();
        app.UseAuthorization();
        app.MapControllers();
        
        app.Run();
    }
}
```

---

## 📁 Project Structure

```
MyApp.Solution/
├── MyApp.Presentation/           (Presentation Layer)
│   ├── Controllers/
│   │   ├── ProductsController.cs
│   │   └── CustomersController.cs
│   ├── DTOs/
│   │   ├── ProductDto.cs
│   │   └── CustomerDto.cs
│   ├── Filters/
│   └── Program.cs
│
├── MyApp.Application/            (Application/Service Layer)
│   ├── Services/
│   │   ├── IProductService.cs
│   │   ├── ProductService.cs
│   │   ├── ICustomerService.cs
│   │   └── CustomerService.cs
│   ├── Mapping/
│   │   └── AutoMapperProfile.cs
│   └── Exceptions/
│       ├── BusinessException.cs
│       └── NotFoundException.cs
│
├── MyApp.Domain/                 (Domain/Business Layer)
│   ├── Entities/
│   │   ├── Product.cs
│   │   ├── Customer.cs
│   │   └── Order.cs
│   ├── Interfaces/
│   │   ├── IProductRepository.cs
│   │   └── ICustomerRepository.cs
│   ├── Services/
│   │   └── PricingService.cs
│   └── Enums/
│       └── CustomerTier.cs
│
└── MyApp.Infrastructure/         (Infrastructure/Data Layer)
    ├── Data/
    │   ├── ApplicationDbContext.cs
    │   └── Migrations/
    ├── Repositories/
    │   ├── ProductRepository.cs
    │   └── CustomerRepository.cs
    └── ExternalServices/
        ├── EmailService.cs
        └── PaymentGateway.cs
```

---

## ✅ When to Use Layered Architecture

✅ **Use when:**
- Building traditional enterprise applications
- Team needs clear separation of concerns
- Requirements are well-defined and stable
- Need standardized approach
- Building monolithic applications
- Team has varying skill levels (layers provide clear boundaries)

❌ **Avoid when:**
- Building highly scalable microservices
- Need extreme flexibility and agility
- Domain logic is highly complex (consider DDD/Hexagonal)
- Cross-cutting concerns dominate

---

## 👍 Advantages

✅ **Simplicity** - Easy to understand and implement  
✅ **Separation of Concerns** - Each layer has specific responsibility  
✅ **Testability** - Layers can be tested independently  
✅ **Maintainability** - Changes isolated to specific layers  
✅ **Team Organization** - Different teams can own different layers  
✅ **Standardization** - Well-known pattern with best practices

---

## 👎 Disadvantages

❌ **Performance** - Layer traversal can add overhead  
❌ **Rigidity** - Strict layer boundaries can be constraining  
❌ **Monolithic Tendency** - Can lead to tightly coupled monoliths  
❌ **Deployment** - All layers deployed together  
❌ **Database-Centric** - Often leads to anemic domain models  
❌ **Scalability** - Cannot scale layers independently

---

## 🔄 Variations

### Strict Layered Architecture
Each layer can only use the layer directly below it.

```
Presentation → Service → Domain → Data
```

### Relaxed Layered Architecture
Layers can skip intermediate layers.

```
Presentation → Service → Domain
            ↘         ↗
              Data ←
```

### Open vs Closed Layers

- **Closed Layer** - Must go through this layer (cannot skip)
- **Open Layer** - Can be bypassed

---

## 🎯 Common Use Cases

- **CRUD Applications** - Simple business applications
- **Enterprise Applications** - Large organizational systems
- **Web Applications** - Traditional web apps (ASP.NET MVC, ASP.NET Core)
- **Line-of-Business Apps** - Internal business tools
- **Content Management Systems** - WordPress-style applications

---

## 🏗️ Best Practices

✅ **Define clear layer responsibilities** - Each layer has one job  
✅ **Use dependency injection** - Wire layers together loosely  
✅ **Keep domain layer pure** - No infrastructure dependencies  
✅ **Use DTOs for layer boundaries** - Don't expose domain entities to UI  
✅ **Implement repository pattern** - Abstract data access  
✅ **Apply SOLID principles** - Especially in service layer  
✅ **Use interfaces** - Enable testing and flexibility  
✅ **Avoid circular dependencies** - Upper layers depend on lower, never reverse

---

## 📚 Related Patterns

- **Repository Pattern** - Data access abstraction
- **Unit of Work** - Manage database transactions
- **DTO (Data Transfer Object)** - Transfer data between layers
- **Service Layer** - Encapsulate business logic
- **MVC (Model-View-Controller)** - Presentation layer pattern