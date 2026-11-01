# Architecting Cloud-Native .NET Apps for Azure

## 1\. Introduction to Cloud-Native Applications

Cloud-native architecture is an approach to designing, constructing, and operating workloads that are **built in the cloud and take full advantage of the cloud computing model**. This approach emphasizes **speed and agility**.

### The Foundational Pillars of Cloud Native

Cloud-native systems rely on six key foundational pillars:

1.  **The Cloud (Infrastructure)**
    - Cloud-native systems use **Platform as a Service (PaaS)** and managed services extensively.
    - They treat underlying infrastructure as **disposable** (provisioned quickly, resized, scaled, or destroyed on demand).
    - They embrace the "Cattle" service model (as opposed to "Pets"), meaning instances are standardized and disposable, promoting **immutable infrastructure**.
2.  **Modern Design**
    - Adheres to the **Twelve-Factor Application** methodology, which provides principles for applications optimized for modern cloud environments.
    - Key factors include maintaining a single **Code Base** per microservice, externalizing **Configurations**, ensuring **Processes** are stateless, favoring horizontal **Concurrency** (scaling out), designing for **Disposability** (fast startup/graceful shutdown), and maintaining **Dev/Prod Parity**.
    - Also emphasizes being **API First**, including built-in **Telemetry**, and robust **Authentication/Authorization**.
    - Follows the **Azure Well-Architected Framework** pillars: Cost management, Operational excellence, Performance efficiency, Reliability, and Security.
3.  **Microservices**
    - An architectural style consisting of distributed, small, independent services.
    - Each microservice encapsulates a **specific business capability**, its own code, data storage technology, dependencies, and programming platform.
    - They provide **agility** and enable **independent deployment and scaling** of application components.
    - Dapr (Distributed Application Runtime) is an open-source application runtime that simplifies the plumbing behind distributed applications through pluggable components.
4.  **Containers**
    - Containers package the application code, dependencies, and runtime into a binary called a **container image**.
    - Containers provide **portability and guarantee consistency** across different environments.
    - **Docker** is the de facto standard for containerizing applications.
    - **Container Orchestrators** (like Kubernetes) manage containers at scale, handling scheduling, health monitoring, failover, scaling, networking, and rolling upgrades. **Kubernetes (K8s)** is the de facto standard orchestrator. **Azure Kubernetes Service (AKS)** is the fully managed platform offering.
5.  **Backing Services**
    - Ancillary resources such as databases, message brokers, caching, monitoring, and identity services.
    - Cloud-native systems favor **managed backing services** (DBaaS/SaaS) from cloud vendors to reduce operational overhead.
    - They should be treated as attached resources, bound dynamically using external configuration (Principle of decoupling from the application code).
6.  **Automation (DevOps)**
    - Encompasses **Infrastructure as Code (IaC)** to declaratively automate platform provisioning (e.g., using Azure Resource Manager, Terraform, or Azure CLI). IaC ensures deployments are automated, consistent, and repeatable.
    - Involves robust **CI/CD pipelines** (Continuous Integration/Continuous Delivery) that enforce separation across the build, release, and run stages, enabling frequent, predictable updates.

* * *

## 2\. Scaling Cloud-Native Applications

Scaling focuses on increasing application capacity to meet user load without compromising performance.

### Scaling with Containers and Orchestrators

- **Scaling Out vs. Scaling Up:** Cloud-native applications favor **scaling out** (horizontal scaling: adding more resources/instances) over scaling up (vertical scaling: adding capacity to a single resource).
- **Kubernetes Scaling:** AKS clusters scale automatically in two dimensions:
    - **Horizontal Pod Autoscaler (HPA):** Monitors resource demand (e.g., CPU usage) and automatically scales the number of running container instances (pods).
    - **Cluster Autoscaler:** Automatically scales the underlying compute nodes (VMs) in the cluster when more compute capacity is needed.
- **Configuration:** Kubernetes deployments should use **declarative configuration** (manifest files) to describe the desired end state, promoting Infrastructure as Code and reliable deployments.
- **Azure Container Hosting Options** (ordered highest abstraction → lowest; **prefer the highest that meets the requirement**):
    - **Azure Container Apps (ACA):** ⭐ **The current default for containerised microservices.** A managed serverless container platform built on Kubernetes, **KEDA** (event-driven autoscaling, including **scale to zero**), **Dapr**, and Envoy ingress — without exposing the Kubernetes API or making you operate a cluster. Choose this unless you can name a capability it lacks.
    - **Azure App Service for Containers:** Suitable for simple production web applications that do not require complex orchestration.
    - **Azure Kubernetes Service (AKS):** For **full** container orchestration where you genuinely need the raw Kubernetes API — custom operators/CRDs, a service mesh you control, specific CNI or node-level configuration, GPU/specialised node pools, StatefulSets — or where the organisation already has a Kubernetes platform team. AKS is not "more advanced ACA"; it is a permanent operational commitment (cluster upgrades, node patching, networking, RBAC, capacity).
    - **Azure Container Instances (ACI):** Serverless single-container solution for short-running workloads or burst tasks; the fastest way to run a simple container. No orchestration or service discovery.

> **Interview note:** defaulting to AKS "for flexibility" is a common red flag. You should have to justify *descending* the abstraction ladder, not justify staying high on it. Compare the operational trade-offs against the [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/).

### Leveraging Serverless Functions

- **Serverless Definition:** A service model where the cloud vendor manages the server infrastructure, and the application team focuses only on code.
- **Azure Functions:** Represents a function, which is a small, lightweight block of code that executes a **single-purpose operation in response to an event**.
- **Appropriate Scenarios:** Ideal for event-driven, asynchronous background tasks (e.g., sending email after an event) or moving slower tasks outside the user interaction loop.
- **Cold Starts:** A primary challenge; the time required to provision a new function instance, which can cause delays.
- **Containerized Functions:** Azure Functions can be wrapped in Docker containers, and when deployed to AKS, **KEDA (Kubernetes-based Event Driven Autoscaling)** can be used to manage scaling from zero instances to meet demand.

* * *

## 3\. Cloud-Native Communication Patterns

Communication in a cloud-native environment is complex due to network calls replacing local method calls.

### Front-End Client Communication

- **API Gateway:** Recommended pattern to route incoming traffic, abstract back-end services from clients, and handle cross-cutting concerns (identity, security).
- **Backend for Frontends (BFF):** A pattern using multiple API Gateways, optimized for specific client types (mobile, web).
- **Azure Services for Gateways:**
    - **Azure Application Gateway:** PaaS service offering basic routing and Layer-7 load balancing; can integrate with AKS via the Ingress Controller.
    - **Azure API Management (APIM):** Full-featured service offering policy enforcement (throttling, caching, security), developer portals, and management dashboards. Recommended for moderate to large-scale systems.
- **Real-Time Communication:** **Azure SignalR Service** is a fully managed service that simplifies pushing content updates directly to connected clients (browser, mobile) over persistent connections.

### Service-to-Service Communication

Cross-service interaction can be categorized as Queries, Commands, or Events.

| Interaction Type | Description | Pattern/Implementation |
| --- | --- | --- |
| **Query** (Request/Response) | Requires an immediate response from the called service. | **Materialized View Pattern** (local data copy) to remove coupling. **Service Aggregator Pattern** to centralize workflow logic. **gRPC** for high-performance synchronous needs. |
| **Command** (Fire-and-Forget) | Calling service requires an action to be executed, but not an immediate response. | **Message Queues** (asynchronous, point-to-point messaging). Azure options: **Storage Queues** (simple, high capacity, low cost) or **Service Bus Queues** (brokered messaging, FIFO, enterprise features). |
| **Event** (Publish/Subscribe) | A service publishes a notification of a state change; multiple interested subscribers react. | **Topics** (one-to-many pattern). Azure options: **Service Bus Topics** (robust, pull model) or **Event Grid** (event-driven, push model, near real-time, deep Azure integration). |
| **Streaming** | Processing a stream of time-ordered, interrelated events (e.g., telemetry). | **Azure Event Hub:** Scalable data streaming platform that collects, transforms, and stores millions of events per second; uses a partitioned consumer model and supports Kafka. |

### gRPC

- **Modern Protocol:** A modern, high-performance framework that evolves the Remote Procedure Call (RPC) protocol.
- **Transport:** Uses **HTTP/2** for its transport protocol, featuring a binary framing protocol and multiplexing.
- **Performance:** Can be up to **8x faster than JSON serialization** and uses messages 60-80% smaller.
- **Serialization:** Employs **Protocol Buffers** (Protobuf), a platform-neutral, efficient serialization format defined using `.proto` files.
- **Usage:** Primary choice for **direct synchronous microservice-to-microservice communication** in polyglot environments where low latency and high throughput are critical.

### Service Mesh

- **Infrastructure Layer:** A configurable infrastructure layer designed to handle complex service-to-service communication, resiliency, and cross-cutting concerns.
- **Sidecar Pattern:** Communication responsibilities are moved into a service proxy (Sidecar Proxy) colocated with each microservice, providing isolation from business logic.
- **Responsibilities:** Handles service discovery, load balancing, health checks, metrics capture, and distributed tracing automatically.
- **Implementation:** **Istio** is a leading service mesh, often using the **Envoy proxy**.

* * *

## 4\. Cloud-Native Data Patterns

### Distributed Data Model

- **Database-per-Microservice:** The classic monolithic database evolves into a distributed data model where each independent microservice encapsulates its own data.
- **Polyglot Persistence:** Microservices use the data store type (relational, document, key-value, graph) best optimized for their specific workload, storage needs, and read/write patterns.

### Handling Consistency in Distributed Data

- **Cross-Service Queries:** Solved using the **Materialized View Pattern**, where a service stores a local, denormalized copy of external data, synchronized asynchronously via events (Pub/Sub).
- **Distributed Transactions:** Solved using the **Saga Pattern**, which executes a series of local transactions and uses **compensating transactions** to roll back changes if any step fails. This moves the system toward **eventual consistency**.
- **High Volume Data Patterns:**
    - **CQRS (Command and Query Responsibility Segregation):** Separates read operations (Query model, Read Store) from write operations (Command model, Write Store) to maximize performance, scalability, and security.
    - **Event Sourcing:** Persists every data change as an immutable event in an Event Store. The current state is derived by replaying events, often projected into a Materialized View for querying.

### Database Types and Azure Offerings

| Database Type | Characteristics | Azure Services |
| --- | --- | --- |
| **Relational (SQL)** | Mature, fixed schema, supports ACID transactions. Typically favours Consistency over Availability during a partition (**CP** — see [Foundations](../01%20Foundations.md#cap-theorem)). | **Azure SQL Database** (Single, Elastic Pool, Managed Instance, Serverless, **Hyperscale** for >4 TB). **Azure Database for PostgreSQL / MySQL — Flexible Server**. ⚠️ *Azure Database for MariaDB has been retired — do not propose it.* |
| **NoSQL** | High performance, schemaless, scalable. Supports Availability and Partition Tolerance. Models include Document, Key-Value, Graph, and Wide-Column. | **Azure Cosmos DB:** Globally distributed, multi-model (supports Mongo, Cassandra, Gremlin APIs), and offers **five tunable consistency models**. |
| **NewSQL** | Emerging technology combining the distributed scalability of NoSQL with the **ACID guarantees** of relational databases. | Open-source options like **CockroachDB**, **TiDB**, and **YugabyteDB** are featured by the CNCF and designed to work natively in Kubernetes. |

### Caching

- Caching copies frequently accessed data to fast storage closer to the application to reduce latency and database contention.
- Cloud-native applications use a **distributed caching** architecture (cache hosted separately as a backing service).
- The common pattern is **Cache-Aside**, where the application queries the cache first, and if data is missing, it retrieves it from the database, writes it to the cache, and returns it.
- **Azure Cache for Redis** is the fully managed, high-throughput Azure PaaS service for distributed caching, supporting advanced features like clustering and geo-replication in the Premium tier.
- **Elasticsearch** is a separate system for distributed search and analytics, useful for complex search capabilities across diverse data types.

* * *

## 5\. Cloud-Native Resiliency

Resiliency is the ability of your system to react to failure and still remain functional, minimizing downtime and disruption in a distributed environment.

### Application Resiliency Patterns

Resiliency policies are applied to request messages to compensate for a service being momentarily unavailable.

- **Polly:** A comprehensive .NET resilience and transient-fault-handling library.
- **Retry Pattern:** Retries failed requests due to transient faults a configurable number of times, typically using an **exponentially increasing backoff time**.
- **Circuit Breaker Pattern:** Blocks traffic to a service immediately after a predefined number of failed calls to allow the service to recover and prevent a cascading failure (self-imposed denial of service).

### Azure Platform Resiliency (Redundancy and Scalability)

- **Redundancy:** Architecting redundancy involves identifying critical paths and ensuring failover mechanisms exist.
    - Deploy multiple service instances.
    - Use load balancers to distribute requests to healthy instances.
    - Plan for **multiregion deployment** using services like Azure Traffic Manager (DNS load balancing) to route traffic across AKS clusters in different regions.
    - Enable **geo-replication** for databases (Cosmos DB, Azure SQL) and container images (ACR) to protect against regional outages.
- **Scalability Design:**
    - Services must be **stateless**.
    - Partition workloads into microservices to scale services independently.
    - Favor **scale-out** (horizontal scaling).
    - Avoid affinity ("sticky sessions")—if state is needed, save it to a distributed cache.
- **Built-in Retries:** Many Azure services (e.g., Cosmos DB, Azure Service Bus, Azure Storage) and their client SDKs include built-in retry mechanisms, which should be understood and leveraged.

* * *

## 6\. Monitoring and Health (Observability)

As services are distributed, centralized monitoring and logging become mandatory.

### Observability Patterns

1.  **Logging:** Capturing information about application activity. In cloud-native apps, centralized logs are preferred over file-based logs. Best practice is to use a **Correlation ID** to link messages across multiple services involved in a single request.
2.  **Monitoring:** Collecting **telemetry and metrics** (response time, latency, CPU load, business metrics) to gauge the holistic health of the system.
3.  **Alerts:** Triggered by monitoring tools when metrics fall outside acceptable levels. Highly mature systems use alerts to trigger **self-healing tasks** (e.g., autoscaling).

### Tools for Observability in Azure

- **Elastic Stack (ELK):** A popular open-source solution for centralized logging and analytics.
    - **Logstash:** Used to gather log information from various sources.
    - **Elasticsearch:** Used to index and store logs, enabling fast querying.
    - **Kibana:** Provides interactive visualizations and web dashboards.
- **Azure Monitor:** An umbrella collection of tools providing visibility into system health.
    - **Application Insights:** Part of Azure Monitor, it gathers application-level metrics, events, and exceptions. It uses the powerful **Kusto Query Language** for analysis and reporting.
    - **Azure Monitor for Containers:** Supports consuming logs and metrics from AKS and other orchestrators, including integration with Prometheus metrics endpoints.
- **OpenTelemetry (OTel)** ⭐ **— the current strategic answer, and the one missing from most candidates' answers.**
    - A **vendor-neutral CNCF standard** for generating traces, metrics, and logs, with first-class .NET support (`System.Diagnostics.Activity` *is* the OTel tracing API in .NET).
    - Microsoft ships the **Azure Monitor OpenTelemetry Distro** (`Azure.Monitor.OpenTelemetry.AspNetCore`), which exports OTel signals into Application Insights.
    - **Why an architect prefers it:** instrumentation lives in your code against an open standard, while the *backend* stays a configuration choice. You can export to Azure Monitor, Prometheus/Grafana, or a third-party APM without re-instrumenting. It also gives consistent context propagation (W3C `traceparent`) across polyglot services.
    - **Guidance:** instrument with OpenTelemetry, export to Azure Monitor. Recommending the classic Application Insights SDK for a greenfield system is dated advice.

    ```csharp
    builder.Services.AddOpenTelemetry()
        .ConfigureResource(r => r.AddService("orders-api"))
        .WithTracing(t => t.AddAspNetCoreInstrumentation()
                           .AddHttpClientInstrumentation()
                           .AddSqlClientInstrumentation())
        .WithMetrics(m => m.AddAspNetCoreInstrumentation()
                           .AddRuntimeInstrumentation())
        .UseAzureMonitor();   // export to Application Insights
    ```

* * *

## 7\. Cloud-Native Identity

Identity management includes authentication (AuthN) and authorization (AuthZ).

- **Modern Identity Model:** Cloud-native solutions use open standards like **OpenID Connect (OIDC)** and **OAuth 2.0** and typically rely on access tokens (JWTs) issued by a **Secure Token Service (STS)**.
- **Microsoft Entra ID** (renamed from *Azure Active Directory / Azure AD* in 2023): Microsoft’s fully managed, cloud-native identity and access management service. For customer/partner-facing (CIAM) scenarios, use **Microsoft Entra External ID**, which supersedes Azure AD B2C for new builds.
- **Duende IdentityServer:** An authentication server implementing OIDC and OAuth 2.0 for ASP.NET Core, commonly used for **Single Sign-On (SSO)** across multiple applications.
  > ⚠️ **Licensing changed — this matters architecturally.** The free, open-source *IdentityServer4* reached end of life and the product is now **Duende IdentityServer**, which is **commercially licensed** above a revenue threshold. Do not propose "IdentityServer" in a design without acknowledging the cost.
  >
  > **Decision guidance:** prefer **Microsoft Entra ID** (workforce) or **Microsoft Entra External ID** (customer/partner) unless you have a concrete requirement they can't meet — full control of the token pipeline, unusual federation, on-premises or sovereign hosting, or deep customisation of the login journey. Self-hosting an identity provider means you now own a security-critical system: patching, key rotation, and breach exposure.

* * *

## 8\. Cloud-Native Security

Security should be a primary concern throughout the entire application lifecycle.

- **Security Mindset:**
    - **Threat Modeling:** Systematically identifying and assessing potential threats.
    - **Principle of Least Privilege (POLP):** Granting any user or process the minimum rights necessary to perform its task.
    - **Penetration Testing:** Using external actors to attempt to breach the system.
    - **Securing the Build:** Ensuring the build server itself is secure and runs checks (like scanning for checked-in credentials or outdated packages).
- **Azure Infrastructure Security:**
    - **Azure Virtual Network (VNet):** Most Azure PaaS resources should be placed in a VNet to establish a private network protected from the wider internet.
    - **Network Security Groups (NSG):** Used to restrict traffic flow between resources within a VNet, enforcing the principle of least privilege in networking.
- **Role-Based Access Control (RBAC):** Used to restrict access to Azure resources. It combines a **Security Principal** (User, Group, Managed Identity) with a **Role** (permissions set) and a **Scope** (the resource boundary). Deny rules take precedence over allow rules.
- **Securing Secrets:**
    - **Azure Key Vault:** Centralized, managed service for securely storing and accessing secrets (API keys, passwords, certificates). Access is controlled via RBAC.
    - **Kubernetes Secrets:** Built-in mechanism for storing small pieces of secret data, though using Key Vault is often more secure.
- **Encryption:**
    - **Encryption in Transit:** Typically achieved using **Transport Layer Security (TLS)** connections for all access to Azure services.
    - **Encryption at Rest:** Data on disk is heavily encrypted. Azure Storage uses FIPS 140-2 compliant 256-bit AES. Azure SQL uses **Transparent Data Encryption (TDE)** and optional **Always Encrypted** for column-level encryption. Cosmos DB enforces AES-256bit encryption by default.

* * *

## 9\. DevOps (CI/CD and IaC)

DevOps practices enable faster, more reliable software releases. Key tooling includes **Azure DevOps** (Boards, Repos, Pipelines) and **GitHub Actions**.

### Source Control

- **Repository Structure:** While successful applications use different layouts, the recommendation is often **Repository per Microservice**. This fosters autonomy, separation of concerns, and simplifies building and maintenance instructions.
- **Standard Directory Structure:** Maintaining a consistent structure across microservice repositories simplifies developer onboarding and enables bulk operations.

### Continuous Integration / Continuous Delivery (CI/CD)

- **Azure Pipelines:** Provides the tools for automated build and release processes.
- **Azure Builds (CI):** Automates the compilation, testing, and packaging of code, often defined in versioned `azure-pipelines.yml` files. The output is a **build artifact**.
- **Azure Releases (CD):** Manages the deployment of artifacts through defined "stages" (e.g., Dev, QA, Production), supporting manual approvals or gates at each step.
- **Best Practice:** Have at least **one build pipeline per microservice** to ensure independent deployment capability.
- **Feature Flags:** A deployment technique using conditional logic to restrict the visibility of new features in production, separating code deployment from feature release. Managed centrally using **Azure App Configuration**.

### Infrastructure as Code (IaC)

IaC automates platform provisioning, ensuring infrastructure is consistent and repeatable.

| IaC Tool | Characteristics | Usage Notes |
| --- | --- | --- |
| **Bicep** ⭐ | Microsoft's **recommended** DSL for Azure. Concise, typed, modular, with strong IDE tooling. Transpiles to ARM JSON — same engine, same idempotency, far better authoring. | **Default for Azure-only estates.** Day-zero support for new resource types. |
| **Azure Resource Manager (ARM) Templates** | JSON-based API provisioning engine built into Azure. **Idempotent** (running the same template multiple times has the same result). | Still the underlying engine, but **authoring raw ARM JSON is no longer recommended** — write Bicep and let it compile down. |
| **Terraform** | Uses HCL (HashiCorp Configuration Language). Cloud-agnostic (supports Azure, AWS, GCP). Maintains explicit **state**. | Preferred for multi-cloud, or where the organisation has standardised on it. State storage/locking is the main operational consideration. |
| **Azure CLI / PowerShell Scripts** | Imperative scripting. Easy learning curve and debugging. | Fine for throwaway environments; requires custom logic for idempotency. Not a substitute for declarative IaC in production. |
| **Azure Developer CLI (`azd`)** | Packages application code + infrastructure (Bicep/Terraform) + pipeline definitions into a repeatable `azd up` workflow. | Useful for standardising scaffolding and the inner-loop→cloud path. |

> ⚠️ **Removed as obsolete.** An earlier version of this note listed **Cloud Native Application
> Bundles (CNAB)** with the **Duffle** tool. That tooling is effectively dead and should not be
> proposed in a current design. For packaging Kubernetes applications use **Helm**; for
> Azure-native provisioning use **Bicep**.

* * *

### Analogy for Understanding Cloud-Native Resilience

Think of building a resilient cloud-native application like operating a modern, global shipping logistics company.

- **Monolithic System (Traditional Company):** You have one central warehouse and one massive delivery truck (the application server). If the truck breaks down, or the warehouse catches fire, the entire business stops. If you need to deliver more, you have to buy a bigger, more expensive truck (scaling up).
- **Cloud-Native System:** You replace the one warehouse with many small, specialized distribution centers (microservices). Instead of one large truck, you use a large fleet of smaller vans (containers/pods) distributed across the country (AKS clusters).
    - **Resiliency (Circuit Breaker/Retry):** If one van breaks down temporarily (transient fault), the logistics company retries the delivery later (Retry Pattern). If a whole distribution center stops responding due to severe flooding, the company stops sending vans there immediately, instead relying on nearby centers until the first one is repaired (Circuit Breaker Pattern).
    - **Scaling:** If demand increases, you don't buy a single, larger van (scaling up); you instantly deploy more small vans and open temporary pop-up distribution points (scaling out/autoscaling).
    - **Service Mesh:** The logistics routes and rules (which van goes where, how fast, safety checks) are handled by a central, automated traffic management system (the Service Mesh/Istio), so the individual van drivers (developers) only worry about the cargo (business logic).
- **Distributed Data:** Each distribution center only holds inventory relevant to its region and specialization, but keeps a local, updated manifest of key products sold by other centers (Materialized View) to speed up local planning without needing to call the other centers every time.