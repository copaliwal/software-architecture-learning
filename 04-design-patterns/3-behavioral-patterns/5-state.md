# State Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

---

## Problem

An object's behavior depends on its state, and it must change behavior at runtime based on that state. Using large conditional statements (if/switch) leads to:
- Complex, hard-to-maintain code
- Violations of Single Responsibility and Open/Closed principles
- Difficult testing of individual states
- State transitions scattered throughout code

---

## Solution

Encapsulate each state in a separate class that implements a common interface. The context object delegates behavior to the current state object, making state transitions explicit and behavior state-specific.

**Key Concept:** "The object will appear to change its class" - same object, different behavior based on internal state object.

---

## Structure

```
Context
├── Maintains reference to current State
├── Delegates operations to current state
└── SetState(state)

State (interface)
└── Handle() - State-specific behavior

ConcreteStateA
├── Implements Handle()
└── Can trigger state transitions

ConcreteStateB
├── Implements Handle()
└── Can trigger state transitions
```

---

## C# Implementation

### Example: Document Workflow

```csharp
// Context
public class Document
{
    public string Name { get; set; }
    public string Content { get; set; }
    private IDocumentState _state;
    
    public Document(string name, string content)
    {
        Name = name;
        Content = content;
        _state = new DraftState();  // Initial state
        Console.WriteLine($"Document '{name}' created in Draft state");
    }
    
    public void SetState(IDocumentState state)
    {
        _state = state;
        Console.WriteLine($"Document state changed to: {state.GetType().Name}");
    }
    
    // Delegate operations to current state
    public void Publish()
    {
        _state.Publish(this);
    }
    
    public void Review()
    {
        _state.Review(this);
    }
    
    public void Approve()
    {
        _state.Approve(this);
    }
    
    public void Reject()
    {
        _state.Reject(this);
    }
}

// State interface
public interface IDocumentState
{
    void Publish(Document document);
    void Review(Document document);
    void Approve(Document document);
    void Reject(Document document);
}

// Concrete State: Draft
public class DraftState : IDocumentState
{
    public void Publish(Document document)
    {
        Console.WriteLine("Cannot publish from Draft. Document must be reviewed first.");
    }
    
    public void Review(Document document)
    {
        Console.WriteLine("Submitting document for review...");
        document.SetState(new UnderReviewState());
    }
    
    public void Approve(Document document)
    {
        Console.WriteLine("Cannot approve a draft document.");
    }
    
    public void Reject(Document document)
    {
        Console.WriteLine("Draft document deleted.");
    }
}

// Concrete State: Under Review
public class UnderReviewState : IDocumentState
{
    public void Publish(Document document)
    {
        Console.WriteLine("Cannot publish while under review.");
    }
    
    public void Review(Document document)
    {
        Console.WriteLine("Document is already under review.");
    }
    
    public void Approve(Document document)
    {
        Console.WriteLine("Document approved! Moving to published state.");
        document.SetState(new PublishedState());
    }
    
    public void Reject(Document document)
    {
        Console.WriteLine("Document rejected. Sending back to draft.");
        document.SetState(new DraftState());
    }
}

// Concrete State: Published
public class PublishedState : IDocumentState
{
    public void Publish(Document document)
    {
        Console.WriteLine("Document is already published.");
    }
    
    public void Review(Document document)
    {
        Console.WriteLine("Published documents cannot be reviewed. Create a new version.");
    }
    
    public void Approve(Document document)
    {
        Console.WriteLine("Document is already approved and published.");
    }
    
    public void Reject(Document document)
    {
        Console.WriteLine("Unpublishing document...");
        document.SetState(new DraftState());
    }
}

// Usage
var doc = new Document("Design Proposal", "This is the content...");

Console.WriteLine("\n--- Workflow ---");
doc.Publish();    // Cannot publish from Draft
doc.Review();     // Draft -> Under Review
doc.Publish();    // Cannot publish while under review
doc.Approve();    // Under Review -> Published
doc.Publish();    // Already published
doc.Reject();     // Published -> Draft

// Output:
// Document 'Design Proposal' created in Draft state
// 
// --- Workflow ---
// Cannot publish from Draft. Document must be reviewed first.
// Submitting document for review...
// Document state changed to: UnderReviewState
// Cannot publish while under review.
// Document approved! Moving to published state.
// Document state changed to: PublishedState
// Document is already published.
// Unpublishing document...
// Document state changed to: DraftState
```

---

## Real-World Example: ATM Machine

```csharp
public class ATM
{
    private IATMState _state;
    private decimal _balance;
    private bool _hasCard;
    
    public ATM(decimal initialBalance)
    {
        _balance = initialBalance;
        _state = new NoCardState();
        _hasCard = false;
    }
    
    public void SetState(IATMState state)
    {
        _state = state;
    }
    
    public void InsertCard()
    {
        _state.InsertCard(this);
    }
    
    public void EjectCard()
    {
        _state.EjectCard(this);
    }
    
    public void EnterPin(string pin)
    {
        _state.EnterPin(this, pin);
    }
    
    public void WithdrawCash(decimal amount)
    {
        _state.WithdrawCash(this, amount);
    }
    
    // Internal methods used by states
    internal bool ValidatePin(string pin) => pin == "1234";
    
    internal bool HasSufficientBalance(decimal amount) => _balance >= amount;
    
    internal void DeductBalance(decimal amount)
    {
        _balance -= amount;
        Console.WriteLine($"Balance: ${_balance}");
    }
    
    internal void MarkCardInserted() => _hasCard = true;
    
    internal void MarkCardEjected() => _hasCard = false;
}

public interface IATMState
{
    void InsertCard(ATM atm);
    void EjectCard(ATM atm);
    void EnterPin(ATM atm, string pin);
    void WithdrawCash(ATM atm, decimal amount);
}

// State 1: No Card
public class NoCardState : IATMState
{
    public void InsertCard(ATM atm)
    {
        Console.WriteLine("Card inserted. Please enter PIN.");
        atm.MarkCardInserted();
        atm.SetState(new HasCardState());
    }
    
    public void EjectCard(ATM atm)
    {
        Console.WriteLine("No card to eject.");
    }
    
    public void EnterPin(ATM atm, string pin)
    {
        Console.WriteLine("Please insert card first.");
    }
    
    public void WithdrawCash(ATM atm, decimal amount)
    {
        Console.WriteLine("Please insert card first.");
    }
}

// State 2: Has Card (waiting for PIN)
public class HasCardState : IATMState
{
    private int _pinAttempts = 0;
    
    public void InsertCard(ATM atm)
    {
        Console.WriteLine("Card already inserted.");
    }
    
    public void EjectCard(ATM atm)
    {
        Console.WriteLine("Card ejected.");
        atm.MarkCardEjected();
        atm.SetState(new NoCardState());
    }
    
    public void EnterPin(ATM atm, string pin)
    {
        if (atm.ValidatePin(pin))
        {
            Console.WriteLine("PIN accepted. You can now withdraw cash.");
            atm.SetState(new AuthenticatedState());
        }
        else
        {
            _pinAttempts++;
            if (_pinAttempts >= 3)
            {
                Console.WriteLine("Too many incorrect PIN attempts. Card blocked.");
                atm.SetState(new CardBlockedState());
            }
            else
            {
                Console.WriteLine($"Incorrect PIN. {3 - _pinAttempts} attempts remaining.");
            }
        }
    }
    
    public void WithdrawCash(ATM atm, decimal amount)
    {
        Console.WriteLine("Please enter PIN first.");
    }
}

// State 3: Authenticated (can withdraw)
public class AuthenticatedState : IATMState
{
    public void InsertCard(ATM atm)
    {
        Console.WriteLine("Card already inserted and authenticated.");
    }
    
    public void EjectCard(ATM atm)
    {
        Console.WriteLine("Thank you! Card ejected.");
        atm.MarkCardEjected();
        atm.SetState(new NoCardState());
    }
    
    public void EnterPin(ATM atm, string pin)
    {
        Console.WriteLine("Already authenticated.");
    }
    
    public void WithdrawCash(ATM atm, decimal amount)
    {
        if (amount <= 0)
        {
            Console.WriteLine("Invalid amount.");
            return;
        }
        
        if (atm.HasSufficientBalance(amount))
        {
            Console.WriteLine($"Dispensing ${amount}...");
            atm.DeductBalance(amount);
            Console.WriteLine("Please take your cash.");
        }
        else
        {
            Console.WriteLine("Insufficient balance.");
        }
    }
}

// State 4: Card Blocked
public class CardBlockedState : IATMState
{
    public void InsertCard(ATM atm)
    {
        Console.WriteLine("This card is blocked. Please contact your bank.");
    }
    
    public void EjectCard(ATM atm)
    {
        Console.WriteLine("Card ejected (blocked).");
        atm.MarkCardEjected();
        atm.SetState(new NoCardState());
    }
    
    public void EnterPin(ATM atm, string pin)
    {
        Console.WriteLine("Card is blocked.");
    }
    
    public void WithdrawCash(ATM atm, decimal amount)
    {
        Console.WriteLine("Card is blocked.");
    }
}

// Usage
var atm = new ATM(1000);

atm.WithdrawCash(100);      // Please insert card first
atm.InsertCard();           // Card inserted
atm.WithdrawCash(100);      // Please enter PIN first
atm.EnterPin("0000");       // Incorrect PIN
atm.EnterPin("1234");       // PIN accepted
atm.WithdrawCash(200);      // Dispensing $200
atm.WithdrawCash(2000);     // Insufficient balance
atm.EjectCard();            // Thank you! Card ejected
```

---

## Order Processing Example

```csharp
public class Order
{
    public int OrderId { get; set; }
    public decimal Total { get; set; }
    private IOrderState _state;
    
    public Order(int orderId, decimal total)
    {
        OrderId = orderId;
        Total = total;
        _state = new NewOrderState();
    }
    
    public void SetState(IOrderState state)
    {
        _state = state;
        Console.WriteLine($"Order {OrderId} -> {state.GetType().Name.Replace("State", "")}");
    }
    
    public void Confirm() => _state.Confirm(this);
    public void Ship() => _state.Ship(this);
    public void Deliver() => _state.Deliver(this);
    public void Cancel() => _state.Cancel(this);
}

public interface IOrderState
{
    void Confirm(Order order);
    void Ship(Order order);
    void Deliver(Order order);
    void Cancel(Order order);
}

public class NewOrderState : IOrderState
{
    public void Confirm(Order order)
    {
        Console.WriteLine("Order confirmed. Processing payment...");
        order.SetState(new ConfirmedState());
    }
    
    public void Ship(Order order) => Console.WriteLine("Cannot ship unconfirmed order.");
    public void Deliver(Order order) => Console.WriteLine("Cannot deliver unconfirmed order.");
    
    public void Cancel(Order order)
    {
        Console.WriteLine("Order cancelled.");
        order.SetState(new CancelledState());
    }
}

public class ConfirmedState : IOrderState
{
    public void Confirm(Order order) => Console.WriteLine("Order already confirmed.");
    
    public void Ship(Order order)
    {
        Console.WriteLine("Order shipped. Tracking number: ABC123");
        order.SetState(new ShippedState());
    }
    
    public void Deliver(Order order) => Console.WriteLine("Cannot deliver unshipped order.");
    
    public void Cancel(Order order)
    {
        Console.WriteLine("Cancelling confirmed order. Refund initiated.");
        order.SetState(new CancelledState());
    }
}

public class ShippedState : IOrderState
{
    public void Confirm(Order order) => Console.WriteLine("Order already confirmed and shipped.");
    public void Ship(Order order) => Console.WriteLine("Order already shipped.");
    
    public void Deliver(Order order)
    {
        Console.WriteLine("Order delivered successfully!");
        order.SetState(new DeliveredState());
    }
    
    public void Cancel(Order order) => Console.WriteLine("Cannot cancel shipped order. Contact support for returns.");
}

public class DeliveredState : IOrderState
{
    public void Confirm(Order order) => Console.WriteLine("Order already delivered.");
    public void Ship(Order order) => Console.WriteLine("Order already delivered.");
    public void Deliver(Order order) => Console.WriteLine("Order already delivered.");
    public void Cancel(Order order) => Console.WriteLine("Cannot cancel delivered order. Request a return instead.");
}

public class CancelledState : IOrderState
{
    public void Confirm(Order order) => Console.WriteLine("Cannot confirm cancelled order.");
    public void Ship(Order order) => Console.WriteLine("Cannot ship cancelled order.");
    public void Deliver(Order order) => Console.WriteLine("Cannot deliver cancelled order.");
    public void Cancel(Order order) => Console.WriteLine("Order already cancelled.");
}

// Usage
var order = new Order(1001, 299.99m);

order.Ship();        // Cannot ship unconfirmed order
order.Confirm();     // New -> Confirmed
order.Confirm();     // Already confirmed
order.Ship();        // Confirmed -> Shipped
order.Cancel();      // Cannot cancel shipped order
order.Deliver();     // Shipped -> Delivered
order.Cancel();      // Cannot cancel delivered order
```

---

## Connection State Example (TCP-like)

```csharp
public class Connection
{
    private IConnectionState _state;
    public string RemoteAddress { get; }
    
    public Connection(string remoteAddress)
    {
        RemoteAddress = remoteAddress;
        _state = new ClosedState();
    }
    
    public void SetState(IConnectionState state)
    {
        _state = state;
        Console.WriteLine($"Connection state: {state.GetType().Name.Replace("State", "")}");
    }
    
    public void Open() => _state.Open(this);
    public void Close() => _state.Close(this);
    public void Send(string data) => _state.Send(this, data);
    public void Receive() => _state.Receive(this);
}

public interface IConnectionState
{
    void Open(Connection connection);
    void Close(Connection connection);
    void Send(Connection connection, string data);
    void Receive(Connection connection);
}

public class ClosedState : IConnectionState
{
    public void Open(Connection connection)
    {
        Console.WriteLine($"Opening connection to {connection.RemoteAddress}...");
        connection.SetState(new ConnectingState());
        Thread.Sleep(1000);  // Simulate connection delay
        connection.SetState(new OpenState());
    }
    
    public void Close(Connection connection) => Console.WriteLine("Connection already closed.");
    public void Send(Connection connection, string data) => Console.WriteLine("Cannot send. Connection is closed.");
    public void Receive(Connection connection) => Console.WriteLine("Cannot receive. Connection is closed.");
}

public class ConnectingState : IConnectionState
{
    public void Open(Connection connection) => Console.WriteLine("Already connecting...");
    public void Close(Connection connection)
    {
        Console.WriteLine("Aborting connection...");
        connection.SetState(new ClosedState());
    }
    
    public void Send(Connection connection, string data) => Console.WriteLine("Connection not established yet.");
    public void Receive(Connection connection) => Console.WriteLine("Connection not established yet.");
}

public class OpenState : IConnectionState
{
    public void Open(Connection connection) => Console.WriteLine("Connection already open.");
    
    public void Close(Connection connection)
    {
        Console.WriteLine("Closing connection...");
        connection.SetState(new ClosedState());
    }
    
    public void Send(Connection connection, string data)
    {
        Console.WriteLine($"Sending: {data}");
    }
    
    public void Receive(Connection connection)
    {
        Console.WriteLine("Receiving data...");
    }
}

// Usage
var conn = new Connection("192.168.1.100:8080");

conn.Send("Hello");     // Cannot send. Connection is closed.
conn.Open();            // Closed -> Connecting -> Open
conn.Send("Hello");     // Sending: Hello
conn.Receive();         // Receiving data...
conn.Close();           // Open -> Closed
conn.Send("World");     // Cannot send. Connection is closed.
```

---

## When to Use State Pattern

✅ **Use when:**
- Object behavior depends on its state
- Operations have large conditional statements based on state
- State transitions are explicit and complex
- States have different valid operations

❌ **Avoid when:**
- Few states with simple transitions
- State logic is trivial
- States don't have different behavior

---

## Advantages

✅ **Single Responsibility** - Each state is a separate class  
✅ **Open/Closed** - Add new states without modifying existing ones  
✅ **Explicit state transitions** - Clear where transitions happen  
✅ **Eliminates conditionals** - No giant if/switch statements

---

## Disadvantages

❌ **Increased classes** - Each state is a separate class  
❌ **Overkill for simple cases** - Simple enums might suffice  
❌ **State explosion** - Many states = many classes

---

## State vs Strategy

| Aspect | State | Strategy |
|--------|-------|----------|
| **Intent** | Change behavior based on state | Select algorithm at runtime |
| **Context awareness** | States know about context and other states | Strategies are independent |
| **Transitions** | States can trigger transitions | Strategies don't change other strategies |
| **Encapsulation** | Encapsulates state-specific behavior | Encapsulates algorithms |

---

## Related Patterns

- **Strategy** - Similar structure; State focuses on state-dependent behavior
- **Singleton** - State objects are often singletons
- **Flyweight** - State objects can be shared if stateless

---

## Common Use Cases

- **Workflow engines** - Document approval, order processing
- **Game development** - Character states (idle, running, jumping)
- **Network protocols** - TCP states (connecting, open, closed)
- **UI controls** - Button states (normal, hover, pressed, disabled)
- **Transaction processing** - Begin, active, committed, aborted
- **Media players** - Playing, paused, stopped

---

## Best Practices

✅ Make state objects **stateless** (share via singleton)  
✅ Clearly define **valid state transitions**  
✅ Use **enums + state classes** together for clarity  
✅ Consider **state machines** for complex workflows  
✅ Document **state diagram** for team understanding  
✅ Validate **invalid state transitions** explicitly