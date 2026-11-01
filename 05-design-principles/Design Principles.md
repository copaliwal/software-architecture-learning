# Design Principles

Comprehensive guide to software design principles that lead to maintainable, scalable, and clean code.

---

## SOLID Principles

See detailed explanation in [SOLID.md](SOLID/SOLID.md)

- **S:** Single Responsibility Principle
- **O:** Open/Closed Principle
- **L:** Liskov Substitution Principle
- **I:** Interface Segregation Principle
- **D:** Dependency Inversion Principle

---

## DRY - Don't Repeat Yourself

**Principle:** Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

**Why?** Code duplication leads to maintenance nightmares. When you need to fix a bug or change logic, you have to update it in multiple places.

### ❌ Bad Example (Violates DRY)

```csharp
public class UserController
{
    public IActionResult RegisterUser(UserDto dto)
    {
        // Validation logic
        if (string.IsNullOrEmpty(dto.Email))
            return BadRequest("Email is required");
        if (!dto.Email.Contains("@"))
            return BadRequest("Invalid email format");
        if (dto.Password.Length < 8)
            return BadRequest("Password must be at least 8 characters");
        
        // Register user...
    }
    
    public IActionResult UpdateUser(UserDto dto)
    {
        // Same validation logic repeated!
        if (string.IsNullOrEmpty(dto.Email))
            return BadRequest("Email is required");
        if (!dto.Email.Contains("@"))
            return BadRequest("Invalid email format");
        if (dto.Password.Length < 8)
            return BadRequest("Password must be at least 8 characters");
        
        // Update user...
    }
}
```

### ✅ Good Example (Follows DRY)

```csharp
public class UserValidator
{
    public ValidationResult Validate(UserDto dto)
    {
        var errors = new List<string>();
        
        if (string.IsNullOrEmpty(dto.Email))
            errors.Add("Email is required");
        else if (!dto.Email.Contains("@"))
            errors.Add("Invalid email format");
        
        if (dto.Password?.Length < 8)
            errors.Add("Password must be at least 8 characters");
        
        return new ValidationResult 
        { 
            IsValid = !errors.Any(), 
            Errors = errors 
        };
    }
}

public class UserController
{
    private readonly UserValidator _validator;
    
    public UserController(UserValidator validator)
    {
        _validator = validator;
    }
    
    public IActionResult RegisterUser(UserDto dto)
    {
        var validationResult = _validator.Validate(dto);
        if (!validationResult.IsValid)
            return BadRequest(validationResult.Errors);
        
        // Register user...
    }
    
    public IActionResult UpdateUser(UserDto dto)
    {
        var validationResult = _validator.Validate(dto);
        if (!validationResult.IsValid)
            return BadRequest(validationResult.Errors);
        
        // Update user...
    }
}
```

### Real-World Example: Configuration

```csharp
// ❌ Bad - Database connection string repeated
public class OrderRepository
{
    public void SaveOrder()
    {
        using var conn = new SqlConnection("Server=prod-db;Database=Orders;User=admin;Password=secret");
        // ...
    }
}

public class ProductRepository
{
    public void SaveProduct()
    {
        using var conn = new SqlConnection("Server=prod-db;Database=Orders;User=admin;Password=secret");
        // ...
    }
}

// ✅ Good - Centralized configuration
public class DatabaseConfig
{
    public static string ConnectionString => 
        ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
}

public class OrderRepository
{
    public void SaveOrder()
    {
        using var conn = new SqlConnection(DatabaseConfig.ConnectionString);
        // ...
    }
}
```

---

## KISS - Keep It Simple, Stupid

**Principle:** Simplicity should be a key goal in design. Avoid unnecessary complexity.

**Why?** Complex code is harder to understand, maintain, and debug. Simple solutions are often more robust.

### ❌ Bad Example (Over-Engineered)

```csharp
// Over-complicated factory with abstract factory pattern for a simple case
public interface IAnimalFactory
{
    IAnimal CreateAnimal();
}

public interface IAnimal
{
    void MakeSound();
}

public class DogFactory : IAnimalFactory
{
    public IAnimal CreateAnimal() => new Dog();
}

public class CatFactory : IAnimalFactory
{
    public IAnimal CreateAnimal() => new Cat();
}

public class AnimalFactoryProvider
{
    public IAnimalFactory GetFactory(string type)
    {
        return type switch
        {
            "Dog" => new DogFactory(),
            "Cat" => new CatFactory(),
            _ => throw new ArgumentException("Invalid animal type")
        };
    }
}

// Usage - Too complex for such a simple task!
var provider = new AnimalFactoryProvider();
var factory = provider.GetFactory("Dog");
var animal = factory.CreateAnimal();
animal.MakeSound();
```

### ✅ Good Example (Simple and Direct)

```csharp
public class Dog
{
    public void MakeSound() => Console.WriteLine("Woof!");
}

public class Cat
{
    public void MakeSound() => Console.WriteLine("Meow!");
}

// Usage - Simple and clear
var dog = new Dog();
dog.MakeSound();
```

### When to Use Patterns vs KISS

```csharp
// ❌ Bad - Unnecessary abstraction for simple calculation
public interface IDiscountCalculator
{
    decimal Calculate(decimal amount);
}

public class DiscountCalculatorFactory
{
    public IDiscountCalculator Create() => new StandardDiscountCalculator();
}

public class StandardDiscountCalculator : IDiscountCalculator
{
    public decimal Calculate(decimal amount) => amount * 0.9m;
}

// ✅ Good - Simple method for simple task
public class OrderService
{
    public decimal ApplyDiscount(decimal amount)
    {
        return amount * 0.9m; // 10% discount
    }
}
```

**Rule of Thumb:** Use patterns when they solve a real problem, not because they're "best practice."

---

## YAGNI - You Aren't Gonna Need It

**Principle:** Don't add functionality until it's necessary. Avoid speculative generality.

**Why?** Building features "just in case" wastes time, adds complexity, and often the feature is never used.

### ❌ Bad Example (Speculative Generality)

```csharp
// Building for future requirements that don't exist yet
public class UserService
{
    // Current requirement: Send email notifications only
    // But developer adds SMS and Push "just in case"
    
    public void NotifyUser(User user, string message, NotificationType type)
    {
        switch (type)
        {
            case NotificationType.Email:
                SendEmail(user.Email, message);
                break;
            case NotificationType.SMS:
                // Not needed yet, but "might need it later"
                SendSMS(user.Phone, message);
                break;
            case NotificationType.PushNotification:
                // Also not needed yet
                SendPush(user.DeviceId, message);
                break;
            case NotificationType.Slack:
                // Really? This too?
                SendSlack(user.SlackId, message);
                break;
        }
    }
    
    private void SendSMS(string phone, string message) 
    { 
        // Unnecessary code to maintain
    }
    
    private void SendPush(string deviceId, string message) 
    { 
        // More unnecessary code
    }
    
    private void SendSlack(string slackId, string message) 
    { 
        // Even more unnecessary code
    }
}
```

### ✅ Good Example (Build What You Need)

```csharp
// Current requirement: Email only
public class UserService
{
    private readonly IEmailService _emailService;
    
    public UserService(IEmailService emailService)
    {
        _emailService = emailService;
    }
    
    public void NotifyUser(User user, string message)
    {
        _emailService.SendEmail(user.Email, message);
    }
}

// When SMS is actually needed, refactor:
// 1. Extract INotificationService interface
// 2. Add SMS implementation
// 3. Use Strategy pattern if multiple types needed
```

### Real-World Example

```csharp
// ❌ Bad - Building a generic data access layer for "flexibility"
public class GenericRepository<T> where T : class
{
    public T GetById(int id) { /* ... */ }
    public IEnumerable<T> GetAll() { /* ... */ }
    public IEnumerable<T> Find(Expression<Func<T, bool>> predicate) { /* ... */ }
    public void Add(T entity) { /* ... */ }
    public void AddRange(IEnumerable<T> entities) { /* ... */ }
    public void Remove(T entity) { /* ... */ }
    public void RemoveRange(IEnumerable<T> entities) { /* ... */ }
    public void Update(T entity) { /* ... */ }
    // ... 20 more methods you might never use
}

// ✅ Good - Build specific repositories as needed
public class OrderRepository
{
    public Order GetById(int id) 
    { 
        return _dbContext.Orders.Find(id); 
    }
    
    public void Add(Order order) 
    { 
        _dbContext.Orders.Add(order); 
    }
    
    // Add more methods when you actually need them
}
```

---

## Separation of Concerns (SoC)

**Principle:** Different concerns should be separated into distinct sections or modules. Each module should address a separate concern.

**Why?** Improves code organization, maintainability, and testability.

### ❌ Bad Example (Mixed Concerns)

```csharp
public class OrderController
{
    public IActionResult CreateOrder(OrderDto orderDto)
    {
        // Validation concern
        if (orderDto.Items == null || !orderDto.Items.Any())
            return BadRequest("Order must have items");
        
        // Business logic concern
        decimal total = 0;
        foreach (var item in orderDto.Items)
        {
            total += item.Price * item.Quantity;
        }
        
        if (total > 10000)
            total *= 0.9m; // Apply discount
        
        // Data access concern
        using var connection = new SqlConnection("connectionString");
        connection.Open();
        var command = new SqlCommand("INSERT INTO Orders...", connection);
        command.ExecuteNonQuery();
        
        // Email concern
        var emailClient = new SmtpClient();
        emailClient.Send("order@example.com", orderDto.Email, "Order Confirmation", "Thanks!");
        
        // Logging concern
        File.AppendAllText("orders.log", $"Order created: {orderDto.Id}");
        
        return Ok();
    }
}
```

### ✅ Good Example (Separated Concerns)

```csharp
// Validation concern
public class OrderValidator
{
    public ValidationResult Validate(OrderDto order)
    {
        if (order.Items == null || !order.Items.Any())
            return ValidationResult.Failed("Order must have items");
        
        return ValidationResult.Success;
    }
}

// Business logic concern
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;
    private readonly ILogger<OrderService> _logger;
    
    public async Task<Order> CreateOrderAsync(OrderDto orderDto)
    {
        var order = new Order
        {
            Items = orderDto.Items,
            Total = CalculateTotal(orderDto.Items)
        };
        
        await _repository.AddAsync(order);
        await _emailService.SendOrderConfirmationAsync(order);
        _logger.LogInformation("Order {OrderId} created", order.Id);
        
        return order;
    }
    
    private decimal CalculateTotal(List<OrderItemDto> items)
    {
        var total = items.Sum(i => i.Price * i.Quantity);
        return total > 10000 ? total * 0.9m : total;
    }
}

// Data access concern
public class OrderRepository : IOrderRepository
{
    private readonly DbContext _dbContext;
    
    public async Task AddAsync(Order order)
    {
        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync();
    }
}

// Email concern
public class EmailService : IEmailService
{
    public async Task SendOrderConfirmationAsync(Order order)
    {
        // Email sending logic
    }
}

// Controller - orchestration only
public class OrderController : ControllerBase
{
    private readonly OrderValidator _validator;
    private readonly OrderService _orderService;
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(OrderDto orderDto)
    {
        var validationResult = _validator.Validate(orderDto);
        if (!validationResult.IsValid)
            return BadRequest(validationResult.Errors);
        
        var order = await _orderService.CreateOrderAsync(orderDto);
        return Ok(order);
    }
}
```

---

## Composition Over Inheritance

**Principle:** Favor object composition over class inheritance for code reuse.

**Why?** Inheritance creates tight coupling. Composition is more flexible and easier to change.

### ❌ Bad Example (Inheritance Abuse)

```csharp
public class Animal
{
    public void Eat() => Console.WriteLine("Eating");
    public void Sleep() => Console.WriteLine("Sleeping");
}

public class Bird : Animal
{
    public void Fly() => Console.WriteLine("Flying");
}

public class Penguin : Bird
{
    // Problem: Penguins can't fly but inherit Fly() method!
    public new void Fly() => throw new NotSupportedException("Penguins can't fly");
}
```

### ✅ Good Example (Composition)

```csharp
public interface IMovable
{
    void Move();
}

public class FlyingBehavior : IMovable
{
    public void Move() => Console.WriteLine("Flying");
}

public class SwimmingBehavior : IMovable
{
    public void Move() => Console.WriteLine("Swimming");
}

public class WalkingBehavior : IMovable
{
    public void Move() => Console.WriteLine("Walking");
}

public class Animal
{
    private readonly IMovable _moveBehavior;
    
    public Animal(IMovable moveBehavior)
    {
        _moveBehavior = moveBehavior;
    }
    
    public void Move() => _moveBehavior.Move();
}

// Usage
var eagle = new Animal(new FlyingBehavior());
eagle.Move(); // Flying

var penguin = new Animal(new SwimmingBehavior());
penguin.Move(); // Swimming
```

---

## Law of Demeter (Principle of Least Knowledge)

**Principle:** A method should only call methods of:
1. Itself
2. Its parameters
3. Objects it creates
4. Its direct component objects

**Why?** Reduces coupling between classes. Avoid "train wrecks" like `obj.getA().getB().getC().doSomething()`.

### ❌ Bad Example (Violates Law of Demeter)

```csharp
public class Order
{
    public Customer Customer { get; set; }
}

public class Customer
{
    public Address Address { get; set; }
}

public class Address
{
    public string City { get; set; }
}

// Train wreck - knows too much about internal structure
public class OrderProcessor
{
    public void Process(Order order)
    {
        var city = order.Customer.Address.City; // Bad!
        Console.WriteLine($"Shipping to {city}");
    }
}
```

### ✅ Good Example (Follows Law of Demeter)

```csharp
public class Order
{
    private readonly Customer _customer;
    
    public Order(Customer customer)
    {
        _customer = customer;
    }
    
    // Encapsulate the navigation
    public string GetShippingCity() => _customer.GetCity();
}

public class Customer
{
    private readonly Address _address;
    
    public Customer(Address address)
    {
        _address = address;
    }
    
    public string GetCity() => _address.City;
}

public class OrderProcessor
{
    public void Process(Order order)
    {
        var city = order.GetShippingCity(); // Clean!
        Console.WriteLine($"Shipping to {city}");
    }
}
```

---

## Clean Architecture Principles

See detailed implementation in [07 Advanced Topics](../07-advanced-topics/07%20Advanced%20Topics.md)

**Layers (from inside out):**
1. **Entities** - Business objects (domain models)
2. **Use Cases** - Application business rules
3. **Interface Adapters** - Controllers, presenters, gateways
4. **Frameworks & Drivers** - UI, database, external services

**Key Rules:**
- Dependencies point inward (inner layers don't know about outer layers)
- Business logic is independent of frameworks, UI, and databases
- Easy to test (mock outer layers)

```csharp
// Domain layer (innermost)
public class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    public OrderStatus Status { get; set; }
}

// Use case layer
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(int id);
}

public class PlaceOrderUseCase
{
    private readonly IOrderRepository _repository;
    
    public async Task<Order> ExecuteAsync(OrderRequest request)
    {
        // Business logic - no dependencies on infrastructure
    }
}

// Infrastructure layer (outermost)
public class SqlOrderRepository : IOrderRepository
{
    public async Task<Order> GetByIdAsync(int id)
    {
        // Database-specific code
    }
}
```

---

## Summary Table

| Principle | Key Question | Example |
|-----------|-------------|---------|
| **DRY** | Am I duplicating logic? | Extract validation to a validator class |
| **KISS** | Is this simpler than it needs to be? | Use direct instantiation instead of factory for simple cases |
| **YAGNI** | Do I actually need this now? | Don't build SMS notifications if only email is required |
| **SoC** | Does this class/method do one thing? | Separate controller, service, repository |
| **Composition over Inheritance** | Can I use composition instead? | Inject behaviors instead of inheriting |
| **Law of Demeter** | Am I reaching through too many objects? | Add methods to hide internal structure |

---

## Applying Principles in .NET

✅ **Use built-in abstractions** - `ILogger`, `IConfiguration`, `IServiceProvider`  
✅ **Leverage dependency injection** - Constructor injection for loose coupling  
✅ **Follow naming conventions** - Clear, descriptive names reduce complexity  
✅ **Write unit tests** - Good design = testable code  
✅ **Code reviews** - Catch violations early
