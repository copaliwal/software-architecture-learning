# Microservices Architecture

## 📖 Definition

- **Microservices** is an **architectural style** where an application is built as a collection of **small, independent services**.
- Each service is **autonomous**, runs in its own process, and communicates via lightweight mechanisms (usually HTTP/REST, gRPC, or messaging).
- Services are **independently deployable** and can be scaled, updated, or replaced without affecting the entire system.


---

## 🎯 Core Characteristics

- **Autonomy:** Each service owns its data and logic.
- **Bounded Contexts:** Services align with business domains (from Domain-Driven Design).
- **Independent Deployment:** Services can be deployed without redeploying the whole system.
- **Polyglot Freedom:** Different services can use different technologies (e.g., .NET for payments, Node.js for catalog).
- **Resilience:** Failures are isolated; the system continues functioning.
- **Scalability:** Services scale independently based on demand.

---

## 🏛 Scope

- **System or suite of applications level.**
- Microservices architecture defines the **shape of the entire system**, not just individual components.
- Patterns (Saga, API Gateway, CQRS) live inside microservices to solve specific problems.

---

## ⚙️ Key Components

- **Service:** Independent unit of functionality (e.g., Order Service, Payment Service).
- **API Gateway:** Entry point for clients, routing requests to services.
- **Service Registry:** Keeps track of available services (e.g., Consul, Eureka).
- **Database per Service:** Each service owns its persistence (no shared DB).
- **Messaging/Event Bus:** Enables asynchronous communication (e.g., Azure Service Bus, Kafka).
- **Observability Tools:** Logging, tracing, monitoring (e.g., Application Insights, Prometheus).

---

## 📚 Common Patterns in Microservices

- **Saga Pattern:** Distributed transaction management.
- **CQRS:** Separate read/write models for scalability.
- **Event Sourcing:** Persist state as events.
- **API Gateway:** Single entry point for clients.
- **Circuit Breaker:** Prevent cascading failures.
- **Sidecar Pattern:** Deploy cross-cutting concerns (logging, monitoring) alongside services.

---

## ✅ Advantages
[^1]
- Independent scaling and deployment.
- Fault isolation (failure in one service doesn’t crash the whole system).
- Technology diversity (polyglot programming).
- Faster development cycles (teams own services).
- Aligns with agile and DevOps practices.

---

## ⚠️ Challenges

- **Complexity:** More moving parts (orchestration, monitoring, CI/CD).
- **Data consistency:** No global transactions; requires Saga/Event Sourcing.
- **Operational overhead:** Requires mature DevOps, monitoring, and automation.
- **Latency:** Network calls between services add overhead.
- **Testing:** End-to-end testing is harder.

---

## 📊 When to Use

- Large, complex applications needing scalability and resilience.
- Systems with multiple teams working independently.
- Applications requiring frequent updates and deployments.
- Cloud-native systems leveraging containers and orchestration (Kubernetes, Azure AKS).

---

## 🚫 When to Avoid

- Small applications or prototypes (monolith is simpler).
- Teams without DevOps maturity (monitoring, CI/CD, automation).
- Systems requiring strict immediate consistency across services.
- Environments with limited infrastructure or operational resources.

---

## 🛠️ Implementation in .NET & Azure

- **.NET Core Web APIs** for services.
- **Azure Kubernetes Service (AKS)** for orchestration.
- **Azure API Management** as API Gateway.
- **Azure Service Bus/Event Grid** for messaging.
- **Cosmos DB / SQL Database per service** for persistence.
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
- **Saga Pattern:** Ensures distributed transaction consistency across order, payment, and inventory.

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
- [ ]  Database per service (no shared DB).
- [ ]  API Gateway configured.
- [ ]  Resilience patterns (retry, circuit breaker).
- [ ]  Observability (logs, metrics, tracing).
- [ ]  Security (OAuth 2.0/OIDC, access-token validation, Microsoft Entra ID).
- [ ]  Documentation (C4 diagrams, ADRs).
- [ ]  Automated testing (unit, integration, end-to-end).

---

[^1]: 
