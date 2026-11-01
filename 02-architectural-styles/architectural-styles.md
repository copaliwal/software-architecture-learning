# Architectural Styles

Architectural styles describe recurring ways to organize a system and constrain how its parts
interact. The terms in this overview do not all belong to the same taxonomy: some are styles,
some are communication or deployment patterns, and some are design approaches. Treat the list as
a selection guide, not a complete or mutually exclusive classification.

## Selected Styles and Related Approaches

| Style or approach | Classification | Description | Scope | Example |
| --- | --- | --- | --- | --- |
| [**Layered (N-tier)**](05-layered-architecture/layered-architecture.md) | Architectural style | Organizes code into layers with defined dependency rules. | Application | Presentation, application, and data-access layers. |
| **Monolithic** | Deployment style | Packages the application as one deployable unit; internal modularity varies. | Application | A single deployable web application. |
| **Modular monolith** | Deployment and module structure | One deployable unit with explicit, enforced internal module boundaries. | Application | An order-management application with separate ordering and inventory modules. |
| [**Microservices**](01-microservices-architecture/microservices-architecture.md) | Architectural and deployment style | Composes independently deployable services around capabilities and data ownership. | System | Catalog, order, and payment services. |
| **Service-Oriented Architecture (SOA)** | Architectural style | Integrates services through explicit contracts; governance and service granularity vary. | Enterprise | Business services shared across multiple applications. |
| [**Serverless**](02-serverless/serverless.md) | Cloud execution approach | Uses provider-managed compute and services, often event-triggered. | Application or workload | A function reacting to an object-storage event. |
| **Client–Server** | Architectural style | Separates requesters from service providers across a boundary. | Application or system | A browser client calling a web API. |
| **Peer-to-Peer (P2P)** | Architectural style | Nodes can act as both clients and providers. | Network or system | Distributed file sharing. |
| [**Event-Driven**](04-event-driven-architecture/event-driven-architecture.md) | Communication style | Components react to events; event channels may use queues, topics, or streams. | System or subsystem | An order event triggering independent consumers. |
| **Publish–Subscribe** | Messaging pattern | Publishers emit messages to topics; subscribers receive matching messages. | Subsystem or system | Topic subscriptions for order notifications. |
| **Microkernel (plug-in)** | Architectural style | A small core delegates variable capabilities to plug-ins. | Application | An editor extended through plug-ins. |
| **Pipe-and-Filter** | Architectural style | Filters transform data passed through a sequence of pipes. | Subsystem or application | A compiler pipeline. |
| **Space-Based** | Architectural style | Distributes processing and in-memory state to reduce centralized data-store bottlenecks. | High-throughput system | A workload partitioned across processing units with replicated in-memory data. |
| **Data-Centered (repository/blackboard)** | Architectural style | Components coordinate through a shared data store or knowledge base. | Application or system | A blackboard used by cooperating analysis components. |
| [**Hexagonal (Ports and Adapters)**](06-hexagonal-architecture/hexagonal-architecture.md) | Application architecture approach | Keeps application rules independent of external technologies through ports and adapters. | Application | Domain logic isolated from persistence and transport adapters. |
| **Clean Architecture** | Application architecture approach | Organizes code so source dependencies point toward policy and use cases. | Application | Framework-independent use cases behind adapters. |
| **Onion Architecture** | Application architecture approach | Places domain policy at the center, with outer infrastructure depending inward. | Application | A domain model surrounded by application and infrastructure layers. |
| [**Domain-Driven Design (DDD)**](03-domain-driven-design-DDD/domain-driven-design-DDD.md) | Modeling and design approach | Uses domain understanding, bounded contexts, and a shared language to manage complexity. | Domain or enterprise | Separate models for claims, policies, and billing. |

These approaches can be combined. For example, a modular monolith may use layered modules,
domain-driven design, and ports-and-adapters boundaries. Choose based on requirements and
constraints rather than treating any one style as a default solution.
