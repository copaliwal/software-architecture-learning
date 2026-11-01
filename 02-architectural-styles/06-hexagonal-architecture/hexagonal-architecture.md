# Hexagonal Architecture (Ports and Adapters)

## 📖 Definition

**Hexagonal Architecture**, also known as **Ports and Adapters**, is an architectural style that aims to create loosely coupled application components that can be connected to their software environment through **ports** and **adapters**. This allows components to be easily tested and swapped.

Introduced by **Alistair Cockburn** in 2005, it's also closely related to **Clean Architecture** and **Onion Architecture**.

---

## 🎯 Core Characteristics

- **Domain-Centric** - Business logic at the core, isolated from external concerns
- **Dependency Inversion** - Dependencies point inward toward domain
- **Ports** - Interfaces that define how application interacts with outside world
- **Adapters** - Implementations that connect ports to external systems
- **Testability** - Core logic testable without external dependencies
- **Technology Agnostic** - Core doesn't know about frameworks, databases, UI

---

## 🏛 Scope

- **Application level**
- Defines how an application's components are organized
- Emphasizes separation between business logic and infrastructure
- Popular in Domain-Driven Design implementations

---

## 🏗️ Architecture Layers

```
┌─────────────────────────────────────────────────────┐
│              EXTERNAL SYSTEMS                       │
│  (Web, CLI, Tests, Database, APIs, Message Queue)  │
└─────────────┬───────────────────────┬───────────────┘
              │                       │
              ↓                       ↓
┌─────────────────────────┐ ┌─────────────────────────┐
│   PRIMARY ADAPTERS      │ │  SECONDARY ADAPTERS     │
│  (Driving/Input)        │ │  (Driven/Output)        │
│  - REST Controllers     │ │  - Database Repos       │
│  - GraphQL Resolvers    │ │  - Email Services       │
│  - CLI Commands         │ │  - External APIs        │
│  - Message Consumers    │ │  - File Systems         │
└───────────┬─────────────┘ └─────────────┬───────────┘
            │                             │
            ↓                             ↓
    ┌───────────────┐           ┌───────────────┐
    │ PRIMARY PORTS │           │SECONDARY PORTS│
    │  (Interfaces) │           │  (Interfaces) │
    └───────┬───────┘           └───────┬───────┘
            │                           │
            ↓                           ↓
    ┌───────────────────────────────────────────┐
    │                                           │
    │         APPLICATION CORE (HEXAGON)        │
    │                                           │
    │  ┌─────────────────────────────────────┐ │
    │  │      Domain Layer (Entities)        │ │
    │  │  - Business Rules                   │ │
    │  │  - Domain Logic                     │ │
    │  │  - Value Objects                    │ │
    │  └─────────────────────────────────────┘ │
    │                                           │
    │  ┌─────────────────────────────────────┐ │
    │  │    Application Layer (Use Cases)    │ │
    │  │  - Application Services             │ │
    │  │  - Command/Query Handlers           │ │
    │  │  - Business Workflows               │ │
    │  └─────────────────────────────────────┘ │
    │                                           │
    └───────────────────────────────────────────┘
```

---

## ⚙️ Key Components

### 1. Domain Layer (Core)

**Responsibility:** Business entities and rules

```csharp
// Domain Entity
public class Order
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();
    
    public decimal TotalAmount => _items.Sum(i => i.Price * i.Quantity);
    
    private Order() { } // For EF Core
    
    public Order(Guid customerId)
    {
        Id = Guid.NewGuid();
        CustomerId = customerId;
        Status = OrderStatus.Draft;
    }
    
    // Business logic
    public void AddItem(Guid productId, string productName, decimal price, int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Cannot add items to non-draft order");
        
        if (quantity <= 0)
            throw new ArgumentException("Quantity must be positive");
        
        var existingItem = _items.FirstOrDefault(i => i.ProductId == productId);
        if (existingItem != null)
        {
            existingItem.IncreaseQuantity(quantity);
        }
        else
        {
            _items.Add(new OrderItem(productId, productName, price, quantity));
        }
    }
    
    public void RemoveItem(Guid productId)
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Cannot remove items from non-draft order");
        
        var item = _items.FirstOrDefault(i => i.ProductId == productId);
        if (item != null)
        {
            _items.Remove(item);
        }
    }
    
    public void Submit()
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Order already submitted");
        
        if (!_items.Any())
            throw new InvalidOperationException("Cannot submit empty order");
        
        Status = OrderStatus.Submitted;
    }
    
    public void Confirm()
    {
        if (Status != OrderStatus.Submitted)
            throw new InvalidOperationException("Only submitted orders can be confirmed");
        
        Status = OrderStatus.Confirmed;
    }
    
    public void Cancel(string reason)
    {
        if (Status == OrderStatus.Shipped || Status == OrderStatus.Delivered)
            throw new InvalidOperationException("Cannot cancel shipped or delivered orders");
        
        Status = OrderStatus.Cancelled;
    }
}

// Value Object
public class OrderItem
{
    public Guid ProductId { get; private set; }
    public string ProductName { get; private set; }
    public decimal Price { get; private set; }
    public int Quantity { get; private set; }
    
    private OrderItem() { } // For EF Core
    
    public OrderItem(Guid productId, string productName, decimal price, int quantity)
    {
        ProductId = productId;
        ProductName = productName;
        Price = price;
        Quantity = quantity;
    }
    
    public void IncreaseQuantity(int amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        
        Quantity += amount;
    }
}

// Enum
public enum OrderStatus
{
    Draft,
    Submitted,
    Confirmed,
    Shipped,
    Delivered,
    Cancelled
}
```

### 2. Application Layer (Use Cases)

**Responsibility:** Orchestrate domain logic, define use cases

```csharp
// Command (Input)
public class CreateOrderCommand
{
    public Guid CustomerId { get; set; }
    public List<OrderItemDto> Items { get; set; }
}

public class OrderItemDto
{
    public Guid ProductId { get; set; }
    public int Quantity { get; set; }
}

// Command Handler (Application Service)
public class CreateOrderCommandHandler
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;
    private readonly IUnitOfWork _unitOfWork;
    private readonly IEventPublisher _eventPublisher;
    
    public CreateOrderCommandHandler(
        IOrderRepository orderRepository,
        IProductRepository productRepository,
        IUnitOfWork unitOfWork,
        IEventPublisher eventPublisher)
    {
        _orderRepository = orderRepository;
        _productRepository = productRepository;
        _unitOfWork = unitOfWork;
        _eventPublisher = eventPublisher;
    }
    
    public async Task<Guid> HandleAsync(CreateOrderCommand command)
    {
        // Create order
        var order = new Order(command.CustomerId);
        
        // Add items
        foreach (var itemDto in command.Items)
        {
            var product = await _productRepository.GetByIdAsync(itemDto.ProductId);
            
            if (product == null)
                throw new NotFoundException($"Product {itemDto.ProductId} not found");
            
            if (!product.IsInStock(itemDto.Quantity))
                throw new InsufficientStockException($"Product {product.Name} is out of stock");
            
            order.AddItem(
                product.Id,
                product.Name,
                product.Price,
                itemDto.Quantity);
        }
        
        // Submit order
        order.Submit();
        
        // Save
        await _orderRepository.AddAsync(order);
        await _unitOfWork.CommitAsync();
        
        // Publish event
        await _eventPublisher.PublishAsync(new OrderCreatedEvent
        {
            OrderId = order.Id,
            CustomerId = order.CustomerId,
            TotalAmount = order.TotalAmount
        });
        
        return order.Id;
    }
}

// Query
public class GetOrderQuery
{
    public Guid OrderId { get; set; }
}

// Query Handler
public class GetOrderQueryHandler
{
    private readonly IOrderRepository _orderRepository;
    
    public GetOrderQueryHandler(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }
    
    public async Task<OrderDto> HandleAsync(GetOrderQuery query)
    {
        var order = await _orderRepository.GetByIdAsync(query.OrderId);
        
        if (order == null)
            throw new NotFoundException($"Order {query.OrderId} not found");
        
        return new OrderDto
        {
            Id = order.Id,
            CustomerId = order.CustomerId,
            Status = order.Status.ToString(),
            Items = order.Items.Select(i => new OrderItemDto
            {
                ProductId = i.ProductId,
                ProductName = i.ProductName,
                Price = i.Price,
                Quantity = i.Quantity
            }).ToList(),
            TotalAmount = order.TotalAmount
        };
    }
}
```

### 3. Ports (Interfaces)

**Secondary Ports (driven by application):**

```csharp
// Repository Port (Interface defined in Domain, implemented in Infrastructure)
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(Guid id);
    Task<IEnumerable<Order>> GetByCustomerIdAsync(Guid customerId);
    Task AddAsync(Order order);
    Task UpdateAsync(Order order);
    Task DeleteAsync(Guid id);
}

public interface IProductRepository
{
    Task<Product> GetByIdAsync(Guid id);
    Task<IEnumerable<Product>> GetAllAsync();
}

public interface IUnitOfWork
{
    Task CommitAsync();
    Task RollbackAsync();
}

// Event Publisher Port
public interface IEventPublisher
{
    Task PublishAsync<T>(T @event) where T : class;
}

// Email Service Port
public interface IEmailService
{
    Task SendOrderConfirmationAsync(Guid customerId, Guid orderId);
    Task SendShippingNotificationAsync(Guid customerId, string trackingNumber);
}

// Payment Service Port
public interface IPaymentService
{
    Task<PaymentResult> ProcessPaymentAsync(Guid orderId, decimal amount, PaymentDetails details);
}
```

**Primary Ports are the application services/handlers themselves**

### 4. Adapters

**Primary Adapters (drive the application):**

```csharp
// REST API Adapter (ASP.NET Core Controller)
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly CreateOrderCommandHandler _createOrderHandler;
    private readonly GetOrderQueryHandler _getOrderHandler;
    
    public OrdersController(
        CreateOrderCommandHandler createOrderHandler,
        GetOrderQueryHandler getOrderHandler)
    {
        _createOrderHandler = createOrderHandler;
        _getOrderHandler = getOrderHandler;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderRequest request)
    {
        var command = new CreateOrderCommand
        {
            CustomerId = request.CustomerId,
            Items = request.Items
        };
        
        var orderId = await _createOrderHandler.HandleAsync(command);
        
        return CreatedAtAction(nameof(GetOrder), new { id = orderId }, null);
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(Guid id)
    {
        var query = new GetOrderQuery { OrderId = id };
        var order = await _getOrderHandler.HandleAsync(query);
        return Ok(order);
    }
}

// CLI Adapter
public class OrderCliAdapter
{
    private readonly CreateOrderCommandHandler _createOrderHandler;
    
    public async Task CreateOrderFromCli(string[] args)
    {
        var customerId = Guid.Parse(args[0]);
        // Parse items from args...
        
        var command = new CreateOrderCommand
        {
            CustomerId = customerId,
            Items = new List<OrderItemDto>()
        };
        
        var orderId = await _createOrderHandler.HandleAsync(command);
        Console.WriteLine($"Order created: {orderId}");
    }
}

// Message Queue Adapter
public class OrderMessageConsumer
{
    private readonly CreateOrderCommandHandler _handler;
    
    public async Task HandleMessage(OrderMessage message)
    {
        var command = new CreateOrderCommand
        {
            CustomerId = message.CustomerId,
            Items = message.Items
        };
        
        await _handler.HandleAsync(command);
    }
}
```

**Secondary Adapters (driven by application):**

```csharp
// SQL Repository Adapter (implements IOrderRepository)
public class SqlOrderRepository : IOrderRepository
{
    private readonly ApplicationDbContext _context;
    
    public SqlOrderRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<Order> GetByIdAsync(Guid id)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    public async Task<IEnumerable<Order>> GetByCustomerIdAsync(Guid customerId)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .Where(o => o.CustomerId == customerId)
            .ToListAsync();
    }
    
    public async Task AddAsync(Order order)
    {
        await _context.Orders.AddAsync(order);
    }
    
    public async Task UpdateAsync(Order order)
    {
        _context.Orders.Update(order);
    }
    
    public async Task DeleteAsync(Guid id)
    {
        var order = await GetByIdAsync(id);
        if (order != null)
        {
            _context.Orders.Remove(order);
        }
    }
}

// In-Memory Repository Adapter (for testing)
public class InMemoryOrderRepository : IOrderRepository
{
    private readonly Dictionary<Guid, Order> _orders = new();
    
    public Task<Order> GetByIdAsync(Guid id)
    {
        _orders.TryGetValue(id, out var order);
        return Task.FromResult(order);
    }
    
    public Task<IEnumerable<Order>> GetByCustomerIdAsync(Guid customerId)
    {
        var orders = _orders.Values.Where(o => o.CustomerId == customerId);
        return Task.FromResult(orders);
    }
    
    public Task AddAsync(Order order)
    {
        _orders[order.Id] = order;
        return Task.CompletedTask;
    }
    
    public Task UpdateAsync(Order order)
    {
        _orders[order.Id] = order;
        return Task.CompletedTask;
    }
    
    public Task DeleteAsync(Guid id)
    {
        _orders.Remove(id);
        return Task.CompletedTask;
    }
}

// SMTP Email Adapter
public class SmtpEmailService : IEmailService
{
    private readonly SmtpClient _smtpClient;
    
    public async Task SendOrderConfirmationAsync(Guid customerId, Guid orderId)
    {
        var message = new MailMessage
        {
            Subject = "Order Confirmation",
            Body = $"Your order {orderId} has been confirmed"
        };
        
        await _smtpClient.SendMailAsync(message);
    }
    
    public async Task SendShippingNotificationAsync(Guid customerId, string trackingNumber)
    {
        var message = new MailMessage
        {
            Subject = "Your order has shipped",
            Body = $"Tracking number: {trackingNumber}"
        };
        
        await _smtpClient.SendMailAsync(message);
    }
}

// Azure Service Bus Event Publisher Adapter
public class ServiceBusEventPublisher : IEventPublisher
{
    private readonly ServiceBusClient _client;
    
    public async Task PublishAsync<T>(T @event) where T : class
    {
        var sender = _client.CreateSender(typeof(T).Name);
        var message = new ServiceBusMessage(JsonSerializer.Serialize(@event));
        await sender.SendMessageAsync(message);
    }
}
```

---

## 📁 Project Structure

```
MyApp.Solution/
├── MyApp.Domain/                    (Core - No dependencies)
│   ├── Entities/
│   │   ├── Order.cs
│   │   ├── Product.cs
│   │   └── Customer.cs
│   ├── ValueObjects/
│   │   ├── OrderItem.cs
│   │   └── Address.cs
│   ├── Enums/
│   │   └── OrderStatus.cs
│   └── Exceptions/
│       ├── InsufficientStockException.cs
│       └── InvalidOrderException.cs
│
├── MyApp.Application/               (Use Cases - Depends on Domain)
│   ├── Commands/
│   │   ├── CreateOrderCommand.cs
│   │   └── CancelOrderCommand.cs
│   ├── CommandHandlers/
│   │   ├── CreateOrderCommandHandler.cs
│   │   └── CancelOrderCommandHandler.cs
│   ├── Queries/
│   │   └── GetOrderQuery.cs
│   ├── QueryHandlers/
│   │   └── GetOrderQueryHandler.cs
│   ├── DTOs/
│   │   └── OrderDto.cs
│   ├── Ports/ (Secondary Ports - Interfaces)
│   │   ├── IOrderRepository.cs
│   │   ├── IEmailService.cs
│   │   ├── IPaymentService.cs
│   │   └── IEventPublisher.cs
│   └── Events/
│       └── OrderCreatedEvent.cs
│
├── MyApp.Infrastructure/            (Secondary Adapters)
│   ├── Persistence/
│   │   ├── ApplicationDbContext.cs
│   │   ├── SqlOrderRepository.cs
│   │   └── Configurations/
│   ├── ExternalServices/
│   │   ├── SmtpEmailService.cs
│   │   ├── StripePaymentService.cs
│   │   └── ServiceBusEventPublisher.cs
│   └── InMemory/ (For testing)
│       └── InMemoryOrderRepository.cs
│
└── MyApp.Api/                       (Primary Adapter - REST)
    ├── Controllers/
    │   ├── OrdersController.cs
    │   └── ProductsController.cs
    ├── Requests/
    │   └── CreateOrderRequest.cs
    ├── Responses/
    │   └── OrderResponse.cs
    └── Program.cs
```

---

## 🔧 Dependency Injection

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Application Layer
builder.Services.AddScoped<CreateOrderCommandHandler>();
builder.Services.AddScoped<GetOrderQueryHandler>();

// Infrastructure Layer - Secondary Adapters
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.AddScoped<IProductRepository, SqlProductRepository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<IEmailService, SmtpEmailService>();
builder.Services.AddScoped<IPaymentService, StripePaymentService>();
builder.Services.AddScoped<IEventPublisher, ServiceBusEventPublisher>();

// For testing, swap implementations:
// builder.Services.AddScoped<IOrderRepository, InMemoryOrderRepository>();

var app = builder.Build();
```

---

## ✅ When to Use Hexagonal Architecture

✅ **Use when:**
- Business logic is complex and valuable
- Need high testability
- Want technology independence
- Multiple interfaces to same domain (REST, CLI, GraphQL)
- Long-term maintainability is critical
- Implementing Domain-Driven Design

❌ **Avoid when:**
- Simple CRUD applications
- Prototypes or short-lived projects
- Team unfamiliar with the pattern
- Rapid delivery more important than architecture

---

## 👍 Advantages

✅ **Technology Independence** - Core logic doesn't depend on frameworks  
✅ **Highly Testable** - Domain can be tested without external dependencies  
✅ **Flexible** - Easy to swap adapters (database, UI, external services)  
✅ **Clear Boundaries** - Well-defined separation of concerns  
✅ **Domain-Focused** - Business logic is first-class citizen  
✅ **Multiple Interfaces** - Same core for web, mobile, CLI

---

## 👎 Disadvantages

❌ **Complexity** - More files, interfaces, and abstractions  
❌ **Learning Curve** - Team needs to understand the pattern  
❌ **Overhead** - May be overkill for simple applications  
❌ **Indirection** - More layers to navigate

---

## 🎯 Common Use Cases

- **Domain-Driven Design** - Natural fit for complex domains
- **Multi-Interface Applications** - Web + Mobile + API
- **Enterprise Applications** - Long-lived, complex business logic
- **Microservices** - Clean service boundaries
- **Applications requiring high testability**

---

## 🏗️ Best Practices

✅ **Keep domain pure** - No framework dependencies in core  
✅ **Define clear ports** - Well-defined interfaces  
✅ **Use dependency injection** - Wire adapters at composition root  
✅ **Test domain in isolation** - Use in-memory adapters  
✅ **Separate commands from queries** - CQRS pattern  
✅ **Use value objects** - Encapsulate domain concepts  
✅ **Apply ubiquitous language** - Domain terms in code

---

## 📚 Related Architectures

- **Clean Architecture** (Robert C. Martin) - Similar concepts, different visualization
- **Onion Architecture** (Jeffrey Palermo) - Emphasizes layers around core
- **Domain-Driven Design** - Often implemented with hexagonal architecture
- **CQRS** - Commonly used within hexagonal architecture