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

## Editorial Notes

The existing material includes updates clarifying CAP and PACELC, replacing fire-and-forget
replication with durable messaging, using Polly v8 resilience APIs, adopting Microsoft Entra
terminology, and distinguishing the Dependency Inversion Principle from dependency injection.

## Scope and Related Material

The Architect Handbook and interview-preparation material referenced by earlier notes are
separate sibling repositories and are not part of this repository's navigable documentation.
