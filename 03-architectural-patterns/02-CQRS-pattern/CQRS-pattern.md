# CQRS Pattern (Command Query Responsibility Segregation)

## Overview

**Category:** Architectural Pattern  
**Intent:** Separate read and write operations into different models, optimizing each for its specific use case.

---

## Problem

Traditional CRUD applications use the same model for both reading and writing data. This creates conflicts:
- **Reads** need different data structures than **writes**
- Complex queries require joins that slow down writes
- Scaling reads and writes independently is difficult
- Domain models become bloated with both command and query concerns

---

## Solution

Split the application into two distinct models:
- **Command Model (Write)** - Handles creates, updates, deletes. Optimized for business logic validation.
- **Query Model (Read)** - Handles reads. Optimized for retrieval and reporting.

---

## Architecture

```
Client
  ├─→ Command (Write) → Command Handler → Write Database
  └─→ Query (Read) → Query Handler → Read Database (or Read Model)
  
  Optional: Event Bus synchronizes Read Model from Write Model
```

---

## Basic C# Implementation

### Simple CQRS (Same Database)

```csharp
// Commands (Write operations)
public class CreateOrderCommand
{
    public int CustomerId { get; set; }
    public List<OrderItem> Items { get; set; }
}

public class UpdateOrderStatusCommand
{
    public int OrderId { get; set; }
    public OrderStatus NewStatus { get; set; }
}

// Command Handlers
public class CreateOrderCommandHandler
{
    private readonly ApplicationDbContext _dbContext;
    
    public CreateOrderCommandHandler(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task<int> HandleAsync(CreateOrderCommand command)
    {
        // Validation
        if (command.Items == null || !command.Items.Any())
            throw new ValidationException("Order must have items");
        
        // Business logic
        var order = new Order
        {
            CustomerId = command.CustomerId,
            OrderDate = DateTime.UtcNow,
            Status = OrderStatus.Pending,
            Items = command.Items,
            Total = command.Items.Sum(i => i.Price * i.Quantity)
        };
        
        // Persist
        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync();
        
        return order.Id;
    }
}

// Queries (Read operations)
public class GetOrderByIdQuery
{
    public int OrderId { get; set; }
}

public class GetOrdersByCustomerQuery
{
    public int CustomerId { get; set; }
}

// Query Models (DTOs optimized for reading)
public class OrderDetailsDto
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal Total { get; set; }
    public List<OrderItemDto> Items { get; set; }
}

// Query Handlers
public class GetOrderByIdQueryHandler
{
    private readonly ApplicationDbContext _dbContext;
    
    public GetOrderByIdQueryHandler(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task<OrderDetailsDto> HandleAsync(GetOrderByIdQuery query)
    {
        // Optimized read query
        return await _dbContext.Orders
            .Where(o => o.Id == query.OrderId)
            .Select(o => new OrderDetailsDto
            {
                Id = o.Id,
                CustomerName = o.Customer.Name,
                OrderDate = o.OrderDate,
                Total = o.Total,
                Items = o.Items.Select(i => new OrderItemDto
                {
                    ProductName = i.Product.Name,
                    Quantity = i.Quantity,
                    Price = i.Price
                }).ToList()
            })
            .FirstOrDefaultAsync();
    }
}

// Controller
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly CreateOrderCommandHandler _createOrderHandler;
    private readonly GetOrderByIdQueryHandler _getOrderHandler;
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(CreateOrderCommand command)
    {
        var orderId = await _createOrderHandler.HandleAsync(command);
        return CreatedAtAction(nameof(GetOrder), new { id = orderId }, null);
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(int id)
    {
        var order = await _getOrderHandler.HandleAsync(new GetOrderByIdQuery { OrderId = id });
        return Ok(order);
    }
}
```

---

## Advanced CQRS with MediatR

**MediatR** is a popular .NET library that implements the mediator pattern, perfect for CQRS.

### Installation
```powershell
dotnet add package MediatR
dotnet add package MediatR.Extensions.Microsoft.DependencyInjection
```

### Implementation

```csharp
using MediatR;

// Commands
public class CreateProductCommand : IRequest<int>
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}

public class CreateProductCommandHandler : IRequestHandler<CreateProductCommand, int>
{
    private readonly ApplicationDbContext _dbContext;
    
    public CreateProductCommandHandler(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task<int> Handle(CreateProductCommand request, CancellationToken cancellationToken)
    {
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price,
            CreatedDate = DateTime.UtcNow
        };
        
        _dbContext.Products.Add(product);
        await _dbContext.SaveChangesAsync(cancellationToken);
        
        return product.Id;
    }
}

// Queries
public class GetProductByIdQuery : IRequest<ProductDto>
{
    public int ProductId { get; set; }
}

public class GetProductByIdQueryHandler : IRequestHandler<GetProductByIdQuery, ProductDto>
{
    private readonly ApplicationDbContext _dbContext;
    
    public GetProductByIdQueryHandler(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task<ProductDto> Handle(GetProductByIdQuery request, CancellationToken cancellationToken)
    {
        return await _dbContext.Products
            .Where(p => p.Id == request.ProductId)
            .Select(p => new ProductDto
            {
                Id = p.Id,
                Name = p.Name,
                Price = p.Price
            })
            .FirstOrDefaultAsync(cancellationToken);
    }
}

// Controller
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public ProductsController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateProduct(CreateProductCommand command)
    {
        var productId = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetProduct), new { id = productId }, null);
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _mediator.Send(new GetProductByIdQuery { ProductId = id });
        return Ok(product);
    }
}

// Startup/Program.cs
builder.Services.AddMediatR(cfg => cfg.RegisterServicesFromAssembly(typeof(Program).Assembly));
```

---

## CQRS with Separate Databases

For high-scale systems, use separate databases for read and write operations.

```csharp
// Write Model - Domain entities
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public DateTime OrderDate { get; set; }
    public OrderStatus Status { get; set; }
    public List<OrderItem> Items { get; set; }
}

// Command Handler (writes to SQL Server)
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, int>
{
    private readonly WriteDbContext _writeDb;
    private readonly IEventBus _eventBus;
    
    public async Task<int> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        var order = new Order { /* ... */ };
        
        _writeDb.Orders.Add(order);
        await _writeDb.SaveChangesAsync(cancellationToken);
        
        // Publish event to sync read database
        await _eventBus.PublishAsync(new OrderCreatedEvent
        {
            OrderId = order.Id,
            CustomerId = order.CustomerId,
            Total = order.Items.Sum(i => i.Price * i.Quantity)
        });
        
        return order.Id;
    }
}

// Read Model - Denormalized for fast queries
public class OrderReadModel
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public string CustomerEmail { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal Total { get; set; }
    public int ItemCount { get; set; }
    public string Status { get; set; }
}

// Event Handler - Updates read database
public class OrderCreatedEventHandler : INotificationHandler<OrderCreatedEvent>
{
    private readonly ReadDbContext _readDb;
    private readonly ICustomerService _customerService;
    
    public async Task Handle(OrderCreatedEvent notification, CancellationToken cancellationToken)
    {
        var customer = await _customerService.GetByIdAsync(notification.CustomerId);
        
        var readModel = new OrderReadModel
        {
            Id = notification.OrderId,
            CustomerName = customer.Name,
            CustomerEmail = customer.Email,
            Total = notification.Total,
            OrderDate = DateTime.UtcNow,
            Status = "Pending"
        };
        
        // Store in read database (could be MongoDB, Elasticsearch, etc.)
        await _readDb.OrderReadModels.AddAsync(readModel, cancellationToken);
        await _readDb.SaveChangesAsync(cancellationToken);
    }
}

// Query Handler (reads from optimized read database)
public class GetOrderByIdQueryHandler : IRequestHandler<GetOrderByIdQuery, OrderReadModel>
{
    private readonly ReadDbContext _readDb;
    
    public async Task<OrderReadModel> Handle(GetOrderByIdQuery request, CancellationToken cancellationToken)
    {
        // Fast read from denormalized model
        return await _readDb.OrderReadModels
            .FirstOrDefaultAsync(o => o.Id == request.OrderId, cancellationToken);
    }
}
```

---

## CQRS with Azure

### Using Azure Service Bus for Event Synchronization

```csharp
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, int>
{
    private readonly WriteDbContext _dbContext;
    private readonly ServiceBusSender _serviceBusSender;
    
    public async Task<int> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        var order = new Order { /* ... */ };
        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync(cancellationToken);
        
        // Publish to Azure Service Bus
        var orderCreatedEvent = new OrderCreatedEvent { OrderId = order.Id, /* ... */ };
        var message = new ServiceBusMessage(JsonSerializer.Serialize(orderCreatedEvent));
        await _serviceBusSender.SendMessageAsync(message, cancellationToken);
        
        return order.Id;
    }
}

// Background service to process events
public class OrderEventProcessor : BackgroundService
{
    private readonly ServiceBusProcessor _processor;
    private readonly ReadDbContext _readDb;
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _processor.ProcessMessageAsync += async args =>
        {
            var orderEvent = JsonSerializer.Deserialize<OrderCreatedEvent>(args.Message.Body);
            
            // Update read model
            var readModel = new OrderReadModel { /* ... */ };
            _readDb.OrderReadModels.Add(readModel);
            await _readDb.SaveChangesAsync(stoppingToken);
            
            await args.CompleteMessageAsync(args.Message);
        };
        
        await _processor.StartProcessingAsync(stoppingToken);
    }
}
```

---

## When to Use CQRS

✅ **Use CQRS when:**
- Complex domain logic on writes, but simple reads
- Read and write workloads differ significantly (read-heavy vs write-heavy)
- Need to scale reads and writes independently
- Different teams work on read vs write sides
- Need different security models for reads vs writes
- Event sourcing is being used

❌ **Avoid CQRS when:**
- Simple CRUD operations
- Reads and writes use the same data model
- Low complexity domain
- Small applications (over-engineering)
- Eventual consistency is unacceptable

---

## Advantages

✅ **Scalability** - Scale reads and writes independently  
✅ **Performance** - Optimize each side for its use case  
✅ **Flexibility** - Use different data stores (SQL for writes, NoSQL for reads)  
✅ **Separation of Concerns** - Clear boundaries between reads and writes  
✅ **Optimized Queries** - Denormalized read models avoid complex joins

---

## Disadvantages

❌ **Complexity** - More code, more infrastructure  
❌ **Eventual Consistency** - Read model may be slightly out of sync  
❌ **Data Duplication** - Same data stored in multiple places  
❌ **Development Overhead** - Maintaining two models

---

## CQRS + Event Sourcing

CQRS pairs well with **Event Sourcing**:

```csharp
// Events (immutable facts)
public class OrderPlacedEvent
{
    public int OrderId { get; set; }
    public int CustomerId { get; set; }
    public DateTime Timestamp { get; set; }
}

// Event Store (write side)
public class EventStore
{
    public async Task AppendAsync(string streamId, IEvent @event)
    {
        // Store event in event store
    }
}

// Read Model Projection
public class OrderReadModelProjection : IEventHandler<OrderPlacedEvent>
{
    public async Task Handle(OrderPlacedEvent @event)
    {
        // Update read model from event
    }
}
```

See [Event Sourcing Pattern](../03-event-sourcing/event-sourcing.md) for details.

---

## Related Patterns

- **Event Sourcing** - Often used together with CQRS
- **SAGA Pattern** - For distributed transactions
- **Mediator Pattern** - MediatR library implements this
- **Repository Pattern** - Separate repositories for read and write

---

## Common Use Cases

- **E-commerce** - Complex order processing, simple product catalog queries
- **Financial Systems** - Strict audit trails on writes, complex reporting on reads
- **IoT Platforms** - High-volume writes (telemetry), analytics queries
- **Social Media** - Post creation vs feed generation
- **Inventory Management** - Real-time stock updates vs reporting dashboards

---

## Tools & Libraries

- **MediatR** - Command/query handling
- **Azure Service Bus** - Event messaging
- **EventStore** - Event sourcing database
- **MongoDB/Elasticsearch** - Read model storage
- **SQL Server** - Write model storage
- **Azure Cosmos DB** - Global read replicas
