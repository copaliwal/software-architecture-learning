# Command Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

---

## Problem

You want to decouple the object that invokes an operation from the one that knows how to perform it. You may also need features like undo/redo, logging, or queuing operations.

---

## Solution

Turn requests into standalone objects that contain all information about the request. This transformation lets you pass requests as method arguments, delay or queue execution, and support undo operations.

---

## Structure

```
Command Interface
└── Execute()
└── Undo() (optional)

ConcreteCommand
├── Receiver reference
├── Execute() - calls receiver methods
└── Undo() - reverses the operation

Invoker
└── Stores and executes commands

Receiver
└── Performs actual work
```

---

## C# Implementation

### Basic Example: Text Editor

```csharp
// Command interface
public interface ICommand
{
    void Execute();
    void Undo();
}

// Receiver
public class TextEditor
{
    private StringBuilder _text = new StringBuilder();
    
    public string GetText() => _text.ToString();
    
    public void AppendText(string text)
    {
        _text.Append(text);
        Console.WriteLine($"Text: {_text}");
    }
    
    public void DeleteText(int length)
    {
        if (length <= _text.Length)
        {
            _text.Remove(_text.Length - length, length);
            Console.WriteLine($"Text: {_text}");
        }
    }
}

// Concrete Commands
public class AppendTextCommand : ICommand
{
    private readonly TextEditor _editor;
    private readonly string _text;
    
    public AppendTextCommand(TextEditor editor, string text)
    {
        _editor = editor;
        _text = text;
    }
    
    public void Execute()
    {
        _editor.AppendText(_text);
    }
    
    public void Undo()
    {
        _editor.DeleteText(_text.Length);
    }
}

public class DeleteTextCommand : ICommand
{
    private readonly TextEditor _editor;
    private readonly int _length;
    private string _deletedText;
    
    public DeleteTextCommand(TextEditor editor, int length)
    {
        _editor = editor;
        _length = length;
    }
    
    public void Execute()
    {
        var currentText = _editor.GetText();
        if (_length <= currentText.Length)
        {
            _deletedText = currentText.Substring(currentText.Length - _length);
            _editor.DeleteText(_length);
        }
    }
    
    public void Undo()
    {
        if (_deletedText != null)
            _editor.AppendText(_deletedText);
    }
}

// Invoker
public class CommandManager
{
    private readonly Stack<ICommand> _undoStack = new();
    private readonly Stack<ICommand> _redoStack = new();
    
    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _undoStack.Push(command);
        _redoStack.Clear(); // Clear redo stack on new command
    }
    
    public void Undo()
    {
        if (_undoStack.Count > 0)
        {
            var command = _undoStack.Pop();
            command.Undo();
            _redoStack.Push(command);
        }
        else
        {
            Console.WriteLine("Nothing to undo");
        }
    }
    
    public void Redo()
    {
        if (_redoStack.Count > 0)
        {
            var command = _redoStack.Pop();
            command.Execute();
            _undoStack.Push(command);
        }
        else
        {
            Console.WriteLine("Nothing to redo");
        }
    }
}

// Usage
var editor = new TextEditor();
var manager = new CommandManager();

manager.ExecuteCommand(new AppendTextCommand(editor, "Hello "));
// Text: Hello 

manager.ExecuteCommand(new AppendTextCommand(editor, "World"));
// Text: Hello World

manager.ExecuteCommand(new DeleteTextCommand(editor, 5));
// Text: Hello 

manager.Undo();
// Text: Hello World

manager.Undo();
// Text: Hello 

manager.Redo();
// Text: Hello World
```

---

## Real-World Example: Smart Home Automation

```csharp
// Receivers
public class Light
{
    public string Location { get; }
    
    public Light(string location)
    {
        Location = location;
    }
    
    public void TurnOn()
    {
        Console.WriteLine($"{Location} light is ON");
    }
    
    public void TurnOff()
    {
        Console.WriteLine($"{Location} light is OFF");
    }
}

public class Thermostat
{
    private int _temperature;
    
    public void SetTemperature(int temperature)
    {
        _temperature = temperature;
        Console.WriteLine($"Temperature set to {_temperature}°C");
    }
    
    public int GetTemperature() => _temperature;
}

public class SecuritySystem
{
    public void Arm()
    {
        Console.WriteLine("Security system ARMED");
    }
    
    public void Disarm()
    {
        Console.WriteLine("Security system DISARMED");
    }
}

// Command Interface
public interface ISmartHomeCommand
{
    void Execute();
    void Undo();
}

// Concrete Commands
public class LightOnCommand : ISmartHomeCommand
{
    private readonly Light _light;
    
    public LightOnCommand(Light light)
    {
        _light = light;
    }
    
    public void Execute() => _light.TurnOn();
    public void Undo() => _light.TurnOff();
}

public class LightOffCommand : ISmartHomeCommand
{
    private readonly Light _light;
    
    public LightOffCommand(Light light)
    {
        _light = light;
    }
    
    public void Execute() => _light.TurnOff();
    public void Undo() => _light.TurnOn();
}

public class SetTemperatureCommand : ISmartHomeCommand
{
    private readonly Thermostat _thermostat;
    private readonly int _newTemperature;
    private int _previousTemperature;
    
    public SetTemperatureCommand(Thermostat thermostat, int temperature)
    {
        _thermostat = thermostat;
        _newTemperature = temperature;
    }
    
    public void Execute()
    {
        _previousTemperature = _thermostat.GetTemperature();
        _thermostat.SetTemperature(_newTemperature);
    }
    
    public void Undo()
    {
        _thermostat.SetTemperature(_previousTemperature);
    }
}

public class ArmSecurityCommand : ISmartHomeCommand
{
    private readonly SecuritySystem _security;
    
    public ArmSecurityCommand(SecuritySystem security)
    {
        _security = security;
    }
    
    public void Execute() => _security.Arm();
    public void Undo() => _security.Disarm();
}

// Macro Command (Composite)
public class MacroCommand : ISmartHomeCommand
{
    private readonly List<ISmartHomeCommand> _commands;
    
    public MacroCommand(params ISmartHomeCommand[] commands)
    {
        _commands = new List<ISmartHomeCommand>(commands);
    }
    
    public void Execute()
    {
        foreach (var command in _commands)
        {
            command.Execute();
        }
    }
    
    public void Undo()
    {
        // Undo in reverse order
        for (int i = _commands.Count - 1; i >= 0; i--)
        {
            _commands[i].Undo();
        }
    }
}

// Invoker - Remote Control
public class RemoteControl
{
    private ISmartHomeCommand _lastCommand;
    
    public void PressButton(ISmartHomeCommand command)
    {
        command.Execute();
        _lastCommand = command;
    }
    
    public void PressUndo()
    {
        if (_lastCommand != null)
        {
            _lastCommand.Undo();
        }
        else
        {
            Console.WriteLine("No command to undo");
        }
    }
}

// Usage
var livingRoomLight = new Light("Living Room");
var bedroomLight = new Light("Bedroom");
var thermostat = new Thermostat();
var security = new SecuritySystem();

var remote = new RemoteControl();

// Individual commands
remote.PressButton(new LightOnCommand(livingRoomLight));
// Living Room light is ON

remote.PressButton(new SetTemperatureCommand(thermostat, 22));
// Temperature set to 22°C

remote.PressUndo();
// Temperature set to 0°C

// Macro Command - "Good Night" mode
var goodNightMode = new MacroCommand(
    new LightOffCommand(livingRoomLight),
    new LightOffCommand(bedroomLight),
    new SetTemperatureCommand(thermostat, 18),
    new ArmSecurityCommand(security)
);

remote.PressButton(goodNightMode);
// Living Room light is OFF
// Bedroom light is OFF
// Temperature set to 18°C
// Security system ARMED

remote.PressUndo();
// Security system DISARMED
// Temperature set to 22°C
// Bedroom light is ON
// Living Room light is ON
```

---

## Real-World Example: Database Transaction Command

```csharp
public interface IDatabaseCommand
{
    void Execute();
    void Rollback();
}

public class InsertCommand : IDatabaseCommand
{
    private readonly string _table;
    private readonly object _data;
    private int _insertedId;
    
    public InsertCommand(string table, object data)
    {
        _table = table;
        _data = data;
    }
    
    public void Execute()
    {
        // Simulate insert
        _insertedId = new Random().Next(1000, 9999);
        Console.WriteLine($"Inserted into {_table}: {_data} (ID: {_insertedId})");
    }
    
    public void Rollback()
    {
        Console.WriteLine($"Deleted from {_table} (ID: {_insertedId})");
    }
}

public class UpdateCommand : IDatabaseCommand
{
    private readonly string _table;
    private readonly int _id;
    private readonly object _newData;
    private object _oldData;
    
    public UpdateCommand(string table, int id, object newData)
    {
        _table = table;
        _id = id;
        _newData = newData;
    }
    
    public void Execute()
    {
        _oldData = "old_value"; // Simulate fetching old data
        Console.WriteLine($"Updated {_table} (ID: {_id}) to {_newData}");
    }
    
    public void Rollback()
    {
        Console.WriteLine($"Reverted {_table} (ID: {_id}) to {_oldData}");
    }
}

public class TransactionManager
{
    private readonly List<IDatabaseCommand> _commands = new();
    
    public void AddCommand(IDatabaseCommand command)
    {
        _commands.Add(command);
    }
    
    public void Commit()
    {
        try
        {
            foreach (var command in _commands)
            {
                command.Execute();
            }
            Console.WriteLine("Transaction committed successfully");
            _commands.Clear();
        }
        catch (Exception)
        {
            Rollback();
        }
    }
    
    public void Rollback()
    {
        Console.WriteLine("Rolling back transaction...");
        for (int i = _commands.Count - 1; i >= 0; i--)
        {
            _commands[i].Rollback();
        }
        _commands.Clear();
    }
}

// Usage
var transaction = new TransactionManager();

transaction.AddCommand(new InsertCommand("Users", new { Name = "John", Email = "john@example.com" }));
transaction.AddCommand(new UpdateCommand("Orders", 123, new { Status = "Shipped" }));

transaction.Commit();
// Inserted into Users: { Name = John, Email = john@example.com } (ID: 5678)
// Updated Orders (ID: 123) to { Status = Shipped }
// Transaction committed successfully
```

---

## Command Queue Pattern

```csharp
public class CommandQueue
{
    private readonly Queue<ICommand> _commandQueue = new();
    
    public void Enqueue(ICommand command)
    {
        _commandQueue.Enqueue(command);
        Console.WriteLine($"Command queued. Queue size: {_commandQueue.Count}");
    }
    
    public void ProcessAll()
    {
        Console.WriteLine($"Processing {_commandQueue.Count} commands...");
        while (_commandQueue.Count > 0)
        {
            var command = _commandQueue.Dequeue();
            command.Execute();
        }
        Console.WriteLine("All commands processed");
    }
}

// Usage - Batch processing
var queue = new CommandQueue();

queue.Enqueue(new AppendTextCommand(editor, "A"));
queue.Enqueue(new AppendTextCommand(editor, "B"));
queue.Enqueue(new AppendTextCommand(editor, "C"));

queue.ProcessAll();
```

---

## When to Use

✅ **Use Command Pattern when:**
- You need undo/redo functionality
- You want to queue, schedule, or log operations
- You need to support macro commands (composite commands)
- You want to parameterize objects with operations
- You need transactional behavior

❌ **Avoid when:**
- Simple callbacks or delegates suffice
- You don't need undo, queuing, or logging
- The overhead isn't justified

---

## Advantages

✅ Decouples invoker from receiver  
✅ Supports undo/redo  
✅ Can queue and log commands  
✅ Supports macro commands (composite)  
✅ Follows Single Responsibility Principle  
✅ Follows Open/Closed Principle

---

## Disadvantages

❌ Increases number of classes  
❌ Can be overkill for simple operations  
❌ May increase memory usage (storing command history)

---

## Related Patterns

- **Memento** - Store command state for undo
- **Composite** - For macro commands
- **Chain of Responsibility** - Commands can be chained
- **Strategy** - Command encapsulates action; Strategy encapsulates algorithm

---

## Common Use Cases in .NET

- **MVVM Commands** - `ICommand` in WPF/UWP (`RelayCommand`, `DelegateCommand`)
- **CQRS** - Command handlers in domain-driven design
- **Task Scheduling** - Background job queues
- **Transaction Management** - Database operations
- **Undo/Redo** - Text editors, graphics applications
- **MediatR** - Command/Query handling in ASP.NET Core
