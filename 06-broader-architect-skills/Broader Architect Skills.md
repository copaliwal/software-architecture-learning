# 06 Broader Architect Skills

Beyond technical design patterns, architects need a diverse skill set spanning security, DevOps, performance optimization, and communication. This section covers essential skills for modern software architects.

---

## Security Architecture

### 1. **Authentication & Authorization**

#### Authentication and Authorization

OAuth 2.0 is an authorization framework; OpenID Connect (OIDC) adds an identity layer. For an
API, prefer an established identity provider to issue access tokens and configure the API to
validate them. A resource server validates tokens; it should not mint its own access tokens.
Keep authentication (who is calling) distinct from authorization (what that caller may do).

```csharp
// ASP.NET Core API validating access tokens from a trusted issuer.
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        })
        .AddJwtBearer(options =>
        {
            options.Authority = "https://your-identity-server.com";
            options.Audience = "your-api";
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,
                ClockSkew = TimeSpan.Zero
            };
        });
        
        services.AddAuthorization(options =>
        {
            options.AddPolicy("RequireAdminRole", policy =>
                policy.RequireRole("Admin"));
            options.AddPolicy("RequireOrderPermission", policy =>
                policy.RequireClaim("permission", "orders.write"));
        });
    }
}

[Authorize(Policy = "RequireAdminRole")]
[HttpDelete("users/{id}")]
public IActionResult DeleteUser(int id)
{
    // Only admins can delete users
}
```

Do not build token issuance by hand for a production identity system. Use a maintained identity
provider or identity platform, validate issuer, audience, signature, and lifetime, and authorize
each operation against the resource being accessed. Do not treat possession of a valid token as
proof that a caller may access every object.

#### Microsoft Entra ID Integration

```csharp
// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("MicrosoftEntra"));

// appsettings.json
{
  "MicrosoftEntra": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id"
  }
}
```

For workforce identities, use Microsoft Entra ID; for customer identity scenarios, evaluate
Microsoft Entra External ID. Store configuration separately from credentials and use managed
identities or workload identity federation where supported.

### 2. **Zero Trust Security Model**

Zero Trust is an operating model, not a middleware class or a single network product. Authenticate
users and workloads, authorize each request for its specific resource, apply least privilege,
assume network location alone does not establish trust, and continuously monitor access. Use
device posture or network context only when obtained from a trusted control plane; a client-supplied
header is not proof of device compliance.

### 3. **Data Protection, Secrets, and API Security**

- Classify data and minimize collection, retention, and exposure.
- Use TLS for data in transit and platform-managed encryption at rest where appropriate. Apply
  application-level encryption when the threat model requires it, with explicit key ownership,
  rotation, recovery, and access controls.
- Store secrets in a managed secret store, restrict access to workloads that need them, rotate
  credentials, and keep secret values out of source, logs, and telemetry.
- Enforce authorization at the resource or object level, validate input, constrain request size
  and query cost, apply rate limits, and return errors that do not disclose sensitive internals.
- Threat-model important trust boundaries and data flows before implementation; revisit the model
  when the architecture or threat environment changes.

### References

- [Microsoft identity platform](https://learn.microsoft.com/en-us/entra/identity-platform/)
- [Access tokens in the Microsoft identity platform](https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens)
- [Configure JWT bearer authentication in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)

---

## DevOps & CI/CD

### 1. **Azure DevOps Pipeline**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.0.x'

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: UseDotNet@2
      inputs:
        version: $(dotnetVersion)
    
    - task: DotNetCoreCLI@2
      displayName: 'Restore packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
    
    - task: DotNetCoreCLI@2
      displayName: 'Build solution'
      inputs:
        command: 'build'
        arguments: '--configuration $(buildConfiguration)'
    
    - task: DotNetCoreCLI@2
      displayName: 'Run unit tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage"'
    
    - task: PublishCodeCoverageResults@1
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
    
    - task: DotNetCoreCLI@2
      displayName: 'Publish application'
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

- stage: DeployDev
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployDevJob
    environment: 'Development'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Connection'
              appName: 'myapp-dev'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'

- stage: DeployProd
  dependsOn: DeployDev
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployProdJob
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Connection'
              appName: 'myapp-prod'
              package: '$(Pipeline.Workspace)/drop/**/*.zip'
              slotName: 'staging'
          
          - task: AzureAppServiceManage@0
            inputs:
              azureSubscription: 'Azure-Connection'
              action: 'Swap Slots'
              webAppName: 'myapp-prod'
              sourceSlot: 'staging'
              targetSlot: 'production'
```

### 2. **Infrastructure as Code (Bicep)**

```bicep
// main.bicep
param location string = resourceGroup().location
param appName string
param environment string

// App Service Plan
resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: '${appName}-plan-${environment}'
  location: location
  sku: {
    name: environment == 'prod' ? 'P1v3' : 'B1'
    tier: environment == 'prod' ? 'Premium' : 'Basic'
  }
}

// Web App
resource webApp 'Microsoft.Web/sites@2022-03-01' = {
  name: '${appName}-${environment}'
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      netFrameworkVersion: 'v8.0'
      appSettings: [
        {
          name: 'ASPNETCORE_ENVIRONMENT'
          value: environment
        }
        {
          name: 'ApplicationInsights__ConnectionString'
          value: appInsights.properties.ConnectionString
        }
      ]
      connectionStrings: [
        {
          name: 'DefaultConnection'
          connectionString: 'Server=tcp:${sqlServer.properties.fullyQualifiedDomainName},1433;Database=${sqlDatabase.name};'
          type: 'SQLAzure'
        }
      ]
    }
  }
}

// Application Insights
resource appInsights 'Microsoft.Insights/components@2020-02-02' = {
  name: '${appName}-insights-${environment}'
  location: location
  kind: 'web'
  properties: {
    Application_Type: 'web'
  }
}

// SQL Server
resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
  name: '${appName}-sql-${environment}'
  location: location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: 'YourSecurePassword123!'
  }
}

resource sqlDatabase 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
  parent: sqlServer
  name: '${appName}-db-${environment}'
  location: location
  sku: {
    name: environment == 'prod' ? 'S1' : 'Basic'
  }
}

output webAppUrl string = webApp.properties.defaultHostName
```

### 3. **Terraform Example**

```hcl
# main.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "rg-${var.app_name}-${var.environment}"
  location = var.location
}

resource "azurerm_service_plan" "plan" {
  name                = "plan-${var.app_name}-${var.environment}"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  os_type             = "Linux"
  sku_name            = var.environment == "prod" ? "P1v3" : "B1"
}

resource "azurerm_linux_web_app" "app" {
  name                = "app-${var.app_name}-${var.environment}"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  service_plan_id     = azurerm_service_plan.plan.id

  site_config {
    application_stack {
      dotnet_version = "8.0"
    }
  }

  app_settings = {
    "ASPNETCORE_ENVIRONMENT" = var.environment
    "ApplicationInsights__ConnectionString" = azurerm_application_insights.insights.connection_string
  }
}

resource "azurerm_application_insights" "insights" {
  name                = "insights-${var.app_name}-${var.environment}"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  application_type    = "web"
}

output "web_app_url" {
  value = azurerm_linux_web_app.app.default_hostname
}
```

---

## Scalability & Performance

### 1. **Caching Strategies**

```csharp
// In-Memory Caching
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly IProductRepository _repository;
    
    public async Task<Product> GetProductAsync(int id)
    {
        var cacheKey = $"product_{id}";
        
        if (!_cache.TryGetValue(cacheKey, out Product product))
        {
            product = await _repository.GetByIdAsync(id);
            
            var cacheOptions = new MemoryCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromMinutes(5))
                .SetAbsoluteExpiration(TimeSpan.FromHours(1));
            
            _cache.Set(cacheKey, product, cacheOptions);
        }
        
        return product;
    }
}

// Distributed Caching (Redis)
public class OrderService
{
    private readonly IDistributedCache _cache;
    
    public async Task<Order> GetOrderAsync(int id)
    {
        var cacheKey = $"order_{id}";
        var cachedOrder = await _cache.GetStringAsync(cacheKey);
        
        if (cachedOrder != null)
        {
            return JsonSerializer.Deserialize<Order>(cachedOrder);
        }
        
        var order = await _repository.GetByIdAsync(id);
        
        await _cache.SetStringAsync(
            cacheKey,
            JsonSerializer.Serialize(order),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            });
        
        return order;
    }
}

// Response Caching
[ResponseCache(Duration = 60, VaryByQueryKeys = new[] { "category" })]
[HttpGet]
public async Task<IActionResult> GetProducts(string category)
{
    var products = await _productService.GetByCategoryAsync(category);
    return Ok(products);
}
```

### 2. **Load Balancing**

```csharp
// Health Check Endpoint
public class HealthCheckController : ControllerBase
{
    [HttpGet("health")]
    public IActionResult Health()
    {
        return Ok(new { status = "healthy", timestamp = DateTime.UtcNow });
    }
    
    [HttpGet("health/ready")]
    public async Task<IActionResult> Ready([FromServices] ApplicationDbContext dbContext)
    {
        try
        {
            await dbContext.Database.CanConnectAsync();
            return Ok(new { status = "ready" });
        }
        catch
        {
            return StatusCode(503, new { status = "not ready" });
        }
    }
}

// Program.cs - Health Checks
builder.Services.AddHealthChecks()
    .AddSqlServer(connectionString, name: "database")
    .AddRedis(redisConnection, name: "redis")
    .AddAzureServiceBusTopic(serviceBusConnection, topicName, name: "servicebus");

app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "application/json";
        var result = JsonSerializer.Serialize(new
        {
            status = report.Status.ToString(),
            checks = report.Entries.Select(e => new
            {
                name = e.Key,
                status = e.Value.Status.ToString(),
                duration = e.Value.Duration.TotalMilliseconds
            })
        });
        await context.Response.WriteAsync(result);
    }
});
```

### 3. **Database Optimization**

```csharp
// Indexing
[Index(nameof(Email), IsUnique = true)]
[Index(nameof(LastName), nameof(FirstName))]
public class User
{
    public int Id { get; set; }
    public string Email { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

// Query Optimization - Projection
public async Task<IEnumerable<ProductDto>> GetProductsAsync()
{
    // ❌ Bad - Loads entire entity
    var products = await _context.Products.ToListAsync();
    return products.Select(p => new ProductDto { Id = p.Id, Name = p.Name });
    
    // ✅ Good - Projects only needed columns
    return await _context.Products
        .Select(p => new ProductDto { Id = p.Id, Name = p.Name })
        .ToListAsync();
}

// Eager Loading vs Lazy Loading
public async Task<Order> GetOrderWithDetailsAsync(int id)
{
    // ✅ Eager loading - Single query with JOIN
    return await _context.Orders
        .Include(o => o.Items)
            .ThenInclude(i => i.Product)
        .Include(o => o.Customer)
        .FirstOrDefaultAsync(o => o.Id == id);
}

// AsNoTracking for read-only queries
public async Task<IEnumerable<Product>> GetProductsForDisplayAsync()
{
    return await _context.Products
        .AsNoTracking()
        .ToListAsync();
}
```

---

## Documentation & Communication

### 1. **C4 Model Diagrams**

```
Level 1 - System Context Diagram:
┌─────────────┐
│   Customer  │──────┐
└─────────────┘      │
                     ↓
                ┌─────────────────┐
                │  E-commerce     │
                │  System         │
                └─────────────────┘
                     ↓
┌─────────────┐      │
│   Payment   │←─────┘
│   Gateway   │
└─────────────┘

Level 2 - Container Diagram:
Customer → Web App → API Gateway → [Product Service]
                                 → [Order Service]
                                 → [Payment Service]
                    ↓
                [SQL Database]
                [Redis Cache]

Level 3 - Component Diagram (Order Service):
API Controller → Order Service → Order Repository → SQL Database
              ↓
         Event Publisher → Service Bus
```

### 2. **Architecture Decision Records (ADRs)**

Already covered in [01 Foundations](../01%20Foundations.md#architecture-decision-records-adrs)

### 3. **API Documentation with Swagger**

```csharp
// Program.cs
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "E-Commerce API",
        Version = "v1",
        Description = "API for managing products and orders",
        Contact = new OpenApiContact
        {
            Name = "Architecture Team",
            Email = "architects@example.com"
        }
    });
    
    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
    
    // JWT Authentication
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header using the Bearer scheme",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });
});

app.UseSwagger();
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "E-Commerce API V1");
    c.RoutePrefix = string.Empty; // Serve at root
});

// Controller with XML comments
/// <summary>
/// Manages product operations
/// </summary>
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    /// <summary>
    /// Retrieves a product by ID
    /// </summary>
    /// <param name="id">The product ID</param>
    /// <returns>The product details</returns>
    /// <response code="200">Product found</response>
    /// <response code="404">Product not found</response>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(ProductDto), 200)]
    [ProducesResponseType(404)]
    public async Task<IActionResult> GetProduct(int id)
    {
        // Implementation
    }
}
```

---

### 4. **Architecture Review and Fitness Functions**

Use lightweight governance to keep important decisions visible without turning review into a
generic approval gate. Review the system context, quality-attribute scenarios, trust boundaries,
data ownership, operational model, and significant trade-offs. Record decisions and revisit them
when their assumptions change.

Fitness functions make selected constraints observable over time. Examples include automated
module-dependency tests, API compatibility checks, security scanning, deployment policy checks,
and production SLO alerts. Each check should have an owner and a clear response when it fails;
automated checks complement, rather than replace, architectural judgment.

## Summary

✅ **Security** - OAuth2, JWT, Zero Trust, encryption  
✅ **DevOps** - CI/CD pipelines, IaC (Bicep, Terraform)  
✅ **Scalability** - Caching (Redis), load balancing, health checks  
✅ **Performance** - Database optimization, indexing, query tuning  
✅ **Documentation** - C4 models, ADRs, Swagger/OpenAPI  
✅ **Communication** - Clear diagrams, technical writing, stakeholder management