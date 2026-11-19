# Software Architecture Learning Knowledge Base

This repository is a learning reference for software architecture, architectural styles and
patterns, design principles, implementation practices, and technology-specific guidance. The
articles are selected learning material, not an exhaustive catalogue. Choose approaches from
requirements, quality attributes, constraints, and operational context.

## 01 — Foundations

- [Foundations](01%20Foundations.md) — architecture and design, drivers and constraints, quality
  attributes, CAP/PACELC, ADRs, trade-offs, evaluation, and fitness functions.

## 02 — Architectural Styles and Approaches

- [Styles overview](02-architectural-styles/architectural-styles.md)
- [Microservices](02-architectural-styles/01-microservices-architecture/microservices-architecture.md)
- [Modular Monolith](02-architectural-styles/07-modular-monolith/modular-monolith.md)
- [Serverless](02-architectural-styles/02-serverless/serverless.md)
- [Domain-Driven Design](02-architectural-styles/03-domain-driven-design-DDD/domain-driven-design-DDD.md)
- [Event-Driven Architecture](02-architectural-styles/04-event-driven-architecture/event-driven-architecture.md)
- [Layered Architecture](02-architectural-styles/05-layered-architecture/layered-architecture.md)
- [Hexagonal Architecture](02-architectural-styles/06-hexagonal-architecture/hexagonal-architecture.md)

## 03 — Architectural Patterns

- [Pattern overview](03-architectural-patterns/architectural-patterns.md)
- [Distributed Systems Patterns](03-architectural-patterns/distributed-systems-patterns.md) —
  deadlines, retry, circuit breaker, bulkhead, rate limiting, idempotency, outbox/inbox, discovery,
  and caching.
- [Enterprise Application Patterns](03-architectural-patterns/enterprise-application-patterns.md) —
  Repository, Unit of Work, Specification, Anti-Corruption Layer, and related boundaries.
- [Integration and Communication Patterns](03-architectural-patterns/integration-and-communication-patterns.md) —
  request/response, REST, GraphQL, gRPC, queues, pub/sub, streams, and webhooks.
- [Saga](03-architectural-patterns/01-SAGA-pattern/SAGA-pattern.md)
- [CQRS](03-architectural-patterns/02-CQRS-pattern/CQRS-pattern.md)
- [Event Sourcing](03-architectural-patterns/03-event-sourcing/event-sourcing.md)
- [API Gateway](03-architectural-patterns/04-api-gateway/api-gateway.md)
- [Distributed Transactions](03-architectural-patterns/distributed-transaction.md)

## 04 — Design Patterns

- [Gang of Four overview](04-design-patterns/gang-of-four.md)
- **Creational:** [Factory Method](04-design-patterns/1-creational-patterns/1-factory-method.md),
  [Abstract Factory](04-design-patterns/1-creational-patterns/2-abstract-factory.md),
  [Factory comparison](04-design-patterns/1-creational-patterns/2.1-abstractfactory-vs-factory-method.md),
  [Builder](04-design-patterns/1-creational-patterns/3-builder-pattern.md),
  [Prototype](04-design-patterns/1-creational-patterns/4-prototype.md),
  [Singleton](04-design-patterns/1-creational-patterns/5-singleton.md)
- **Structural:** [Adapter](04-design-patterns/2-structural-patterns/1-adapter.md),
  [Bridge](04-design-patterns/2-structural-patterns/2-bridge.md),
  [Composite](04-design-patterns/2-structural-patterns/3-composite.md),
  [Decorator](04-design-patterns/2-structural-patterns/4-decorator.md),
  [Facade](04-design-patterns/2-structural-patterns/5-facade.md),
  [Flyweight](04-design-patterns/2-structural-patterns/6-flyweight.md),
  [Proxy](04-design-patterns/2-structural-patterns/7-proxy.md)
- **Behavioral:** [Observer](04-design-patterns/3-behavioral-patterns/1-observer.md),
  [Strategy](04-design-patterns/3-behavioral-patterns/2-strategy.md),
  [Command](04-design-patterns/3-behavioral-patterns/3-command.md),
  [Chain of Responsibility](04-design-patterns/3-behavioral-patterns/4-chain-of-responsibility.md),
  [State](04-design-patterns/3-behavioral-patterns/5-state.md),
  [Template Method](04-design-patterns/3-behavioral-patterns/6-template-method.md),
  [Mediator](04-design-patterns/3-behavioral-patterns/7-mediator.md),
  [Memento](04-design-patterns/3-behavioral-patterns/8-memento.md),
  [Interpreter](04-design-patterns/3-behavioral-patterns/9-interpreter.md),
  [Iterator](04-design-patterns/3-behavioral-patterns/11-iterator.md),
  [Visitor](04-design-patterns/3-behavioral-patterns/10-visitor.md)

## 05 — Design Principles

- [Design Principles](05-design-principles/Design%20Principles.md) — DRY, KISS, YAGNI, separation
  of concerns, and related principles.
- [SOLID](05-design-principles/SOLID/SOLID.md)

## 06 — Broader Architect Skills

- [Broader Architect Skills](06-broader-architect-skills/Broader%20Architect%20Skills.md) —
  security, delivery, performance, and architecture communication.

## 07 — Advanced Topics

- [Advanced Topics](07-advanced-topics/07%20Advanced%20Topics.md)
- [Cloud-Native .NET on Azure](07-advanced-topics/Architecting%20Cloud-Native%20NET%20Apps%20for%20Azure.md)

## 08 — Software Development Process

- [Branching Strategy](08-software-development-process/Branching%20Strategy.md)

## 09 — Containers and Docker

- [Docker and Containerization](09-containers-and-docker/docker-and-containerization.md) —
  containers vs VMs, Docker architecture, images/layers, multi-stage builds, ASP.NET Core images,
  networking, volumes, Compose, troubleshooting, security, Azure deployment, Docker vs Kubernetes.

## 10 — ASP.NET Core Web API Internals

- [ASP.NET Core Web API Internals](10-aspnet-core-web-api-internals/aspnet-core-web-api-internals.md) —
  pipeline and middleware, routing, controllers vs Minimal APIs, DI/Options/logging, exception
  handling, filters, binding/validation, hosted services, `IHttpClientFactory` and socket
  exhaustion, performance and production troubleshooting.

## 11 — Cloud Migration and Modernization

- [Cloud Migration and Legacy Modernization](11-cloud-migration-and-modernization/cloud-migration-and-legacy-modernization.md) —
  the 8 Rs, Strangler Fig, monolith to microservices, database migration and CDC, zero-downtime
  releases (blue-green, canary, parallel run), rollback, disaster recovery, hybrid cloud, cost.

## 12 — Webhooks and Event-Based Integration

- [Webhooks and Event-Based Integration](12-webhooks-and-event-based-integration/webhooks-and-event-based-integration.md) —
  webhook vs polling vs queues, provider/consumer design, HMAC signatures, idempotency, retries,
  ordering, async processing with Service Bus, monitoring and security.

## 13 — Azure Service Bus and Messaging Patterns

- [Azure Service Bus and Messaging Patterns](13-azure-service-bus-and-messaging-patterns/azure-service-bus-and-messaging-patterns.md) —
  messaging fundamentals, queues/topics/sessions, settlement, DLQ, duplicate detection, scaling,
  outbox/inbox, competing consumers, replay, Service Bus vs Event Grid vs Event Hubs.

## 14 — Azure Services and Cloud Architecture

- [Azure Services and Cloud Architecture](14-azure-services-and-cloud-architecture/azure-services-and-cloud-architecture.md) —
  core Azure services, service selection trade-offs, identity and Key Vault, networking and private
  endpoints, resilience and DR, Well-Architected Framework, cost optimization.

## 15 — Inbox, Outbox and Reliable Messaging

- [Inbox, Outbox and Reliable Messaging](15-inbox-outbox-and-reliable-messaging/inbox-outbox-and-reliable-messaging.md) —
  dual-write problem, transactional outbox (polling and CDC), atomic claiming, inbox idempotency,
  Service Bus delivery, why exactly-once is hard, idempotency keys for external APIs, DLQ and replay.

## 16 — Advanced Architecture and Microservices

- [Advanced Architecture and Microservices](16-advanced-architecture-and-microservices/advanced-architecture-and-microservices.md) —
  interview-oriented capstone: a map from every architecture and microservices subtopic to its deep
  dive, plus full coverage of Clean/Onion architecture, service decomposition, CAP in practice,
  resilience, observability, versioning, distributed monoliths and trade-off decisions.

## 17 — Database Performance Optimization and Troubleshooting

- [Part 1: SQL Server internals, query tuning and concurrency](17-database-performance-optimization/01-sql-server-internals-query-tuning-and-concurrency.md) — execution lifecycle,
  plans, indexes, statistics, parameter sniffing, SARGability, pagination, N+1, isolation levels,
  locking, deadlocks, row versioning, connection pooling.
- [Part 2: Database architecture, scaling and troubleshooting](17-database-performance-optimization/02-database-architecture-scaling-and-troubleshooting.md) — partitioning,
  sharding, replicas, Redis caching patterns, read/write separation, HA, Query Store, Extended
  Events, DMVs, wait stats and a production troubleshooting runbook.
- [Finding and fixing database performance issues](17-database-performance-optimization/03-finding-and-fixing-database-performance-issues.md) —
  end-to-end method: locate the bottleneck from app and database side, improvement ladder, proving
  the fix, prevention, and worked case studies.
- [OFFSET/FETCH and pagination deep dive](17-database-performance-optimization/04-offset-fetch-and-pagination-deep-dive.md) —
  why OFFSET is slow, how to make it cheaper, keyset pagination, cheap counts and API design.

## Using the Knowledge Base

Start with [Foundations](01%20Foundations.md) to identify drivers and trade-offs, then use the
style and pattern indexes to compare approaches. The collection is selective rather than
exhaustive; use the linked primary references for implementation details that vary by platform
or version.

## Scope and Related Material

The Architect Handbook and interview-preparation material referenced by earlier notes are
separate sibling repositories and are not part of this repository's navigable documentation.
