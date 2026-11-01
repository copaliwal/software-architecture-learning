# 01 Foundations

## Hierarchy of Software Architecture Concepts

Think of architecture knowledge as a pyramid - from high-level to low-level:

| Level | Concept | Description | Example |
| --- | --- | --- | --- |
| **Top** | **Software Architecture** | The overall structure of a system: how components interact, constraints, and qualities (scalability, security, performance). | A microservices-based e-commerce platform deployed on Azure. |
| ↓ | **Architectural Styles** | Broad categories of system organization. Defines the "shape" of the system. | Layered architecture, microservices, event-driven, client-server. |
| ↓ | **Architectural Patterns** | Proven solutions to recurring architectural problems. More specific than styles. | CQRS, Saga, API Gateway, Event Sourcing. |
| ↓ | **Design Patterns** | Reusable solutions to common design problems at the code/component level. | Singleton, Factory, Observer, Repository. |
| **Bottom** | **Software Design Principles** | Guidelines for writing maintainable, flexible, and clean code. | SOLID, DRY, KISS, Separation of Concerns. |

## **Hierarchy summary:**

*   **Architecture** (big picture)
    *   **Style** (system shape)
        *   **Pattern** (solution templates)
            *   **Design Pattern** (code-level solutions)
                *   **Principles** (guidelines).

This is a teaching aid, not a strict taxonomy: styles, architectural patterns, and principles
overlap and operate at different scopes. For example, a modular monolith can use layered
organization, ports and adapters, and ordinary object-level design patterns at the same time.

---

## Software Architecture Basics

*   **Definition:** High‑level structure of a software system, defining components, interactions, and constraints.
*   **Goal:** Ensure scalability, maintainability, security, and performance.
*   **Example:** An e‑commerce platform with separate services for catalog, orders, and payments.

## Architecture vs. Design

Architecture concerns the system's important structures and decisions: boundaries, deployment,
data ownership, communication, and the quality attributes those choices enable. Design also
includes the more local decisions made inside those boundaries, such as object responsibilities
and algorithms. The boundary is contextual rather than absolute: a decision is architectural when
its consequences materially affect system qualities, teams, operations, or future change.

## Architectural Drivers and Constraints

Architecture starts with the forces that shape the solution, not with a preferred technology or
style.

| Driver | Questions to answer | Example |
|---|---|---|
| Business goals and stakeholders | What outcome matters, to whom, and by when? | Launch a new region without interrupting existing customers. |
| Functional requirements | What capabilities and workflows must the system provide? | Customers can place, cancel, and track orders. |
| Quality attributes | How well must those capabilities work, under what conditions? | 99.95% monthly availability; p95 order submission under 300 ms. |
| Constraints | What limits the viable options? | Data residency, an existing identity provider, team skills, budget, or a mandated platform. |

Make quality requirements measurable with scenarios (see [Quality Attribute Scenarios](#quality-attribute-scenarios)).
Distinguish a genuine constraint from a preference; preferences can be compared as options, while
constraints require a documented exception or a changed requirement.

---

## Architect Role vs Developer

*   **Developer:** Focuses on implementing features, writing code, and fixing bugs.
*   **Architect:** Focuses on system design, trade‑offs, technology choices, and aligning with business goals.
*   **Example:** Developer implements a payment API; architect decides whether payments are handled via microservices or integrated into a monolith.

---

## Quality Attributes

*   **Scalability:** Handle growth (horizontal scaling).
*   **Availability:** System uptime (99.9% SLA).
*   **Security:** Protect data (OAuth 2.0 / OIDC, Microsoft Entra ID — formerly Azure AD).
*   **Maintainability:** Easy to update.
*   **Performance:** Fast response times.
*   **Trade‑off Example:** High security may reduce performance due to encryption overhead.

---

## CAP Theorem

> ⚠️ **The "pick 2 of 3" version of CAP is wrong**, and stating it that way in an interview is a
> tell. CAP is a statement about what happens **during a network partition**, not a menu of three
> properties you shop from.

**Accurate statement:** *When a network partition occurs*, a distributed system must choose between
**consistency** and **availability**. When there is no partition — the normal case — you can have
both.

| Property | Precise meaning |
|----------|-----------------|
| **C — Consistency** | **Linearizability**: every read returns the most recent completed write, as if there were a single copy. ⚠️ This is *not* the "C" in ACID (which means "constraints hold"). |
| **A — Availability** | Every request to a **non-failed** node returns a non-error response — even if it may be stale. |
| **P — Partition Tolerance** | The system continues to operate when the network drops or delays messages between nodes. |

### Why "CA" is a category error

Partitions are not a design choice — they are a property of networks. You cannot opt out of them,
so **P is not optional** in any distributed system. That makes CAP a binary choice (CP or AP), not
a three-way one.

A single-node SQL Server is not "CA" — it is simply **not distributed**, so CAP has nothing to say
about it. (A single node also has an availability ceiling of one machine.)

### PACELC — the extension that matters more in practice

CAP only describes behaviour during a partition, which is rare. **PACELC** covers the common case:

> **if (P)** artition → choose **A**vailability or **C**onsistency;
> **E**lse → choose **L**atency or **C**onsistency.

The "else" branch is the one you live with every day: **stronger consistency costs latency**,
because more replicas must acknowledge a write before it's confirmed.

| System | Classification | Reading |
|---|---|---|
| Azure Cosmos DB (Strong) | PC/EC | Consistent under partition *and* accepts latency normally |
| Azure Cosmos DB (Session/Eventual) | PA/EL | Available under partition, low latency normally |
| Azure SQL (single region) | PC/EC | Prioritises consistency |
| Cassandra / DynamoDB | PA/EL | Available and fast, eventually consistent |

**Interview line:** *"CAP is the partition-time choice; PACELC is the everyday one. Most of my
consistency decisions are actually the 'else' branch — trading latency for consistency — because
partitions are rare and latency is constant."*

### Trade-offs

**In practice, partition tolerance is mandatory** in distributed systems (network failures will happen), so you must choose between:

#### CP (Consistency + Partition Tolerance)
- **Sacrifices:** Availability during network partitions
- **Use When:** Data accuracy is critical (financial transactions, inventory)
- **Examples:** MongoDB (with strong consistency), HBase, Redis (with synchronous replication)
- **Scenario:** Banking system - better to be unavailable than show incorrect balance

#### AP (Availability + Partition Tolerance)
- **Sacrifices:** Consistency (eventual consistency)
- **Use When:** System uptime is more important than immediate consistency
- **Examples:** Cassandra, DynamoDB, Cosmos DB (in eventual consistency mode)
- **Scenario:** Social media feed - okay if likes/comments are slightly delayed

#### CA (Consistency + Availability)
- **Reality:** Not achievable in true distributed systems with network partitions
- **Examples:** Single-node databases (not distributed)
- **Note:** If you don't need partition tolerance, you don't need a distributed system

### Practical Example

```csharp
// CP System - Prioritize Consistency
public class OrderService_CP
{
    public async Task<bool> PlaceOrder(Order order)
    {
        // Wait for all replicas to confirm before responding
        var confirmed = await _database.WriteWithQuorum(order);
        
        if (!confirmed)
        {
            // Sacrifice availability - reject the request
            throw new ServiceUnavailableException("System temporarily unavailable");
        }
        
        return true;
    }
}

// AP System - Prioritize Availability
public class SocialMediaService_AP
{
    public async Task<bool> PostComment(Comment comment, CancellationToken ct)
    {
        // Write locally AND record the replication intent in ONE transaction (outbox).
        // Availability comes from not waiting for remote replicas — NOT from firing
        // background work and hoping.
        await _database.WriteWithOutboxAsync(comment, ct);
        return true; // Available now; other regions converge shortly.
    }
}
```

> ⚠️ **Anti-pattern to avoid here.** A tempting but broken way to write the above is:
>
> ```csharp
> _ = Task.Run(() => _database.ReplicateAsync(comment)); // ❌ do not do this
> ```
>
> This is a **silent data-loss bug**, not eventual consistency:
> - Exceptions are unobserved — the replication fails and nobody ever knows.
> - The work is not tied to host lifetime, so it is killed mid-flight on shutdown, scale-in,
>   or deployment.
> - There is no retry, no ordering, and no record that the work was ever owed.
>
> Eventual consistency means *"the write is durably recorded and will converge"*, not
> *"we started a thread."* Use the **transactional outbox** (durable intent, at-least-once relay)
> or a queue. See [Distributed Systems Patterns](03-architectural-patterns/distributed-systems-patterns.md#transactional-outbox).

---

## Architecture Decision Records (ADRs)

**ADRs** document important architectural decisions with their context and consequences.

### ADR Template

```markdown
# ADR-001: Use Microservices Architecture

**Status:** Accepted

**Context:**
Our monolithic e-commerce application is becoming difficult to scale and deploy. 
Different teams are blocked waiting for each other's changes. Database is a 
bottleneck during peak traffic.

**Decision:**
Migrate to microservices architecture with separate services for:
- Product Catalog
- Order Management
- Payment Processing
- User Management

**Consequences:**

Positive:
+ Independent scaling of services
+ Faster deployments (team autonomy)
+ Technology diversity (use best tool for each job)
+ Better fault isolation

Negative:
- Increased operational complexity
- Need for service mesh/API gateway
- Distributed transaction challenges
- Higher infrastructure costs initially

**Alternatives Considered:**
1. Modular Monolith - Rejected due to scaling limitations
2. Serverless - Rejected due to cold start concerns for latency-sensitive operations
```

### Example in .NET Project

```csharp
// docs/architecture/decisions/ADR-002-use-azure-service-bus.md
/*
# ADR-002: Use Azure Service Bus for Async Communication

**Status:** Accepted

**Context:**
Microservices need reliable asynchronous communication for events like 
OrderPlaced, PaymentProcessed, etc. Need guaranteed delivery and 
dead-letter queue support.

**Decision:**
Use Azure Service Bus with Topics/Subscriptions model

**Implementation:**
*/

public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public async Task PlaceOrderAsync(Order order, CancellationToken ct)
    {
        // Persist the order and an outbox record in the same database transaction.
        var orderPlacedEvent = new OrderPlacedEvent 
        { 
            OrderId = order.Id, 
            CustomerId = order.CustomerId 
        };
        await _repository.SaveOrderAndOutboxMessageAsync(order, orderPlacedEvent, ct);

        // A separate outbox dispatcher publishes committed records and retries failures.
        // Consumers must tolerate duplicate delivery.
    }
}
```

The direct `save, then publish` sequence is not atomic: a process can fail after the database
commit but before the broker accepts the message. Use a transactional outbox when both effects
must be reliably coordinated; see [Distributed Systems Patterns](03-architectural-patterns/distributed-systems-patterns.md#transactional-outbox).

### When to Create ADRs

✅ Technology stack choices (database, cloud provider, frameworks)  
✅ Architectural style/pattern decisions  
✅ Significant design trade-offs  
✅ Security or compliance-related decisions  
✅ Major refactoring approaches  

❌ Routine code changes  
❌ Bug fixes  
❌ Minor implementation details

---

## Trade-off Analysis Framework

Architecture is about making **informed trade-offs**. Use this framework for decision-making:

### 1. **Identify Quality Attributes**

| Quality Attribute | Description | Measurement |
|------------------|-------------|-------------|
| **Performance** | Response time, throughput | Latency < 200ms, 10K requests/sec |
| **Scalability** | Handle growth | Support 10x user growth |
| **Availability** | Uptime | 99.9% SLA (8.76 hours downtime/year) |
| **Security** | Protect from threats | Pass security audit, encryption at rest |
| **Maintainability** | Ease of change | New feature in 1 sprint vs 3 sprints |
| **Cost** | Infrastructure + development | $10K/month vs $50K/month |

### 2. **Prioritize Requirements**

Use **MoSCoW** method:
- **Must have:** Non-negotiable (e.g., PCI compliance for payments)
- **Should have:** Important but not critical (e.g., real-time analytics)
- **Could have:** Nice to have (e.g., AI recommendations)
- **Won't have:** Out of scope (e.g., blockchain integration)

### 3. **Analyze Trade-offs**

#### Example: Caching Strategy

| Approach | Performance | Consistency | Complexity | Cost |
|----------|------------|------------|------------|------|
| **No Cache** | ❌ Slow | ✅ Always consistent | ✅ Simple | ✅ Low |
| **In-Memory Cache** | ✅ Fast | ⚠️ Eventually consistent | ⚠️ Medium | ✅ Low |
| **Distributed Cache (Redis)** | ✅ Very fast | ⚠️ Eventually consistent | ❌ Complex | ❌ High |

**Decision Matrix:**
```csharp
// Scenario 1: Product catalog (read-heavy, rare updates)
// Winner: Distributed Cache (Redis)
public class ProductCatalogService
{
    private readonly IDistributedCache _cache;
    
    public async Task<Product> GetProductAsync(int id)
    {
        var cacheKey = $"product:{id}";
        var cached = await _cache.GetStringAsync(cacheKey);
        
        if (cached != null)
            return JsonSerializer.Deserialize<Product>(cached);
        
        var product = await _database.GetProductAsync(id);
        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(product),
            new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10) });
        
        return product;
    }
}

// Scenario 2: Real-time inventory (consistency critical)
// Winner: No Cache (or very short TTL)
public class InventoryService
{
    public async Task<int> GetAvailableQuantityAsync(int productId)
    {
        // Always fetch from database - consistency is critical
        return await _database.GetInventoryCountAsync(productId);
    }
}
```

### 4. **Document Decision**

Create an ADR (see above) to capture:
- Context and requirements
- Options evaluated
- Trade-offs analyzed
- Final decision and rationale
- Consequences (positive and negative)

---

## Quality Attribute Scenarios

Use **concrete scenarios** instead of vague requirements:

### Template
**Source** → **Stimulus** → **Artifact** → **Environment** → **Response** → **Measure**

### Examples

#### Performance Scenario
- **Source:** User
- **Stimulus:** Searches for a product
- **Artifact:** Search API
- **Environment:** Normal load (1000 concurrent users)
- **Response:** Return search results
- **Measure:** 95th percentile < 200ms

```csharp
// Implementation
[ResponseCache(Duration = 60)] // Cache for 60 seconds
public async Task<IActionResult> Search(string query)
{
    var stopwatch = Stopwatch.StartNew();
    
    var results = await _searchService.SearchAsync(query);
    
    stopwatch.Stop();
    _telemetry.TrackMetric("SearchLatency", stopwatch.ElapsedMilliseconds);
    
    return Ok(results);
}
```

#### Availability Scenario
- **Source:** Hardware failure
- **Stimulus:** Primary database goes down
- **Artifact:** Order service
- **Environment:** Peak traffic
- **Response:** Failover to secondary database
- **Measure:** < 5 seconds downtime

```csharp
// Implementation with retry and circuit breaker — Polly v8 (ResiliencePipeline).
// NOTE: the older Policy.Handle(...).WaitAndRetryAsync(...).WrapAsync(...) API is v7 and
// is superseded. v8 also adds UseJitter, which v7 lacked by default.
public class OrderRepository
{
    private readonly ResiliencePipeline _pipeline;

    public OrderRepository()
    {
        _pipeline = new ResiliencePipelineBuilder()
            .AddRetry(new RetryStrategyOptions
            {
                ShouldHandle      = new PredicateBuilder().Handle<SqlException>(e => e.IsTransient),
                MaxRetryAttempts  = 3,
                BackoffType       = DelayBackoffType.Exponential,
                UseJitter         = true,   // prevents synchronised retry storms
                Delay             = TimeSpan.FromMilliseconds(200)
            })
            .AddCircuitBreaker(new CircuitBreakerStrategyOptions
            {
                ShouldHandle      = new PredicateBuilder().Handle<SqlException>(),
                FailureRatio      = 0.5,                        // ratio, not raw count
                MinimumThroughput = 20,
                SamplingDuration  = TimeSpan.FromSeconds(30),
                BreakDuration     = TimeSpan.FromMinutes(1)
            })
            .AddTimeout(TimeSpan.FromSeconds(2))
            .Build();
    }

    public async Task<Order?> GetOrderAsync(int id, CancellationToken ct) =>
        await _pipeline.ExecuteAsync(
            async token => await _dbContext.Orders.FindAsync([id], token), ct);
}
```

> **Ordering matters:** retry sits *above* the circuit breaker so the breaker observes each attempt
> and can open mid-retry; the timeout is innermost so one hung attempt can't consume the whole
> budget. For HTTP dependencies in .NET 8+, prefer
> `AddHttpClient<T>().AddStandardResilienceHandler()`, which wires this ordering for you.
> See [Distributed Systems Patterns](03-architectural-patterns/distributed-systems-patterns.md).

#### Security Scenario
- **Source:** Malicious user
- **Stimulus:** Attempts SQL injection
- **Artifact:** API endpoint
- **Environment:** Production
- **Response:** Reject request, log attempt
- **Measure:** Security tests verify that untrusted input cannot alter query structure; the API
  returns the documented validation response and emits a security event without logging secrets.

```csharp
// Implementation with parameterized queries
public async Task<User> GetUserAsync(string email)
{
    // ✅ Safe - parameterized query
    return await _dbContext.Users
        .Where(u => u.Email == email)
        .FirstOrDefaultAsync();
    
    // ❌ Unsafe - string concatenation
    // var sql = $"SELECT * FROM Users WHERE Email = '{email}'";
}
```

---

## Architecture Evaluation and Governance

Architecture evaluation checks whether a design can meet its stated drivers before and after
implementation. A useful review traces each important decision to a requirement, explores
alternatives and failure modes, and records risks, owners, and follow-up actions. Reviews are most
effective when they include developers, operations, security, and domain stakeholders.

Architecture fitness functions make selected constraints or quality goals repeatable to verify.
They can be automated tests (for example, module dependency rules), deployment checks, security
scans, or production SLO alerts. They do not prove that an architecture is good; they detect
specific forms of drift and should be paired with periodic human review. Record consequential
decisions and their trade-offs in [ADRs](#architecture-decision-records-adrs).

---

## Summary: Foundational Concepts

✅ **Understand the hierarchy** - Architecture → Styles → Patterns → Design Patterns → Principles  
✅ **Master quality attributes** - Scalability, availability, performance, security, maintainability  
✅ **Apply CAP theorem** - Choose between consistency and availability in distributed systems  
✅ **Document decisions** - Use ADRs for important architectural choices  
✅ **Analyze trade-offs** - Every decision has consequences; make informed choices  
✅ **Use concrete scenarios** - Define quality attributes with measurable criteria

---