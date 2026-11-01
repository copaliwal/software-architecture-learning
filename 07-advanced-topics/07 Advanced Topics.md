# 07 Advanced Topics

Deep dive into advanced architectural concepts and patterns for enterprise-scale applications.

---

## Domain-Driven Design (DDD)

### Core Concepts

**Domain-Driven Design** focuses on modeling software based on the business domain with close collaboration between technical and domain experts.

#### 1. **Ubiquitous Language**

A shared language between developers and domain experts.

```csharp
// ❌ Bad - Technical language
public class DataRecord
{
    public int Id { get; set; }
    public string Field1 { get; set; }
    public decimal Field2 { get; set; }
}

// ✅ Good - Domain language
public class Order
{
    public OrderId Id { get; set; }
    public CustomerId CustomerId { get; set; }
    public Money Total { get; set; }
    public OrderStatus Status { get; set; }
}
```

#### 2. **Bounded Contexts**

Explicit boundaries where a particular domain model applies.

```csharp
// Ordering Context
namespace ECommerce.Ordering
{
    public class Product
    {
        public ProductId Id { get; set; }
        public string Name { get; set; }
        public Money Price { get; set; }
    }
}

// Inventory Context - Different view of Product
namespace ECommerce.Inventory
{
    public class Product
    {
        public ProductId Id { get; set; }
        public int StockLevel { get; set; }
        public WarehouseLocation Location { get; set; }
    }
}
```

#### 3. **Aggregates**

Cluster of domain objects treated as a single unit. Each aggregate has a root entity.

```csharp
// Order is the Aggregate Root
public class Order
{
    private readonly List<OrderLine> _orderLines = new();
    
    public OrderId Id { get; private set; }
    public CustomerId CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    
    // Read-only collection
    public IReadOnlyCollection<OrderLine> OrderLines => _orderLines.AsReadOnly();
    
    // Business logic encapsulated
    public void AddOrderLine(ProductId productId, int quantity, Money unitPrice)
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Cannot modify non-draft order");
        
        if (quantity <= 0)
            throw new ArgumentException("Quantity must be positive");
        
        var existingLine = _orderLines.FirstOrDefault(l => l.ProductId == productId);
        if (existingLine != null)
        {
            existingLine.IncreaseQuantity(quantity);
        }
        else
        {
            _orderLines.Add(new OrderLine(productId, quantity, unitPrice));
        }
    }
    
    public void Submit()
    {
        if (!_orderLines.Any())
            throw new InvalidOperationException("Cannot submit empty order");
        
        Status = OrderStatus.Pending;
        
        // Raise domain event
        AddDomainEvent(new OrderSubmittedEvent(Id, CustomerId));
    }
}

// Entity within the aggregate
public class OrderLine
{
    public ProductId ProductId { get; private set; }
    public int Quantity { get; private set; }
    public Money UnitPrice { get; private set; }
    
    public Money LineTotal => new Money(UnitPrice.Amount * Quantity, UnitPrice.Currency);
    
    internal void IncreaseQuantity(int amount)
    {
        Quantity += amount;
    }
}
```

#### 4. **Value Objects**

Objects defined by their attributes rather than identity. Immutable.

```csharp
public record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
        
        return new Money(Amount + other.Amount, Currency);
    }
    
    public Money Multiply(decimal factor)
    {
        return new Money(Amount * factor, Currency);
    }
}

public record Address(
    string Street,
    string City,
    string State,
    string ZipCode,
    string Country)
{
    public bool IsValid()
    {
        return !string.IsNullOrWhiteSpace(Street) &&
               !string.IsNullOrWhiteSpace(City) &&
               !string.IsNullOrWhiteSpace(ZipCode);
    }
}

// Usage
var price1 = new Money(100, "USD");
var price2 = new Money(50, "USD");
var total = price1.Add(price2); // Money(150, "USD")
```

#### 5. **Domain Events**

Events that capture something important that happened in the domain.

```csharp
public abstract class DomainEvent
{
    public Guid EventId { get; } = Guid.NewGuid();
    public DateTime OccurredOn { get; } = DateTime.UtcNow;
}

public class OrderSubmittedEvent : DomainEvent
{
    public OrderId OrderId { get; }
    public CustomerId CustomerId { get; }
    
    public OrderSubmittedEvent(OrderId orderId, CustomerId customerId)
    {
        OrderId = orderId;
        CustomerId = customerId;
    }
}

public class OrderShippedEvent : DomainEvent
{
    public OrderId OrderId { get; }
    public string TrackingNumber { get; }
    
    public OrderShippedEvent(OrderId orderId, string trackingNumber)
    {
        OrderId = orderId;
        TrackingNumber = trackingNumber;
    }
}

// Domain Event Handler
public class OrderSubmittedEventHandler : INotificationHandler<OrderSubmittedEvent>
{
    private readonly IOrderSummaryProjection _orderSummary;
    
    public async Task Handle(OrderSubmittedEvent notification, CancellationToken cancellationToken)
    {
        // Local database work only; it participates in the unit-of-work transaction.
        await _orderSummary.ApplyAsync(notification, cancellationToken);
    }
}
```

MediatR notifications are in-process dispatch, not durable messaging. Dispatch local domain-event
handlers within the database unit of work when their changes must commit atomically. For external
side effects such as inventory calls, email, or broker publication, persist an integration message
through a [transactional outbox](../03-architectural-patterns/distributed-systems-patterns.md#transactional-outbox)
and process it asynchronously.

#### 6. **Repositories**

Abstractions for accessing aggregates.

```csharp
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(OrderId id);
    Task<IEnumerable<Order>> GetByCustomerAsync(CustomerId customerId);
    Task AddAsync(Order order);
    Task UpdateAsync(Order order);
}

public class OrderRepository : IOrderRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<Order> GetByIdAsync(OrderId id)
    {
        return await _context.Orders
            .Include(o => o.OrderLines)
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    public async Task AddAsync(Order order)
    {
        await using var transaction = await _context.Database.BeginTransactionAsync();
        _context.Orders.Add(order);

        await DispatchDomainEventsAsync(order);
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();
        order.ClearDomainEvents();
    }
    
    private async Task DispatchDomainEventsAsync(Order order)
    {
        var domainEvents = order.DomainEvents.ToList();
        
        foreach (var domainEvent in domainEvents)
        {
            await _mediator.Publish(domainEvent);
        }
    }
}
```

#### 7. **Domain Services**

Operations that don't naturally fit in entities or value objects.

```csharp
public class OrderPricingService
{
    public Money CalculateTotalWithDiscount(Order order, Customer customer)
    {
        var subtotal = order.OrderLines
            .Select(line => line.LineTotal)
            .Aggregate((a, b) => a.Add(b));
        
        // Apply customer-specific discount
        decimal discountPercentage = customer.CustomerType switch
        {
            CustomerType.Regular => 0,
            CustomerType.Premium => 0.05m,
            CustomerType.VIP => 0.10m,
            _ => 0
        };
        
        var discount = subtotal.Multiply(discountPercentage);
        return subtotal.Add(discount.Multiply(-1)); // Subtract discount
    }
}
```

---

## Enterprise Integration Patterns

### 1. **Orchestration vs Choreography**

#### Orchestration (Centralized Control)

```csharp
public class OrderOrchestrator
{
    private readonly IPaymentService _paymentService;
    private readonly IInventoryService _inventoryService;
    private readonly IShippingService _shippingService;
    
    public async Task ProcessOrderAsync(Order order)
    {
        try
        {
            // Step 1: Process payment
            var paymentResult = await _paymentService.ChargeAsync(
                order.CustomerId, 
                order.Total);
            
            if (!paymentResult.Success)
                throw new PaymentFailedException();
            
            // Step 2: Reserve inventory
            await _inventoryService.ReserveAsync(order.OrderLines);
            
            // Step 3: Create shipment
            var trackingNumber = await _shippingService.CreateShipmentAsync(order);
            
            // Step 4: Update order status
            order.MarkAsShipped(trackingNumber);
        }
        catch (Exception ex)
        {
            // Centralized error handling and compensation
            await CompensateAsync(order);
            throw;
        }
    }
    
    private async Task CompensateAsync(Order order)
    {
        // Rollback operations
        await _paymentService.RefundAsync(order.CustomerId, order.Total);
        await _inventoryService.ReleaseAsync(order.OrderLines);
    }
}
```

This is illustrative pseudocode, not a complete saga implementation. A production orchestrator
persists progress, compensates only steps known to have completed, makes commands idempotent, and
retries or escalates failed compensations. A timeout does not prove that a remote step failed.

#### Choreography (Event-Driven)

```csharp
// No central orchestrator - services react to events

// Order Service
public class OrderService
{
    public async Task SubmitOrderAsync(Order order, CancellationToken ct)
    {
        order.Submit();
        // Atomically persist the order and its integration-event outbox record.
        await _repository.SaveOrderAndOutboxMessageAsync(
            order,
            new OrderSubmittedEvent(order.Id, order.Total),
            ct);
        
    }
}

// Payment Service - Subscribes to OrderSubmittedEvent
public class PaymentEventHandler : IEventHandler<OrderSubmittedEvent>
{
    public async Task Handle(OrderSubmittedEvent @event)
    {
        var result = await _paymentService.ChargeAsync(@event.CustomerId, @event.Total);
        
        if (result.Success)
        {
            await _eventBus.PublishAsync(new PaymentProcessedEvent(@event.OrderId));
        }
        else
        {
            await _eventBus.PublishAsync(new PaymentFailedEvent(@event.OrderId));
        }
    }
}

// Inventory Service - Subscribes to PaymentProcessedEvent
public class InventoryEventHandler : IEventHandler<PaymentProcessedEvent>
{
    public async Task Handle(PaymentProcessedEvent @event)
    {
        await _inventoryService.ReserveAsync(@event.OrderId);
        await _eventBus.PublishAsync(new InventoryReservedEvent(@event.OrderId));
    }
}

// Shipping Service - Subscribes to InventoryReservedEvent
public class ShippingEventHandler : IEventHandler<InventoryReservedEvent>
{
    public async Task Handle(InventoryReservedEvent @event)
    {
        var trackingNumber = await _shippingService.CreateShipmentAsync(@event.OrderId);
        await _eventBus.PublishAsync(new OrderShippedEvent(@event.OrderId, trackingNumber));
    }
}
```

An outbox dispatcher publishes committed records after the order method returns. Each service
that changes its database and emits another integration event should use the same durable
handoff; consumers should be idempotent. See [Distributed Systems Patterns](../03-architectural-patterns/distributed-systems-patterns.md).

### 2. **Message Patterns**

#### Request-Reply

```csharp
public class RequestReplyService
{
    private readonly ServiceBusClient _client;
    
    public async Task<PaymentResponse> ProcessPaymentAsync(PaymentRequest request)
    {
        var sender = _client.CreateSender("payment-requests");
        var receiver = _client.CreateReceiver("payment-responses");
        
        var correlationId = Guid.NewGuid().ToString();
        
        var message = new ServiceBusMessage(JsonSerializer.Serialize(request))
        {
            CorrelationId = correlationId,
            ReplyTo = "payment-responses"
        };
        
        await sender.SendMessageAsync(message);
        
        // Wait for response
        var response = await receiver.ReceiveMessageAsync(TimeSpan.FromSeconds(30));
        
        if (response.CorrelationId == correlationId)
        {
            return JsonSerializer.Deserialize<PaymentResponse>(response.Body);
        }
        
        throw new TimeoutException("No response received");
    }
}
```

#### Publish-Subscribe

```csharp
// Publisher
public class EventPublisher
{
    private readonly ServiceBusClient _client;
    
    public async Task PublishAsync<T>(T @event) where T : class
    {
        var sender = _client.CreateSender("order-events");
        
        var message = new ServiceBusMessage(JsonSerializer.Serialize(@event))
        {
            Subject = typeof(T).Name,
            ContentType = "application/json"
        };
        
        await sender.SendMessageAsync(message);
    }
}

// Subscriber
public class EventSubscriber : BackgroundService
{
    private readonly ServiceBusProcessor _processor;
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _processor.ProcessMessageAsync += async args =>
        {
            var eventType = args.Message.Subject;
            var eventData = args.Message.Body.ToString();
            
            // Route to appropriate handler
            await RouteEventAsync(eventType, eventData);
            
            await args.CompleteMessageAsync(args.Message);
        };
        
        await _processor.StartProcessingAsync(stoppingToken);
    }
}
```

---

## Cloud-Native Architecture

### 1. **Containers & Kubernetes**

```dockerfile
# Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp.csproj", "./"]
RUN dotnet restore "MyApp.csproj"
COPY . .
RUN dotnet build "MyApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

```yaml
# kubernetes-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myregistry.azurecr.io/myapp:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: connection-string
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

### 2. **Sidecar Pattern**

```yaml
# Pod with sidecar for logging
apiVersion: v1
kind: Pod
metadata:
  name: myapp-with-sidecar
spec:
  containers:
  - name: myapp
    image: myapp:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/myapp
  
  - name: log-forwarder
    image: fluent/fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/myapp
    - name: fluentd-config
      mountPath: /fluentd/etc
  
  volumes:
  - name: shared-logs
    emptyDir: {}
  - name: fluentd-config
    configMap:
      name: fluentd-config
```

---

## Observability

### Application Insights Integration

```csharp
// Program.cs
builder.Services.AddApplicationInsightsTelemetry();

// Custom telemetry
public class OrderService
{
    private readonly TelemetryClient _telemetry;
    
    public async Task ProcessOrderAsync(Order order)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            await _repository.SaveAsync(order);
            
            // Track custom metric
            _telemetry.TrackMetric("OrderProcessingTime", stopwatch.ElapsedMilliseconds);
            
            // Track custom event
            _telemetry.TrackEvent("OrderProcessed", new Dictionary<string, string>
            {
                { "OrderId", order.Id.ToString() },
                { "Total", order.Total.Amount.ToString() }
            });
        }
        catch (Exception ex)
        {
            _telemetry.TrackException(ex);
            throw;
        }
    }
}
```

---

## Risk Management

### Technical Debt Tracking

```csharp
// TODO: TECH-DEBT-001 - Refactor to use async/await
// Current sync calls blocking threads under high load
// Priority: High | Estimated effort: 3 days
public Order GetOrder(int id)
{
    return _repository.GetById(id);
}

// TODO: TECH-DEBT-002 - Extract to separate service
// This method violates SRP and should be in OrderPricingService
// Priority: Medium | Estimated effort: 1 day
public decimal CalculateDiscount(Order order)
{
    // Complex discount logic
}
```

---

## Summary

✅ **DDD** - Bounded contexts, aggregates, value objects, domain events  
✅ **Integration** - Orchestration vs choreography, messaging patterns  
✅ **Cloud-Native** - Containers, Kubernetes, sidecar pattern  
✅ **Observability** - Logging, tracing, monitoring (Application Insights)  
✅ **Risk Management** - Technical debt tracking, trade-off evaluation

---

See also: [Architecting Cloud-Native .NET Apps for Azure](Architecting%20Cloud-Native%20NET%20Apps%20for%20Azure.md)