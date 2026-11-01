# Microservices Architecture

## 📖 Definition

- **Microservices** is an **architectural style** where a system is built as a collection of services organized around capabilities or bounded contexts.
- Services run as independently deployable units and communicate across network boundaries (for example, HTTP APIs or messaging).
- Services can be deployed, scaled, or changed independently when contracts, data ownership, and operational dependencies permit it.


---

## 🎯 Core Characteristics

- **Autonomy:** Each service owns its data and logic.
- **Bounded Contexts:** Services align with business domains (from Domain-Driven Design).
- **Independent Deployment:** Services can be deployed without redeploying the whole system.
- **Polyglot Freedom:** Different services can use different technologies (e.g., .NET for payments, Node.js for catalog).
- **Resilience:** Boundaries can limit failure propagation, but do not guarantee that a service failure leaves the overall system unaffected.
- **Scalability:** Services scale independently based on demand.

---

## 🏛 Scope

- **System or suite of applications level.**
- Microservices architecture defines the **shape of the entire system**, not just individual components.
- Patterns (Saga, API Gateway, CQRS) live inside microservices to solve specific problems.

---

## ⚙️ Key Components

- **Service:** Independent unit of functionality (e.g., Order Service, Payment Service).
- **API Gateway:** An optional entry point for client-facing routing and cross-cutting policies.
- **Service Registry:** Keeps track of available services (e.g., Consul, Eureka).
- **Data ownership:** Each service owns its data and schema contract; logical ownership does not require a separate database server for every service.
- **Messaging/Event Bus:** Enables asynchronous communication (e.g., Azure Service Bus, Kafka).
- **Observability Tools:** Logging, tracing, monitoring (e.g., Application Insights, Prometheus).

---

## 📚 Common Patterns in Microservices

- **Saga Pattern:** Distributed transaction management.
- **CQRS:** Separate read/write models for scalability.
- **Event Sourcing:** An optional persistence model for systems that need an event history and can support its operational costs.
- **API Gateway:** Optional routing and policy boundary for selected client traffic.
- **Circuit Breaker:** Prevent cascading failures.
- **Sidecar Pattern:** Deploy cross-cutting concerns (logging, monitoring) alongside services.

---

## ✅ Advantages
- Independent scaling and deployment.
- Potential to limit the scope of some failures when dependencies, timeouts, and degraded behavior are designed accordingly.
- Technology diversity (polyglot programming).
- Faster development cycles (teams own services).
- Aligns with agile and DevOps practices.

---

## ⚠️ Challenges

- **Complexity:** More moving parts (orchestration, monitoring, CI/CD).
- **Data consistency:** Cross-service workflows cannot rely on one local ACID transaction. Sagas or other explicit workflow/consistency strategies may be appropriate; Event Sourcing is optional and is not required for distributed consistency.
- **Operational overhead:** Requires mature DevOps, monitoring, and automation.
- **Latency:** Network calls between services add overhead.
- **Testing:** End-to-end testing is harder.

---

## 📊 When to Use

- Large, complex applications needing scalability and resilience.
- Systems with multiple teams working independently.
- Applications requiring frequent updates and deployments.
- Cloud-native systems leveraging containers and orchestration (Kubernetes, Azure AKS).
- Capabilities have clear ownership boundaries and the organization can operate independently deployed services.

---

## 🚫 When to Avoid

- Small applications or prototypes (monolith is simpler).
- Systems whose requirements can be met more simply by a well-modularized monolith.
- Teams without DevOps maturity (monitoring, CI/CD, automation).
- Workflows requiring frequent cross-service ACID transactions without acceptable coordination or eventual-consistency trade-offs.
- Environments with limited infrastructure or operational resources.

---

## 🛠️ Implementation in .NET & Azure

- **ASP.NET Core APIs** for services.
- **Azure Kubernetes Service (AKS)** for orchestration.
- **Azure API Management** as API Gateway.
- **Azure Service Bus/Event Grid** for messaging.
- A persistence technology selected for each service's consistency, workload, and operational requirements.
- **Azure Monitor + Application Insights** for observability.
- **CI/CD pipelines** with GitHub Actions or Azure DevOps.

---

## 📖 Example: E‑Commerce System

- **Catalog Service:** Manages product listings.
- **Order Service:** Handles order creation.
- **Payment Service:** Processes payments.
- **Inventory Service:** Manages stock.
- **Shipping Service:** Schedules deliveries.
- **API Gateway:** Routes client requests.
- **Saga Pattern:** Coordinates steps across order, payment, and inventory using local transactions and compensations; the workflow is eventually consistent.

---

## 🔍 Observability & Governance

- **Tracing:** Correlation IDs across services.
- **Logging:** Structured logs per service.
- **Metrics:** Service health, latency, throughput.
- **Documentation:** C4 diagrams, ADRs for decisions.
- **Governance:** Service contracts, versioning, backward compatibility.

---

## 📌 Checklist for Production Readiness

- [ ]  Independent deployment pipelines per service.
- [ ]  Data ownership and service boundaries enforced (avoid shared schemas that bypass ownership).
- [ ]  Client entry point and gateway policy selected to fit the system; a gateway is not mandatory for every service.
- [ ]  Resilience patterns (retry, circuit breaker).
- [ ]  Observability (logs, metrics, tracing).
- [ ]  Security (OAuth 2.0/OIDC, access-token validation, Microsoft Entra ID).
- [ ]  Documentation (C4 diagrams, ADRs).
- [ ]  Automated testing (unit, integration, end-to-end).

---
