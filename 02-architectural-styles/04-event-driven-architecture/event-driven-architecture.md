# Event-Driven Architecture (EDA)

## 📖 Definition

**Event-Driven Architecture (EDA)** is an architectural style where system components communicate through the production, detection, consumption, and reaction to events. An event represents a significant change in state or an important occurrence in the system.

---

## 🎯 Core Characteristics

- **Loose Coupling** - Producers and consumers don't need to know about each other
- **Asynchronous Communication** - Events are processed asynchronously
- **Event-First Design** - System behavior is driven by events, not direct calls
- **Scalability** - Components can scale independently
- **Real-Time Processing** - Events are processed as they occur
- **Reactive** - System reacts to state changes rather than polling

---

## 🏛 Scope

- **System/Enterprise level**
- EDA is an architectural style that defines how the entire system or major subsystems communicate
- Often combined with microservices, serverless, or other patterns

---

## ⚙️ Key Components

### 1. Event Producers
Components that generate events when something significant happens.

```csharp
// Example: Order Service producing events
public class OrderService
{
    private readonly IEventBus _eventBus;
    
    public async Task CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order
        {
            Id = Guid.NewGuid(),
            CustomerId = request.CustomerId,
            TotalAmount = request.TotalAmount,
            Status = OrderStatus.Created
        };
        
        await _orderRepository.AddAsync(order);
        
        // Publish event
        await _eventBus.PublishAsync(new OrderCreatedEvent
        {
            OrderId = order.Id,
            CustomerId = order.CustomerId,
            TotalAmount = order.TotalAmount,
            CreatedAt = DateTime.UtcNow
        });
    }
}
```

### 2. Event Consumers
Components that subscribe to and react to events.

```csharp
// Example: Inventory Service consuming events
public class OrderCreatedEventHandler : IEventHandler<OrderCreatedEvent>
{
    private readonly IInventoryService _inventoryService;
    
    public async Task HandleAsync(OrderCreatedEvent @event)
    {
        Console.WriteLine($"Reserving inventory for order {@event.OrderId}");
        await _inventoryService.ReserveItemsAsync(@event.OrderId);
    }
}

// Example: Notification Service consuming events
public class OrderNotificationHandler : IEventHandler<OrderCreatedEvent>
{
    private readonly IEmailService _emailService;
    
    public async Task HandleAsync(OrderCreatedEvent @event)
    {
        Console.WriteLine($"Sending order confirmation for {@event.OrderId}");
        await _emailService.SendOrderConfirmationAsync(@event.CustomerId, @event.OrderId);
    }
}
```

### 3. Event Bus/Broker
Infrastructure that routes events from producers to consumers.

```csharp
// Event Bus Interface
public interface IEventBus
{
    Task PublishAsync<T>(T @event) where T : class;
    void Subscribe<T, TH>() where T : class where TH : IEventHandler<T>;
}

// Azure Service Bus Implementation
public class AzureServiceBusEventBus : IEventBus
{
    private readonly ServiceBusClient _serviceBusClient;
    private readonly IServiceProvider _serviceProvider;
    
    public async Task PublishAsync<T>(T @event) where T : class
    {
        var topicName = typeof(T).Name;
        var sender = _serviceBusClient.CreateSender(topicName);
        
        var message = new ServiceBusMessage(JsonSerializer.Serialize(@event))
        {
            MessageId = Guid.NewGuid().ToString(),
            ContentType = "application/json"
        };
        
        await sender.SendMessageAsync(message);
    }
    
    public void Subscribe<T, TH>() where T : class where TH : IEventHandler<T>
    {
        var topicName = typeof(T).Name;
        var subscriptionName = typeof(TH).Name;
        
        var processor = _serviceBusClient.CreateProcessor(topicName, subscriptionName);
        
        processor.ProcessMessageAsync += async args =>
        {
            var @event = JsonSerializer.Deserialize<T>(args.Message.Body.ToString());
            var handler = _serviceProvider.GetRequiredService<TH>();
            await handler.HandleAsync(@event);
            await args.CompleteMessageAsync(args.Message);
        };
        
        processor.StartProcessingAsync();
    }
}
```

The producer examples above are intentionally simplified. Saving business state and then
publishing directly creates a dual-write failure window: either step can succeed while the other
fails. When publication must reliably follow a database change, write an outbox record in the
same local transaction and publish it with a relay; see
[Transactional Outbox](../../03-architectural-patterns/distributed-systems-patterns.md#transactional-outbox).
Consumers should also tolerate duplicate delivery.

### 4. Event Store (Optional)
Persistent storage for events (used in Event Sourcing).

```csharp
public class EventStore
{
    private readonly DbContext _context;
    
    public async Task SaveEventAsync(DomainEvent @event)
    {
        var eventRecord = new EventRecord
        {
            Id = Guid.NewGuid(),
            EventType = @event.GetType().Name,
            AggregateId = @event.AggregateId,
            Data = JsonSerializer.Serialize(@event),
            Timestamp = DateTime.UtcNow
        };
        
        await _context.Events.AddAsync(eventRecord);
        await _context.SaveChangesAsync();
    }
    
    public async Task<IEnumerable<DomainEvent>> GetEventsAsync(Guid aggregateId)
    {
        var records = await _context.Events
            .Where(e => e.AggregateId == aggregateId)
            .OrderBy(e => e.Timestamp)
            .ToListAsync();
        
        return records.Select(r => DeserializeEvent(r.EventType, r.Data));
    }
}
```

---

## 🔧 Implementation Patterns

### 1. Event Notification
Simple events notifying that something happened.

```csharp
// Event
public class ProductPriceChangedEvent
{
    public Guid ProductId { get; set; }
    public decimal OldPrice { get; set; }
    public decimal NewPrice { get; set; }
    public DateTime ChangedAt { get; set; }
}

// Publisher
public class ProductService
{
    public async Task UpdatePriceAsync(Guid productId, decimal newPrice)
    {
        var product = await _repository.GetByIdAsync(productId);
        var oldPrice = product.Price;
        
        product.Price = newPrice;
        await _repository.UpdateAsync(product);
        
        await _eventBus.PublishAsync(new ProductPriceChangedEvent
        {
            ProductId = productId,
            OldPrice = oldPrice,
            NewPrice = newPrice,
            ChangedAt = DateTime.UtcNow
        });
    }
}

// Subscriber
public class PriceHistoryHandler : IEventHandler<ProductPriceChangedEvent>
{
    public async Task HandleAsync(ProductPriceChangedEvent @event)
    {
        await _priceHistoryRepository.AddAsync(new PriceHistory
        {
            ProductId = @event.ProductId,
            OldPrice = @event.OldPrice,
            NewPrice = @event.NewPrice,
            ChangedAt = @event.ChangedAt
        });
    }
}
```

### 2. Event-Carried State Transfer
Events carry full state, reducing need for querying.

```csharp
public class CustomerUpdatedEvent
{
    public Guid CustomerId { get; set; }
    public string FullName { get; set; }
    public string Email { get; set; }
    public Address Address { get; set; }
    public CustomerTier Tier { get; set; }
    // Carry complete customer state
}

public class CustomerCacheHandler : IEventHandler<CustomerUpdatedEvent>
{
    private readonly IDistributedCache _cache;
    
    public async Task HandleAsync(CustomerUpdatedEvent @event)
    {
        // Update local cache with full customer data
        await _cache.SetStringAsync(
            $"customer:{@event.CustomerId}",
            JsonSerializer.Serialize(@event),
            new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1) }
        );
    }
}
```

### 3. Event Sourcing
Events are the source of truth for application state.

```csharp
// Events
public abstract class OrderEvent
{
    public Guid OrderId { get; set; }
    public DateTime Timestamp { get; set; }
}

public class OrderPlacedEvent : OrderEvent
{
    public Guid CustomerId { get; set; }
    public List<OrderItem> Items { get; set; }
}

public class OrderShippedEvent : OrderEvent
{
    public string TrackingNumber { get; set; }
}

public class OrderCancelledEvent : OrderEvent
{
    public string Reason { get; set; }
}

// Aggregate
public class Order
{
    public Guid Id { get; private set; }
    public OrderStatus Status { get; private set; }
    private readonly List<OrderEvent> _events = new();
    
    public void PlaceOrder(Guid customerId, List<OrderItem> items)
    {
        var @event = new OrderPlacedEvent
        {
            OrderId = Id,
            CustomerId = customerId,
            Items = items,
            Timestamp = DateTime.UtcNow
        };
        
        Apply(@event);
        _events.Add(@event);
    }
    
    public void Ship(string trackingNumber)
    {
        if (Status != OrderStatus.Confirmed)
            throw new InvalidOperationException("Cannot ship unconfirmed order");
        
        var @event = new OrderShippedEvent
        {
            OrderId = Id,
            TrackingNumber = trackingNumber,
            Timestamp = DateTime.UtcNow
        };
        
        Apply(@event);
        _events.Add(@event);
    }
    
    private void Apply(OrderPlacedEvent @event)
    {
        Id = @event.OrderId;
        Status = OrderStatus.Placed;
    }
    
    private void Apply(OrderShippedEvent @event)
    {
        Status = OrderStatus.Shipped;
    }
    
    public IEnumerable<OrderEvent> GetUncommittedEvents() => _events;
}
```

---

## 🌩️ Azure Implementation

### Azure Event Hub (High Throughput)

```csharp
public class EventHubPublisher
{
    private readonly EventHubProducerClient _producerClient;
    
    public EventHubPublisher(string connectionString, string eventHubName)
    {
        _producerClient = new EventHubProducerClient(connectionString, eventHubName);
    }
    
    public async Task PublishAsync<T>(T @event)
    {
        using var eventBatch = await _producerClient.CreateBatchAsync();
        
        var eventData = new EventData(JsonSerializer.SerializeToUtf8Bytes(@event));
        
        if (!eventBatch.TryAdd(eventData))
        {
            throw new Exception("Event is too large for the batch");
        }
        
        await _producerClient.SendAsync(eventBatch);
    }
}

public class EventHubConsumer
{
    public async Task StartProcessingAsync(string connectionString, string eventHubName, string consumerGroup)
    {
        var storageClient = new BlobContainerClient(storageConnectionString, "checkpoints");
        
        var processor = new EventProcessorClient(
            storageClient,
            consumerGroup,
            connectionString,
            eventHubName);
        
        processor.ProcessEventAsync += async args =>
        {
            var eventData = args.Data.EventBody.ToObjectFromJson<OrderCreatedEvent>();
            await HandleEventAsync(eventData);
            await args.UpdateCheckpointAsync();
        };
        
        processor.ProcessErrorAsync += args =>
        {
            Console.WriteLine($"Error: {args.Exception.Message}");
            return Task.CompletedTask;
        };
        
        await processor.StartProcessingAsync();
    }
}
```

### Azure Service Bus (Message Ordering)

```csharp
public class ServiceBusEventPublisher
{
    private readonly ServiceBusClient _client;
    
    public async Task PublishAsync<T>(T @event, string sessionId = null)
    {
        var topicName = typeof(T).Name;
        var sender = _client.CreateSender(topicName);
        
        var message = new ServiceBusMessage(JsonSerializer.Serialize(@event))
        {
            ContentType = "application/json",
            SessionId = sessionId  // For message ordering
        };
        
        await sender.SendMessageAsync(message);
    }
}
```

### Azure Event Grid (Event Routing)

```csharp
public class EventGridPublisher
{
    private readonly EventGridPublisherClient _client;
    
    public async Task PublishAsync<T>(T @event)
    {
        var cloudEvent = new CloudEvent(
            source: "/myapp/orders",
            type: typeof(T).Name,
            jsonSerializableData: @event);
        
        await _client.SendEventAsync(cloudEvent);
    }
}
```

---

## ✅ When to Use Event-Driven Architecture

✅ **Use when:**
- System needs high scalability and loose coupling
- Real-time processing or reactions required
- Multiple systems need to respond to same event
- Asynchronous processing is acceptable
- Building microservices or distributed systems
- Need audit trails or event history

❌ **Avoid when:**
- Strong consistency required (consider eventual consistency carefully)
- Synchronous responses needed immediately
- Simple CRUD applications
- Debugging complexity is a concern
- Team lacks experience with distributed systems

---

## 👍 Advantages

✅ **Loose Coupling** - Components don't depend on each other  
✅ **Scalability** - Easy to scale producers/consumers independently  
✅ **Flexibility** - Add new consumers without changing producers  
✅ **Real-Time Processing** - React to changes immediately  
✅ **Resilience** - Failures isolated to individual consumers  
✅ **Audit Trail** - Natural event log for compliance

---

## 👎 Disadvantages

❌ **Complexity** - Harder to understand flow than request/response  
❌ **Eventual Consistency** - Data may be temporarily inconsistent  
❌ **Debugging Difficulty** - Harder to trace event chains  
❌ **Guaranteed Delivery** - Need infrastructure for at-least-once/exactly-once  
❌ **Ordering** - Event order may not be guaranteed  
❌ **Error Handling** - Need dead-letter queues, retry policies

---

## 🎯 Common Use Cases

- **E-commerce** - Order processing, inventory updates, payment confirmation
- **IoT** - Sensor data ingestion, telemetry processing
- **Real-time Analytics** - Click streams, user behavior tracking
- **Notifications** - Email, SMS, push notifications
- **Workflow Orchestration** - Multi-step business processes
- **Data Synchronization** - Keeping distributed caches/databases in sync

---

## 🔄 EDA vs Other Styles

| Aspect | Event-Driven | Request-Response | Message Queue |
|--------|--------------|------------------|---------------|
| **Communication** | Asynchronous, broadcast | Synchronous, direct | Asynchronous, point-to-point |
| **Coupling** | Very loose | Tight | Loose |
| **Consumers** | Multiple (pub/sub) | One (caller) | One (queue consumer) |
| **Use Case** | Real-time reactions | Immediate response needed | Background jobs |

---

## 🏗️ Best Practices

✅ **Design idempotent consumers** - Handle duplicate events gracefully  
✅ **Use event versioning** - Plan for schema evolution  
✅ **Implement dead-letter queues** - Handle failed messages  
✅ **Monitor event lag** - Ensure consumers keep up  
✅ **Use correlation IDs** - Trace events across services  
✅ **Define event schemas** - Use contracts (JSON Schema, Protobuf)  
✅ **Implement circuit breakers** - Protect against cascading failures  
✅ **Use event store for audit** - Keep event history for compliance

---

## 📚 Related Patterns

- **CQRS** - Often paired with EDA for command/query separation
- **Event Sourcing** - Events as source of truth
- **Saga Pattern** - Coordinate distributed transactions with events
- **Pub/Sub Pattern** - Core communication mechanism in EDA
- **Outbox Pattern** - Ensure reliable event publishing