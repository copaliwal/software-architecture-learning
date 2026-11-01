# Distributed Transactions and Coordination

## What Needs Coordination?

A workflow that changes data owned by more than one resource cannot usually use one local ACID
transaction. Choose a strategy from the business invariant, participant capabilities, failure
semantics, and acceptable user-visible states. These approaches solve different problems; they
are not interchangeable implementations of one universal transaction.

## Coordination Approaches

| Approach | How it works | Trade-offs and suitable use |
| --- | --- | --- |
| **Saga** | A sequence of local transactions. A later failure triggers compensating business actions for completed steps. Coordination may be orchestration or choreography. | Allows participants to commit independently, but exposes intermediate states and compensation is not a literal rollback. Use for long-running business workflows that can be eventually consistent. |
| **Two-phase commit (2PC)** | A coordinator asks participants to prepare, then directs them to commit or abort. | Can provide atomic commit when every participant and the transaction manager support the protocol. Participants may hold resources while waiting; coordinator recovery and blocking behavior matter. Use only where the environment and availability requirements support it. |
| **Try-Confirm/Cancel (TCC)** | Each participant reserves or prepares a business resource, then confirms or cancels that reservation. | Makes resource reservation explicit, but requires participant APIs, expiration/recovery rules, and idempotent operations. It does not automatically provide global ACID isolation or stronger consistency than every Saga. |

### Patterns That Support, but Do Not Replace, Coordination

- **Transactional outbox:** Atomically stores a business update and the intent to publish a message
  in one local database transaction. A relay publishes it later, normally with at-least-once
  delivery; consumers must handle duplicates. This solves reliable publication from one database,
  not atomic commit across the database and broker. See
  [Distributed Systems Patterns](distributed-systems-patterns.md#transactional-outbox).
- **Event Sourcing:** Persists domain changes as an event sequence and rebuilds state from that
  history. It can support audit and temporal requirements and can be combined with a Saga, but it
  is not itself a distributed transaction protocol. See
  [Event Sourcing](03-event-sourcing/event-sourcing.md).
- **CQRS:** Separates command and query models. It may make read-model consistency explicit but
  does not coordinate writes across services. See [CQRS](02-CQRS-pattern/CQRS-pattern.md).

## Choosing a Strategy

1. State the invariant and identify which component owns each piece of data.
2. Decide whether intermediate states are visible and how long they may persist.
3. Determine whether participants support a shared commit protocol, reservations, or only local
   transactions.
4. Define timeout, retry, duplicate, compensation, and recovery behavior before implementation.
5. Make workflow status observable and provide a safe way to reconcile or repair incomplete work.

Use a Saga when a business workflow can tolerate intermediate states and has meaningful
compensations. Consider 2PC only when the resource managers, operational environment, and failure
requirements make its blocking and recovery model acceptable. Use TCC when participants can
reserve domain resources and the business needs explicit confirm/cancel steps.