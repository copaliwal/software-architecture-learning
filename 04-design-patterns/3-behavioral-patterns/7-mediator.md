# Mediator Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly.

---

## Problem

When many objects communicate with each other directly, the system becomes:
- **Tightly coupled** - Objects depend on many other objects
- **Hard to maintain** - Changes ripple through multiple classes
- **Complex** - Communication logic is scattered everywhere
- **Difficult to reuse** - Objects can't be used independently

Direct object-to-object communication creates a tangled web of dependencies.

---

## Solution

Create a mediator object that centralizes communication. Objects:
1. Don't communicate directly with each other
2. Send messages to the mediator
3. The mediator forwards messages to appropriate recipients

This reduces dependencies from N² (many-to-many) to N (many-to-one).

---

## Structure

```
Mediator (interface)
└── Notify(sender, event)

ConcreteMediator
├── Maintains references to components
└── Implements communication logic

Component
├── Holds reference to mediator
└── Notifies mediator of events

ConcreteComponentA
└── Calls mediator.Notify()

ConcreteComponentB
└── Calls mediator.Notify()
```

---

## C# Implementation

### Example: Chat Room

```csharp
// Mediator interface
public interface IChatMediator
{
    void SendMessage(string message, User user);
    void AddUser(User user);
}

// Concrete Mediator
public class ChatRoom : IChatMediator
{
    private readonly List<User> _users = new();
    
    public void AddUser(User user)
    {
        _users.Add(user);
        Console.WriteLine($"{user.Name} joined the chat room");
    }
    
    public void SendMessage(string message, User sender)
    {
        // Broadcast to all users except sender
        foreach (var user in _users)
        {
            if (user != sender)
            {
                user.Receive(message, sender.Name);
            }
        }
    }
}

// Component (Colleague)
public class User
{
    public string Name { get; }
    private readonly IChatMediator _mediator;
    
    public User(string name, IChatMediator mediator)
    {
        Name = name;
        _mediator = mediator;
    }
    
    public void Send(string message)
    {
        Console.WriteLine($"{Name} sends: {message}");
        _mediator.SendMessage(message, this);
    }
    
    public void Receive(string message, string from)
    {
        Console.WriteLine($"{Name} received from {from}: {message}");
    }
}

// Usage
var chatRoom = new ChatRoom();

var alice = new User("Alice", chatRoom);
var bob = new User("Bob", chatRoom);
var charlie = new User("Charlie", chatRoom);

chatRoom.AddUser(alice);
chatRoom.AddUser(bob);
chatRoom.AddUser(charlie);

Console.WriteLine();

alice.Send("Hello everyone!");
Console.WriteLine();

bob.Send("Hi Alice!");
Console.WriteLine();

charlie.Send("Good morning!");

// Output:
// Alice joined the chat room
// Bob joined the chat room
// Charlie joined the chat room
// 
// Alice sends: Hello everyone!
// Bob received from Alice: Hello everyone!
// Charlie received from Alice: Hello everyone!
// 
// Bob sends: Hi Alice!
// Alice received from Bob: Hi Alice!
// Charlie received from Bob: Hi Alice!
// 
// Charlie sends: Good morning!
// Alice received from Charlie: Good morning!
// Bob received from Charlie: Good morning!
```

---

## Real-World Example: Air Traffic Control

```csharp
// Mediator
public interface IATCMediator
{
    void RequestLanding(Aircraft aircraft);
    void RequestTakeoff(Aircraft aircraft);
    void RegisterAircraft(Aircraft aircraft);
}

// Concrete Mediator
public class AirTrafficControl : IATCMediator
{
    private readonly List<Aircraft> _aircrafts = new();
    private bool _runwayOccupied = false;
    
    public void RegisterAircraft(Aircraft aircraft)
    {
        _aircrafts.Add(aircraft);
        Console.WriteLine($"[ATC] {aircraft.FlightNumber} registered");
    }
    
    public void RequestLanding(Aircraft aircraft)
    {
        Console.WriteLine($"[ATC] {aircraft.FlightNumber} requesting landing...");
        
        if (_runwayOccupied)
        {
            aircraft.ReceiveMessage("Landing denied. Runway occupied. Please circle.");
        }
        else
        {
            _runwayOccupied = true;
            aircraft.ReceiveMessage("Landing approved. Runway clear.");
            
            // Notify other aircraft
            foreach (var other in _aircrafts)
            {
                if (other != aircraft)
                {
                    other.ReceiveMessage($"Runway now occupied by {aircraft.FlightNumber}");
                }
            }
        }
    }
    
    public void RequestTakeoff(Aircraft aircraft)
    {
        Console.WriteLine($"[ATC] {aircraft.FlightNumber} requesting takeoff...");
        
        if (_runwayOccupied)
        {
            aircraft.ReceiveMessage("Takeoff denied. Runway occupied.");
        }
        else
        {
            _runwayOccupied = true;
            aircraft.ReceiveMessage("Takeoff approved. Cleared for departure.");
            
            // Notify other aircraft
            foreach (var other in _aircrafts)
            {
                if (other != aircraft)
                {
                    other.ReceiveMessage($"{aircraft.FlightNumber} is taking off");
                }
            }
            
            // Simulate takeoff completion
            Task.Delay(2000).ContinueWith(_ =>
            {
                _runwayOccupied = false;
                Console.WriteLine($"[ATC] Runway now clear");
            });
        }
    }
}

// Component
public class Aircraft
{
    public string FlightNumber { get; }
    private readonly IATCMediator _mediator;
    
    public Aircraft(string flightNumber, IATCMediator mediator)
    {
        FlightNumber = flightNumber;
        _mediator = mediator;
    }
    
    public void RequestLanding()
    {
        _mediator.RequestLanding(this);
    }
    
    public void RequestTakeoff()
    {
        _mediator.RequestTakeoff(this);
    }
    
    public void ReceiveMessage(string message)
    {
        Console.WriteLine($"[{FlightNumber}] {message}");
    }
}

// Usage
var atc = new AirTrafficControl();

var flight123 = new Aircraft("AA123", atc);
var flight456 = new Aircraft("UA456", atc);
var flight789 = new Aircraft("DL789", atc);

atc.RegisterAircraft(flight123);
atc.RegisterAircraft(flight456);
atc.RegisterAircraft(flight789);

Console.WriteLine();

flight123.RequestLanding();
Console.WriteLine();

flight456.RequestLanding();  // Will be denied
Console.WriteLine();

flight789.RequestTakeoff();  // Will be denied
```

---

## UI Example: Form Components

```csharp
// Mediator
public interface IFormMediator
{
    void ComponentChanged(FormComponent component, string value);
}

// Concrete Mediator
public class RegistrationFormMediator : IFormMediator
{
    private TextBox _usernameTextBox;
    private TextBox _passwordTextBox;
    private CheckBox _termsCheckBox;
    private Button _submitButton;
    
    public void SetComponents(TextBox username, TextBox password, CheckBox terms, Button submit)
    {
        _usernameTextBox = username;
        _passwordTextBox = password;
        _termsCheckBox = terms;
        _submitButton = submit;
        
        // Initial state
        UpdateSubmitButton();
    }
    
    public void ComponentChanged(FormComponent component, string value)
    {
        Console.WriteLine($"[Mediator] {component.Name} changed to: {value}");
        
        // Coordinate component interactions
        if (component == _usernameTextBox)
        {
            // Update password requirements based on username
            if (value.Length < 3)
            {
                _usernameTextBox.SetError("Username too short");
            }
            else
            {
                _usernameTextBox.ClearError();
            }
        }
        
        if (component == _passwordTextBox)
        {
            // Validate password strength
            if (value.Length < 8)
            {
                _passwordTextBox.SetError("Password must be at least 8 characters");
            }
            else
            {
                _passwordTextBox.ClearError();
            }
        }
        
        // Update submit button state
        UpdateSubmitButton();
    }
    
    private void UpdateSubmitButton()
    {
        var isValid = !string.IsNullOrEmpty(_usernameTextBox.Value) &&
                      !string.IsNullOrEmpty(_passwordTextBox.Value) &&
                      _termsCheckBox.IsChecked &&
                      !_usernameTextBox.HasError &&
                      !_passwordTextBox.HasError;
        
        _submitButton.SetEnabled(isValid);
        Console.WriteLine($"[Mediator] Submit button {(isValid ? "enabled" : "disabled")}");
    }
}

// Component base
public abstract class FormComponent
{
    public string Name { get; }
    protected IFormMediator Mediator { get; }
    
    protected FormComponent(string name, IFormMediator mediator)
    {
        Name = name;
        Mediator = mediator;
    }
}

// Concrete Components
public class TextBox : FormComponent
{
    public string Value { get; private set; }
    public bool HasError { get; private set; }
    
    public TextBox(string name, IFormMediator mediator) : base(name, mediator) { }
    
    public void SetValue(string value)
    {
        Value = value;
        Mediator.ComponentChanged(this, value);
    }
    
    public void SetError(string error)
    {
        HasError = true;
        Console.WriteLine($"[{Name}] Error: {error}");
    }
    
    public void ClearError()
    {
        HasError = false;
    }
}

public class CheckBox : FormComponent
{
    public bool IsChecked { get; private set; }
    
    public CheckBox(string name, IFormMediator mediator) : base(name, mediator) { }
    
    public void SetChecked(bool isChecked)
    {
        IsChecked = isChecked;
        Mediator.ComponentChanged(this, isChecked.ToString());
    }
}

public class Button : FormComponent
{
    private bool _enabled;
    
    public Button(string name, IFormMediator mediator) : base(name, mediator) { }
    
    public void SetEnabled(bool enabled)
    {
        _enabled = enabled;
        Console.WriteLine($"[{Name}] {(enabled ? "Enabled" : "Disabled")}");
    }
    
    public void Click()
    {
        if (_enabled)
        {
            Console.WriteLine($"[{Name}] Clicked - Form submitted!");
        }
        else
        {
            Console.WriteLine($"[{Name}] Cannot click - button disabled");
        }
    }
}

// Usage
var mediator = new RegistrationFormMediator();

var usernameBox = new TextBox("Username", mediator);
var passwordBox = new TextBox("Password", mediator);
var termsCheckbox = new CheckBox("Terms & Conditions", mediator);
var submitButton = new Button("Submit", mediator);

mediator.SetComponents(usernameBox, passwordBox, termsCheckbox, submitButton);

Console.WriteLine("\n--- User Interaction ---\n");

usernameBox.SetValue("ab");           // Too short
Console.WriteLine();

usernameBox.SetValue("john_doe");     // Valid
Console.WriteLine();

passwordBox.SetValue("123");          // Too short
Console.WriteLine();

passwordBox.SetValue("secure123");    // Valid
Console.WriteLine();

termsCheckbox.SetChecked(true);       // Accept terms
Console.WriteLine();

submitButton.Click();                 // Now enabled, can submit
```

---

## MediatR Library Example (ASP.NET Core)

```csharp
// MediatR is a popular implementation of Mediator pattern in .NET

// Request
public class CreateUserCommand : IRequest<int>
{
    public string Name { get; set; }
    public string Email { get; set; }
}

// Handler
public class CreateUserCommandHandler : IRequestHandler<CreateUserCommand, int>
{
    private readonly IUserRepository _repository;
    private readonly IEmailService _emailService;
    
    public CreateUserCommandHandler(IUserRepository repository, IEmailService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
    
    public async Task<int> Handle(CreateUserCommand request, CancellationToken cancellationToken)
    {
        var user = new User
        {
            Name = request.Name,
            Email = request.Email
        };
        
        await _repository.AddAsync(user);
        await _emailService.SendWelcomeEmail(user.Email);
        
        return user.Id;
    }
}

// Controller (sends request to mediator)
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    private readonly IMediator _mediator;
    
    public UsersController(IMediator mediator)
    {
        _mediator = mediator;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserCommand command)
    {
        var userId = await _mediator.Send(command);
        return Ok(new { userId });
    }
}

// Notification (pub/sub through mediator)
public class UserCreatedNotification : INotification
{
    public int UserId { get; set; }
    public string Email { get; set; }
}

// Multiple handlers can listen to same notification
public class SendWelcomeEmailHandler : INotificationHandler<UserCreatedNotification>
{
    public async Task Handle(UserCreatedNotification notification, CancellationToken cancellationToken)
    {
        Console.WriteLine($"Sending welcome email to {notification.Email}");
    }
}

public class LogUserCreationHandler : INotificationHandler<UserCreatedNotification>
{
    public async Task Handle(UserCreatedNotification notification, CancellationToken cancellationToken)
    {
        Console.WriteLine($"Logging user creation: {notification.UserId}");
    }
}

// Publish notification
await _mediator.Publish(new UserCreatedNotification 
{ 
    UserId = userId, 
    Email = email 
});
```

---

## Smart Home Example

```csharp
public interface ISmartHomeMediator
{
    void Notify(SmartDevice sender, string eventName);
}

public class SmartHomeHub : ISmartHomeMediator
{
    private SecurityCamera _camera;
    private SmartLock _lock;
    private LightSystem _lights;
    private Thermostat _thermostat;
    
    public void RegisterDevices(SecurityCamera camera, SmartLock lock, LightSystem lights, Thermostat thermostat)
    {
        _camera = camera;
        _lock = lock;
        _lights = lights;
        _thermostat = thermostat;
    }
    
    public void Notify(SmartDevice sender, string eventName)
    {
        Console.WriteLine($"[Hub] Event from {sender.Name}: {eventName}");
        
        if (sender == _lock && eventName == "UNLOCKED")
        {
            // Door unlocked - turn on lights, adjust temperature
            _lights.TurnOn();
            _thermostat.SetTemperature(72);
            _camera.StartRecording();
        }
        else if (sender == _lock && eventName == "LOCKED")
        {
            // Door locked - turn off lights, lower temperature
            _lights.TurnOff();
            _thermostat.SetTemperature(65);
            _camera.StopRecording();
        }
        else if (sender == _camera && eventName == "MOTION_DETECTED")
        {
            // Motion detected - turn on lights if door is locked
            _lights.TurnOn();
            Console.WriteLine("[Hub] Sending alert: Motion detected while away!");
        }
    }
}

public abstract class SmartDevice
{
    public string Name { get; }
    protected ISmartHomeMediator Mediator { get; }
    
    protected SmartDevice(string name, ISmartHomeMediator mediator)
    {
        Name = name;
        Mediator = mediator;
    }
}

public class SmartLock : SmartDevice
{
    public SmartLock(ISmartHomeMediator mediator) : base("Smart Lock", mediator) { }
    
    public void Unlock()
    {
        Console.WriteLine("[Smart Lock] Unlocking...");
        Mediator.Notify(this, "UNLOCKED");
    }
    
    public void Lock()
    {
        Console.WriteLine("[Smart Lock] Locking...");
        Mediator.Notify(this, "LOCKED");
    }
}

public class LightSystem : SmartDevice
{
    public LightSystem(ISmartHomeMediator mediator) : base("Light System", mediator) { }
    
    public void TurnOn() => Console.WriteLine("[Lights] Turned ON");
    public void TurnOff() => Console.WriteLine("[Lights] Turned OFF");
}

public class Thermostat : SmartDevice
{
    public Thermostat(ISmartHomeMediator mediator) : base("Thermostat", mediator) { }
    
    public void SetTemperature(int temp) => Console.WriteLine($"[Thermostat] Set to {temp}°F");
}

public class SecurityCamera : SmartDevice
{
    public SecurityCamera(ISmartHomeMediator mediator) : base("Security Camera", mediator) { }
    
    public void StartRecording() => Console.WriteLine("[Camera] Recording started");
    public void StopRecording() => Console.WriteLine("[Camera] Recording stopped");
    
    public void DetectMotion()
    {
        Console.WriteLine("[Camera] Motion detected!");
        Mediator.Notify(this, "MOTION_DETECTED");
    }
}

// Usage
var hub = new SmartHomeHub();

var lock = new SmartLock(hub);
var lights = new LightSystem(hub);
var thermostat = new Thermostat(hub);
var camera = new SecurityCamera(hub);

hub.RegisterDevices(camera, lock, lights, thermostat);

Console.WriteLine("--- Arriving Home ---\n");
lock.Unlock();

Console.WriteLine("\n--- Leaving Home ---\n");
lock.Lock();

Console.WriteLine("\n--- Motion Detected While Away ---\n");
camera.DetectMotion();
```

---

## When to Use Mediator Pattern

✅ **Use when:**
- Set of objects communicate in complex ways
- Reusing objects is difficult due to many dependencies
- Behavior distributed between classes should be customizable
- Too many connections between objects

❌ **Avoid when:**
- Objects have simple, well-defined interactions
- Only two objects communicate
- Mediator becomes a "god object" doing too much

---

## Advantages

✅ **Reduced coupling** - Components don't reference each other  
✅ **Centralized control** - Communication logic in one place  
✅ **Simplified components** - Components have less responsibility  
✅ **Easier to understand** - Clear communication flow  
✅ **Reusability** - Components can be reused independently

---

## Disadvantages

❌ **God object risk** - Mediator can become too complex  
❌ **Single point of failure** - Everything depends on mediator  
❌ **Harder to change** - Mediator knows about all components

---

## Mediator vs Observer

| Aspect | Mediator | Observer |
|--------|----------|----------|
| **Communication** | Two-way (components and mediator) | One-way (subject to observers) |
| **Coupling** | Components know mediator | Observers know subject |
| **Purpose** | Coordinate complex interactions | Notify multiple observers of changes |
| **Usage** | UI components, ATC | Event handling, data binding |

---

## Related Patterns

- **Observer** - Mediator can use Observer for component notification
- **Facade** - Simplifies interface; Mediator adds collaborative behavior
- **Singleton** - Mediator is often a singleton

---

## Common Use Cases

- **UI frameworks** - Form validation, component coordination
- **Chat applications** - Chat rooms, messaging
- **Air traffic control** - Aircraft coordination
- **Smart home systems** - Device automation
- **Game development** - Entity coordination
- **CQRS** - MediatR library for commands/queries

---

## Best Practices

✅ Keep mediator **focused** - avoid god object  
✅ Use **events** for decoupled notification  
✅ Consider **multiple mediators** for different concerns  
✅ Document **communication patterns** clearly  
✅ Use **dependency injection** for mediator  
✅ Combine with **Observer** for publish/subscribe scenarios