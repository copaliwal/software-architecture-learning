# Event Sourcing Pattern

## Overview

**Category:** Architectural Pattern  
**Intent:** Store the state of an application as a sequence of events rather than just the current state. Every change is captured as an immutable event.

---

## Problem

Traditional systems store only the **current state** of data:
- Lost history - can't see how data evolved
- Limited audit trail
- Difficult to replay or debug past states
- Can't reconstruct state at any point in time
- Temporal queries are complex

---

## Solution

Instead of storing current state, store **all changes as events**:
- Events are **immutable** facts (something that happened)
- Current state is derived by replaying events
- Complete audit trail automatically maintained
- Can rebuild state at any point in time

---

## Architecture

```
Command → Event → Event Store
                     ↓
                  Event Stream
                     ↓
            Read Model (Projection)
```

**Key Components:**
1. **Events** - Immutable records of state changes
2. **Event Store** - Append-only database for events
3. **Aggregates** - Domain entities that produce events
4. **Projections** - Build read models from events

---

## Basic C# Implementation

### Events

```csharp
// Base event interface
public interface IEvent
{
    Guid EventId { get; }
    DateTime Timestamp { get; }
}

// Concrete events
public class OrderPlacedEvent : IEvent
{
    public Guid EventId { get; set; } = Guid.NewGuid();
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
    
    public int OrderId { get; set; }
    public int CustomerId { get; set; }
    public List<OrderItem> Items { get; set; }
    public decimal Total { get; set; }
}

public class OrderShippedEvent : IEvent
{
    public Guid EventId { get; set; } = Guid.NewGuid();
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
    
    public int OrderId { get; set; }
    public string TrackingNumber { get; set; }
}

public class OrderCancelledEvent : IEvent
{
    public Guid EventId { get; set; } = Guid.NewGuid();
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
    
    public int OrderId { get; set; }
    public string Reason { get; set; }
}
```

### Aggregate (Domain Entity)

```csharp
public class OrderAggregate
{
    private readonly List<IEvent> _changes = new();
    
    public int Id { get; private set; }
    public int CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    public decimal Total { get; private set; }
    
    // For rehydration from event store
    public void LoadFromHistory(IEnumerable<IEvent> events)
    {
        foreach (var @event in events)
        {
            ApplyEvent(@event, isNew: false);
        }
    }
    
    // Commands - produce events
    public void PlaceOrder(int customerId, List<OrderItem> items)
    {
        var total = items.Sum(i => i.Price * i.Quantity);
        
        var @event = new OrderPlacedEvent
        {
            OrderId = Id,
            CustomerId = customerId,
            Items = items,
            Total = total
        };
        
        ApplyEvent(@event, isNew: true);
    }
    
    public void ShipOrder(string trackingNumber)
    {
        if (Status != OrderStatus.Pending)
            throw new InvalidOperationException("Can only ship pending orders");
        
        var @event = new OrderShippedEvent
        {
            OrderId = Id,
            TrackingNumber = trackingNumber
        };
        
        ApplyEvent(@event, isNew: true);
    }
    
    public void CancelOrder(string reason)
    {
        if (Status == OrderStatus.Cancelled)
            throw new InvalidOperationException("Order already cancelled");
        
        var @event = new OrderCancelledEvent
        {
            OrderId = Id,
            Reason = reason
        };
        
        ApplyEvent(@event, isNew: true);
    }
    
    // Apply events to update state
    private void ApplyEvent(IEvent @event, bool isNew)
    {
        switch (@event)
        {
            case OrderPlacedEvent e:
                Id = e.OrderId;
                CustomerId = e.CustomerId;
                Total = e.Total;
                Status = OrderStatus.Pending;
                break;
                
            case OrderShippedEvent e:
                Status = OrderStatus.Shipped;
                break;
                
            case OrderCancelledEvent e:
                Status = OrderStatus.Cancelled;
                break;
        }
        
        if (isNew)
            _changes.Add(@event);
    }
    
    // Get uncommitted events
    public IEnumerable<IEvent> GetUncommittedChanges()
    {
        return _changes.AsReadOnly();
    }
    
    public void MarkChangesAsCommitted()
    {
        _changes.Clear();
    }
}

public enum OrderStatus
{
    Pending,
    Shipped,
    Delivered,
    Cancelled
}
```

### Event Store

```csharp
public interface IEventStore
{
    Task SaveEventsAsync(string streamId, IEnumerable<IEvent> events, int expectedVersion);
    Task<IEnumerable<IEvent>> GetEventsAsync(string streamId);
}

public class InMemoryEventStore : IEventStore
{
    private readonly Dictionary<string, List<IEvent>> _eventStreams = new();
    
    public Task SaveEventsAsync(string streamId, IEnumerable<IEvent> events, int expectedVersion)
    {
        if (!_eventStreams.ContainsKey(streamId))
            _eventStreams[streamId] = new List<IEvent>();
        
        var stream = _eventStreams[streamId];
        
        // Optimistic concurrency check
        if (stream.Count != expectedVersion)
            throw new ConcurrencyException($"Expected version {expectedVersion}, but found {stream.Count}");
        
        stream.AddRange(events);
        
        return Task.CompletedTask;
    }
    
    public Task<IEnumerable<IEvent>> GetEventsAsync(string streamId)
    {
        if (!_eventStreams.ContainsKey(streamId))
            return Task.FromResult(Enumerable.Empty<IEvent>());
        
        return Task.FromResult<IEnumerable<IEvent>>(_eventStreams[streamId]);
    }
}

// SQL Server implementation
public class SqlEventStore : IEventStore
{
    private readonly string _connectionString;
    
    public async Task SaveEventsAsync(string streamId, IEnumerable<IEvent> events, int expectedVersion)
    {
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        using var transaction = connection.BeginTransaction();
        
        try
        {
            foreach (var @event in events)
            {
                var command = new SqlCommand(
                    @"INSERT INTO EventStore (StreamId, EventType, EventData, Timestamp, Version)
                      VALUES (@StreamId, @EventType, @EventData, @Timestamp, @Version)",
                    connection, transaction);
                
                command.Parameters.AddWithValue("@StreamId", streamId);
                command.Parameters.AddWithValue("@EventType", @event.GetType().Name);
                command.Parameters.AddWithValue("@EventData", JsonSerializer.Serialize(@event));
                command.Parameters.AddWithValue("@Timestamp", @event.Timestamp);
                command.Parameters.AddWithValue("@Version", expectedVersion++);
                
                await command.ExecuteNonQueryAsync();
            }
            
            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
    
    public async Task<IEnumerable<IEvent>> GetEventsAsync(string streamId)
    {
        var events = new List<IEvent>();
        
        using var connection = new SqlConnection(_connectionString);
        await connection.OpenAsync();
        
        var command = new SqlCommand(
            "SELECT EventType, EventData FROM EventStore WHERE StreamId = @StreamId ORDER BY Version",
            connection);
        command.Parameters.AddWithValue("@StreamId", streamId);
        
        using var reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            var eventType = reader.GetString(0);
            var eventData = reader.GetString(1);
            
            var type = Type.GetType($"YourNamespace.{eventType}");
            var @event = JsonSerializer.Deserialize(eventData, type) as IEvent;
            
            events.Add(@event);
        }
        
        return events;
    }
}
```

### Repository

```csharp
public interface IOrderRepository
{
    Task<OrderAggregate> GetByIdAsync(int orderId);
    Task SaveAsync(OrderAggregate order);
}

public class EventSourcedOrderRepository : IOrderRepository
{
    private readonly IEventStore _eventStore;
    
    public EventSourcedOrderRepository(IEventStore eventStore)
    {
        _eventStore = eventStore;
    }
    
    public async Task<OrderAggregate> GetByIdAsync(int orderId)
    {
        var streamId = $"order-{orderId}";
        var events = await _eventStore.GetEventsAsync(streamId);
        
        var order = new OrderAggregate();
        order.LoadFromHistory(events);
        
        return order;
    }
    
    public async Task SaveAsync(OrderAggregate order)
    {
        var streamId = $"order-{order.Id}";
        var events = order.GetUncommittedChanges();
        
        if (events.Any())
        {
            await _eventStore.SaveEventsAsync(streamId, events, expectedVersion: 0);
            order.MarkChangesAsCommitted();
        }
    }
}
```

### Usage

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public async Task PlaceOrderAsync(PlaceOrderCommand command)
    {
        var order = new OrderAggregate();
        order.PlaceOrder(command.CustomerId, command.Items);
        
        await _repository.SaveAsync(order);
        
        // Events are now stored in event store
    }
    
    public async Task ShipOrderAsync(int orderId, string trackingNumber)
    {
        var order = await _repository.GetByIdAsync(orderId);
        order.ShipOrder(trackingNumber);
        
        await _repository.SaveAsync(order);
    }
    
    public async Task CancelOrderAsync(int orderId, string reason)
    {
        var order = await _repository.GetByIdAsync(orderId);
        order.CancelOrder(reason);
        
        await _repository.SaveAsync(order);
    }
}
```

---

## Projections (Read Models)

Build denormalized read models from events:

```csharp
public class OrderReadModel
{
    public int OrderId { get; set; }
    public string CustomerName { get; set; }
    public decimal Total { get; set; }
    public string Status { get; set; }
    public DateTime OrderDate { get; set; }
    public DateTime? ShippedDate { get; set; }
}

public class OrderProjection
{
    private readonly ReadDbContext _readDb;
    
    public async Task HandleAsync(OrderPlacedEvent @event)
    {
        var readModel = new OrderReadModel
        {
            OrderId = @event.OrderId,
            Total = @event.Total,
            Status = "Pending",
            OrderDate = @event.Timestamp
        };
        
        _readDb.OrderReadModels.Add(readModel);
        await _readDb.SaveChangesAsync();
    }
    
    public async Task HandleAsync(OrderShippedEvent @event)
    {
        var readModel = await _readDb.OrderReadModels
            .FirstOrDefaultAsync(o => o.OrderId == @event.OrderId);
        
        if (readModel != null)
        {
            readModel.Status = "Shipped";
            readModel.ShippedDate = @event.Timestamp;
            await _readDb.SaveChangesAsync();
        }
    }
    
    public async Task HandleAsync(OrderCancelledEvent @event)
    {
        var readModel = await _readDb.OrderReadModels
            .FirstOrDefaultAsync(o => o.OrderId == @event.OrderId);
        
        if (readModel != null)
        {
            readModel.Status = "Cancelled";
            await _readDb.SaveChangesAsync();
        }
    }
}
```

---

## Snapshots (Performance Optimization)

For long event streams, use snapshots to avoid replaying thousands of events:

```csharp
public class OrderSnapshot
{
    public int OrderId { get; set; }
    public int Version { get; set; }
    public OrderStatus Status { get; set; }
    public decimal Total { get; set; }
    public DateTime SnapshotDate { get; set; }
}

public class SnapshotStore
{
    public async Task<OrderSnapshot> GetSnapshotAsync(int orderId)
    {
        // Load latest snapshot from database
    }
    
    public async Task SaveSnapshotAsync(OrderSnapshot snapshot)
    {
        // Save snapshot to database
    }
}

// Repository with snapshots
public async Task<OrderAggregate> GetByIdAsync(int orderId)
{
    var snapshot = await _snapshotStore.GetSnapshotAsync(orderId);
    var order = new OrderAggregate();
    
    if (snapshot != null)
    {
        order.LoadFromSnapshot(snapshot);
        // Only load events after the snapshot
        var events = await _eventStore.GetEventsAsync($"order-{orderId}", snapshot.Version);
        order.LoadFromHistory(events);
    }
    else
    {
        // Load all events
        var events = await _eventStore.GetEventsAsync($"order-{orderId}");
        order.LoadFromHistory(events);
    }
    
    return order;
}
```

---

## Using EventStore DB

**EventStore** is a purpose-built database for event sourcing.

```csharp
// Install package
// dotnet add package EventStore.Client.Grpc.Streams

public class EventStoreDbEventStore : IEventStore
{
    private readonly EventStoreClient _client;
    
    public EventStoreDbEventStore(EventStoreClient client)
    {
        _client = client;
    }
    
    public async Task SaveEventsAsync(string streamId, IEnumerable<IEvent> events, int expectedVersion)
    {
        var eventData = events.Select(e => new EventData(
            Uuid.NewUuid(),
            e.GetType().Name,
            JsonSerializer.SerializeToUtf8Bytes(e)
        ));
        
        await _client.AppendToStreamAsync(
            streamId,
            StreamRevision.FromInt64(expectedVersion),
            eventData
        );
    }
    
    public async Task<IEnumerable<IEvent>> GetEventsAsync(string streamId)
    {
        var result = _client.ReadStreamAsync(
            Direction.Forwards,
            streamId,
            StreamPosition.Start
        );
        
        var events = new List<IEvent>();
        
        await foreach (var resolvedEvent in result)
        {
            var eventType = Type.GetType($"YourNamespace.{resolvedEvent.Event.EventType}");
            var @event = JsonSerializer.Deserialize(resolvedEvent.Event.Data.Span, eventType) as IEvent;
            events.Add(@event);
        }
        
        return events;
    }
}
```

---

## When to Use Event Sourcing

✅ **Use Event Sourcing when:**
- Complete audit trail is required (financial, healthcare, legal)
- Need to replay events or rebuild state
- Temporal queries (state at any point in time)
- Event-driven architecture
- Domain events are first-class citizens
- Debugging production issues by replaying events

❌ **Avoid when:**
- Simple CRUD operations
- No need for audit trail
- Can't tolerate eventual consistency
- Team lacks experience with ES
- Read operations vastly outnumber writes

---

## Advantages

✅ **Complete Audit Trail** - Every change is recorded  
✅ **Temporal Queries** - State at any point in time  
✅ **Event Replay** - Rebuild state, fix bugs retroactively  
✅ **Event-Driven Architecture** - Natural fit for microservices  
✅ **Debugging** - Replay production issues  
✅ **Business Insights** - Analyze event patterns

---

## Disadvantages

❌ **Complexity** - Steeper learning curve  
❌ **Eventual Consistency** - Read models lag behind  
❌ **Storage** - Events accumulate over time  
❌ **Schema Evolution** - Handling event versioning  
❌ **Querying** - Need projections for complex queries

---

## Event Versioning

```csharp
// V1
public class OrderPlacedEventV1 : IEvent
{
    public int OrderId { get; set; }
    public decimal Total { get; set; }
}

// V2 - Added CustomerId
public class OrderPlacedEventV2 : IEvent
{
    public int OrderId { get; set; }
    public int CustomerId { get; set; }
    public decimal Total { get; set; }
}

// Upcasting - Convert old events to new format
public class EventUpcast er
{
    public IEvent Upcast(IEvent @event)
    {
        return @event switch
        {
            OrderPlacedEventV1 v1 => new OrderPlacedEventV2
            {
                OrderId = v1.OrderId,
                CustomerId = 0, // Default value
                Total = v1.Total
            },
            _ => @event
        };
    }
}
```

---

## Related Patterns

- **CQRS** - Often used together
- **Saga Pattern** - Distributed transactions
- **Event-Driven Architecture** - Events as first-class citizens
- **Domain-Driven Design** - Aggregates produce events

---

## Common Use Cases

- **Banking** - Transaction history, audit compliance
- **E-commerce** - Order lifecycle tracking
- **Healthcare** - Patient record changes
- **Supply Chain** - Shipment tracking
- **Gaming** - Player actions, game state
- **Collaboration Tools** - Document change history (Git-like)

---

## Tools & Libraries

- **EventStore DB** - Purpose-built event store database
- **Marten** - .NET document database with event sourcing
- **NEventStore** - .NET event sourcing library
- **Equinox** - Event sourcing for .NET
- **SQL Server** - Can be used as event store
- **Azure Cosmos DB** - Change feed for event streaming
