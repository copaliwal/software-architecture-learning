# Architectural Patterns and Related Approaches

This page is a navigation map, not a complete catalogue. Styles describe broad system
organization; patterns address recurring design, integration, or operational problems. Several
approaches can be combined in one system.

## Architecture and Design Approaches

| Topic | What it describes | Guide |
| --- | --- | --- |
| Architectural styles | Deployment and interaction shapes such as layered, monolithic, and event-driven systems. | [Architectural Styles](../02-architectural-styles/architectural-styles.md) |
| Domain-Driven Design | A modeling approach for complex business domains and bounded contexts. | [Domain-Driven Design](../02-architectural-styles/03-domain-driven-design-DDD/domain-driven-design-DDD.md) |
| Modular monolith | One deployment with explicit internal module boundaries. | [Modular Monolith](../02-architectural-styles/07-modular-monolith/modular-monolith.md) |
| Object-oriented patterns | Reusable solutions for object collaboration and responsibilities. | [Gang of Four patterns](../04-design-patterns/gang-of-four.md) |

## Enterprise Application Patterns

| Problem | Related patterns | Guide |
| --- | --- | --- |
| Isolating persistence and local commits | Repository, Unit of Work | [Enterprise Application Patterns](enterprise-application-patterns.md) |
| Protecting a domain from an external model | Anti-Corruption Layer, Adapter | [Enterprise Application Patterns](enterprise-application-patterns.md) |
| Coordinating reads and writes | CQRS | [CQRS](02-CQRS-pattern/CQRS-pattern.md) |
| Persisting state changes as history | Event Sourcing | [Event Sourcing](03-event-sourcing/event-sourcing.md) |

## Distributed Workflows and Integration

| Problem | Related patterns | Guide |
| --- | --- | --- |
| Coordinating work across service-owned data | Saga, two-phase commit, Try-Confirm/Cancel | [Distributed Transactions](distributed-transaction.md) |
| Handling timeouts, retries, duplicate delivery, and overload | Timeout, retry, circuit breaker, bulkhead, outbox/inbox, idempotency | [Distributed Systems Patterns](distributed-systems-patterns.md) |
| Choosing synchronous or asynchronous contracts | REST, GraphQL, gRPC, queues, publish/subscribe, streams, webhooks | [Integration and Communication Patterns](integration-and-communication-patterns.md) |
| Routing and composing client-facing APIs | API Gateway | [API Gateway](04-api-gateway/api-gateway.md) |

## Other Concepts in the Framework

These concepts were present in the former overview and remain useful navigation terms. Most are
brief pointers rather than standalone guides in this repository.

| Area | Concepts | Current coverage |
| --- | --- | --- |
| System organization | Monolith, three-tier, client-server, peer-to-peer, self-contained systems, microkernel, Pipe-and-Filter | [Architectural Styles](../02-architectural-styles/architectural-styles.md) |
| Application and UI structure | Layered, Hexagonal, Clean, Onion, MVC, MVVM, MVP | [Architectural Styles](../02-architectural-styles/architectural-styles.md); UI patterns are overview-only. |
| Runtime and migration | 12-Factor App, FaaS, sidecar, ambassador, service mesh, Strangler Fig | Overview-only; serverless and cloud-native material is in [Advanced Topics](../07-advanced-topics/07%20Advanced%20Topics.md). |
| API composition and security | Aggregator, Backend for Frontend, Zero Trust, identity federation, policy enforcement point | API Gateway and [Broader Architect Skills](../06-broader-architect-skills/Broader%20Architect%20Skills.md); several are overview-only. |
| Data organization | Database per service, shared database, sharding, partitioning, caching | Distributed data and caching guidance is in [Distributed Systems Patterns](distributed-systems-patterns.md). |

## Choosing Among Approaches

Start from the quality attributes, constraints, and business invariants. A microservices system
may use DDD to discover boundaries, a Saga to coordinate a workflow, an outbox to publish durable
messages, and a gateway for selected client traffic. None of those decisions implies that the
others are required.

Do not treat a shared database as a substitute for service data ownership, event sourcing as a
general-purpose transaction protocol, or a pattern name as a guarantee of scalability or
resilience. For decision context and architecture evaluation, see
[Foundations](../01%20Foundations.md).
