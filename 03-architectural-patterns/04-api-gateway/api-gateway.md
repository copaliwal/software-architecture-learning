# API Gateway Pattern

## Overview

**Category:** Architectural Pattern  
**Intent:** Provide a single entry point for client applications to interact with multiple microservices. Acts as a reverse proxy to route requests to appropriate services.

---

## Problem

In a microservices architecture:
- Clients need to call multiple services directly
- Each service has different protocols, authentication
- Cross-cutting concerns (auth, logging, rate limiting) are duplicated
- Client needs to know about internal service structure
- Network chattiness (multiple round trips)

---

## Solution

Introduce an **API Gateway** as a single entry point that:
- Routes requests to appropriate microservices
- Aggregates responses from multiple services
- Handles cross-cutting concerns centrally
- Provides protocol translation
- Shields clients from service changes

---

## Architecture

```
Client (Mobile/Web)
       ↓
   API Gateway
       ↓
  ┌────┴────┬────────┬────────┐
  ↓         ↓        ↓        ↓
Product  Order   Payment  User
Service  Service Service  Service
```

---

## C# Implementation with Ocelot

**Ocelot** is a popular .NET API Gateway library.

### Installation

```powershell
dotnet new webapi -n ApiGateway
cd ApiGateway
dotnet add package Ocelot
dotnet add package Ocelot.Cache.CacheManager
```

### Configuration (ocelot.json)

```json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/products/{everything}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "productservice.example.com",
          "Port": 443
        }
      ],
      "UpstreamPathTemplate": "/products/{everything}",
      "UpstreamHttpMethod": [ "GET", "POST", "PUT", "DELETE" ]
    },
    {
      "DownstreamPathTemplate": "/api/orders/{everything}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "orderservice.example.com",
          "Port": 443
        }
      ],
      "UpstreamPathTemplate": "/orders/{everything}",
      "UpstreamHttpMethod": [ "GET", "POST" ],
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer"
      },
      "RateLimitOptions": {
        "EnableRateLimiting": true,
        "Period": "1m",
        "Limit": 100
      }
    },
    {
      "DownstreamPathTemplate": "/api/users/{everything}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "userservice.example.com",
          "Port": 443
        }
      ],
      "UpstreamPathTemplate": "/users/{everything}",
      "UpstreamHttpMethod": [ "GET", "POST" ],
      "FileCacheOptions": {
        "TtlSeconds": 30
      }
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "https://api.example.com",
    "RateLimitOptions": {
      "DisableRateLimitHeaders": false,
      "QuotaExceededMessage": "Rate limit exceeded",
      "HttpStatusCode": 429
    }
  }
}
```

### Program.cs

```csharp
using Ocelot.DependencyInjection;
using Ocelot.Middleware;

var builder = WebApplication.CreateBuilder(args);

// Add Ocelot configuration
builder.Configuration.AddJsonFile("ocelot.json", optional: false, reloadOnChange: true);

// Add Ocelot services
builder.Services.AddOcelot();

// Add authentication
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.Authority = "https://your-identity-server.com";
        options.Audience = "api";
    });

var app = builder.Build();

// Use Ocelot middleware
await app.UseOcelot();

app.Run();
```

---

## Custom API Gateway Implementation

For more control, build a custom gateway:

```csharp
[ApiController]
[Route("api/gateway")]
public class GatewayController : ControllerBase
{
    private readonly HttpClient _httpClient;
    private readonly IConfiguration _configuration;
    
    public GatewayController(IHttpClientFactory httpClientFactory, IConfiguration configuration)
    {
        _httpClient = httpClientFactory.CreateClient();
        _configuration = configuration;
    }
    
    // Aggregate data from multiple services
    [HttpGet("product-details/{productId}")]
    public async Task<IActionResult> GetProductDetails(int productId)
    {
        // Call Product Service
        var productResponse = await _httpClient.GetAsync(
            $"{_configuration["Services:Product"]}/api/products/{productId}");
        var product = await productResponse.Content.ReadFromJsonAsync<ProductDto>();
        
        // Call Review Service
        var reviewResponse = await _httpClient.GetAsync(
            $"{_configuration["Services:Review"]}/api/reviews/product/{productId}");
        var reviews = await reviewResponse.Content.ReadFromJsonAsync<List<ReviewDto>>();
        
        // Call Inventory Service
        var inventoryResponse = await _httpClient.GetAsync(
            $"{_configuration["Services:Inventory"]}/api/inventory/{productId}");
        var inventory = await inventoryResponse.Content.ReadFromJsonAsync<InventoryDto>();
        
        // Aggregate response
        var result = new
        {
            Product = product,
            Reviews = reviews,
            AvailableQuantity = inventory.Quantity,
            InStock = inventory.Quantity > 0
        };
        
        return Ok(result);
    }
    
    // Simple proxy
    [HttpGet("products")]
    public async Task<IActionResult> GetProducts([FromQuery] string category)
    {
        var response = await _httpClient.GetAsync(
            $"{_configuration["Services:Product"]}/api/products?category={category}");
        
        response.EnsureSuccessStatusCode();
        var products = await response.Content.ReadFromJsonAsync<List<ProductDto>>();
        
        return Ok(products);
    }
    
    // Request transformation
    [HttpPost("orders")]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderRequest request)
    {
        // Enrich request with user context
        var userId = User.FindFirst("sub")?.Value;
        
        var orderRequest = new
        {
            UserId = userId,
            Items = request.Items,
            ShippingAddress = request.ShippingAddress,
            Timestamp = DateTime.UtcNow
        };
        
        var response = await _httpClient.PostAsJsonAsync(
            $"{_configuration["Services:Order"]}/api/orders",
            orderRequest);
        
        if (response.IsSuccessStatusCode)
        {
            var order = await response.Content.ReadFromJsonAsync<OrderDto>();
            return CreatedAtAction(nameof(GetOrderById), new { id = order.Id }, order);
        }
        
        return StatusCode((int)response.StatusCode);
    }
}
```

---

## Advanced Features

### 1. **Authentication & Authorization**

```csharp
public class GatewayAuthenticationMiddleware
{
    private readonly RequestDelegate _next;
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Extract token
        var token = context.Request.Headers["Authorization"].FirstOrDefault()?.Split(" ").Last();
        
        if (string.IsNullOrEmpty(token))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Unauthorized");
            return;
        }
        
        // Validate token
        var isValid = await ValidateTokenAsync(token);
        if (!isValid)
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Invalid token");
            return;
        }
        
        await _next(context);
    }
    
    private async Task<bool> ValidateTokenAsync(string token)
    {
        // Call identity service or validate JWT locally
        return true;
    }
}
```

### 2. **Rate Limiting**

```csharp
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    
    public async Task InvokeAsync(HttpContext context, IMemoryCache cache)
    {
        var clientId = context.Connection.RemoteIpAddress?.ToString();
        var cacheKey = $"rate_limit_{clientId}";
        
        if (cache.TryGetValue(cacheKey, out int requestCount))
        {
            if (requestCount >= 100) // 100 requests per minute
            {
                context.Response.StatusCode = 429;
                await context.Response.WriteAsync("Too many requests");
                return;
            }
            
            cache.Set(cacheKey, requestCount + 1, TimeSpan.FromMinutes(1));
        }
        else
        {
            cache.Set(cacheKey, 1, TimeSpan.FromMinutes(1));
        }
        
        await _next(context);
    }
}
```

### 3. **Circuit Breaker with Polly**

```csharp
public class ResilientGatewayController : ControllerBase
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _circuitBreakerPolicy;
    
    public ResilientGatewayController(IHttpClientFactory httpClientFactory)
    {
        _httpClient = httpClientFactory.CreateClient();
        
        _circuitBreakerPolicy = Policy
            .Handle<HttpRequestException>()
            .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .CircuitBreakerAsync(
                handledEventsAllowedBeforeBreaking: 3,
                durationOfBreak: TimeSpan.FromSeconds(30),
                onBreak: (result, duration) =>
                {
                    Console.WriteLine($"Circuit broken for {duration.TotalSeconds}s");
                },
                onReset: () => Console.WriteLine("Circuit reset"));
    }
    
    [HttpGet("products/{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        try
        {
            var response = await _circuitBreakerPolicy.ExecuteAsync(() =>
                _httpClient.GetAsync($"https://productservice/api/products/{id}"));
            
            var product = await response.Content.ReadFromJsonAsync<ProductDto>();
            return Ok(product);
        }
        catch (BrokenCircuitException)
        {
            return StatusCode(503, "Service temporarily unavailable");
        }
    }
}
```

### 4. **Caching**

```csharp
public class CachingGatewayController : ControllerBase
{
    private readonly IDistributedCache _cache;
    private readonly HttpClient _httpClient;
    
    [HttpGet("products/{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var cacheKey = $"product_{id}";
        
        // Try cache first
        var cachedProduct = await _cache.GetStringAsync(cacheKey);
        if (cachedProduct != null)
        {
            var product = JsonSerializer.Deserialize<ProductDto>(cachedProduct);
            return Ok(product);
        }
        
        // Cache miss - call service
        var response = await _httpClient.GetAsync($"https://productservice/api/products/{id}");
        var productDto = await response.Content.ReadFromJsonAsync<ProductDto>();
        
        // Store in cache
        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(productDto),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            });
        
        return Ok(productDto);
    }
}
```

### 5. **Request/Response Transformation**

```csharp
public class TransformationMiddleware
{
    private readonly RequestDelegate _next;
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Transform request
        if (context.Request.Path.StartsWithSegments("/v1/products"))
        {
            // Rewrite to new path
            context.Request.Path = context.Request.Path.Value.Replace("/v1/", "/v2/");
        }
        
        // Capture original response body
        var originalBodyStream = context.Response.Body;
        
        using var responseBody = new MemoryStream();
        context.Response.Body = responseBody;
        
        await _next(context);
        
        // Transform response
        context.Response.Body.Seek(0, SeekOrigin.Begin);
        var responseText = await new StreamReader(context.Response.Body).ReadToEndAsync();
        
        // Modify response (e.g., add metadata)
        var transformedResponse = new
        {
            Data = JsonSerializer.Deserialize<object>(responseText),
            Metadata = new
            {
                Timestamp = DateTime.UtcNow,
                Version = "2.0"
            }
        };
        
        var modifiedResponse = JsonSerializer.Serialize(transformedResponse);
        
        context.Response.Body = originalBodyStream;
        await context.Response.WriteAsync(modifiedResponse);
    }
}
```

---

## Azure API Management (APIM)

For production, use managed services like Azure API Management:

```csharp
// apim-policy.xml
<policies>
    <inbound>
        <base />
        <rate-limit-by-key calls="100" renewal-period="60" counter-key="@(context.Request.IpAddress)" />
        <authentication-jwt
            header-name="Authorization"
            failed-validation-httpcode="401"
            require-scheme="Bearer">
            <issuer-signing-keys>
                <key>{{jwt-signing-key}}</key>
            </issuer-signing-keys>
        </authentication-jwt>
        <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" />
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
        <cache-store duration="60" />
        <set-header name="X-Powered-By" exists-action="delete" />
    </outbound>
    <on-error>
        <base />
        <return-response>
            <set-status code="500" reason="Internal Server Error" />
            <set-body>@{
                return new JObject(
                    new JProperty("error", context.LastError.Message)
                ).ToString();
            }</set-body>
        </return-response>
    </on-error>
</policies>
```

---

## Backend for Frontend (BFF) Pattern

Different gateways for different clients:

```
Mobile App  →  Mobile Gateway  →  Microservices
Web App     →  Web Gateway     →  Microservices
Admin Panel →  Admin Gateway   →  Microservices
```

```csharp
// Mobile-specific gateway
[ApiController]
[Route("mobile/api")]
public class MobileGatewayController : ControllerBase
{
    [HttpGet("dashboard")]
    public async Task<IActionResult> GetDashboard()
    {
        // Return only essential data for mobile
        return Ok(new
        {
            RecentOrders = await GetRecentOrdersSummary(),
            Recommendations = await GetTopRecommendations(5)
        });
    }
}

// Web-specific gateway
[ApiController]
[Route("web/api")]
public class WebGatewayController : ControllerBase
{
    [HttpGet("dashboard")]
    public async Task<IActionResult> GetDashboard()
    {
        // Return comprehensive data for web
        return Ok(new
        {
            RecentOrders = await GetDetailedOrders(),
            Recommendations = await GetRecommendations(20),
            Analytics = await GetUserAnalytics(),
            Notifications = await GetNotifications()
        });
    }
}
```

---

## When to Use API Gateway

✅ **Use API Gateway when:**
- Microservices architecture
- Multiple client types (mobile, web, IoT)
- Need centralized authentication/authorization
- Want to reduce client complexity
- Need protocol translation (REST, gRPC, WebSockets)
- Require rate limiting, caching, logging

❌ **Avoid when:**
- Monolithic application
- Single client type
- Simple internal services
- Gateway becomes a bottleneck

---

## Advantages

✅ **Single Entry Point** - Simplifies client integration  
✅ **Centralized Cross-Cutting Concerns** - Auth, logging, rate limiting  
✅ **Protocol Translation** - REST to gRPC, HTTP to WebSocket  
✅ **Response Aggregation** - Combine multiple service calls  
✅ **Reduces Client Complexity** - Clients don't need to know internal structure

---

## Disadvantages

❌ **Single Point of Failure** - Gateway down = all services down  
❌ **Performance Bottleneck** - All traffic goes through gateway  
❌ **Increased Latency** - Additional network hop  
❌ **Complexity** - Another component to maintain

---

## Best Practices

✅ Keep gateway lightweight (avoid business logic)  
✅ Use circuit breakers to prevent cascading failures  
✅ Implement caching for read-heavy operations  
✅ Use service discovery (Consul, Eureka)  
✅ Monitor gateway performance  
✅ Use API versioning  
✅ Implement request/response logging  
✅ Deploy multiple gateway instances for HA

---

## Related Patterns

- **Backend for Frontend (BFF)** - Specialized gateways per client
- **Service Mesh** - Sidecar proxies for service-to-service communication
- **CQRS** - Separate read and write gateways
- **Circuit Breaker** - Resilience pattern

---

## Common Use Cases

- **E-commerce** - Aggregate product, pricing, inventory data
- **Mobile Apps** - Optimize payloads for mobile clients
- **Microservices** - Single entry point for distributed services
- **Third-Party Integrations** - Facade for external APIs
- **Multi-Tenant Systems** - Route requests based on tenant

---

## Tools & Libraries

- **Ocelot** - .NET API Gateway
- **Azure API Management** - Managed service
- **Kong** - Open-source API Gateway
- **AWS API Gateway** - AWS managed service
- **NGINX** - Reverse proxy / API Gateway
- **Envoy** - Service mesh proxy
- **YARP** - Reverse proxy toolkit for .NET
