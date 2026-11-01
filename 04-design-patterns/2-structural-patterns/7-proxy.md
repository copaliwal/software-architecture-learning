# Proxy Pattern

## Overview

**Category:** Structural Pattern  
**Intent:** Provide a surrogate or placeholder for another object to control access to it.

---

## Problem

You need to control access to an object for various reasons:
- **Lazy initialization** - Object is expensive to create
- **Access control** - Restrict access based on permissions
- **Remote access** - Object exists in different address space
- **Logging/auditing** - Track access to the object
- **Caching** - Cache expensive operations

Creating the real object directly doesn't allow this control.

---

## Solution

Create a proxy class that:
1. Implements the same interface as the real object
2. Maintains a reference to the real object
3. Controls access to the real object
4. Forwards requests to the real object when appropriate

Clients interact with the proxy instead of the real object.

---

## Types of Proxies

| Type | Purpose | Example |
|------|---------|---------|
| **Virtual Proxy** | Lazy initialization of expensive objects | Loading large images on demand |
| **Protection Proxy** | Access control and permissions | Admin-only operations |
| **Remote Proxy** | Represents object in different address space | gRPC, WCF services |
| **Smart Proxy** | Additional functionality (logging, caching) | Reference counting, auditing |
| **Cache Proxy** | Cache expensive operations | API response caching |

---

## C# Implementation

### 1. Virtual Proxy (Lazy Loading)

```csharp
// Subject interface
public interface IImage
{
    void Display();
    int GetFileSize();
}

// Real Subject - Expensive to create
public class HighResolutionImage : IImage
{
    private readonly string _filename;
    private readonly byte[] _imageData;
    
    public HighResolutionImage(string filename)
    {
        _filename = filename;
        Console.WriteLine($"Loading high-resolution image: {filename}");
        
        // Simulate expensive loading operation (5MB image)
        Thread.Sleep(2000);  // Simulate network/disk delay
        _imageData = new byte[5_000_000];
        
        Console.WriteLine($"Image {filename} loaded into memory (5MB)");
    }
    
    public void Display()
    {
        Console.WriteLine($"Displaying {_filename}");
    }
    
    public int GetFileSize() => _imageData.Length;
}

// Proxy - Delays creation until needed
public class ImageProxy : IImage
{
    private readonly string _filename;
    private HighResolutionImage? _realImage;
    
    public ImageProxy(string filename)
    {
        _filename = filename;
        Console.WriteLine($"ImageProxy created for {filename} (not loaded yet)");
    }
    
    public void Display()
    {
        // Lazy loading - create real object only when needed
        if (_realImage == null)
        {
            Console.WriteLine("First access - loading real image...");
            _realImage = new HighResolutionImage(_filename);
        }
        
        _realImage.Display();
    }
    
    public int GetFileSize()
    {
        if (_realImage == null)
        {
            _realImage = new HighResolutionImage(_filename);
        }
        
        return _realImage.GetFileSize();
    }
}

// Usage
Console.WriteLine("Creating image proxies...");
var image1 = new ImageProxy("photo1.jpg");
var image2 = new ImageProxy("photo2.jpg");
var image3 = new ImageProxy("photo3.jpg");

Console.WriteLine("\nProxies created instantly. No images loaded yet.\n");

Console.WriteLine("User scrolls to photo2...");
image2.Display();  // Only now photo2 is loaded

Console.WriteLine("\nUser scrolls to photo2 again...");
image2.Display();  // Reuses already loaded image

// Output:
// Creating image proxies...
// ImageProxy created for photo1.jpg (not loaded yet)
// ImageProxy created for photo2.jpg (not loaded yet)
// ImageProxy created for photo3.jpg (not loaded yet)
// 
// Proxies created instantly. No images loaded yet.
// 
// User scrolls to photo2...
// First access - loading real image...
// Loading high-resolution image: photo2.jpg
// Image photo2.jpg loaded into memory (5MB)
// Displaying photo2.jpg
```

---

### 2. Protection Proxy (Access Control)

```csharp
// Subject interface
public interface IBankAccount
{
    void Deposit(decimal amount);
    void Withdraw(decimal amount);
    decimal GetBalance();
    void ViewTransactionHistory();
}

// Real Subject
public class BankAccount : IBankAccount
{
    private decimal _balance;
    private readonly List<string> _transactions = new();
    
    public void Deposit(decimal amount)
    {
        _balance += amount;
        _transactions.Add($"Deposited: ${amount}");
        Console.WriteLine($"Deposited ${amount}. New balance: ${_balance}");
    }
    
    public void Withdraw(decimal amount)
    {
        if (_balance >= amount)
        {
            _balance -= amount;
            _transactions.Add($"Withdrew: ${amount}");
            Console.WriteLine($"Withdrew ${amount}. New balance: ${_balance}");
        }
        else
        {
            Console.WriteLine("Insufficient funds");
        }
    }
    
    public decimal GetBalance() => _balance;
    
    public void ViewTransactionHistory()
    {
        Console.WriteLine("--- Transaction History ---");
        foreach (var transaction in _transactions)
        {
            Console.WriteLine(transaction);
        }
    }
}

// Protection Proxy - Enforces access control
public class BankAccountProxy : IBankAccount
{
    private readonly BankAccount _realAccount;
    private readonly string _userRole;
    
    public BankAccountProxy(BankAccount account, string userRole)
    {
        _realAccount = account;
        _userRole = userRole;
    }
    
    public void Deposit(decimal amount)
    {
        // All users can deposit
        _realAccount.Deposit(amount);
    }
    
    public void Withdraw(decimal amount)
    {
        if (_userRole == "Owner" || _userRole == "Admin")
        {
            _realAccount.Withdraw(amount);
        }
        else
        {
            Console.WriteLine("Access Denied: Only account owner can withdraw");
        }
    }
    
    public decimal GetBalance()
    {
        if (_userRole == "Owner" || _userRole == "Admin" || _userRole == "Viewer")
        {
            return _realAccount.GetBalance();
        }
        
        Console.WriteLine("Access Denied: Insufficient permissions to view balance");
        return 0;
    }
    
    public void ViewTransactionHistory()
    {
        if (_userRole == "Owner" || _userRole == "Admin")
        {
            _realAccount.ViewTransactionHistory();
        }
        else
        {
            Console.WriteLine("Access Denied: Only account owner can view transaction history");
        }
    }
}

// Usage
var account = new BankAccount();

// Owner access
var ownerProxy = new BankAccountProxy(account, "Owner");
ownerProxy.Deposit(1000);
ownerProxy.Withdraw(200);
ownerProxy.ViewTransactionHistory();

Console.WriteLine();

// Viewer access (read-only)
var viewerProxy = new BankAccountProxy(account, "Viewer");
Console.WriteLine($"Balance: ${viewerProxy.GetBalance()}");
viewerProxy.Withdraw(100);  // Denied
viewerProxy.ViewTransactionHistory();  // Denied

Console.WriteLine();

// Guest access (minimal permissions)
var guestProxy = new BankAccountProxy(account, "Guest");
guestProxy.Deposit(50);  // Allowed
guestProxy.GetBalance();  // Denied
```

---

### 3. Smart Proxy (Caching & Logging)

```csharp
// Subject interface
public interface IProductService
{
    Product GetProductById(int id);
    List<Product> GetProductsByCategory(string category);
}

// Model
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
}

// Real Subject - Expensive database operations
public class ProductService : IProductService
{
    public Product GetProductById(int id)
    {
        Console.WriteLine($"[DB Query] Fetching product {id} from database...");
        Thread.Sleep(1000);  // Simulate database delay
        
        return new Product
        {
            Id = id,
            Name = $"Product {id}",
            Category = "Electronics",
            Price = 99.99m
        };
    }
    
    public List<Product> GetProductsByCategory(string category)
    {
        Console.WriteLine($"[DB Query] Fetching all products in category '{category}'...");
        Thread.Sleep(2000);  // Simulate expensive query
        
        return new List<Product>
        {
            new() { Id = 1, Name = "Product 1", Category = category, Price = 99.99m },
            new() { Id = 2, Name = "Product 2", Category = category, Price = 149.99m }
        };
    }
}

// Smart Proxy - Adds caching and logging
public class ProductServiceProxy : IProductService
{
    private readonly ProductService _realService;
    private readonly Dictionary<int, (Product Product, DateTime CachedAt)> _productCache = new();
    private readonly Dictionary<string, (List<Product> Products, DateTime CachedAt)> _categoryCache = new();
    private readonly TimeSpan _cacheExpiration = TimeSpan.FromMinutes(5);
    
    public ProductServiceProxy(ProductService realService)
    {
        _realService = realService;
    }
    
    public Product GetProductById(int id)
    {
        // Check cache
        if (_productCache.ContainsKey(id))
        {
            var cached = _productCache[id];
            if (DateTime.Now - cached.CachedAt < _cacheExpiration)
            {
                Console.WriteLine($"[Cache Hit] Returning cached product {id}");
                return cached.Product;
            }
            else
            {
                Console.WriteLine($"[Cache Expired] Cache entry for product {id} expired");
                _productCache.Remove(id);
            }
        }
        
        // Cache miss - fetch from real service
        Console.WriteLine($"[Cache Miss] Product {id} not in cache");
        var product = _realService.GetProductById(id);
        
        // Store in cache
        _productCache[id] = (product, DateTime.Now);
        Console.WriteLine($"[Cache Store] Product {id} cached for {_cacheExpiration.TotalMinutes} minutes");
        
        return product;
    }
    
    public List<Product> GetProductsByCategory(string category)
    {
        if (_categoryCache.ContainsKey(category))
        {
            var cached = _categoryCache[category];
            if (DateTime.Now - cached.CachedAt < _cacheExpiration)
            {
                Console.WriteLine($"[Cache Hit] Returning cached category '{category}'");
                return cached.Products;
            }
        }
        
        Console.WriteLine($"[Cache Miss] Category '{category}' not in cache");
        var products = _realService.GetProductsByCategory(category);
        _categoryCache[category] = (products, DateTime.Now);
        
        return products;
    }
}

// Usage
var realService = new ProductService();
var proxy = new ProductServiceProxy(realService);

Console.WriteLine("First request:");
var product1 = proxy.GetProductById(1);  // Cache miss - DB query

Console.WriteLine("\nSecond request (same product):");
var product2 = proxy.GetProductById(1);  // Cache hit - no DB query

Console.WriteLine("\nThird request (different product):");
var product3 = proxy.GetProductById(2);  // Cache miss - DB query

// Output:
// First request:
// [Cache Miss] Product 1 not in cache
// [DB Query] Fetching product 1 from database...
// [Cache Store] Product 1 cached for 5 minutes
// 
// Second request (same product):
// [Cache Hit] Returning cached product 1
// 
// Third request (different product):
// [Cache Miss] Product 2 not in cache
// [DB Query] Fetching product 2 from database...
```

---

## ASP.NET Core Example: Repository Proxy with Logging

```csharp
public interface IUserRepository
{
    Task<User> GetByIdAsync(int id);
    Task<List<User>> GetAllAsync();
    Task AddAsync(User user);
    Task UpdateAsync(User user);
    Task DeleteAsync(int id);
}

// Real repository
public class UserRepository : IUserRepository
{
    private readonly DbContext _context;
    
    public UserRepository(DbContext context)
    {
        _context = context;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        return await _context.Set<User>().FindAsync(id);
    }
    
    public async Task<List<User>> GetAllAsync()
    {
        return await _context.Set<User>().ToListAsync();
    }
    
    public async Task AddAsync(User user)
    {
        await _context.Set<User>().AddAsync(user);
        await _context.SaveChangesAsync();
    }
    
    public async Task UpdateAsync(User user)
    {
        _context.Set<User>().Update(user);
        await _context.SaveChangesAsync();
    }
    
    public async Task DeleteAsync(int id)
    {
        var user = await GetByIdAsync(id);
        if (user != null)
        {
            _context.Set<User>().Remove(user);
            await _context.SaveChangesAsync();
        }
    }
}

// Proxy with logging and performance tracking
public class UserRepositoryProxy : IUserRepository
{
    private readonly IUserRepository _realRepository;
    private readonly ILogger<UserRepositoryProxy> _logger;
    
    public UserRepositoryProxy(IUserRepository realRepository, ILogger<UserRepositoryProxy> logger)
    {
        _realRepository = realRepository;
        _logger = logger;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        _logger.LogInformation("Fetching user with ID: {UserId}", id);
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var user = await _realRepository.GetByIdAsync(id);
            stopwatch.Stop();
            
            _logger.LogInformation(
                "User {UserId} fetched successfully in {ElapsedMs}ms",
                id, stopwatch.ElapsedMilliseconds);
            
            return user;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            _logger.LogError(ex, "Error fetching user {UserId} after {ElapsedMs}ms", 
                id, stopwatch.ElapsedMilliseconds);
            throw;
        }
    }
    
    public async Task<List<User>> GetAllAsync()
    {
        _logger.LogInformation("Fetching all users");
        var stopwatch = Stopwatch.StartNew();
        
        var users = await _realRepository.GetAllAsync();
        stopwatch.Stop();
        
        _logger.LogInformation(
            "Fetched {UserCount} users in {ElapsedMs}ms",
            users.Count, stopwatch.ElapsedMilliseconds);
        
        return users;
    }
    
    public async Task AddAsync(User user)
    {
        _logger.LogInformation("Adding new user: {UserEmail}", user.Email);
        await _realRepository.AddAsync(user);
        _logger.LogInformation("User {UserEmail} added successfully", user.Email);
    }
    
    public async Task UpdateAsync(User user)
    {
        _logger.LogInformation("Updating user ID: {UserId}", user.Id);
        await _realRepository.UpdateAsync(user);
        _logger.LogInformation("User {UserId} updated successfully", user.Id);
    }
    
    public async Task DeleteAsync(int id)
    {
        _logger.LogWarning("Deleting user ID: {UserId}", id);
        await _realRepository.DeleteAsync(id);
        _logger.LogInformation("User {UserId} deleted successfully", id);
    }
}

// Dependency injection setup
services.AddScoped<DbContext, ApplicationDbContext>();
services.AddScoped<UserRepository>();
services.AddScoped<IUserRepository>(sp =>
{
    var realRepo = sp.GetRequiredService<UserRepository>();
    var logger = sp.GetRequiredService<ILogger<UserRepositoryProxy>>();
    return new UserRepositoryProxy(realRepo, logger);
});
```

---

## Remote Proxy Example (gRPC)

```csharp
// gRPC service definition (proto file)
// service UserService {
//   rpc GetUser(GetUserRequest) returns (UserResponse);
// }

// Local proxy for remote gRPC service
public class UserServiceProxy : IUserService
{
    private readonly UserService.UserServiceClient _grpcClient;
    
    public UserServiceProxy(string serverAddress)
    {
        var channel = GrpcChannel.ForAddress(serverAddress);
        _grpcClient = new UserService.UserServiceClient(channel);
    }
    
    public async Task<User> GetUserAsync(int userId)
    {
        // Convert local call to remote gRPC call
        var request = new GetUserRequest { UserId = userId };
        var response = await _grpcClient.GetUserAsync(request);
        
        // Convert gRPC response to local domain model
        return new User
        {
            Id = response.Id,
            Name = response.Name,
            Email = response.Email
        };
    }
}

// Client code - doesn't know it's calling a remote service
var userService = new UserServiceProxy("https://localhost:5001");
var user = await userService.GetUserAsync(123);
Console.WriteLine($"User: {user.Name}");
```

---

## When to Use Proxy Pattern

✅ **Use Proxy when:**
- Need lazy initialization of expensive objects
- Access control/authorization required
- Remote object access (different process/machine)
- Logging, auditing, or metrics needed
- Caching expensive operations
- Reference counting or smart references

❌ **Avoid when:**
- Direct access is sufficient
- Proxy overhead outweighs benefits
- Simple objects with no special requirements

---

## Advantages

✅ **Controlled access** - Control when/how real object is accessed  
✅ **Lazy initialization** - Create expensive objects only when needed  
✅ **Separation of concerns** - Proxy handles cross-cutting concerns  
✅ **Transparent** - Clients use same interface  
✅ **Open/Closed Principle** - Add proxies without modifying real object

---

## Disadvantages

❌ **Added complexity** - Extra layer of indirection  
❌ **Performance overhead** - Additional method calls  
❌ **Delayed response** - Lazy loading may cause unexpected delays  
❌ **Harder debugging** - Extra layer makes stack traces longer

---

## Proxy vs Decorator vs Adapter

| Pattern | Purpose | Same Interface | Manages Lifecycle |
|---------|---------|----------------|-------------------|
| **Proxy** | Control access to object | Yes | Often manages creation |
| **Decorator** | Add responsibilities | Yes | No, wraps existing object |
| **Adapter** | Convert interface | No (adapts) | No |

---

## Related Patterns

- **Decorator** - Similar structure, but decorator adds behavior while proxy controls access
- **Adapter** - Adapts interface; proxy has same interface
- **Facade** - Simplifies interface; proxy has same interface
- **Singleton** - Proxy can ensure single instance

---

## Common Use Cases

- **Virtual proxy** - Lazy loading images, documents, large datasets
- **Protection proxy** - Role-based access control, authentication
- **Remote proxy** - RPC, gRPC, WCF, web services
- **Smart reference** - Reference counting, logging, caching
- **Copy-on-write** - Defer copying until modification
- **Firewall proxy** - Network security, filtering

---

## .NET Framework Examples

```csharp
// Lazy<T> - Built-in virtual proxy
var lazyValue = new Lazy<ExpensiveObject>(() => new ExpensiveObject());
// Object not created yet

var value = lazyValue.Value;  // Now created on first access

// HttpClient - Acts as remote proxy
var httpClient = new HttpClient();
var response = await httpClient.GetAsync("https://api.example.com/users");
// Proxy handles HTTP communication

// DispatchProxy - Create dynamic proxies at runtime
public class LoggingProxy<T> : DispatchProxy
{
    private T _target;
    private ILogger _logger;
    
    public static T Create(T target, ILogger logger)
    {
        object proxy = Create<T, LoggingProxy<T>>();
        ((LoggingProxy<T>)proxy).SetTarget(target, logger);
        return (T)proxy;
    }
    
    private void SetTarget(T target, ILogger logger)
    {
        _target = target;
        _logger = logger;
    }
    
    protected override object Invoke(MethodInfo targetMethod, object[] args)
    {
        _logger.LogInformation($"Calling {targetMethod.Name}");
        var result = targetMethod.Invoke(_target, args);
        _logger.LogInformation($"Finished {targetMethod.Name}");
        return result;
    }
}
```

---

## Best Practices

✅ Implement **same interface** as real object  
✅ Use proxy for **cross-cutting concerns** (logging, caching, security)  
✅ Consider **DispatchProxy** for dynamic proxies in .NET  
✅ Make proxies **thread-safe** if shared  
✅ Document **performance characteristics** (lazy loading delays)  
✅ Use **dependency injection** to inject proxies transparently
