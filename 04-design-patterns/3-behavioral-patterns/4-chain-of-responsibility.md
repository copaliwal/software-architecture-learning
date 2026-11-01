# Chain of Responsibility Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

---

## Problem

You need to process a request through multiple handlers, but:
- The sender shouldn't know which handler will process the request
- Multiple handlers might be capable of handling the request
- The set of handlers can change dynamically
- You want to avoid tight coupling between sender and receiver

Directly calling specific handlers creates tight coupling and inflexibility.

---

## Solution

Create a chain of handler objects. Each handler:
1. Has a reference to the next handler in the chain
2. Decides whether to process the request or pass it to the next handler
3. Can optionally continue passing after partial processing

The client initiates the request at the first handler in the chain.

---

## Structure

```
Handler (abstract)
├── SetNext(handler)
├── HandleRequest()
└── _nextHandler

ConcreteHandlerA
├── HandleRequest() - Process or pass to next

ConcreteHandlerB
├── HandleRequest() - Process or pass to next

Client
└── Sends request to first handler
```

---

## C# Implementation

### Example: Support Ticket System

```csharp
// Request
public class SupportTicket
{
    public string Issue { get; set; }
    public string Priority { get; set; }  // Low, Medium, High, Critical
    public string Description { get; set; }
    
    public SupportTicket(string issue, string priority, string description)
    {
        Issue = issue;
        Priority = priority;
        Description = description;
    }
}

// Abstract Handler
public abstract class SupportHandler
{
    protected SupportHandler? _nextHandler;
    
    public SupportHandler SetNext(SupportHandler handler)
    {
        _nextHandler = handler;
        return handler;  // Allows chaining: handler1.SetNext(handler2).SetNext(handler3)
    }
    
    public abstract void HandleTicket(SupportTicket ticket);
    
    protected void PassToNext(SupportTicket ticket)
    {
        if (_nextHandler != null)
        {
            _nextHandler.HandleTicket(ticket);
        }
        else
        {
            Console.WriteLine($"No handler available for ticket: {ticket.Issue}");
        }
    }
}

// Concrete Handler 1 - Level 1 Support
public class Level1Support : SupportHandler
{
    public override void HandleTicket(SupportTicket ticket)
    {
        if (ticket.Priority == "Low")
        {
            Console.WriteLine($"[Level 1 Support] Handling: {ticket.Issue}");
            Console.WriteLine($"  Resolution: {ticket.Description}");
        }
        else
        {
            Console.WriteLine($"[Level 1 Support] Escalating ticket: {ticket.Issue}");
            PassToNext(ticket);
        }
    }
}

// Concrete Handler 2 - Level 2 Support
public class Level2Support : SupportHandler
{
    public override void HandleTicket(SupportTicket ticket)
    {
        if (ticket.Priority == "Medium")
        {
            Console.WriteLine($"[Level 2 Support] Handling: {ticket.Issue}");
            Console.WriteLine($"  Resolution: {ticket.Description}");
        }
        else
        {
            Console.WriteLine($"[Level 2 Support] Escalating ticket: {ticket.Issue}");
            PassToNext(ticket);
        }
    }
}

// Concrete Handler 3 - Senior Support
public class SeniorSupport : SupportHandler
{
    public override void HandleTicket(SupportTicket ticket)
    {
        if (ticket.Priority == "High")
        {
            Console.WriteLine($"[Senior Support] Handling: {ticket.Issue}");
            Console.WriteLine($"  Resolution: {ticket.Description}");
        }
        else
        {
            Console.WriteLine($"[Senior Support] Escalating to manager: {ticket.Issue}");
            PassToNext(ticket);
        }
    }
}

// Concrete Handler 4 - Manager
public class ManagerSupport : SupportHandler
{
    public override void HandleTicket(SupportTicket ticket)
    {
        // Manager handles everything that reaches here
        Console.WriteLine($"[Manager] Handling critical ticket: {ticket.Issue}");
        Console.WriteLine($"  Priority: {ticket.Priority}");
        Console.WriteLine($"  Description: {ticket.Description}");
    }
}

// Usage
var level1 = new Level1Support();
var level2 = new Level2Support();
var senior = new SeniorSupport();
var manager = new ManagerSupport();

// Build the chain
level1.SetNext(level2)
      .SetNext(senior)
      .SetNext(manager);

// Test tickets
var tickets = new[]
{
    new SupportTicket("Password reset", "Low", "User forgot password"),
    new SupportTicket("Application crash", "Medium", "App crashes on startup"),
    new SupportTicket("Performance issue", "High", "System running slow"),
    new SupportTicket("Security breach", "Critical", "Unauthorized access detected")
};

foreach (var ticket in tickets)
{
    Console.WriteLine($"\n--- Processing Ticket: {ticket.Issue} ({ticket.Priority}) ---");
    level1.HandleTicket(ticket);
}

// Output:
// --- Processing Ticket: Password reset (Low) ---
// [Level 1 Support] Handling: Password reset
//   Resolution: User forgot password
// 
// --- Processing Ticket: Application crash (Medium) ---
// [Level 1 Support] Escalating ticket: Application crash
// [Level 2 Support] Handling: Application crash
//   Resolution: App crashes on startup
// 
// --- Processing Ticket: Performance issue (High) ---
// [Level 1 Support] Escalating ticket: Performance issue
// [Level 2 Support] Escalating ticket: Performance issue
// [Senior Support] Handling: Performance issue
//   Resolution: System running slow
// 
// --- Processing Ticket: Security breach (Critical) ---
// [Level 1 Support] Escalating ticket: Security breach
// [Level 2 Support] Escalating ticket: Security breach
// [Senior Support] Escalating to manager: Security breach
// [Manager] Handling critical ticket: Security breach
```

---

## ASP.NET Core Middleware Example

```csharp
// ASP.NET Core middleware IS the Chain of Responsibility pattern!

public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        // Each middleware is a handler in the chain
        app.UseExceptionHandler("/error");  // Handler 1
        app.UseHttpsRedirection();          // Handler 2
        app.UseAuthentication();            // Handler 3
        app.UseAuthorization();             // Handler 4
        app.UseRouting();                   // Handler 5
        app.UseEndpoints(endpoints =>       // Handler 6
        {
            endpoints.MapControllers();
        });
    }
}

// Custom middleware example
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;
    
    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;  // Next handler in chain
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation($"Request: {context.Request.Method} {context.Request.Path}");
        
        // Process and pass to next handler
        await _next(context);
        
        _logger.LogInformation($"Response: {context.Response.StatusCode}");
    }
}

public class AuthenticationMiddleware
{
    private readonly RequestDelegate _next;
    
    public AuthenticationMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var token = context.Request.Headers["Authorization"].FirstOrDefault();
        
        if (string.IsNullOrEmpty(token))
        {
            context.Response.StatusCode = 401;
            await context.Response.WriteAsync("Unauthorized");
            return;  // Stop chain here
        }
        
        // Valid token - continue chain
        await _next(context);
    }
}

public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private static readonly Dictionary<string, (int Count, DateTime Reset)> _requests = new();
    private const int MaxRequests = 100;
    private static readonly TimeSpan TimeWindow = TimeSpan.FromMinutes(1);
    
    public RateLimitingMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var clientIp = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
        
        if (_requests.ContainsKey(clientIp))
        {
            var (count, reset) = _requests[clientIp];
            
            if (DateTime.Now < reset)
            {
                if (count >= MaxRequests)
                {
                    context.Response.StatusCode = 429;
                    await context.Response.WriteAsync("Too many requests");
                    return;  // Stop chain
                }
                
                _requests[clientIp] = (count + 1, reset);
            }
            else
            {
                _requests[clientIp] = (1, DateTime.Now.Add(TimeWindow));
            }
        }
        else
        {
            _requests[clientIp] = (1, DateTime.Now.Add(TimeWindow));
        }
        
        await _next(context);
    }
}

// Register middleware in order
app.UseMiddleware<RequestLoggingMiddleware>();
app.UseMiddleware<RateLimitingMiddleware>();
app.UseMiddleware<AuthenticationMiddleware>();
```

---

## Approval Workflow Example

```csharp
// Request
public class PurchaseRequest
{
    public string Item { get; set; }
    public decimal Amount { get; set; }
    public string Requestor { get; set; }
    
    public PurchaseRequest(string item, decimal amount, string requestor)
    {
        Item = item;
        Amount = amount;
        Requestor = requestor;
    }
}

// Abstract Approver
public abstract class Approver
{
    protected Approver? _nextApprover;
    protected decimal _approvalLimit;
    
    public Approver(decimal approvalLimit)
    {
        _approvalLimit = approvalLimit;
    }
    
    public void SetNext(Approver approver)
    {
        _nextApprover = approver;
    }
    
    public abstract void ProcessRequest(PurchaseRequest request);
}

// Team Lead - Can approve up to $1,000
public class TeamLead : Approver
{
    public TeamLead() : base(1000) { }
    
    public override void ProcessRequest(PurchaseRequest request)
    {
        if (request.Amount <= _approvalLimit)
        {
            Console.WriteLine($"[Team Lead] Approved ${request.Amount} for {request.Item}");
        }
        else if (_nextApprover != null)
        {
            Console.WriteLine($"[Team Lead] Cannot approve ${request.Amount}. Forwarding to manager...");
            _nextApprover.ProcessRequest(request);
        }
    }
}

// Manager - Can approve up to $10,000
public class Manager : Approver
{
    public Manager() : base(10000) { }
    
    public override void ProcessRequest(PurchaseRequest request)
    {
        if (request.Amount <= _approvalLimit)
        {
            Console.WriteLine($"[Manager] Approved ${request.Amount} for {request.Item}");
        }
        else if (_nextApprover != null)
        {
            Console.WriteLine($"[Manager] Cannot approve ${request.Amount}. Forwarding to director...");
            _nextApprover.ProcessRequest(request);
        }
    }
}

// Director - Can approve up to $100,000
public class Director : Approver
{
    public Director() : base(100000) { }
    
    public override void ProcessRequest(PurchaseRequest request)
    {
        if (request.Amount <= _approvalLimit)
        {
            Console.WriteLine($"[Director] Approved ${request.Amount} for {request.Item}");
        }
        else if (_nextApprover != null)
        {
            Console.WriteLine($"[Director] Cannot approve ${request.Amount}. Forwarding to CEO...");
            _nextApprover.ProcessRequest(request);
        }
    }
}

// CEO - Can approve anything
public class CEO : Approver
{
    public CEO() : base(decimal.MaxValue) { }
    
    public override void ProcessRequest(PurchaseRequest request)
    {
        Console.WriteLine($"[CEO] Approved ${request.Amount} for {request.Item}");
    }
}

// Usage
var teamLead = new TeamLead();
var manager = new Manager();
var director = new Director();
var ceo = new CEO();

// Build chain
teamLead.SetNext(manager);
manager.SetNext(director);
director.SetNext(ceo);

// Test requests
var requests = new[]
{
    new PurchaseRequest("Office supplies", 500, "John"),
    new PurchaseRequest("New laptops", 5000, "Sarah"),
    new PurchaseRequest("Server hardware", 50000, "Mike"),
    new PurchaseRequest("Company acquisition", 5000000, "Board")
};

foreach (var request in requests)
{
    Console.WriteLine($"\n--- Request: {request.Item} (${request.Amount}) ---");
    teamLead.ProcessRequest(request);
}
```

---

## Logging Pipeline Example

```csharp
public enum LogLevel
{
    Debug,
    Info,
    Warning,
    Error,
    Critical
}

public class LogMessage
{
    public LogLevel Level { get; set; }
    public string Message { get; set; }
    public DateTime Timestamp { get; set; } = DateTime.Now;
}

// Abstract Logger
public abstract class Logger
{
    protected Logger? _nextLogger;
    protected LogLevel _logLevel;
    
    protected Logger(LogLevel logLevel)
    {
        _logLevel = logLevel;
    }
    
    public Logger SetNext(Logger logger)
    {
        _nextLogger = logger;
        return logger;
    }
    
    public void Log(LogMessage message)
    {
        if (message.Level >= _logLevel)
        {
            Write(message);
        }
        
        _nextLogger?.Log(message);  // Always pass to next (multiple handlers can process)
    }
    
    protected abstract void Write(LogMessage message);
}

// Console Logger
public class ConsoleLogger : Logger
{
    public ConsoleLogger(LogLevel level) : base(level) { }
    
    protected override void Write(LogMessage message)
    {
        var color = message.Level switch
        {
            LogLevel.Debug => ConsoleColor.Gray,
            LogLevel.Info => ConsoleColor.White,
            LogLevel.Warning => ConsoleColor.Yellow,
            LogLevel.Error => ConsoleColor.Red,
            LogLevel.Critical => ConsoleColor.DarkRed,
            _ => ConsoleColor.White
        };
        
        Console.ForegroundColor = color;
        Console.WriteLine($"[Console] {message.Timestamp:HH:mm:ss} [{message.Level}] {message.Message}");
        Console.ResetColor();
    }
}

// File Logger
public class FileLogger : Logger
{
    private readonly string _filePath;
    
    public FileLogger(LogLevel level, string filePath) : base(level)
    {
        _filePath = filePath;
    }
    
    protected override void Write(LogMessage message)
    {
        var logEntry = $"{message.Timestamp:yyyy-MM-dd HH:mm:ss} [{message.Level}] {message.Message}";
        File.AppendAllText(_filePath, logEntry + Environment.NewLine);
        Console.WriteLine($"[File] Logged to {_filePath}");
    }
}

// Email Logger (for critical errors)
public class EmailLogger : Logger
{
    private readonly string _emailTo;
    
    public EmailLogger(LogLevel level, string emailTo) : base(level)
    {
        _emailTo = emailTo;
    }
    
    protected override void Write(LogMessage message)
    {
        // Simulate sending email
        Console.WriteLine($"[Email] Sending alert to {_emailTo}");
        Console.WriteLine($"  Subject: {message.Level} Alert");
        Console.WriteLine($"  Body: {message.Message}");
    }
}

// Usage
var consoleLogger = new ConsoleLogger(LogLevel.Debug);
var fileLogger = new FileLogger(LogLevel.Warning, "app.log");
var emailLogger = new EmailLogger(LogLevel.Critical, "admin@company.com");

// Build chain - all loggers process messages
consoleLogger.SetNext(fileLogger)
             .SetNext(emailLogger);

// Test different log levels
consoleLogger.Log(new LogMessage { Level = LogLevel.Debug, Message = "Debug message" });
consoleLogger.Log(new LogMessage { Level = LogLevel.Info, Message = "Info message" });
consoleLogger.Log(new LogMessage { Level = LogLevel.Warning, Message = "Warning message" });
consoleLogger.Log(new LogMessage { Level = LogLevel.Error, Message = "Error occurred" });
consoleLogger.Log(new LogMessage { Level = LogLevel.Critical, Message = "System failure!" });
```

---

## When to Use Chain of Responsibility

✅ **Use when:**
- Multiple objects can handle a request
- Handler is not known in advance
- Set of handlers changes dynamically
- Request should be handled by one OR multiple handlers
- Want to decouple sender from receivers

❌ **Avoid when:**
- Only one handler is appropriate
- Request must be handled (no guarantee in CoR)
- Performance is critical (chain traversal has overhead)

---

## Advantages

✅ **Decoupling** - Sender doesn't know which handler processes request  
✅ **Flexibility** - Add/remove handlers dynamically  
✅ **Single Responsibility** - Each handler has one job  
✅ **Open/Closed** - Add handlers without modifying existing code

---

## Disadvantages

❌ **No guarantee** - Request might not be handled  
❌ **Debugging difficulty** - Hard to trace through chain  
❌ **Performance** - Request may traverse entire chain  
❌ **Configuration complexity** - Setting up chain correctly

---

## Chain Variants

| Variant | Behavior | Example |
|---------|----------|---------|
| **Pure CoR** | Only one handler processes | Approval workflow |
| **Impure CoR** | Multiple handlers can process | Logging pipeline |
| **Conditional** | Handler decides to pass or not | Middleware |

---

## Related Patterns

- **Command** - Both encapsulate requests; CoR focuses on passing through handlers
- **Decorator** - Similar structure but decorator adds behavior; CoR handles request
- **Composite** - Can be used to implement chain as tree structure

---

## Common Use Cases

- **Middleware** - ASP.NET Core, Express.js
- **Event handling** - UI event bubbling
- **Approval workflows** - Purchase orders, expense reports
- **Validation** - Multiple validation rules
- **Logging** - Multiple log destinations
- **Authentication/Authorization** - Multiple auth mechanisms

---

## Best Practices

✅ Provide **default handler** at end of chain  
✅ Make handlers **stateless** when possible  
✅ Use **fluent interface** for chain building  
✅ Document **chain order** clearly  
✅ Consider **breaking chain** when request is fully handled  
✅ Log when **request reaches end** without being handled