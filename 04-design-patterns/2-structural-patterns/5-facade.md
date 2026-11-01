# Facade Pattern

## Overview

**Category:** Structural Pattern  
**Intent:** Provide a unified, simplified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

---

## Problem

A subsystem consists of many complex classes with intricate interactions. Clients need to:
- Understand the entire subsystem to use it
- Manage multiple dependencies
- Deal with complex API calls
- Navigate through detailed implementation

---

## Solution

Create a **Facade** class that provides a simple interface to the complex subsystem. The facade:
- Wraps complex subsystem functionality
- Provides simplified methods
- Hides implementation details
- Reduces client coupling

---

## Structure

```
Client → Facade → [Subsystem Classes]
                  - ClassA
                  - ClassB
                  - ClassC
```

---

## C# Implementation

### Example: Home Theater System

```csharp
// Complex subsystem classes
public class DVDPlayer
{
    public void On() => Console.WriteLine("DVD Player ON");
    public void Play(string movie) => Console.WriteLine($"Playing movie: {movie}");
    public void Stop() => Console.WriteLine("DVD Player stopped");
    public void Off() => Console.WriteLine("DVD Player OFF");
}

public class Projector
{
    public void On() => Console.WriteLine("Projector ON");
    public void WideScreenMode() => Console.WriteLine("Projector in widescreen mode");
    public void Off() => Console.WriteLine("Projector OFF");
}

public class SoundSystem
{
    public void On() => Console.WriteLine("Sound System ON");
    public void SetVolume(int level) => Console.WriteLine($"Sound volume set to {level}");
    public void SetSurroundSound() => Console.WriteLine("Surround sound activated");
    public void Off() => Console.WriteLine("Sound System OFF");
}

public class Lights
{
    public void Dim(int level) => Console.WriteLine($"Lights dimmed to {level}%");
    public void On() => Console.WriteLine("Lights ON");
}

public class Screen
{
    public void Down() => Console.WriteLine("Screen down");
    public void Up() => Console.WriteLine("Screen up");
}

// Facade - Simplified interface
public class HomeTheaterFacade
{
    private readonly DVDPlayer _dvdPlayer;
    private readonly Projector _projector;
    private readonly SoundSystem _soundSystem;
    private readonly Lights _lights;
    private readonly Screen _screen;
    
    public HomeTheaterFacade(
        DVDPlayer dvdPlayer,
        Projector projector,
        SoundSystem soundSystem,
        Lights lights,
        Screen screen)
    {
        _dvdPlayer = dvdPlayer;
        _projector = projector;
        _soundSystem = soundSystem;
        _lights = lights;
        _screen = screen;
    }
    
    // Simplified method - hides complexity
    public void WatchMovie(string movie)
    {
        Console.WriteLine("\nGet ready to watch a movie...\n");
        
        _lights.Dim(10);
        _screen.Down();
        _projector.On();
        _projector.WideScreenMode();
        _soundSystem.On();
        _soundSystem.SetVolume(5);
        _soundSystem.SetSurroundSound();
        _dvdPlayer.On();
        _dvdPlayer.Play(movie);
    }
    
    public void EndMovie()
    {
        Console.WriteLine("\nShutting down movie theater...\n");
        
        _dvdPlayer.Stop();
        _dvdPlayer.Off();
        _soundSystem.Off();
        _projector.Off();
        _screen.Up();
        _lights.On();
    }
}

// Usage
var homeTheater = new HomeTheaterFacade(
    new DVDPlayer(),
    new Projector(),
    new SoundSystem(),
    new Lights(),
    new Screen()
);

// Simple interface for complex operation
homeTheater.WatchMovie("Inception");
// Output:
// Get ready to watch a movie...
// Lights dimmed to 10%
// Screen down
// Projector ON
// Projector in widescreen mode
// Sound System ON
// Sound volume set to 5
// Surround sound activated
// DVD Player ON
// Playing movie: Inception

homeTheater.EndMovie();
// Output:
// Shutting down movie theater...
// DVD Player stopped
// DVD Player OFF
// Sound System OFF
// Projector OFF
// Screen up
// Lights ON
```

---

## Real-World Example: Order Processing System

```csharp
// Complex subsystem
public class InventoryService
{
    public bool CheckStock(int productId, int quantity)
    {
        Console.WriteLine($"Checking stock for product {productId}");
        return true;
    }
    
    public void ReserveInventory(int productId, int quantity)
    {
        Console.WriteLine($"Reserved {quantity} units of product {productId}");
    }
}

public class PaymentService
{
    public bool ProcessPayment(string cardNumber, decimal amount)
    {
        Console.WriteLine($"Processing payment of ${amount}");
        return true;
    }
}

public class ShippingService
{
    public string CreateShipment(string address, List<int> productIds)
    {
        Console.WriteLine($"Creating shipment to {address}");
        return "TRACK-12345";
    }
}

public class NotificationService
{
    public void SendOrderConfirmation(string email, string orderId)
    {
        Console.WriteLine($"Sending confirmation email to {email} for order {orderId}");
    }
}

public class InvoiceService
{
    public void GenerateInvoice(string orderId, decimal amount)
    {
        Console.WriteLine($"Generating invoice for order {orderId}");
    }
}

// Facade
public class OrderFacade
{
    private readonly InventoryService _inventoryService;
    private readonly PaymentService _paymentService;
    private readonly ShippingService _shippingService;
    private readonly NotificationService _notificationService;
    private readonly InvoiceService _invoiceService;
    
    public OrderFacade()
    {
        _inventoryService = new InventoryService();
        _paymentService = new PaymentService();
        _shippingService = new ShippingService();
        _notificationService = new NotificationService();
        _invoiceService = new InvoiceService();
    }
    
    public bool PlaceOrder(OrderRequest order)
    {
        Console.WriteLine($"\n=== Processing Order {order.OrderId} ===\n");
        
        // Step 1: Check inventory
        foreach (var item in order.Items)
        {
            if (!_inventoryService.CheckStock(item.ProductId, item.Quantity))
            {
                Console.WriteLine("Order failed: Out of stock");
                return false;
            }
        }
        
        // Step 2: Process payment
        if (!_paymentService.ProcessPayment(order.CardNumber, order.TotalAmount))
        {
            Console.WriteLine("Order failed: Payment declined");
            return false;
        }
        
        // Step 3: Reserve inventory
        foreach (var item in order.Items)
        {
            _inventoryService.ReserveInventory(item.ProductId, item.Quantity);
        }
        
        // Step 4: Create shipment
        var trackingNumber = _shippingService.CreateShipment(
            order.ShippingAddress,
            order.Items.Select(i => i.ProductId).ToList()
        );
        
        // Step 5: Generate invoice
        _invoiceService.GenerateInvoice(order.OrderId, order.TotalAmount);
        
        // Step 6: Send confirmation
        _notificationService.SendOrderConfirmation(order.Email, order.OrderId);
        
        Console.WriteLine($"\nOrder {order.OrderId} placed successfully!");
        Console.WriteLine($"Tracking number: {trackingNumber}\n");
        
        return true;
    }
}

public class OrderRequest
{
    public string OrderId { get; set; }
    public string Email { get; set; }
    public string CardNumber { get; set; }
    public string ShippingAddress { get; set; }
    public decimal TotalAmount { get; set; }
    public List<OrderItem> Items { get; set; }
}

public class OrderItem
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
}

// Usage - Simple!
var orderFacade = new OrderFacade();

var order = new OrderRequest
{
    OrderId = "ORD-001",
    Email = "customer@example.com",
    CardNumber = "1234-5678-9012-3456",
    ShippingAddress = "123 Main St, City, State",
    TotalAmount = 299.99m,
    Items = new List<OrderItem>
    {
        new() { ProductId = 101, Quantity = 2 },
        new() { ProductId = 202, Quantity = 1 }
    }
};

orderFacade.PlaceOrder(order);
```

---

## ASP.NET Core Example: Database Facade

```csharp
// Complex subsystem - Entity Framework
public class ApplicationDbContext : DbContext
{
    public DbSet<User> Users { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<Product> Products { get; set; }
}

// Repository layer
public class UserRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }
}

public class OrderRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<List<Order>> GetByUserIdAsync(int userId)
    {
        return await _context.Orders
            .Where(o => o.UserId == userId)
            .ToListAsync();
    }
}

// Facade - Simplified service layer
public class UserProfileFacade
{
    private readonly UserRepository _userRepository;
    private readonly OrderRepository _orderRepository;
    private readonly ILogger<UserProfileFacade> _logger;
    
    public UserProfileFacade(
        UserRepository userRepository,
        OrderRepository orderRepository,
        ILogger<UserProfileFacade> logger)
    {
        _userRepository = userRepository;
        _orderRepository = orderRepository;
        _logger = logger;
    }
    
    public async Task<UserProfileDto> GetUserProfileAsync(int userId)
    {
        try
        {
            // Complex orchestration hidden behind simple method
            var user = await _userRepository.GetByIdAsync(userId);
            var orders = await _orderRepository.GetByUserIdAsync(userId);
            
            return new UserProfileDto
            {
                UserId = user.Id,
                Name = user.Name,
                Email = user.Email,
                TotalOrders = orders.Count,
                TotalSpent = orders.Sum(o => o.Total),
                RecentOrders = orders.Take(5).ToList()
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error fetching user profile for user {UserId}", userId);
            throw;
        }
    }
}

// Controller - Very simple!
[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly UserProfileFacade _userProfileFacade;
    
    [HttpGet("{id}/profile")]
    public async Task<IActionResult> GetProfile(int id)
    {
        var profile = await _userProfileFacade.GetUserProfileAsync(id);
        return Ok(profile);
    }
}
```

---

## Third-Party API Facade

```csharp
// Facade to simplify external payment gateway
public class PaymentGatewayFacade
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly IConfiguration _configuration;
    
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        var client = _httpClientFactory.CreateClient("PaymentGateway");
        
        // Complex API calls hidden
        var tokenResponse = await GetAuthTokenAsync(client);
        var validationResponse = await ValidateCardAsync(client, tokenResponse.Token, request.CardNumber);
        
        if (!validationResponse.IsValid)
        {
            return new PaymentResult { Success = false, Error = "Invalid card" };
        }
        
        var chargeResponse = await ChargeCardAsync(client, tokenResponse.Token, request);
        
        return new PaymentResult
        {
            Success = chargeResponse.Success,
            TransactionId = chargeResponse.TransactionId
        };
    }
    
    private async Task<AuthToken> GetAuthTokenAsync(HttpClient client)
    {
        // Complex authentication logic
        var authRequest = new { /* ... */ };
        var response = await client.PostAsJsonAsync("/auth/token", authRequest);
        return await response.Content.ReadFromJsonAsync<AuthToken>();
    }
    
    private async Task<CardValidation> ValidateCardAsync(HttpClient client, string token, string cardNumber)
    {
        // Complex validation logic
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        var response = await client.PostAsJsonAsync("/validate", new { cardNumber });
        return await response.Content.ReadFromJsonAsync<CardValidation>();
    }
    
    private async Task<ChargeResponse> ChargeCardAsync(HttpClient client, string token, PaymentRequest request)
    {
        // Complex charge logic
        var response = await client.PostAsJsonAsync("/charge", request);
        return await response.Content.ReadFromJsonAsync<ChargeResponse>();
    }
}
```

---

## When to Use Facade Pattern

✅ **Use Facade when:**
- A subsystem is complex with many interdependent classes
- You want to provide a simple interface to a complex library/framework
- You need to decouple client code from subsystem implementation
- Migrating from a legacy system (facade wraps old code)
- Integrating third-party libraries with complex APIs

❌ **Avoid when:**
- The subsystem is already simple
- You need access to all subsystem features (facade might hide needed functionality)
- Over-simplification loses necessary flexibility

---

## Advantages

✅ **Simplifies client code** - Easy-to-use interface  
✅ **Reduces coupling** - Clients don't depend on subsystem classes  
✅ **Provides flexibility** - Can change subsystem without affecting clients  
✅ **Improves readability** - Clear, high-level operations  
✅ **Encapsulates complexity** - Hides implementation details

---

## Disadvantages

❌ **Can become a god object** - If facade does too much  
❌ **May hide important details** - Clients lose fine-grained control  
❌ **Additional layer** - Slight performance overhead

---

## Facade vs Other Patterns

| Pattern | Difference |
|---------|-----------|
| **Adapter** | Changes interface to match client expectations; Facade simplifies interface |
| **Proxy** | Same interface as subject; Facade provides different (simplified) interface |
| **Mediator** | Facilitates communication between objects; Facade simplifies subsystem access |

---

## Related Patterns

- **Abstract Factory** - Can use facade to simplify factory usage
- **Singleton** - Facade is often implemented as singleton
- **Mediator** - Both reduce coupling, but mediator focuses on component communication

---

## Common Use Cases

- **Service layers** in ASP.NET Core (simplify repository interactions)
- **Third-party API wrappers** (simplify complex external APIs)
- **Legacy system integration** (facade wraps old code)
- **Complex library usage** (Entity Framework, payment gateways)
- **Microservices** (API Gateway acts as facade)

---

## Best Practices

✅ Keep facade methods focused on specific use cases  
✅ Don't try to hide everything - allow direct subsystem access when needed  
✅ Use dependency injection for subsystem components  
✅ Document what the facade simplifies  
✅ Consider creating multiple facades for different client needs
