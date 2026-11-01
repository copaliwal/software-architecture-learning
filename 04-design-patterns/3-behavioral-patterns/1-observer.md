# Observer Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Also Known As:** Publish-Subscribe, Event-Listener

---

## Problem

You need to notify multiple objects about state changes in another object without tightly coupling them together. Direct dependencies make the code rigid and hard to extend.

---

## Solution

The Observer pattern establishes a subscription mechanism where:
- **Subject (Observable)** - maintains a list of observers and notifies them of state changes
- **Observers** - register with the subject and receive notifications when changes occur

---

## Structure

```
Subject (Observable)
├── Attach(Observer)
├── Detach(Observer)
└── Notify()

ConcreteSubject
└── State changes trigger Notify()

Observer Interface
└── Update()

ConcreteObserver
└── Update() implementation
```

---

## C# Implementation

### Basic Implementation

```csharp
// Observer interface
public interface IObserver
{
    void Update(string message);
}

// Subject (Observable)
public class Subject
{
    private List<IObserver> _observers = new List<IObserver>();
    
    public void Attach(IObserver observer)
    {
        _observers.Add(observer);
    }
    
    public void Detach(IObserver observer)
    {
        _observers.Remove(observer);
    }
    
    protected void Notify(string message)
    {
        foreach (var observer in _observers)
        {
            observer.Update(message);
        }
    }
}

// Concrete Subject
public class WeatherStation : Subject
{
    private float _temperature;
    
    public float Temperature
    {
        get => _temperature;
        set
        {
            _temperature = value;
            Notify($"Temperature changed to {_temperature}°C");
        }
    }
}

// Concrete Observers
public class PhoneDisplay : IObserver
{
    public void Update(string message)
    {
        Console.WriteLine($"[Phone Display] {message}");
    }
}

public class WindowDisplay : IObserver
{
    public void Update(string message)
    {
        Console.WriteLine($"[Window Display] {message}");
    }
}

// Usage
var weatherStation = new WeatherStation();

var phoneDisplay = new PhoneDisplay();
var windowDisplay = new WindowDisplay();

weatherStation.Attach(phoneDisplay);
weatherStation.Attach(windowDisplay);

weatherStation.Temperature = 25.5f; 
// Output:
// [Phone Display] Temperature changed to 25.5°C
// [Window Display] Temperature changed to 25.5°C

weatherStation.Detach(phoneDisplay);
weatherStation.Temperature = 30.0f;
// Output:
// [Window Display] Temperature changed to 30.0°C
```

---

## Real-World Example: Stock Price Monitoring

```csharp
// Observer interface with typed data
public interface IStockObserver
{
    void OnPriceChanged(string symbol, decimal price);
}

// Subject
public class Stock
{
    private readonly List<IStockObserver> _observers = new();
    private decimal _price;
    
    public string Symbol { get; }
    
    public decimal Price
    {
        get => _price;
        set
        {
            if (_price != value)
            {
                _price = value;
                NotifyObservers();
            }
        }
    }
    
    public Stock(string symbol, decimal initialPrice)
    {
        Symbol = symbol;
        _price = initialPrice;
    }
    
    public void Subscribe(IStockObserver observer)
    {
        if (!_observers.Contains(observer))
            _observers.Add(observer);
    }
    
    public void Unsubscribe(IStockObserver observer)
    {
        _observers.Remove(observer);
    }
    
    private void NotifyObservers()
    {
        foreach (var observer in _observers)
        {
            observer.OnPriceChanged(Symbol, Price);
        }
    }
}

// Concrete Observers
public class StockPortfolio : IStockObserver
{
    private readonly string _ownerName;
    
    public StockPortfolio(string ownerName)
    {
        _ownerName = ownerName;
    }
    
    public void OnPriceChanged(string symbol, decimal price)
    {
        Console.WriteLine($"[Portfolio - {_ownerName}] {symbol} is now ${price}");
    }
}

public class PriceAlert : IStockObserver
{
    private readonly decimal _threshold;
    
    public PriceAlert(decimal threshold)
    {
        _threshold = threshold;
    }
    
    public void OnPriceChanged(string symbol, decimal price)
    {
        if (price > _threshold)
        {
            Console.WriteLine($"⚠️ ALERT: {symbol} exceeded ${_threshold}! Current: ${price}");
        }
    }
}

// Usage
var appleStock = new Stock("AAPL", 150.00m);

var myPortfolio = new StockPortfolio("John Doe");
var priceAlert = new PriceAlert(180.00m);

appleStock.Subscribe(myPortfolio);
appleStock.Subscribe(priceAlert);

appleStock.Price = 175.00m;
// Output: [Portfolio - John Doe] AAPL is now $175.00

appleStock.Price = 185.00m;
// Output: 
// [Portfolio - John Doe] AAPL is now $185.00
// ⚠️ ALERT: AAPL exceeded $180.00! Current: $185.00
```

---

## .NET Built-in Observer Pattern

### Using `IObservable<T>` and `IObserver<T>`

```csharp
using System;
using System.Reactive.Linq;

// Subject using IObservable
public class TemperatureSensor : IObservable<float>
{
    private List<IObserver<float>> _observers = new();
    
    public IDisposable Subscribe(IObserver<float> observer)
    {
        if (!_observers.Contains(observer))
            _observers.Add(observer);
        
        return new Unsubscriber(_observers, observer);
    }
    
    public void SetTemperature(float temp)
    {
        foreach (var observer in _observers)
        {
            observer.OnNext(temp);
        }
    }
    
    // Helper class for unsubscribing
    private class Unsubscriber : IDisposable
    {
        private List<IObserver<float>> _observers;
        private IObserver<float> _observer;
        
        public Unsubscriber(List<IObserver<float>> observers, IObserver<float> observer)
        {
            _observers = observers;
            _observer = observer;
        }
        
        public void Dispose()
        {
            if (_observer != null && _observers.Contains(_observer))
                _observers.Remove(_observer);
        }
    }
}

// Observer
public class TemperatureDisplay : IObserver<float>
{
    private IDisposable _unsubscriber;
    
    public void Subscribe(IObservable<float> provider)
    {
        _unsubscriber = provider.Subscribe(this);
    }
    
    public void OnNext(float value)
    {
        Console.WriteLine($"Current temperature: {value}°C");
    }
    
    public void OnError(Exception error)
    {
        Console.WriteLine($"Error: {error.Message}");
    }
    
    public void OnCompleted()
    {
        Console.WriteLine("Temperature monitoring completed.");
        _unsubscriber?.Dispose();
    }
}

// Usage
var sensor = new TemperatureSensor();
var display = new TemperatureDisplay();

display.Subscribe(sensor);

sensor.SetTemperature(20.5f); // Current temperature: 20.5°C
sensor.SetTemperature(22.0f); // Current temperature: 22.0°C
```

---

## C# Events (Built-in Observer Pattern)

```csharp
// Using C# events (most common in .NET)
public class Button
{
    // Event declaration
    public event EventHandler<ButtonClickedEventArgs> Clicked;
    
    public void Click()
    {
        OnClicked(new ButtonClickedEventArgs { Timestamp = DateTime.Now });
    }
    
    protected virtual void OnClicked(ButtonClickedEventArgs e)
    {
        Clicked?.Invoke(this, e);
    }
}

public class ButtonClickedEventArgs : EventArgs
{
    public DateTime Timestamp { get; set; }
}

// Observers (event handlers)
public class Logger
{
    public void OnButtonClicked(object sender, ButtonClickedEventArgs e)
    {
        Console.WriteLine($"[Logger] Button clicked at {e.Timestamp}");
    }
}

public class Analytics
{
    public void OnButtonClicked(object sender, ButtonClickedEventArgs e)
    {
        Console.WriteLine($"[Analytics] Recording click event at {e.Timestamp}");
    }
}

// Usage
var button = new Button();
var logger = new Logger();
var analytics = new Analytics();

// Subscribe to the event
button.Clicked += logger.OnButtonClicked;
button.Clicked += analytics.OnButtonClicked;

button.Click();
// Output:
// [Logger] Button clicked at 12/16/2025 10:30:00 AM
// [Analytics] Recording click event at 12/16/2025 10:30:00 AM

// Unsubscribe
button.Clicked -= logger.OnButtonClicked;

button.Click();
// Output:
// [Analytics] Recording click event at 12/16/2025 10:30:01 AM
```

---

## When to Use

✅ **Use Observer Pattern when:**
- Multiple objects need to react to state changes in another object
- You want loose coupling between objects
- The number of observers is dynamic (can change at runtime)
- You're implementing event-driven systems
- Building UI applications with data binding

❌ **Avoid when:**
- Notification order matters (observers are notified in arbitrary order)
- You need guaranteed delivery or transactional notifications
- The relationship is always one-to-one (use direct method calls instead)

---

## Advantages

✅ Loose coupling between subject and observers  
✅ Supports broadcast communication  
✅ Dynamic relationships (add/remove observers at runtime)  
✅ Follows Open/Closed Principle (easy to add new observers)

---

## Disadvantages

❌ Observers are notified in random order  
❌ Memory leaks if observers aren't unsubscribed  
❌ Can cause unexpected updates if not careful  
❌ Debugging can be harder (indirect flow of execution)

---

## Related Patterns

- **Mediator** - Similar but centralizes communication through a mediator object
- **Publish-Subscribe** - More decoupled version using message brokers
- **Event Sourcing** - Uses events to track state changes

---

## Common Use Cases in .NET

- **WPF/XAML Data Binding** - `INotifyPropertyChanged`
- **ASP.NET Core SignalR** - Real-time notifications
- **Reactive Extensions (Rx.NET)** - `IObservable<T>` / `IObserver<T>`
- **C# Events** - Button clicks, property changes
- **Domain Events in DDD** - Domain state change notifications
