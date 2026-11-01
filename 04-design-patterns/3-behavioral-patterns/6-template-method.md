# Template Method Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Define the skeleton of an algorithm in a base class, letting subclasses override specific steps without changing the algorithm's structure.

---

## Problem

You have multiple classes with similar algorithms that differ only in certain steps. Duplicating the entire algorithm in each class leads to:
- Code duplication
- Maintenance nightmares (change in one place, must update everywhere)
- Inconsistent implementations
- Violation of DRY principle

---

## Solution

Extract the algorithm into a base class with a template method that defines the structure. Individual steps are implemented as separate methods (some abstract, some with default implementations) that subclasses can override.

**Key Concept:** "Don't call us, we'll call you" (Hollywood Principle) - the base class controls the algorithm flow and calls subclass methods.

---

## Structure

```
AbstractClass
├── TemplateMethod() - Defines algorithm skeleton (final/sealed)
│   ├── Step1()
│   ├── Step2() - Abstract (must override)
│   ├── Step3() - Hook (optional override)
│   └── Step4()
└── Primitive operations

ConcreteClass1
└── Step2() - Implementation

ConcreteClass2
└── Step2() - Different implementation
```

---

## C# Implementation

### Example: Data Processing Pipeline

```csharp
// Abstract class with template method
public abstract class DataProcessor
{
    // Template method - defines algorithm skeleton (sealed so subclasses can't change flow)
    public void ProcessData()
    {
        Console.WriteLine("=== Data Processing Started ===\n");
        
        LoadData();
        
        if (ValidateData())
        {
            TransformData();
            SaveData();
            
            // Hook - optional step
            if (ShouldNotify())
            {
                SendNotification();
            }
        }
        else
        {
            Console.WriteLine("Validation failed. Processing aborted.\n");
        }
        
        Console.WriteLine("=== Data Processing Completed ===\n");
    }
    
    // Abstract methods - must be implemented by subclasses
    protected abstract void LoadData();
    protected abstract void TransformData();
    protected abstract void SaveData();
    
    // Hook methods - default implementation (optional override)
    protected virtual bool ValidateData()
    {
        Console.WriteLine("Performing default validation...");
        return true;
    }
    
    protected virtual bool ShouldNotify()
    {
        return false;  // By default, don't notify
    }
    
    protected virtual void SendNotification()
    {
        Console.WriteLine("Sending notification...");
    }
}

// Concrete implementation 1: CSV Processor
public class CSVDataProcessor : DataProcessor
{
    protected override void LoadData()
    {
        Console.WriteLine("[CSV] Loading data from CSV file...");
    }
    
    protected override void TransformData()
    {
        Console.WriteLine("[CSV] Transforming CSV data to object model...");
    }
    
    protected override void SaveData()
    {
        Console.WriteLine("[CSV] Saving data to database...");
    }
    
    protected override bool ValidateData()
    {
        Console.WriteLine("[CSV] Validating CSV structure...");
        return true;
    }
}

// Concrete implementation 2: JSON Processor
public class JSONDataProcessor : DataProcessor
{
    protected override void LoadData()
    {
        Console.WriteLine("[JSON] Loading data from API endpoint...");
    }
    
    protected override void TransformData()
    {
        Console.WriteLine("[JSON] Deserializing JSON to objects...");
    }
    
    protected override void SaveData()
    {
        Console.WriteLine("[JSON] Saving to NoSQL database...");
    }
    
    protected override bool ShouldNotify()
    {
        return true;  // Always notify for JSON processing
    }
    
    protected override void SendNotification()
    {
        Console.WriteLine("[JSON] Sending email notification...");
    }
}

// Concrete implementation 3: XML Processor
public class XMLDataProcessor : DataProcessor
{
    protected override void LoadData()
    {
        Console.WriteLine("[XML] Loading XML from message queue...");
    }
    
    protected override void TransformData()
    {
        Console.WriteLine("[XML] Parsing XML using XDocument...");
    }
    
    protected override void SaveData()
    {
        Console.WriteLine("[XML] Writing to file system...");
    }
    
    protected override bool ValidateData()
    {
        Console.WriteLine("[XML] Validating against XSD schema...");
        // Simulate validation failure
        return false;
    }
}

// Usage
Console.WriteLine("Processing CSV:");
var csvProcessor = new CSVDataProcessor();
csvProcessor.ProcessData();

Console.WriteLine("\nProcessing JSON:");
var jsonProcessor = new JSONDataProcessor();
jsonProcessor.ProcessData();

Console.WriteLine("\nProcessing XML:");
var xmlProcessor = new XMLDataProcessor();
xmlProcessor.ProcessData();

// Output:
// Processing CSV:
// === Data Processing Started ===
// [CSV] Loading data from CSV file...
// [CSV] Validating CSV structure...
// [CSV] Transforming CSV data to object model...
// [CSV] Saving data to database...
// === Data Processing Completed ===
// 
// Processing JSON:
// === Data Processing Started ===
// [JSON] Loading data from API endpoint...
// Performing default validation...
// [JSON] Deserializing JSON to objects...
// [JSON] Saving to NoSQL database...
// [JSON] Sending email notification...
// === Data Processing Completed ===
// 
// Processing XML:
// === Data Processing Started ===
// [XML] Loading XML from message queue...
// [XML] Validating against XSD schema...
// Validation failed. Processing aborted.
// === Data Processing Completed ===
```

---

## ASP.NET Core Example: Base Controller

```csharp
// Template: Base API Controller with common behavior
public abstract class BaseApiController<T> : ControllerBase where T : class
{
    protected readonly ILogger _logger;
    
    protected BaseApiController(ILogger logger)
    {
        _logger = logger;
    }
    
    // Template Method - common GET flow
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        try
        {
            _logger.LogInformation($"Fetching {typeof(T).Name} with ID: {id}");
            
            // Validate
            if (!await ValidateGetRequest(id))
            {
                return BadRequest("Invalid request");
            }
            
            // Fetch data
            var entity = await FetchEntity(id);
            
            if (entity == null)
            {
                return NotFound();
            }
            
            // Transform (hook)
            var dto = TransformToDto(entity);
            
            // Log (hook)
            LogAccess(id);
            
            return Ok(dto);
        }
        catch (Exception ex)
        {
            return HandleError(ex);
        }
    }
    
    // Abstract methods - must implement
    protected abstract Task<T> FetchEntity(int id);
    protected abstract object TransformToDto(T entity);
    
    // Hook methods - optional override
    protected virtual Task<bool> ValidateGetRequest(int id)
    {
        return Task.FromResult(id > 0);
    }
    
    protected virtual void LogAccess(int id)
    {
        // Default: do nothing
    }
    
    protected virtual IActionResult HandleError(Exception ex)
    {
        _logger.LogError(ex, "An error occurred");
        return StatusCode(500, "Internal server error");
    }
}

// Concrete: User Controller
public class UsersController : BaseApiController<User>
{
    private readonly IUserRepository _repository;
    
    public UsersController(IUserRepository repository, ILogger<UsersController> logger)
        : base(logger)
    {
        _repository = repository;
    }
    
    protected override async Task<User> FetchEntity(int id)
    {
        return await _repository.GetByIdAsync(id);
    }
    
    protected override object TransformToDto(User entity)
    {
        return new UserDto
        {
            Id = entity.Id,
            Name = entity.Name,
            Email = entity.Email
            // Don't expose password
        };
    }
    
    protected override void LogAccess(int id)
    {
        _logger.LogInformation($"User {id} accessed by {User.Identity.Name}");
    }
}

// Concrete: Product Controller with custom validation
public class ProductsController : BaseApiController<Product>
{
    private readonly IProductRepository _repository;
    
    public ProductsController(IProductRepository repository, ILogger<ProductsController> logger)
        : base(logger)
    {
        _repository = repository;
    }
    
    protected override async Task<Product> FetchEntity(int id)
    {
        return await _repository.GetByIdAsync(id);
    }
    
    protected override object TransformToDto(Product entity)
    {
        return new ProductDto
        {
            Id = entity.Id,
            Name = entity.Name,
            Price = entity.Price,
            InStock = entity.Quantity > 0
        };
    }
    
    protected override async Task<bool> ValidateGetRequest(int id)
    {
        // Custom validation: ensure user has permission
        if (!User.IsInRole("Admin") && !User.IsInRole("Viewer"))
        {
            _logger.LogWarning($"Unauthorized access attempt to product {id}");
            return false;
        }
        
        return await base.ValidateGetRequest(id);
    }
}
```

---

## Report Generation Example

```csharp
public abstract class ReportGenerator
{
    // Template method
    public void GenerateReport()
    {
        Console.WriteLine($"\n=== Generating {GetReportType()} Report ===\n");
        
        var data = CollectData();
        
        if (ApplyFilters(data))
        {
            var processed = ProcessData(data);
            var formatted = FormatReport(processed);
            
            OutputReport(formatted);
            
            // Hook
            if (ShouldEmail())
            {
                SendEmail(formatted);
            }
        }
        
        Console.WriteLine($"\n=== {GetReportType()} Report Complete ===\n");
    }
    
    // Abstract methods
    protected abstract string GetReportType();
    protected abstract List<object> CollectData();
    protected abstract List<object> ProcessData(List<object> data);
    protected abstract string FormatReport(List<object> data);
    protected abstract void OutputReport(string report);
    
    // Hook methods
    protected virtual bool ApplyFilters(List<object> data)
    {
        Console.WriteLine("Applying default filters...");
        return true;
    }
    
    protected virtual bool ShouldEmail()
    {
        return false;
    }
    
    protected virtual void SendEmail(string report)
    {
        Console.WriteLine("Sending report via email...");
    }
}

// Sales Report
public class SalesReportGenerator : ReportGenerator
{
    protected override string GetReportType() => "Sales";
    
    protected override List<object> CollectData()
    {
        Console.WriteLine("Collecting sales data from database...");
        return new List<object> { "Sale1", "Sale2", "Sale3" };
    }
    
    protected override List<object> ProcessData(List<object> data)
    {
        Console.WriteLine("Calculating total sales, averages, and trends...");
        return data;
    }
    
    protected override string FormatReport(List<object> data)
    {
        Console.WriteLine("Formatting as PDF with charts...");
        return "SALES_REPORT_PDF";
    }
    
    protected override void OutputReport(string report)
    {
        Console.WriteLine($"Saving report to: C:\\Reports\\{report}.pdf");
    }
    
    protected override bool ShouldEmail() => true;
}

// Inventory Report
public class InventoryReportGenerator : ReportGenerator
{
    protected override string GetReportType() => "Inventory";
    
    protected override List<object> CollectData()
    {
        Console.WriteLine("Collecting inventory data from warehouse system...");
        return new List<object> { "Item1", "Item2", "Item3" };
    }
    
    protected override List<object> ProcessData(List<object> data)
    {
        Console.WriteLine("Identifying low stock items...");
        return data;
    }
    
    protected override string FormatReport(List<object> data)
    {
        Console.WriteLine("Formatting as Excel spreadsheet...");
        return "INVENTORY_REPORT_XLSX";
    }
    
    protected override void OutputReport(string report)
    {
        Console.WriteLine($"Saving report to: C:\\Reports\\{report}.xlsx");
    }
    
    protected override bool ApplyFilters(List<object> data)
    {
        Console.WriteLine("Filtering: Only items with stock < 10");
        return data.Count > 0;
    }
}

// Usage
var salesReport = new SalesReportGenerator();
salesReport.GenerateReport();

var inventoryReport = new InventoryReportGenerator();
inventoryReport.GenerateReport();
```

---

## Unit Test Base Class Example

```csharp
// Template for integration tests
public abstract class IntegrationTestBase
{
    protected HttpClient Client { get; private set; }
    protected WebApplicationFactory<Program> Factory { get; private set; }
    
    // Template method
    public async Task RunTest()
    {
        try
        {
            Setup();
            SeedDatabase();
            
            await ExecuteTest();
            
            VerifyResults();
        }
        finally
        {
            Cleanup();
        }
    }
    
    // Abstract - test logic
    protected abstract Task ExecuteTest();
    protected abstract void VerifyResults();
    
    // Hook - optional database seeding
    protected virtual void SeedDatabase()
    {
        // Default: no seeding
    }
    
    // Common setup
    private void Setup()
    {
        Factory = new WebApplicationFactory<Program>();
        Client = Factory.CreateClient();
    }
    
    // Common cleanup
    private void Cleanup()
    {
        Client?.Dispose();
        Factory?.Dispose();
    }
}

// Concrete test
public class UserApiTest : IntegrationTestBase
{
    private HttpResponseMessage _response;
    
    protected override void SeedDatabase()
    {
        // Seed test users
        Console.WriteLine("Seeding test users...");
    }
    
    protected override async Task ExecuteTest()
    {
        _response = await Client.GetAsync("/api/users/1");
    }
    
    protected override void VerifyResults()
    {
        Assert.Equal(HttpStatusCode.OK, _response.StatusCode);
    }
}
```

---

## Game AI Example

```csharp
public abstract class AICharacter
{
    // Template method - AI decision loop
    public void MakeTurn()
    {
        Console.WriteLine($"\n{GetCharacterName()}'s turn:");
        
        AnalyzeSituation();
        
        var action = DecideAction();
        
        ExecuteAction(action);
        
        if (ShouldCommunicate())
        {
            Communicate();
        }
    }
    
    // Abstract methods
    protected abstract string GetCharacterName();
    protected abstract string DecideAction();
    protected abstract void ExecuteAction(string action);
    
    // Hook methods
    protected virtual void AnalyzeSituation()
    {
        Console.WriteLine("Analyzing battlefield...");
    }
    
    protected virtual bool ShouldCommunicate()
    {
        return false;
    }
    
    protected virtual void Communicate()
    {
        Console.WriteLine("...");
    }
}

// Warrior AI
public class WarriorAI : AICharacter
{
    protected override string GetCharacterName() => "Warrior";
    
    protected override string DecideAction()
    {
        Console.WriteLine("Deciding: Attack the nearest enemy!");
        return "ATTACK";
    }
    
    protected override void ExecuteAction(string action)
    {
        Console.WriteLine("⚔️ Swinging sword at enemy!");
    }
    
    protected override bool ShouldCommunicate() => true;
    
    protected override void Communicate()
    {
        Console.WriteLine("💬 'For honor!'");
    }
}

// Mage AI
public class MageAI : AICharacter
{
    protected override string GetCharacterName() => "Mage";
    
    protected override void AnalyzeSituation()
    {
        Console.WriteLine("Analyzing: Checking mana and spell cooldowns...");
    }
    
    protected override string DecideAction()
    {
        Console.WriteLine("Deciding: Cast fireball!");
        return "CAST_SPELL";
    }
    
    protected override void ExecuteAction(string action)
    {
        Console.WriteLine("🔥 Casting fireball!");
    }
}

// Usage
var warrior = new WarriorAI();
warrior.MakeTurn();

var mage = new MageAI();
mage.MakeTurn();
```

---

## When to Use Template Method

✅ **Use when:**
- Multiple classes have similar algorithms with different steps
- Want to control the algorithm structure in one place
- Subclasses should implement specific steps but not change flow
- Common behavior should be extracted to avoid duplication

❌ **Avoid when:**
- Algorithm steps vary significantly between implementations
- Need more flexibility in algorithm structure
- Too many hook methods make base class complex

---

## Advantages

✅ **Code reuse** - Common algorithm structure in one place  
✅ **Consistency** - All subclasses follow same flow  
✅ **Open/Closed** - Add new implementations without changing base  
✅ **Inversion of Control** - Base class calls subclass methods (Hollywood Principle)

---

## Disadvantages

❌ **Rigid structure** - Can't change algorithm flow in subclasses  
❌ **Liskov Substitution** - Can be violated if not careful  
❌ **Debugging complexity** - Flow jumps between base and derived classes  
❌ **Maintenance** - Changes to base class affect all subclasses

---

## Hook Methods vs Abstract Methods

| Method Type | Purpose | Implementation | Override Required |
|-------------|---------|----------------|-------------------|
| **Abstract** | Core algorithm steps | None (abstract) | Yes |
| **Hook** | Optional extension points | Default (empty or basic) | No |

---

## Related Patterns

- **Strategy** - Template Method uses inheritance; Strategy uses composition
- **Factory Method** - Often called within Template Method
- **Hollywood Principle** - "Don't call us, we'll call you"

---

## Common Use Cases

- **Frameworks** - ASP.NET Core controllers, test frameworks
- **Data processing** - ETL pipelines (Extract, Transform, Load)
- **Report generation** - Different formats, same structure
- **Game loops** - Initialize, update, render, cleanup
- **Build systems** - Compile, link, package
- **Testing** - Setup, execute, verify, teardown

---

## Best Practices

✅ Make template method **sealed/final** to prevent overriding  
✅ Clearly distinguish **abstract** methods (required) from **hooks** (optional)  
✅ Document the **algorithm structure** and expected behavior  
✅ Use **protected** visibility for primitive operations  
✅ Minimize **number of hooks** to avoid complexity  
✅ Consider **Strategy pattern** if algorithm structure varies too much