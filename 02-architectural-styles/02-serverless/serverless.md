# Serverless

The **Serverless architectural style** is a cloud computing execution model where applications run in stateless compute containers managed entirely by the cloud provider. Developers focus on writing business logic, while the provider handles provisioning, scaling, and maintenance.

## 📖 Definition

- **Serverless architecture** allows developers to build and run applications without managing servers.
- Despite the name, servers still exist — but they are **abstracted away** by the cloud provider.
- Code runs in **stateless, event-triggered functions** (e.g., AWS Lambda, Azure Functions, Google Cloud Functions).
- Infrastructure tasks like scaling, patching, and monitoring are handled automatically.

* * *

## 🎯 Core Characteristics

- **Event-driven:** Functions are triggered by events (HTTP requests, file uploads, database changes).
- **Stateless execution:** Each function invocation is independent; state must be externalized (e.g., databases, caches).
- **Automatic scaling:** Cloud provider scales functions up/down based on demand.
- **Pay-per-use:** Billing is based on execution time and resource usage, not idle servers.
- **Managed infrastructure:** No need to provision or maintain servers.

* * *

## 🏛 Scope

- **Application/system level.**
- Serverless is an **architectural style** for building entire applications or subsystems.
- Patterns (e.g., **Fan-out/Fan-in**, **Queue-based load leveling**) live inside serverless systems to solve specific problems.

* * *

## ⚙️ Key Components

- **Functions-as-a-Service (FaaS):** Core compute units (Azure Functions, AWS Lambda).
- **Backend-as-a-Service (BaaS):** Managed services like authentication (Microsoft Entra External ID), databases (Cosmos DB, DynamoDB), and storage (Azure Blob Storage).
- **Event Sources:** Triggers such as HTTP requests, message queues, file uploads.
- **Monitoring & Logging:** Cloud-native observability tools (Azure Monitor, AWS CloudWatch).

* * *

## 📚 Common Patterns in Serverless

- **Function Chaining:** Sequence of functions executing a workflow.
- **Fan-out/Fan-in:** Parallel execution of functions, then aggregation of results.
- **Queue-based Load Leveling:** Queue absorbs spikes, functions process messages at steady pace.
- **Event-driven Processing:** Functions react to domain events (e.g., file uploaded → process image).
- **API Gateway Integration:** Functions exposed via API Gateway for client access.

* * *

## ✅ Advantages

- **No server management:** Focus on business logic.
- **Cost efficiency:** Pay only for execution time.
- **Elastic scaling:** Automatically handles traffic spikes.
- **Rapid development:** Faster prototyping and deployment.
- **Integration with cloud services:** Easy to connect with databases, storage, messaging.

* * *

## ⚠️ Challenges

- **Cold starts:** Delay when functions are invoked after inactivity.
- **Statelessness:** Must externalize state (adds complexity).
- **Vendor lock-in:** Tied to provider’s ecosystem (Azure, AWS, GCP).
- **Debugging complexity:** Distributed, event-driven workflows are harder to trace.
- **Limited execution time:** Functions often have max runtime (e.g., 5–15 minutes).

* * *

## 📊 When to Use

- Event-driven applications (file processing, notifications).
- APIs and lightweight backends.
- Real-time data processing (IoT, analytics).
- Automation tasks (scheduled jobs, workflows).
- Prototypes and MVPs needing rapid iteration.

* * *

## 🚫 When to Avoid

- Long-running processes (batch jobs, video rendering).
- Applications requiring strict control over infrastructure.
- Systems needing persistent state within execution.
- Workloads with predictable, constant demand (may be cheaper with dedicated servers).

* * *

## 🛠️ Implementation in .NET & Azure

- **Azure Functions:** Core FaaS offering.
- **Azure Logic Apps:** Workflow automation.
- **Azure Event Grid / Service Bus:** Event-driven triggers.
- **Cosmos DB / Blob Storage:** External state management.
- **Azure API Management:** Expose functions as APIs.
- **Azure Monitor + Application Insights:** Observability.

* * *

## 📖 Example: Image Processing System

1.  User uploads image to Azure Blob Storage.
2.  Event Grid triggers Azure Function.
3.  Function processes image (resize, watermark).
4.  Result stored in Blob Storage.
5.  Notification sent via Service Bus.

* * *

## 📌 Checklist for Production Readiness

- [ ] Functions designed to be stateless and idempotent.
- [ ] External state managed in DB/cache.
- [ ] Cold start mitigation (provisioned concurrency).
- [ ] Monitoring and tracing enabled.
- [ ] Security (OAuth2, JWT, managed identities).
- [ ] Cost monitoring and optimization.
- [ ] Documentation of workflows and event sources.

* * *

**Summary:**

Serverless is an **architectural style** that abstracts infrastructure, enabling developers to focus on business logic. It’s ideal for **event-driven, scalable, cost-efficient applications**, but unsuitable for **long-running or stateful workloads**.

* * *

## References

- [Azure Functions overview — Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview)