# Memento Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Capture and externalize an object's internal state without violating encapsulation, allowing the object to be restored to this state later.

---

## Problem

You need to implement undo/redo functionality or save checkpoints, but:
- Exposing internal state violates encapsulation
- Direct access to private fields breaks object design
- Objects need to restore previous states
- Saving full object snapshots is complex

Directly saving/restoring object state exposes implementation details.

---

## Solution

Create a memento object that stores a snapshot of the originator's state. The memento:
1. Is opaque to everyone except the originator
2. Can only be accessed by the originator
3. Stored and retrieved by a caretaker

**Three participants:**
- **Originator** - Creates memento with current state, restores from memento
- **Memento** - Stores originator's state (immutable)
- **Caretaker** - Keeps mementos but never modifies them

---

## Structure

```
Originator
├── CreateMemento() - Save state
├── RestoreFromMemento(memento) - Restore state
└── Internal state

Memento
├── GetState() - Private, only Originator can access
└── Stores snapshot of Originator's state

Caretaker
├── Memento history
├── Save(memento)
└── Undo() / Redo()
```

---

## C# Implementation

### Example: Text Editor with Undo

```csharp
// Memento - Stores editor state
public class EditorMemento
{
    public string Content { get; }
    public int CursorPosition { get; }
    public DateTime Timestamp { get; }
    
    public EditorMemento(string content, int cursorPosition)
    {
        Content = content;
        CursorPosition = cursorPosition;
        Timestamp = DateTime.Now;
    }
}

// Originator - The text editor
public class TextEditor
{
    private string _content = "";
    private int _cursorPosition = 0;
    
    public void Type(string text)
    {
        _content += text;
        _cursorPosition = _content.Length;
        Console.WriteLine($"Content: '{_content}' (Cursor at {_cursorPosition})");
    }
    
    public void Delete(int count)
    {
        if (_content.Length >= count)
        {
            _content = _content.Substring(0, _content.Length - count);
            _cursorPosition = _content.Length;
            Console.WriteLine($"Content: '{_content}' (Cursor at {_cursorPosition})");
        }
    }
    
    public void SetCursor(int position)
    {
        _cursorPosition = Math.Min(position, _content.Length);
    }
    
    // Create memento (save state)
    public EditorMemento Save()
    {
        Console.WriteLine($"[Save] Saving state: '{_content}'");
        return new EditorMemento(_content, _cursorPosition);
    }
    
    // Restore from memento
    public void Restore(EditorMemento memento)
    {
        _content = memento.Content;
        _cursorPosition = memento.CursorPosition;
        Console.WriteLine($"[Restore] Restored to: '{_content}' (from {memento.Timestamp:HH:mm:ss})");
    }
}

// Caretaker - Manages undo/redo
public class EditorHistory
{
    private readonly Stack<EditorMemento> _undoStack = new();
    private readonly Stack<EditorMemento> _redoStack = new();
    
    public void Save(EditorMemento memento)
    {
        _undoStack.Push(memento);
        _redoStack.Clear();  // Clear redo stack on new save
    }
    
    public EditorMemento Undo()
    {
        if (_undoStack.Count > 0)
        {
            var memento = _undoStack.Pop();
            _redoStack.Push(memento);
            return memento;
        }
        
        Console.WriteLine("Nothing to undo");
        return null;
    }
    
    public EditorMemento Redo()
    {
        if (_redoStack.Count > 0)
        {
            var memento = _redoStack.Pop();
            _undoStack.Push(memento);
            return memento;
        }
        
        Console.WriteLine("Nothing to redo");
        return null;
    }
    
    public void ShowHistory()
    {
        Console.WriteLine($"\n=== History ===");
        Console.WriteLine($"Undo stack: {_undoStack.Count} states");
        Console.WriteLine($"Redo stack: {_redoStack.Count} states");
    }
}

// Usage
var editor = new TextEditor();
var history = new EditorHistory();

// Type and save
editor.Type("Hello");
history.Save(editor.Save());

editor.Type(" World");
history.Save(editor.Save());

editor.Type("!");
history.Save(editor.Save());

history.ShowHistory();

// Undo
Console.WriteLine("\n--- Undo ---");
var memento = history.Undo();
if (memento != null) editor.Restore(memento);

Console.WriteLine("\n--- Undo ---");
memento = history.Undo();
if (memento != null) editor.Restore(memento);

// Redo
Console.WriteLine("\n--- Redo ---");
memento = history.Redo();
if (memento != null) editor.Restore(memento);

history.ShowHistory();

// Output:
// Content: 'Hello' (Cursor at 5)
// [Save] Saving state: 'Hello'
// Content: 'Hello World' (Cursor at 11)
// [Save] Saving state: 'Hello World'
// Content: 'Hello World!' (Cursor at 12)
// [Save] Saving state: 'Hello World!'
// 
// === History ===
// Undo stack: 3 states
// Redo stack: 0 states
// 
// --- Undo ---
// [Restore] Restored to: 'Hello World!' (from 10:30:15)
// ...
```

---

## Game Checkpoint Example

```csharp
// Memento - Game state snapshot
public class GameMemento
{
    public int Level { get; }
    public int Score { get; }
    public int Health { get; }
    public Vector3 Position { get; }
    public DateTime SavedAt { get; }
    
    public GameMemento(int level, int score, int health, Vector3 position)
    {
        Level = level;
        Score = score;
        Health = health;
        Position = position;
        SavedAt = DateTime.Now;
    }
}

public record Vector3(float X, float Y, float Z);

// Originator - Game state
public class GameState
{
    public int Level { get; private set; } = 1;
    public int Score { get; private set; } = 0;
    public int Health { get; private set; } = 100;
    public Vector3 Position { get; private set; } = new Vector3(0, 0, 0);
    
    public void Play(int scoreGained, int damageTaken)
    {
        Score += scoreGained;
        Health -= damageTaken;
        Position = new Vector3(Position.X + 10, Position.Y, Position.Z);
        
        Console.WriteLine($"Playing... Score: {Score}, Health: {Health}, Pos: ({Position.X}, {Position.Y}, {Position.Z})");
        
        if (Health <= 0)
        {
            Console.WriteLine("💀 Game Over!");
        }
    }
    
    public void LevelUp()
    {
        Level++;
        Health = 100;  // Restore health
        Console.WriteLine($"🎉 Level Up! Now at Level {Level}");
    }
    
    public GameMemento CreateCheckpoint()
    {
        Console.WriteLine($"💾 Checkpoint created - Level {Level}, Score {Score}");
        return new GameMemento(Level, Score, Health, Position);
    }
    
    public void LoadCheckpoint(GameMemento memento)
    {
        Level = memento.Level;
        Score = memento.Score;
        Health = memento.Health;
        Position = memento.Position;
        Console.WriteLine($"📂 Checkpoint loaded - Level {Level}, Score {Score}, Health {Health}");
    }
}

// Caretaker - Save system
public class SaveSystem
{
    private readonly Dictionary<string, GameMemento> _saves = new();
    
    public void Save(string saveName, GameMemento memento)
    {
        _saves[saveName] = memento;
        Console.WriteLine($"[SaveSystem] Saved to slot: {saveName}");
    }
    
    public GameMemento Load(string saveName)
    {
        if (_saves.ContainsKey(saveName))
        {
            Console.WriteLine($"[SaveSystem] Loading from slot: {saveName}");
            return _saves[saveName];
        }
        
        Console.WriteLine($"[SaveSystem] No save found: {saveName}");
        return null;
    }
    
    public void ListSaves()
    {
        Console.WriteLine("\n=== Save Slots ===");
        foreach (var save in _saves)
        {
            var m = save.Value;
            Console.WriteLine($"{save.Key}: Level {m.Level}, Score {m.Score}, Saved {m.SavedAt:HH:mm:ss}");
        }
    }
}

// Usage
var game = new GameState();
var saveSystem = new SaveSystem();

// Play
game.Play(scoreGained: 100, damageTaken: 20);
game.Play(scoreGained: 50, damageTaken: 30);

// Save checkpoint
saveSystem.Save("Checkpoint1", game.CreateCheckpoint());

// Continue playing
game.Play(scoreGained: 200, damageTaken: 40);
game.LevelUp();

// Save another checkpoint
saveSystem.Save("Checkpoint2", game.CreateCheckpoint());

// Die
game.Play(scoreGained: 50, damageTaken: 200);

// Load checkpoint
Console.WriteLine("\n--- Loading Checkpoint ---");
var checkpoint = saveSystem.Load("Checkpoint2");
if (checkpoint != null)
{
    game.LoadCheckpoint(checkpoint);
}

saveSystem.ListSaves();
```

---

## Drawing App Example

```csharp
// Memento - Canvas snapshot
public class CanvasMemento
{
    private readonly List<Shape> _shapes;
    
    public CanvasMemento(IEnumerable<Shape> shapes)
    {
        // Deep copy to prevent external modification
        _shapes = shapes.Select(s => s.Clone()).ToList();
    }
    
    public IEnumerable<Shape> GetShapes()
    {
        return _shapes.Select(s => s.Clone()).ToList();
    }
}

public abstract class Shape
{
    public string Color { get; set; }
    public abstract Shape Clone();
}

public class Circle : Shape
{
    public int Radius { get; set; }
    
    public override Shape Clone()
    {
        return new Circle { Color = this.Color, Radius = this.Radius };
    }
    
    public override string ToString() => $"Circle(r={Radius}, color={Color})";
}

public class Rectangle : Shape
{
    public int Width { get; set; }
    public int Height { get; set; }
    
    public override Shape Clone()
    {
        return new Rectangle { Color = this.Color, Width = this.Width, Height = this.Height };
    }
    
    public override string ToString() => $"Rectangle({Width}x{Height}, color={Color})";
}

// Originator - Canvas
public class Canvas
{
    private readonly List<Shape> _shapes = new();
    
    public void AddShape(Shape shape)
    {
        _shapes.Add(shape);
        Console.WriteLine($"Added: {shape}");
    }
    
    public void RemoveLastShape()
    {
        if (_shapes.Count > 0)
        {
            var removed = _shapes[^1];
            _shapes.RemoveAt(_shapes.Count - 1);
            Console.WriteLine($"Removed: {removed}");
        }
    }
    
    public void Display()
    {
        Console.WriteLine($"\n=== Canvas ({_shapes.Count} shapes) ===");
        foreach (var shape in _shapes)
        {
            Console.WriteLine($"  {shape}");
        }
    }
    
    public CanvasMemento Save()
    {
        Console.WriteLine("[Canvas] Saving state...");
        return new CanvasMemento(_shapes);
    }
    
    public void Restore(CanvasMemento memento)
    {
        _shapes.Clear();
        _shapes.AddRange(memento.GetShapes());
        Console.WriteLine("[Canvas] State restored");
    }
}

// Caretaker
public class CanvasHistory
{
    private readonly Stack<CanvasMemento> _history = new();
    private const int MaxHistorySize = 10;
    
    public void Save(CanvasMemento memento)
    {
        _history.Push(memento);
        
        // Limit history size
        if (_history.Count > MaxHistorySize)
        {
            var temp = new Stack<CanvasMemento>();
            for (int i = 0; i < MaxHistorySize; i++)
            {
                temp.Push(_history.Pop());
            }
            _history.Clear();
            while (temp.Count > 0)
            {
                _history.Push(temp.Pop());
            }
        }
    }
    
    public CanvasMemento Undo()
    {
        if (_history.Count > 0)
        {
            return _history.Pop();
        }
        
        Console.WriteLine("No more undo history");
        return null;
    }
}

// Usage
var canvas = new Canvas();
var history = new CanvasHistory();

// Draw shapes and save
canvas.AddShape(new Circle { Radius = 10, Color = "Red" });
history.Save(canvas.Save());

canvas.AddShape(new Rectangle { Width = 20, Height = 30, Color = "Blue" });
history.Save(canvas.Save());

canvas.AddShape(new Circle { Radius = 5, Color = "Green" });
history.Save(canvas.Save());

canvas.Display();

// Undo
Console.WriteLine("\n--- Undo ---");
var memento = history.Undo();
if (memento != null) canvas.Restore(memento);
canvas.Display();
```

---

## Transaction Example (Database)

```csharp
// Memento - Database transaction snapshot
public class TransactionMemento
{
    public Dictionary<string, object> Data { get; }
    
    public TransactionMemento(Dictionary<string, object> data)
    {
        // Deep copy
        Data = new Dictionary<string, object>(data);
    }
}

// Originator - Database context
public class DatabaseContext
{
    private Dictionary<string, object> _data = new();
    
    public void Set(string key, object value)
    {
        _data[key] = value;
        Console.WriteLine($"[DB] SET {key} = {value}");
    }
    
    public object Get(string key)
    {
        return _data.ContainsKey(key) ? _data[key] : null;
    }
    
    public void Display()
    {
        Console.WriteLine("\n=== Database State ===");
        foreach (var kvp in _data)
        {
            Console.WriteLine($"  {kvp.Key} = {kvp.Value}");
        }
    }
    
    public TransactionMemento BeginTransaction()
    {
        Console.WriteLine("[DB] Transaction started");
        return new TransactionMemento(_data);
    }
    
    public void Commit()
    {
        Console.WriteLine("[DB] Transaction committed");
    }
    
    public void Rollback(TransactionMemento memento)
    {
        _data = new Dictionary<string, object>(memento.Data);
        Console.WriteLine("[DB] Transaction rolled back");
    }
}

// Usage
var db = new DatabaseContext();

db.Set("User1", "Alice");
db.Set("User2", "Bob");
db.Display();

// Start transaction
var transaction = db.BeginTransaction();

// Make changes
db.Set("User3", "Charlie");
db.Set("User1", "Alice Updated");
db.Display();

// Rollback
Console.WriteLine("\n--- Rolling Back ---");
db.Rollback(transaction);
db.Display();

// Start new transaction and commit
transaction = db.BeginTransaction();
db.Set("User3", "Charlie");
db.Commit();
db.Display();
```

---

## When to Use Memento Pattern

✅ **Use when:**
- Need to implement undo/redo functionality
- Must save/restore object state
- Direct access to state would violate encapsulation
- Need checkpoints or snapshots

❌ **Avoid when:**
- State is simple (just use properties)
- Serialization is sufficient
- Memory consumption is a concern (large states)
- State changes are infrequent

---

## Advantages

✅ **Preserves encapsulation** - Internal state not exposed  
✅ **Simplifies originator** - Doesn't manage history  
✅ **Undo/redo** - Easy to implement  
✅ **Snapshots** - Save checkpoints easily

---

## Disadvantages

❌ **Memory overhead** - Each memento stores full state  
❌ **Performance** - Creating mementos can be expensive  
❌ **Lifetime management** - Caretaker must manage mementos  
❌ **Serialization complexity** - Deep copying can be tricky

---

## Memento Variants

| Variant | Description | Example |
|---------|-------------|---------|
| **Full Memento** | Complete state snapshot | Game save |
| **Incremental Memento** | Only changed fields | Git commits |
| **Compressed Memento** | Compressed state | Large documents |

---

## Related Patterns

- **Command** - Can use Memento for undo (Command stores Memento)
- **Iterator** - Can use Memento to save iteration state
- **Prototype** - Similar cloning, but Memento focuses on state restoration

---

## Common Use Cases

- **Text editors** - Undo/redo functionality
- **Games** - Save games, checkpoints
- **Drawing apps** - Canvas history
- **Database transactions** - Rollback capability
- **Version control** - State snapshots
- **Configuration** - Restore previous settings

---

## Best Practices

✅ Make mementos **immutable**  
✅ Use **deep copying** to prevent external modification  
✅ Consider **serialization** for persistence  
✅ Limit **history size** to prevent memory issues  
✅ Use **compression** for large states  
✅ Consider **Command pattern** for more complex undo  
✅ Document **what state is saved** in memento