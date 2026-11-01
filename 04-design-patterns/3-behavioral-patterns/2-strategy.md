# Strategy Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

---

## Problem

You have multiple ways to perform an operation, and you want to select the algorithm at runtime without using complex conditional logic (`if-else` or `switch` statements).

---

## Solution

Extract each algorithm into separate classes that implement a common interface. The context class holds a reference to a strategy interface and delegates the work to the linked strategy object.

---

## Structure

```
Context
├── SetStrategy(IStrategy)
└── ExecuteStrategy()

IStrategy (Interface)
└── Execute()

ConcreteStrategyA
└── Execute() implementation

ConcreteStrategyB
└── Execute() implementation
```

---

## C# Implementation

### Basic Example: Payment Processing

```csharp
// Strategy interface
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}

// Concrete Strategies
public class CreditCardPayment : IPaymentStrategy
{
    private readonly string _cardNumber;
    
    public CreditCardPayment(string cardNumber)
    {
        _cardNumber = cardNumber;
    }
    
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} using Credit Card ending in {_cardNumber.Substring(_cardNumber.Length - 4)}");
    }
}

public class PayPalPayment : IPaymentStrategy
{
    private readonly string _email;
    
    public PayPalPayment(string email)
    {
        _email = email;
    }
    
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} using PayPal account {_email}");
    }
}

public class CryptoPayment : IPaymentStrategy
{
    private readonly string _walletAddress;
    
    public CryptoPayment(string walletAddress)
    {
        _walletAddress = walletAddress;
    }
    
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Paid ${amount} in Bitcoin to wallet {_walletAddress}");
    }
}

// Context
public class ShoppingCart
{
    private IPaymentStrategy _paymentStrategy;
    
    public void SetPaymentStrategy(IPaymentStrategy strategy)
    {
        _paymentStrategy = strategy;
    }
    
    public void Checkout(decimal amount)
    {
        if (_paymentStrategy == null)
            throw new InvalidOperationException("Payment strategy not set");
            
        _paymentStrategy.Pay(amount);
    }
}

// Usage
var cart = new ShoppingCart();

// Pay with credit card
cart.SetPaymentStrategy(new CreditCardPayment("1234-5678-9012-3456"));
cart.Checkout(99.99m);
// Output: Paid $99.99 using Credit Card ending in 3456

// Switch to PayPal
cart.SetPaymentStrategy(new PayPalPayment("user@example.com"));
cart.Checkout(49.99m);
// Output: Paid $49.99 using PayPal account user@example.com

// Switch to Crypto
cart.SetPaymentStrategy(new CryptoPayment("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"));
cart.Checkout(199.99m);
// Output: Paid $199.99 in Bitcoin to wallet 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
```

---

## Real-World Example: Sorting Strategies

```csharp
// Strategy interface
public interface ISortStrategy<T> where T : IComparable<T>
{
    void Sort(List<T> data);
}

// Concrete Strategies
public class BubbleSort<T> : ISortStrategy<T> where T : IComparable<T>
{
    public void Sort(List<T> data)
    {
        Console.WriteLine("Sorting using Bubble Sort...");
        for (int i = 0; i < data.Count - 1; i++)
        {
            for (int j = 0; j < data.Count - i - 1; j++)
            {
                if (data[j].CompareTo(data[j + 1]) > 0)
                {
                    var temp = data[j];
                    data[j] = data[j + 1];
                    data[j + 1] = temp;
                }
            }
        }
    }
}

public class QuickSort<T> : ISortStrategy<T> where T : IComparable<T>
{
    public void Sort(List<T> data)
    {
        Console.WriteLine("Sorting using Quick Sort...");
        QuickSortRecursive(data, 0, data.Count - 1);
    }
    
    private void QuickSortRecursive(List<T> data, int low, int high)
    {
        if (low < high)
        {
            int pi = Partition(data, low, high);
            QuickSortRecursive(data, low, pi - 1);
            QuickSortRecursive(data, pi + 1, high);
        }
    }
    
    private int Partition(List<T> data, int low, int high)
    {
        T pivot = data[high];
        int i = low - 1;
        
        for (int j = low; j < high; j++)
        {
            if (data[j].CompareTo(pivot) < 0)
            {
                i++;
                var temp = data[i];
                data[i] = data[j];
                data[j] = temp;
            }
        }
        
        var temp1 = data[i + 1];
        data[i + 1] = data[high];
        data[high] = temp1;
        
        return i + 1;
    }
}

public class MergeSort<T> : ISortStrategy<T> where T : IComparable<T>
{
    public void Sort(List<T> data)
    {
        Console.WriteLine("Sorting using Merge Sort...");
        MergeSortRecursive(data, 0, data.Count - 1);
    }
    
    private void MergeSortRecursive(List<T> data, int left, int right)
    {
        if (left < right)
        {
            int mid = (left + right) / 2;
            MergeSortRecursive(data, left, mid);
            MergeSortRecursive(data, mid + 1, right);
            Merge(data, left, mid, right);
        }
    }
    
    private void Merge(List<T> data, int left, int mid, int right)
    {
        // Merge implementation
        var leftArray = data.GetRange(left, mid - left + 1);
        var rightArray = data.GetRange(mid + 1, right - mid);
        
        int i = 0, j = 0, k = left;
        
        while (i < leftArray.Count && j < rightArray.Count)
        {
            if (leftArray[i].CompareTo(rightArray[j]) <= 0)
                data[k++] = leftArray[i++];
            else
                data[k++] = rightArray[j++];
        }
        
        while (i < leftArray.Count)
            data[k++] = leftArray[i++];
            
        while (j < rightArray.Count)
            data[k++] = rightArray[j++];
    }
}

// Context
public class DataSorter<T> where T : IComparable<T>
{
    private ISortStrategy<T> _sortStrategy;
    
    public void SetSortStrategy(ISortStrategy<T> strategy)
    {
        _sortStrategy = strategy;
    }
    
    public void Sort(List<T> data)
    {
        if (_sortStrategy == null)
            throw new InvalidOperationException("Sort strategy not set");
            
        _sortStrategy.Sort(data);
    }
}

// Usage
var numbers = new List<int> { 64, 34, 25, 12, 22, 11, 90 };
var sorter = new DataSorter<int>();

// Small dataset - use Bubble Sort
if (numbers.Count < 10)
{
    sorter.SetSortStrategy(new BubbleSort<int>());
}
else if (numbers.Count < 1000)
{
    sorter.SetSortStrategy(new QuickSort<int>());
}
else
{
    sorter.SetSortStrategy(new MergeSort<int>());
}

sorter.Sort(numbers);
Console.WriteLine(string.Join(", ", numbers));
```

---

## Real-World Example: Shipping Cost Calculation

```csharp
public interface IShippingStrategy
{
    decimal CalculateCost(decimal weight, decimal distance);
}

public class StandardShipping : IShippingStrategy
{
    public decimal CalculateCost(decimal weight, decimal distance)
    {
        return weight * 0.5m + distance * 0.1m;
    }
}

public class ExpressShipping : IShippingStrategy
{
    public decimal CalculateCost(decimal weight, decimal distance)
    {
        return (weight * 0.8m + distance * 0.2m) * 1.5m; // 50% premium
    }
}

public class InternationalShipping : IShippingStrategy
{
    public decimal CalculateCost(decimal weight, decimal distance)
    {
        return weight * 2.0m + distance * 0.5m + 25m; // Base international fee
    }
}

public class Order
{
    private IShippingStrategy _shippingStrategy;
    
    public decimal Weight { get; set; }
    public decimal Distance { get; set; }
    
    public void SetShippingStrategy(IShippingStrategy strategy)
    {
        _shippingStrategy = strategy;
    }
    
    public decimal CalculateShippingCost()
    {
        return _shippingStrategy.CalculateCost(Weight, Distance);
    }
}

// Usage
var order = new Order { Weight = 10, Distance = 500 };

order.SetShippingStrategy(new StandardShipping());
Console.WriteLine($"Standard: ${order.CalculateShippingCost()}"); // $55.00

order.SetShippingStrategy(new ExpressShipping());
Console.WriteLine($"Express: ${order.CalculateShippingCost()}"); // $165.00

order.SetShippingStrategy(new InternationalShipping());
Console.WriteLine($"International: ${order.CalculateShippingCost()}"); // $295.00
```

---

## Strategy Pattern with Dependency Injection (ASP.NET Core)

```csharp
// Startup.cs or Program.cs
services.AddScoped<IPaymentStrategy, CreditCardPayment>();
services.AddScoped<IPaymentStrategy, PayPalPayment>();

// Using a factory to select strategy
public class PaymentStrategyFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public PaymentStrategyFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public IPaymentStrategy GetStrategy(string paymentType)
    {
        return paymentType switch
        {
            "CreditCard" => _serviceProvider.GetService<CreditCardPayment>(),
            "PayPal" => _serviceProvider.GetService<PayPalPayment>(),
            _ => throw new ArgumentException("Invalid payment type")
        };
    }
}

// Controller
public class CheckoutController : Controller
{
    private readonly PaymentStrategyFactory _factory;
    
    public CheckoutController(PaymentStrategyFactory factory)
    {
        _factory = factory;
    }
    
    [HttpPost]
    public IActionResult ProcessPayment(string paymentType, decimal amount)
    {
        var strategy = _factory.GetStrategy(paymentType);
        strategy.Pay(amount);
        return Ok();
    }
}
```

---

## When to Use

✅ **Use Strategy Pattern when:**
- You have multiple algorithms for a specific task
- You want to avoid complex conditional statements (`if-else`, `switch`)
- Algorithms should be selected at runtime
- You want to isolate algorithm implementation details from client code
- Different variants of an algorithm exist

❌ **Avoid when:**
- You only have one or two algorithms (over-engineering)
- The algorithms are unlikely to change
- The strategy selection logic is simple and stable

---

## Advantages

✅ Eliminates conditional statements  
✅ Follows Open/Closed Principle (easy to add new strategies)  
✅ Runtime algorithm selection  
✅ Better testability (each strategy can be tested independently)  
✅ Promotes composition over inheritance

---

## Disadvantages

❌ Increases number of classes  
❌ Clients must be aware of different strategies  
❌ Can be overkill for simple scenarios  

---

## Strategy vs State Pattern

| Aspect | Strategy | State |
|--------|----------|-------|
| **Purpose** | Select algorithm | Manage state transitions |
| **Who decides** | Client chooses strategy | Context/state decides next state |
| **Independence** | Strategies don't know each other | States may know about transitions |
| **Example** | Payment methods | Order status (Pending → Shipped → Delivered) |

---

## Related Patterns

- **Factory Method** - Can be used to create strategies
- **Template Method** - Defines algorithm skeleton; Strategy defines entire algorithm
- **State** - Similar structure but different intent

---

## Common Use Cases in .NET

- **Validation Strategies** - Different validation rules
- **Compression Algorithms** - ZIP, GZIP, BZIP2
- **Authentication Strategies** - OAuth, JWT, Basic Auth
- **Logging Strategies** - File, Database, Cloud
- **Pricing Strategies** - Discount calculations
- **Serialization** - JSON, XML, Binary
