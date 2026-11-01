# Distributed transaction

## 📊 Comparison of Distributed Transaction Approaches

| Approach | Type | How It Works | Pros | Cons | Best Use Cases | Avoid When |
| --- | --- | --- | --- | --- | --- | --- |
| **Saga Pattern** | **Architectural Pattern** | Breaks a long transaction into local transactions with compensating actions. Two types: **Orchestration** (central controller) and **Choreography** (event-driven). | Scales well in microservices, avoids global locks, resilient | Eventual consistency only, debugging can be complex | E‑commerce orders, booking systems, workflows across multiple services | Systems needing strict immediate consistency (e.g., financial ledgers) |
| **Two‑Phase Commit (2PC)** | **Protocol** | Coordinator ensures all participants agree before commit. If one fails, rollback occurs. | Strong consistency, simple conceptual model | Blocking, poor scalability, single point of failure | Small distributed systems with few participants | Large-scale microservices, high‑latency networks |
| **Event Sourcing** | **Architectural Pattern** | Persist all state changes as events; rebuild state by replaying events. | Full audit/history, easy to debug, integrates with CQRS | Complex to implement, storage overhead | Financial systems, audit trails, systems needing replay | Simple apps where only current state matters |
| **Transactional Outbox** | **Pattern** | Write events and DB changes in same transaction; a relay publishes events reliably. | Ensures consistency between DB and message broker, simple to implement | Extra infrastructure (outbox table, relay), eventual consistency | Microservices needing reliable event publishing | Systems without messaging or where DB overhead is critical |
| **Try‑Confirm/Cancel (TCC)** | **Pattern** | Each service reserves resources (Try), confirms if successful, cancels otherwise. | Stronger consistency than Saga, explicit resource reservation | Complex to design, requires idempotent operations | Payment systems, resource booking | Simple workflows, systems without reservation semantics |

---

## 🔑 Key Insights

*   **Saga** → Best for **microservices** where eventual consistency is acceptable.
*   **2PC** → Best for **small distributed systems** needing strict consistency, but not scalable.
*   **Event Sourcing** → Best for **audit/history** requirements.
*   **Transactional Outbox** → Best for **reliable event publishing** in microservices.
*   **TCC** → Best for **resource reservation workflows** (e.g., booking, payments).

---

## ✨ Takeaway

*   **Saga is an architectural pattern** designed for distributed transactions in microservices.
*   It trades strict consistency for scalability and resilience.
*   Alternatives like **2PC, Event Sourcing, Outbox, TCC** each solve different aspects of the distributed transaction problem.

---