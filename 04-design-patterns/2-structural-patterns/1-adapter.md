The **Adapter** design pattern is used when you need to convert the interface of a class into another interface that a client expects. This pattern allows classes with incompatible interfaces to work together by providing a wrapper class that adapts one interface to another.

### Scenario:

You have a modern payment gateway interface, but your system still uses an old payment processor. You can use the Adapter pattern to make the old payment processor compatible with the modern interface without changing the existing code.

### 1. **Existing Class (Old Payment Processor)**

This represents a class with an incompatible interface that needs to be adapted.

```csharp
// Old payment processor class with an incompatible interface
public class OldPaymentProcessor
{
    public void ProcessPayment(string account, decimal amount)
    {
        Console.WriteLine($"Processing payment of {amount:C} for account {account} using the old payment processor.");
    }
}
``` 

### 2. **Target Interface (Modern Payment Gateway)**

This defines the interface that the client expects to work with.

```csharp
// Target interface (modern payment gateway)
public interface IPaymentGateway
{
    void MakePayment(string customerId, decimal amount);
}
```

### 3. **Adapter**

This is the adapter class that converts the interface of `OldPaymentProcessor` into the interface expected by the `IPaymentGateway`.

```csharp
// Adapter class
public class PaymentAdapter : IPaymentGateway
{
    private readonly OldPaymentProcessor _oldPaymentProcessor;

    public PaymentAdapter(OldPaymentProcessor oldPaymentProcessor)
    {
        _oldPaymentProcessor = oldPaymentProcessor;
    }

    // Adapting the old interface to the modern interface
    public void MakePayment(string customerId, decimal amount)
    {
        // In the old system, customerId is referred to as account
        _oldPaymentProcessor.ProcessPayment(customerId, amount);
    }
}
```

### 4. **Client Code**

The client uses the `IPaymentGateway` interface, but through the adapter, it can work with the `OldPaymentProcessor` without knowing the details.

```csharp
class Program
{
    static void Main(string[] args)
    {
        // The old payment processor that needs to be adapted
        OldPaymentProcessor oldPaymentProcessor = new OldPaymentProcessor();

        // Using the adapter to make the old payment processor compatible with the new interface
        IPaymentGateway paymentGateway = new PaymentAdapter(oldPaymentProcessor);

        // Client interacts with the new interface (IPaymentGateway)
        paymentGateway.MakePayment("Customer123", 100.50m);
    }
} 
```

### **Output**:

    Processing payment of $100.50 for account Customer123 using the old payment processor. 

### Explanation:

1.  **Old Payment Processor (`OldPaymentProcessor`)**: This class has a method `ProcessPayment()`, which accepts the account and amount, but its interface is incompatible with what the modern system expects.
2.  **Modern Payment Gateway (`IPaymentGateway`)**: This is the interface that the client expects to work with. It has a method `MakePayment()` which accepts a `customerId` and `amount`.
3.  **Adapter (`PaymentAdapter`)**: The adapter converts the method call from `MakePayment()` into `ProcessPayment()` by wrapping the old payment processor.
4.  **Client**: The client interacts with the `IPaymentGateway` interface, but thanks to the adapter, it can work with the old payment processor.

### Why Use Adapter:

-   The Adapter pattern allows you to **reuse existing incompatible classes** without modifying their code.
-   It helps integrate **legacy systems** or third-party libraries into a new system by adapting their interfaces to match what the client expects.
