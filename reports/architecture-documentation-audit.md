# Architecture Documentation Audit

**Audit date:** 2026-11-01

## Scope and Method

The repository was recursively inventoried before changes. The baseline contained 47 Markdown
files across foundations, architectural styles, architectural and distributed patterns, GoF
patterns, design principles, architect skills, advanced topics, and development process.
Documents were categorized and scanned against the requested architecture framework; detailed
content review focused on the indexes, distributed-systems guidance, style guides, and articles
changed in this pass.

The repository already had an uncommitted edit to
[`distributed-systems-patterns.md`](../03-architectural-patterns/distributed-systems-patterns.md)
at the start of the work. That edit was preserved and not overwritten.

## Inventory and Outcome

| Measure | Result |
| --- | ---: |
| Markdown files discovered before changes | 47 |
| Existing documents included in the repository-wide inventory and coverage scan | 47 |
| Existing documents enhanced in this pass | 11 |
| New topic guides created | 3 |
| Audit report created | 1 |
| Markdown files after this pass, including this report | 51 |
| Documents renamed | 0 |
| Broken internal file links found after changes | 0 |

### Existing Documents Enhanced

- [`README.md`](../README.md) — added links to the new material and replaced editorial notes with
  reader guidance and an audit link.
- [`01 Foundations.md`](../01%20Foundations.md) — removed over-generalized vendor-level
  CAP/PACELC labels and clarified consistency trade-offs.
- [`architectural-styles.md`](../02-architectural-styles/architectural-styles.md) — linked the
  modular-monolith guide.
- [`microservices-architecture.md`](../02-architectural-styles/01-microservices-architecture/microservices-architecture.md)
  — qualified fault isolation, data ownership, optional patterns, and transaction trade-offs.
- [`serverless.md`](../02-architectural-styles/02-serverless/serverless.md) — removed conversational
  draft text and added an official Azure Functions reference.
- [`domain-driven-design-DDD.md`](../02-architectural-styles/03-domain-driven-design-DDD/domain-driven-design-DDD.md)
  — prioritized primary and official sources and removed the conversational follow-up.
- [`event-driven-architecture.md`](../02-architectural-styles/04-event-driven-architecture/event-driven-architecture.md)
  — documented the dual-write failure window and linked the outbox guidance.
- [`architectural-patterns.md`](../03-architectural-patterns/architectural-patterns.md) — refocused
  the page as a navigation map and separated styles, enterprise patterns, and distributed
  integration concerns.
- [`distributed-transaction.md`](../03-architectural-patterns/distributed-transaction.md) —
  distinguished Saga, 2PC, and TCC from supporting patterns such as outbox, CQRS, and event
  sourcing.
- [`gang-of-four.md`](../04-design-patterns/gang-of-four.md) — completed the behavioral-pattern
  index with Iterator.
- [`07 Advanced Topics.md`](../07-advanced-topics/07%20Advanced%20Topics.md) — clarified that its
  DDD material is a .NET-oriented example companion to the canonical DDD guide.

### New Topic Guides

- [`modular-monolith.md`](../02-architectural-styles/07-modular-monolith/modular-monolith.md) —
  boundaries, deployment model, trade-offs, example, and comparison with microservices.
- [`enterprise-application-patterns.md`](../03-architectural-patterns/enterprise-application-patterns.md)
  — Repository, Unit of Work, Specification, Anti-Corruption Layer, and related application
  patterns.
- [`11-iterator.md`](../04-design-patterns/3-behavioral-patterns/11-iterator.md) — GoF intent,
  participants, trade-offs, and a .NET enumeration example.

## Knowledge Coverage

### Existing Strengths

- Foundations cover architecture versus design, drivers and constraints, quality attributes,
  ADRs, trade-off analysis, quality-attribute scenarios, evaluation, and governance.
- Architectural styles include detailed guides for layered, microservices, serverless,
  event-driven, and hexagonal approaches, plus an overview of related styles.
- The GoF collection now has guides for all 23 patterns.
- Distributed-systems guidance covers deadlines, retries, circuit breakers, bulkheads, rate
  limiting, idempotency, outbox/inbox, discovery, caching, overload, consistency, and operations.
- Security, cloud-native delivery, CI/CD, infrastructure as code, C4, ADRs, fitness functions,
  observability, and technical debt appear in broader guides and should be cross-linked rather
  than duplicated wholesale.

### Gaps Added or Partially Closed

- Added the missing Iterator guide.
- Added a standalone modular-monolith guide; it is a useful alternative to an unconstrained
  monolith or premature microservices.
- Added a selected enterprise-pattern guide for Repository, Unit of Work, Specification, and
  Anti-Corruption Layer, while linking related DDD, DTO, Service Layer, and DI concepts.
- Clarified that event sourcing is not a distributed transaction protocol, an outbox is not a
  distributed commit, and a Saga does not provide immediate cross-service consistency.

### Remaining Knowledge Gaps

The framework is a reference, not a requirement to create an article for every term. The
following topics remain absent as standalone, in-depth guides or are represented mainly by
overview mentions:

- Clean Architecture, Onion Architecture, SOA, Microkernel, Pipe-and-Filter, and Space-Based
  Architecture.
- Backend for Frontend and deeper service-mesh or service-discovery implementation guidance.
- Threat modeling, architecture fitness-function implementation, and architecture governance
  practices beyond their current summaries.
- Dedicated deployment and operations coverage for blue-green/canary releases, disaster
  recovery, and recovery objectives.
- More detailed, independently navigable quality-attribute guides and measurable scenario
  examples.

Prioritize these only where the intended audience or repository roadmap calls for them; several
are already named in indexes or covered partially in broader articles.

## Overlap and Organization

- The architectural-styles and architectural-patterns pages previously mixed related taxonomies.
  The styles page remains the selection overview; the patterns page is now a navigation map to
  detailed guides. The former overview's useful pointers to UI, runtime, security, and data
  concepts were retained in a clearly labeled reference section rather than deleted.
- DDD appears in both its canonical overview and the advanced .NET examples. The advanced section
  now links to the canonical guide and is explicitly framed as an implementation companion; its
  code examples remain useful.
- Overview mentions of related concepts are retained when they help navigation. No existing
  document was deleted or renamed.

## Validation

- **Internal Markdown file links:** 0 broken paths found. Link checks excluded fenced code
  examples; heading-fragment targets were not exhaustively checked.
- **Fenced code blocks:** no unpaired triple-backtick fences.
- **Mermaid:** 5 diagram blocks are present after the changes. They were reviewed for obvious
  syntax issues; no Mermaid parser is configured in the repository, so parser validation was not
  run.
- **Whitespace/diff check:** `git diff --check` passed.
- **Automated documentation tests:** none were configured in the repository.

## Recommended Follow-Up

1. Add standalone guides only for the remaining topics that match the audience's learning goals.
2. Consider a lightweight Markdown-link and Mermaid validation check in CI if the documentation
   changes frequently.
3. Keep examples version-scoped where APIs or cloud-service behavior may change, and revisit
   external references periodically.
