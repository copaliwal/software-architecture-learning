# SOLID Principles

The **SOLID** principles are five object-oriented design principles that make software designs more understandable, flexible, and maintainable. They were promoted by Robert C. Martin (Uncle Bob).

---

## 1. **S - Single Responsibility Principle (SRP)**

**Definition:** A class should have only one reason to change, meaning it should have only one job or responsibility.

**Why?** When a class has multiple responsibilities, changes to one responsibility can affect the others, making the code fragile and hard to maintain.

### 🎯 Architect-level clarification: "one reason to change" means *one actor*

 "One responsibility" is the most misapplied phrase in software design — taken literally it leads
 to classes with a single method and a codebase nobody can navigate.

 **Robert C. Martin's own clarification:** a "reason to change" means **a person or group who
 requests the change** — an *actor*. The principle is really:

 *A module should be responsible to one, and only one, actor.*

 **Why this reframing matters.** Consider a payroll `Employee` class with `CalculatePay()`,
 `ReportHours()`, and `Save()`. Three methods, three actors:

| Method | Actor |
|---|---|
| `CalculatePay()` | **Finance** (the CFO's rules) |
| `ReportHours()` | **HR** (the COO's reports) |
| `Save()` | **DBA / architecture** |

 They're all "about an employee", so a naive reading says one responsibility. But when Finance
 changes the overtime formula, that edit sits in a class HR also depends on — and you can break
 HR's reports while satisfying Finance. **The coupling is organisational, not technical.**

 **The interview version:** *"SRP isn't about how many methods a class has — it's about how many
 stakeholders can force it to change. If two different departments can each demand an edit to the
 same class, that class has two responsibilities regardless of how cohesive it looks."*

 This is a frequent probe, because "one job" is the answer everyone gives and "one actor" is the
 answer that shows you've read past the acronym.
### ❌ Bad Example (Violates SRP)

```csharp
public class UserService
{
    public void RegisterUser(User user)
    {
        // Validate user
        if (string.IsNullOrEmpty(user.Email))
            throw new Exception("Email is required");
        
        // Save to database
        using (var connection = new SqlConnection("connectionString"))
        {
            // Database logic
        }
        
        // Send email
        var emailClient = new SmtpClient();
        emailClient.Send("welcome@example.com", user.Email, "Welcome!", "Thank you for registering");
    }
}
```

**Problems:** This class handles validation, database operations, AND email sending. Changes to any of these will require modifying this class.

### ✅ Good Example (Follows SRP)

```csharp
// Each class has a single responsibility
public class UserValidator
{
    public void Validate(User user)
    {
        if (string.IsNullOrEmpty(user.Email))
            throw new ValidationException("Email is required");
    }
}

public class UserRepository
{
    public void Save(User user)
    {
        using (var connection = new SqlConnection("connectionString"))
        {
            // Database logic
        }
    }
}

public class EmailService
{
    public void SendWelcomeEmail(User user)
    {
        var emailClient = new SmtpClient();
        emailClient.Send("welcome@example.com", user.Email, "Welcome!", "Thank you for registering");
    }
}

public class UserRegistrationService
{
    private readonly UserValidator _validator;
    private readonly UserRepository _repository;
    private readonly EmailService _emailService;

    public UserRegistrationService(UserValidator validator, UserRepository repository, EmailService emailService)
    {
        _validator = validator;
        _repository = repository;
        _emailService = emailService;
    }

    public void RegisterUser(User user)
    {
        _validator.Validate(user);
        _repository.Save(user);
        _emailService.SendWelcomeEmail(user);
    }
}
```

**Benefits:** Each class now has one reason to change. Validation logic changes don't affect email sending, etc.

---

## 2. **O - Open/Closed Principle (OCP)**

**Definition:** Software entities (classes, modules, functions) should be **open for extension** but **closed for modification**.

**Why?** You should be able to add new functionality without changing existing code, reducing the risk of breaking existing features.

### ❌ Bad Example (Violates OCP)

```csharp
public class DiscountCalculator
{
    public decimal Calculate(decimal amount, string customerType)
    {
        if (customerType == "Regular")
            return amount * 0.95m; // 5% discount
        else if (customerType == "Premium")
            return amount * 0.90m; // 10% discount
        else if (customerType == "VIP")
            return amount * 0.80m; // 20% discount
        
        return amount;
    }
}
```

**Problem:** Adding a new customer type requires modifying the existing `Calculate` method.

### ✅ Good Example (Follows OCP)

```csharp
// Abstract base
public abstract class CustomerDiscount
{
    public abstract decimal ApplyDiscount(decimal amount);
}

// Concrete implementations
public class RegularCustomerDiscount : CustomerDiscount
{
    public override decimal ApplyDiscount(decimal amount)
    {
        return amount * 0.95m; // 5% discount
    }
}

public class PremiumCustomerDiscount : CustomerDiscount
{
    public override decimal ApplyDiscount(decimal amount)
    {
        return amount * 0.90m; // 10% discount
    }
}

public class VIPCustomerDiscount : CustomerDiscount
{
    public override decimal ApplyDiscount(decimal amount)
    {
        return amount * 0.80m; // 20% discount
    }
}

// Calculator uses abstraction
public class DiscountCalculator
{
    public decimal Calculate(decimal amount, CustomerDiscount discount)
    {
        return discount.ApplyDiscount(amount);
    }
}

// Usage
var calculator = new DiscountCalculator();
var amount = calculator.Calculate(100m, new VIPCustomerDiscount());
```

**Benefits:** To add a new customer type (e.g., "Gold"), you just create a new class. No need to modify existing code.

---

## 3. **L - Liskov Substitution Principle (LSP)**

**Definition:** Objects of a superclass should be replaceable with objects of a subclass without breaking the application.

**Why?** Ensures that derived classes extend base classes without changing their behavior in unexpected ways.

> ### 🎯 Architect-level: LSP is a statement about **contracts**, not shapes
>
> Rectangle/Square (below) is the example *every* candidate gives. It demonstrates the violation
> but doesn't give you a **test** for it. The contract formulation does — a subtype must:
>
> | Rule | Meaning | Violation looks like |
> |---|---|---|
> | **Preconditions may not be strengthened** | The subtype can't demand *more* of callers than the base did | Base accepts any `int`; override throws on negatives |
> | **Postconditions may not be weakened** | The subtype must deliver *at least* what the base promised | Base guarantees a sorted result; override returns unsorted |
> | **Invariants must be preserved** | Base-class truths stay true | Base guarantees `Balance >= 0`; subtype allows overdraft |
> | **History constraint** | The subtype must not allow state changes the base forbids | Mutating a field the base treats as immutable |
> | **Exceptions** | No new exception types callers weren't told to expect | `NotImplementedException` in an override ⚠️ |
>
> **The practical smell:** if calling code needs `if (x is SomeSubtype)` or a `try/catch` for a
> subtype-specific exception, LSP is already broken. The abstraction is lying about what it
> promises.
>
> **The .NET example everyone has actually hit:** `ReadOnlyCollection<T>` implements `IList<T>`,
> whose `Add` is in the contract — but `ReadOnlyCollection<T>.Add` throws
> `NotSupportedException`. That is a genuine LSP violation **in the BCL**, and it's why
> `IReadOnlyList<T>` was later introduced. Citing this shows you can see the principle in real
> code rather than in shape puzzles.
>
> **Interview line:** *"LSP is about behavioural substitutability, not inheritance hierarchy. The
> test isn't 'is a Square a Rectangle' — it's 'can I swap the subtype in without the caller
> needing to know'. If a caller has to type-check, the subtype broke the contract."*

### ❌ Bad Example (Violates LSP)

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    
    public int GetArea()
    {
        return Width * Height;
    }
}

public class Square : Rectangle
{
    public override int Width
    {
        get => base.Width;
        set
        {
            base.Width = value;
            base.Height = value; // Violates LSP!
        }
    }
    
    public override int Height
    {
        get => base.Height;
        set
        {
            base.Width = value; // Violates LSP!
            base.Height = value;
        }
    }
}

// Usage
Rectangle rect = new Square();
rect.Width = 5;
rect.Height = 10;
Console.WriteLine(rect.GetArea()); // Expected: 50, Actual: 100!
```

**Problem:** A `Square` is-a `Rectangle` mathematically, but in code, the behavior is different. Setting width/height independently doesn't work as expected.

### ✅ Good Example (Follows LSP)

```csharp
// Common abstraction
public interface IShape
{
    int GetArea();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }
    
    public int GetArea()
    {
        return Width * Height;
    }
}

public class Square : IShape
{
    public int Side { get; set; }
    
    public int GetArea()
    {
        return Side * Side;
    }
}

// Usage
IShape shape1 = new Rectangle { Width = 5, Height = 10 };
IShape shape2 = new Square { Side = 5 };

Console.WriteLine(shape1.GetArea()); // 50
Console.WriteLine(shape2.GetArea()); // 25
```

**Benefits:** Both `Rectangle` and `Square` can be used interchangeably through `IShape` without unexpected behavior.

---

## 4. **I - Interface Segregation Principle (ISP)**

**Definition:** No client should be forced to depend on methods it does not use. Split large interfaces into smaller, more specific ones.

**Why?** Prevents classes from implementing unnecessary methods, reducing coupling and improving clarity.

### ❌ Bad Example (Violates ISP)

```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Human : IWorker
{
    public void Work() { /* ... */ }
    public void Eat() { /* ... */ }
    public void Sleep() { /* ... */ }
}

public class Robot : IWorker
{
    public void Work() { /* ... */ }
    public void Eat() { throw new NotImplementedException(); } // Robots don't eat!
    public void Sleep() { throw new NotImplementedException(); } // Robots don't sleep!
}
```

**Problem:** `Robot` is forced to implement `Eat()` and `Sleep()` even though they don't make sense.

### ✅ Good Example (Follows ISP)

```csharp
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work() { /* ... */ }
    public void Eat() { /* ... */ }
    public void Sleep() { /* ... */ }
}

public class Robot : IWorkable
{
    public void Work() { /* ... */ }
    // No need to implement Eat or Sleep
}
```

**Benefits:** Each class only implements the interfaces it needs.

---

## 5. **D - Dependency Inversion Principle (DIP)**

**Definition:** 
- High-level modules should not depend on low-level modules. Both should depend on abstractions.
- Abstractions should not depend on details. Details should depend on abstractions.

**Why?** Reduces coupling between components, making the system more flexible and testable.

### ❌ Bad Example (Violates DIP)

```csharp
public class EmailService
{
    public void SendEmail(string message)
    {
        // Send email logic
    }
}

public class Notification
{
    private EmailService _emailService = new EmailService(); // Direct dependency!
    
    public void Send(string message)
    {
        _emailService.SendEmail(message);
    }
}
```

**Problem:** `Notification` is tightly coupled to `EmailService`. If you want to send SMS instead, you need to modify `Notification`.

### ✅ Good Example (Follows DIP)

```csharp
// Abstraction
public interface IMessageService
{
    void Send(string message);
}

// Low-level modules
public class EmailService : IMessageService
{
    public void Send(string message)
    {
        // Send email logic
    }
}

public class SmsService : IMessageService
{
    public void Send(string message)
    {
        // Send SMS logic
    }
}

// High-level module depends on abstraction
public class Notification
{
    private readonly IMessageService _messageService;
    
    public Notification(IMessageService messageService)
    {
        _messageService = messageService;
    }
    
    public void Send(string message)
    {
        _messageService.Send(message);
    }
}

// Usage with Dependency Injection
var emailNotification = new Notification(new EmailService());
emailNotification.Send("Hello via Email");

var smsNotification = new Notification(new SmsService());
smsNotification.Send("Hello via SMS");
```

**Benefits:** `Notification` doesn't care about the concrete implementation. You can easily swap `EmailService` for `SmsService` or add new services without modifying `Notification`.

> ### ⚠️ Architect-level: DIP is **not** dependency injection — and this is a deliberate interview trap
>
> The example above is correct, but on its own it demonstrates **dependency injection**, a
> *technique*. DIP is a **principle** about direction of ownership, and the two are routinely
> conflated. Interviewers ask precisely because the confusion is so common.
>
> | | Dependency **Injection** | Dependency **Inversion** |
> |---|---|---|
> | Kind of thing | Technique / mechanism | Design principle |
> | Question it answers | *How* does a class receive its collaborators? | *Which module owns the abstraction?* |
> | Achieved by | Constructor/property injection, a DI container | Placing the interface with the **consumer**, not the implementer |
>
> **You can do DI and still violate DIP.** Watch the direction:
>
> ```csharp
> // ❌ DI, but NOT dependency inversion.
> // IMessageService is DEFINED in the Infrastructure layer, next to its implementation.
> // Domain must reference Infrastructure to see the interface — the dependency still
> // points from high-level policy DOWN to low-level detail. Nothing was inverted.
> namespace MyApp.Infrastructure.Email;
> public interface IMessageService { void Send(string message); }
> public class SmtpService : IMessageService { /* ... */ }
>
> namespace MyApp.Domain;
> using MyApp.Infrastructure.Email;   // ⚠️ domain → infrastructure. Wrong direction.
> public class Notification(IMessageService svc) { /* injected, but not inverted */ }
> ```
>
> ```csharp
> // ✅ Genuine dependency INVERSION.
> // The interface lives with the CONSUMER that needs it. The domain owns the contract.
> namespace MyApp.Domain;
> public interface IMessageService { void Send(string message); }  // owned by the domain
> public class Notification(IMessageService svc) { /* ... */ }
>
> namespace MyApp.Infrastructure.Email;
> using MyApp.Domain;                 // ✅ infrastructure → domain. Inverted.
> public class SmtpService : IMessageService { /* ... */ }
> ```
>
> Both compile. Both use DI. Only the second **inverts** the dependency: at build time the arrow
> now points *inward*, from detail to policy. That reversal is the entire content of the "I" in
> SOLID — and it is exactly what makes **Clean / Onion / Hexagonal architecture** possible. Those
> styles are DIP applied at the architectural scale: the domain defines the ports, infrastructure
> implements them, and the domain project references nothing outward.
>
> **The tell:** if your `Domain` project has a `using` for your `Infrastructure` project, you have
> DI without DIP, no matter how many interfaces you have.
>
> **Interview line:** *"DI is how an object gets its dependencies. DIP is about who owns the
> abstraction — the high-level module defines the interface it needs, and the low-level module
> implements it. You can have a container wiring up a hundred interfaces and still violate DIP if
> those interfaces live in the infrastructure layer."*

---

## Summary Table

| Principle | Key Question | Goal |
|-----------|-------------|------|
| **Single Responsibility** | Does this class have only one reason to change? | One class, one job |
| **Open/Closed** | Can I add new functionality without modifying existing code? | Extend, don't modify |
| **Liskov Substitution** | Can I replace a parent class with a child class without issues? | Maintain expected behavior |
| **Interface Segregation** | Does this interface force clients to implement unused methods? | Small, focused interfaces |
| **Dependency Inversion** | Do high-level modules depend on abstractions or concrete classes? | Depend on abstractions |

---

## Benefits of Following SOLID

✅ **Maintainability** - Easier to understand and modify  
✅ **Testability** - Components can be tested in isolation  
✅ **Flexibility** - Easy to extend and adapt to new requirements  
✅ **Reduced Coupling** - Components are less dependent on each other  
✅ **Better Code Organization** - Clear separation of concerns

---

## Real-World Application in .NET

- **ASP.NET Core Dependency Injection** — the *mechanism* for DI. It enables DIP but does **not** guarantee it: inversion depends on **where you define the interface** (see the DIP callout above).
- **Repository Pattern** — SRP (separating data access) — though see the caution below on generic repositories over EF Core.
- **Strategy Pattern** — OCP (adding new strategies without modifying existing code)
- **Interface-based Programming** — ISP and DIP combined
- **Clean Architecture / Onion Architecture** — **DIP applied at architectural scale**: the domain defines ports, infrastructure implements them, and the dependency arrow points inward.

---

## ⚠️ When SOLID becomes over-engineering

**A senior candidate who cannot name the cost of these principles reads as dogmatic.** SOLID is a
set of heuristics for managing *change*. Where change is unlikely, applying them adds indirection
and buys nothing.

| Over-application | What it looks like | Better |
|---|---|---|
| **SRP taken to extremes** | Dozens of one-method classes; understanding one feature means opening fifteen files | Cohesion is a virtue too. Group by *actor* and reason to change, not by counting methods |
| **OCP speculation** | Abstractions and strategy hierarchies built for variation that never arrives | **YAGNI wins.** Add the abstraction on the *second* concrete case, when you can see the real axis of variation |
| **ISP shredding** | Every interface has one method; no coherent contract remains | Split when a *real* client is forced to depend on what it doesn't use — not preemptively |
| **DIP everywhere** | An interface with exactly one implementation, forever, for every class | Abstractions have a carrying cost. `SqlOrderRepository` with no second implementation and no test seam is pure indirection |
| **Generic `IRepository<T>` over EF Core** | A repository wrapping `DbSet<T>` and leaking `IQueryable` | `DbContext` **is** a Unit of Work and `DbSet<T>` **is** a repository. Wrapping them usually hides EF's capabilities and adds nothing testable |

### The two questions to apply before abstracting

1. **What change am I insulating against, and how likely is it?** If you can't name the change, the
   abstraction is speculative.
2. **What does this cost a reader?** Every interface adds a hop between intent and implementation.
   Indirection is not free — it is paid on every future read of the code.

**Interview line:** *"SOLID manages the cost of change, so it's only worth paying where change is
likely. I apply it hardest at boundaries — domain to infrastructure, module to module — and much
more lightly inside a stable module. An interface with one implementation and no test seam is
usually indirection I'll pay for on every read and never collect on."*

Saying this unprompted is a strong seniority signal: it shows you treat principles as **trade-offs
with costs**, not commandments.

---

## Interview questions

### Q1. What does "one reason to change" actually mean?
**Evaluating:** whether you've read past the acronym.
**Strong answer:** One **actor** — one person or group who can request a change. Two departments
able to force edits to the same class means two responsibilities, however cohesive it looks. It's
about organisational coupling, not method count.

### Q2. Give an LSP violation from a real library, not Rectangle/Square.
**Strong answer:** `ReadOnlyCollection<T>` implements `IList<T>` but throws `NotSupportedException`
from `Add`. Callers holding an `IList<T>` can't substitute it safely. It's why `IReadOnlyList<T>`
exists. The general test: if callers must type-check or catch subtype-specific exceptions, the
contract is broken.

### Q3. Is dependency injection the same as dependency inversion?
**Strong answer:** No — and this is the trap. DI is a technique for supplying collaborators;
DIP is a principle about which module **owns the abstraction**. You can inject a hundred interfaces
and still violate DIP if they're all defined in the infrastructure layer, because the dependency
still points from policy to detail. Inversion happens when the consumer defines the contract.

### Q4. When would you *not* apply SOLID?
**Strong answer:** When the change it insulates against is unlikely. OCP abstractions built for
variation that never arrives are pure cost; YAGNI generally beats speculative OCP, so I add the
abstraction on the second concrete case. A generic repository over EF Core is the standard example
of the anti-pattern — `DbContext` is already a Unit of Work. I apply SOLID hardest at boundaries and
lightly inside stable modules.

---

## Flashcards

**Q:** SRP — "one reason to change" means what, precisely?
**A:** One **actor** (person/group who can request the change), not one method or one job.

**Q:** The four LSP contract rules?
**A:** Preconditions not strengthened; postconditions not weakened; invariants preserved; no surprising new exceptions.

**Q:** DI vs DIP?
**A:** DI = how an object receives collaborators (technique). DIP = the consumer owns the abstraction (principle). DI without DIP is possible and common.

**Q:** Where must the interface live for genuine dependency inversion?
**A:** With the **high-level consumer** (the domain), not next to the implementation.

**Q:** Which architecture styles are DIP at scale?
**A:** Clean, Onion, Hexagonal — domain defines ports, infrastructure implements, dependencies point inward.

**Q:** Most common SOLID over-application in .NET?
**A:** Generic `IRepository<T>` over EF Core — `DbContext` is already a Unit of Work and `DbSet<T>` a repository.