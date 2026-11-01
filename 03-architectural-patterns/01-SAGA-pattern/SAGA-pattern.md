# SAGA Pattern

A practical, system-level way to manage distributed transactions across multiple services without a global transaction manager. Sagas achieve eventual consistency by sequencing local transactions and invoking compensating actions on failure.

* * *

## Core concept and motivations

- **Problem:**
    
    Multiple services participate in a business workflow (order, payment, inventory, shipping). A single ACID transaction across services/databases is infeasible or harms scalability.
    
- **Solution:**
    
    - Local transactions per service, each committing independently.
    - On failure, previously executed steps are undone via **compensating transactions**.
    - Consistency is **eventual**, not immediate.
- **Scope:**
    
    **Subsystem or service-group level**, typically in microservices or event-driven architectures.
    
- **Outcomes:**
    
    - **Resilience:** Avoids global locks and coordinator bottlenecks.
    - **Scalability:** Aligns with asynchronous, distributed systems.
    - **Traceability:** Explicit business steps and compensations.

* * *

## Saga types

### Orchestration

- **Idea:**
    
    A central **orchestrator** coordinates the workflow, dispatching commands and listening for replies.
    
- **Pros:**
    
    - **Clarity:** Single place for process logic.
    - **Debuggability:** Easier tracing and monitoring.
    - **Determinism:** Explicit transitions and error handling.
- **Cons:**
    
    - **Single point of failure:** Must ensure HA and statelessness.
    - **Coupling:** Orchestrator holds knowledge of participants.
    - **Evolution cost:** Changes funnel through orchestrator.
- **Use when:**
    
    **Complex, branching workflows**; strong need for visibility; strict governance.
    
- **Avoid when:**
    
    Team prefers decentralized autonomy; workflows are simple and event-native.
    

### Choreography

- **Idea:**
    
    No central brain. Services publish and subscribe to **domain events** and react based on local policies.
    
- **Pros:**
    
    - **Decoupling:** No central dependency.
    - **Scalability:** Natural fit for event-driven systems.
    - **Autonomy:** Teams own their service logic.
- **Cons:**
    
    - **Observability:** Harder to trace end-to-end.
    - **Emergent behavior:** Risk of cycles or unintended triggers.
    - **Consistency management:** More discipline needed for contracts and idempotency.
- **Use when:**
    
    **Simple, linear workflows**; high service autonomy; mature event backbone.
    
- **Avoid when:**
    
    Complex branching/compensation rules; strong audit/traceability requirements.
    

* * *

## Design elements

- **Local transaction:**
    
    Each step performs a durable change in its own store.
    
- **Compensating transaction:**
    
    Inverse action to undo effects (e.g., un-reserve stock, refund payment).
    
- **State machine:**
    
    **Explicit states** (Pending, Reserved, Paid, Shipped, Cancelled) and **transitions** (success/failure).
    
- **Idempotency:**
    
    Ensure commands/events can be safely retried without duplicating effects.
    
- **Message delivery semantics:**
    
    Design for **at-least-once** delivery; implement deduplication.
    
- **Timeouts and retries:**
    
    Configure **retry policies** and **timeouts** per step to avoid stuck sagas.
    
- **Isolation levels:**
    
    Avoid long-held locks; prefer optimistic concurrency.
    
- **Compensation strategy:**
    
    Choose between full inverse operations or logical compensation (e.g., issue credit note).
    

* * *

## Typical workflow example

- **Order placement saga (orchestrated):**
    1.  **Order service:** Create order (Pending).
    2.  **Inventory:** Reserve items.
    3.  **Payment:** Charge card.
    4.  **Shipping:** Schedule shipment.
    5.  **Order service:** Mark as Completed.
- **Failures and compensations:**
    - If payment fails: **Inventory compensation** (release stock), **Order → Cancelled**.
    - If shipping fails: **Payment compensation** (refund), **Inventory compensation** (release), **Order → Cancelled**.

* * *

## Implementation options in .NET/Azure

### Messaging backbone

- **Azure Service Bus:**
    
    **Queues** for commands, **topics** for events; dead-letter queues for failures.
    
- **Azure Event Hubs:**
    
    High-throughput event ingestion for choreography.
    
- **Azure Storage Queues:**
    
    Simpler, cost-effective queueing for small workloads.
    

### Orchestrator choices

- **Durable Functions (Azure Functions):**
    - **Orchestrator function** coordinates activity functions.
    - Built-in **retries**, **timeouts**, **state persistence**.
    - Ideal for serverless orchestration.
- **Workflow engines:**
    - **Dapr Workflows**, **MassTransit** saga state machines, **NServiceBus** sagas.
- **Custom orchestrator (ASP.NET Core):**
    - State machine with persistent saga state (SQL/Cosmos DB).
    - Background services triggering transitions.

### Sample orchestrated saga with Durable Functions (C#)

```csharp
[FunctionName("OrderSaga_Orchestrator")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext ctx)
{
    var order = ctx.GetInput<OrderRequest>();
    try
    {
        await ctx.CallActivityAsync("ReserveInventory", order);
        await ctx.CallActivityAsync("ChargePayment", order);
        await ctx.CallActivityAsync("ScheduleShipping", order);

        await ctx.CallActivityAsync("CompleteOrder", order);
    }
    catch (FunctionFailedException ex)
    {
        // Compensation chain (reverse order)
        await ctx.CallActivityAsync("CancelShipping", order);
        await ctx.CallActivityAsync("RefundPayment", order);
        await ctx.CallActivityAsync("ReleaseInventory", order);

        await ctx.CallActivityAsync("CancelOrder", new CancelRequest(order.Id, ex.Message));
    }
}

```

### Transactional outbox for reliable event publishing

- **Pattern:** Write domain change and outgoing event in the same DB transaction; a **relay** publishes events to the broker.

```csharp
// Within service's transaction
await db.Orders.AddAsync(order);
await db.Outbox.AddAsync(new OutboxMessage {
    Id = Guid.NewGuid(),
    Type = "OrderPlaced",
    Payload = JsonSerializer.Serialize(order),
    CreatedAt = DateTime.UtcNow
});
await db.SaveChangesAsync();

// Background relay
var messages = await db.Outbox.Where(m => !m.Published).ToListAsync();
foreach (var msg in messages)
{
    await bus.PublishAsync(msg.Type, msg.Payload);
    msg.Published = true;
}
await db.SaveChangesAsync();

```

* * *

## Choreography example (events)

- **Sequence:**
    - **OrderPlaced** → Inventory reserves → emits **InventoryReserved**.
    - **InventoryReserved** → Payment charges → emits **PaymentSucceeded**.
    - **PaymentSucceeded** → Shipping schedules → emits **ShipmentScheduled**.
    - **Any failure event** triggers compensations by upstream services.
- **Event contract design:**
    - **Schema versioning** and **compatibility**.
    - Include **correlation IDs** and **causation IDs**.
    - Use **immutable payloads**.

* * *

## Reliability and resilience

- **Idempotency keys:**
    - **Label:** Use business IDs and dedup tables/caches.
    - **Effect:** Safely handle retries and at-least-once delivery.
- **Circuit breakers and retries:**
    - **Label:** Apply Polly policies in HTTP/AMQP clients.
    - **Effect:** Avoid cascading failures and transient issues.
- **Timeouts and escalation:**
    - **Label:** Detect stuck sagas, mark as **TimedOut**, trigger compensation.
- **Poison message handling:**
    - **Label:** DLQ routing; alert and manual review flows.

* * *

## State, storage, and concurrency

- **Saga state store:**
    - **Choices:** SQL, Cosmos DB, Redis, or orchestrator-managed storage.
    - **Contents:** Current state, step results, timestamps, correlation IDs.
- **Concurrency control:**
    - **Label:** Optimistic concurrency with ETags/rowversion.
    - **Effect:** Prevent double progression or conflicting updates.
- **Event sourcing integration:**
    - **Label:** Store events per saga instance; rebuild timeline for audits.

* * *

## Observability and operations

- **Tracing:**
    - **Label:** Propagate **correlation IDs** across services.
    - **Effect:** End-to-end visibility in Application Insights/OpenTelemetry.
- **Logging:**
    - **Label:** Structured logs with sagaId, step, state, outcome.
    - **Effect:** Faster root cause analysis.
- **Metrics:**
    - **Label:** Success rate, compensation rate, average duration, timeout count.
    - **Effect:** Health and SLA tracking.
- **Dashboards:**
    - **Label:** Per-saga state distribution; DLQ depth; retry counts.

* * *

## Testing strategy

- **Contract tests:**
    - **Label:** Validate event schemas and command interfaces.
    - **Effect:** Prevent breaking changes between services.
- **Workflow tests:**
    - **Label:** Simulate success/failure paths end-to-end.
    - **Effect:** Verify compensations and idempotency.
- **Chaos testing:**
    - **Label:** Inject timeouts, message duplication, service outages.
    - **Effect:** Confirm resilience.
- **Performance tests:**
    - **Label:** Measure latency and throughput under load; tune batch sizes and retries.

* * *

## Governance and documentation

- **Architecture Decision Records (ADRs):**
    - **Label:** Record why/when saga was chosen; outline compensation rules.
- **C4 diagrams:**
    - **Label:** Context, container, and component diagrams with message flows.
- **Runbooks:**
    - **Label:** Steps for manual compensation, DLQ handling, replay procedures.
- **Versioning:**
    - **Label:** Event schema evolution policy; backward compatibility guarantees.

* * *

## When to use

- **Distributed workflows** across multiple services/databases.
- **Eventual consistency acceptable**; business can tolerate short delays.
- **High scalability and resilience** prioritized over strict atomicity.
- **Clear compensations exist** for each step.

## When to avoid

- **Strict immediate consistency** mandatory (e.g., double-entry ledger posting).
- **Compensations are impossible** or legally problematic.
- **Operational maturity is low** (lack of monitoring, tracing, DLQ handling).
- **Simple monolith** with single DB and ACID transactions.

* * *

## Common pitfalls and anti-patterns

- **Missing idempotency:**
    
    Duplicate processing on retries causing over-charging or over-reserving.
    
- **Implicit compensations:**
    
    Not implementing true inverse operations; relying on manual fixes.
    
- **Chatty orchestrators:**
    
    Tight coupling and performance bottlenecks; prefer coarse-grained commands.
    
- **Event storms:**
    
    Too granular events causing noise and complexity; consolidate events.
    
- **Hidden side-effects:**
    
    Services changing state outside the saga contract; break determinism.
    

* * *

## Decision guide

- **If you need audit/history:** Consider **Event Sourcing**; use Saga for cross-service process coordination.
- **If you need strong consistency with few participants:** Consider **2PC** or **TCC** with reservations.
- **If you need reliable event publishing:** Add **Transactional Outbox** to your Saga/messaging.
- **If workflow is complex:** Prefer **Orchestration**. If simple and decoupled, prefer **Choreography**.

* * *

## Minimal templates

### Orchestrated saga step contract

- **Command:**
    - **Label:** ReserveInventoryCommand
    - **Fields:** orderId, items\[\], correlationId
- **Reply:**
    - **Label:** InventoryReservedEvent | InventoryReservationFailedEvent
    - **Fields:** orderId, reservedItems\[\], reason?, correlationId, causationId

### Compensation policy snippet

```csharp
public record CompensationPolicy(string Step, Func<Task> Compensate, Func<bool> ShouldCompensate);

var policy = new List<CompensationPolicy> {
    new("Shipping", () => CancelShipping(orderId), () => shippingScheduled),
    new("Payment", () => RefundPayment(orderId), () => paymentCharged),
    new("Inventory", () => ReleaseInventory(orderId), () => inventoryReserved)
};
```

* * *