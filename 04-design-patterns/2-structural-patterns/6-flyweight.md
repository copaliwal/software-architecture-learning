# Flyweight Pattern

## Overview

**Category:** Structural Pattern  
**Intent:** Use sharing to support large numbers of fine-grained objects efficiently by sharing common state between objects.

---

## Problem

Creating large numbers of similar objects consumes excessive memory. When objects share most of their state, creating separate instances for each is wasteful.

**Example:** A text editor with millions of characters - storing font, size, and color for each character separately would use enormous memory.

---

## Solution

Split object state into:
- **Intrinsic state** - Shared among objects (stored in flyweight)
- **Extrinsic state** - Unique to each object (passed by client)

Share flyweight objects with identical intrinsic state. Clients pass extrinsic state when calling methods.

---

## Structure

```
FlyweightFactory
├── Creates and manages flyweights
├── Returns existing flyweight if available
└── Creates new flyweight only if needed

Flyweight
├── Stores intrinsic (shared) state
└── Accepts extrinsic state as parameters

Client
└── Maintains extrinsic state
```

---

## C# Implementation

### Example: Text Editor Characters

```csharp
// Flyweight - Stores shared state (font, size, color)
public class CharacterStyle
{
    public string Font { get; }
    public int Size { get; }
    public string Color { get; }
    
    public CharacterStyle(string font, int size, string color)
    {
        Font = font;
        Size = size;
        Color = color;
        
        Console.WriteLine($"Creating new CharacterStyle: {font}, {size}pt, {color}");
    }
    
    // Operation that uses both intrinsic and extrinsic state
    public void Display(char character, int position)
    {
        Console.WriteLine($"Char: '{character}' at position {position} | Font: {Font}, Size: {Size}pt, Color: {Color}");
    }
}

// Flyweight Factory
public class CharacterStyleFactory
{
    private readonly Dictionary<string, CharacterStyle> _styles = new();
    
    public CharacterStyle GetStyle(string font, int size, string color)
    {
        var key = $"{font}_{size}_{color}";
        
        if (!_styles.ContainsKey(key))
        {
            _styles[key] = new CharacterStyle(font, size, color);
        }
        else
        {
            Console.WriteLine($"Reusing existing CharacterStyle: {font}, {size}pt, {color}");
        }
        
        return _styles[key];
    }
    
    public int GetTotalStyles() => _styles.Count;
}

// Client - Maintains extrinsic state (actual character and position)
public class Character
{
    private readonly char _char;  // Extrinsic state
    private readonly int _position;  // Extrinsic state
    private readonly CharacterStyle _style;  // Reference to flyweight (intrinsic state)
    
    public Character(char c, int position, CharacterStyle style)
    {
        _char = c;
        _position = position;
        _style = style;
    }
    
    public void Display()
    {
        _style.Display(_char, _position);
    }
}

// Text Document
public class TextEditor
{
    private readonly List<Character> _characters = new();
    private readonly CharacterStyleFactory _styleFactory = new();
    
    public void AddCharacter(char c, string font, int size, string color)
    {
        var style = _styleFactory.GetStyle(font, size, color);
        var character = new Character(c, _characters.Count, style);
        _characters.Add(character);
    }
    
    public void DisplayDocument()
    {
        Console.WriteLine("\n--- Document Content ---");
        foreach (var character in _characters)
        {
            character.Display();
        }
        Console.WriteLine($"\nTotal unique styles created: {_styleFactory.GetTotalStyles()}");
        Console.WriteLine($"Total characters: {_characters.Count}\n");
    }
}

// Usage
var editor = new TextEditor();

// Type "Hello" - all using same font/size/color
editor.AddCharacter('H', "Arial", 12, "Black");
editor.AddCharacter('e', "Arial", 12, "Black");
editor.AddCharacter('l', "Arial", 12, "Black");
editor.AddCharacter('l', "Arial", 12, "Black");
editor.AddCharacter('o', "Arial", 12, "Black");

// Change style for " World"
editor.AddCharacter(' ', "Arial", 12, "Black");
editor.AddCharacter('W', "Arial", 16, "Red");
editor.AddCharacter('o', "Arial", 16, "Red");
editor.AddCharacter('r', "Arial", 16, "Red");
editor.AddCharacter('l', "Arial", 16, "Red");
editor.AddCharacter('d', "Arial", 16, "Red");

editor.DisplayDocument();

// Output:
// Creating new CharacterStyle: Arial, 12pt, Black
// Reusing existing CharacterStyle: Arial, 12pt, Black
// Reusing existing CharacterStyle: Arial, 12pt, Black
// Reusing existing CharacterStyle: Arial, 12pt, Black
// Reusing existing CharacterStyle: Arial, 12pt, Black
// Reusing existing CharacterStyle: Arial, 12pt, Black
// Creating new CharacterStyle: Arial, 16pt, Red
// Reusing existing CharacterStyle: Arial, 16pt, Red
// ... (Only 2 styles created for 11 characters!)
```

---

## Real-World Example: Game Tree Rendering

```csharp
// Flyweight - Shared tree model data
public class TreeType
{
    public string Name { get; }
    public string Color { get; }
    public string Texture { get; }
    private readonly byte[] _meshData;  // Large data
    
    public TreeType(string name, string color, string texture)
    {
        Name = name;
        Color = color;
        Texture = texture;
        _meshData = LoadMeshData(name);  // Expensive operation
        
        Console.WriteLine($"Loading tree type: {name} (Heavy resource loaded)");
    }
    
    private byte[] LoadMeshData(string name)
    {
        // Simulate loading large 3D model data
        return new byte[1000000];  // 1MB per tree type
    }
    
    public void Render(int x, int y, int scale)
    {
        Console.WriteLine($"Rendering {Name} tree at ({x}, {y}) with scale {scale}");
    }
}

// Flyweight Factory
public class TreeFactory
{
    private static readonly Dictionary<string, TreeType> _treeTypes = new();
    
    public static TreeType GetTreeType(string name, string color, string texture)
    {
        var key = $"{name}_{color}_{texture}";
        
        if (!_treeTypes.ContainsKey(key))
        {
            _treeTypes[key] = new TreeType(name, color, texture);
        }
        
        return _treeTypes[key];
    }
    
    public static int GetTotalTreeTypes() => _treeTypes.Count;
}

// Context - Extrinsic state
public class Tree
{
    private readonly int _x;  // Unique position
    private readonly int _y;
    private readonly int _scale;  // Unique scale
    private readonly TreeType _type;  // Shared flyweight
    
    public Tree(int x, int y, int scale, TreeType type)
    {
        _x = x;
        _y = y;
        _scale = scale;
        _type = type;
    }
    
    public void Render()
    {
        _type.Render(_x, _y, _scale);
    }
}

// Forest - Contains many trees
public class Forest
{
    private readonly List<Tree> _trees = new();
    
    public void PlantTree(int x, int y, int scale, string name, string color, string texture)
    {
        var treeType = TreeFactory.GetTreeType(name, color, texture);
        var tree = new Tree(x, y, scale, treeType);
        _trees.Add(tree);
    }
    
    public void Render()
    {
        Console.WriteLine($"\n--- Rendering Forest ---");
        foreach (var tree in _trees)
        {
            tree.Render();
        }
        Console.WriteLine($"\nTotal trees: {_trees.Count}");
        Console.WriteLine($"Unique tree types: {TreeFactory.GetTotalTreeTypes()}");
        Console.WriteLine($"Memory saved: {(_trees.Count - TreeFactory.GetTotalTreeTypes()) * 1} MB\n");
    }
}

// Usage - Plant 10,000 trees using only 3 types
var forest = new Forest();

// Plant 10,000 trees (but only 3 types)
for (int i = 0; i < 10000; i++)
{
    var random = new Random(i);
    var treeTypes = new[] 
    {
        ("Oak", "Green", "oak_texture.png"),
        ("Pine", "DarkGreen", "pine_texture.png"),
        ("Birch", "LightGreen", "birch_texture.png")
    };
    
    var treeType = treeTypes[i % 3];
    forest.PlantTree(
        random.Next(0, 1000),  // x
        random.Next(0, 1000),  // y
        random.Next(1, 5),     // scale
        treeType.Item1,
        treeType.Item2,
        treeType.Item3
    );
}

forest.Render();
// Without Flyweight: 10,000 MB memory (1MB x 10,000 trees)
// With Flyweight: 3 MB memory (1MB x 3 types)
// Memory saved: 9,997 MB!
```

---

## ASP.NET Core Example: Connection Pool (Flyweight-like)

```csharp
// Flyweight - Shared database connection
public class DatabaseConnection
{
    public string ConnectionString { get; }
    public Guid ConnectionId { get; }
    
    public DatabaseConnection(string connectionString)
    {
        ConnectionString = connectionString;
        ConnectionId = Guid.NewGuid();
        Console.WriteLine($"Creating new DB connection: {ConnectionId}");
    }
    
    public async Task<List<Product>> ExecuteQueryAsync(string query)
    {
        // Simulate query execution
        await Task.Delay(100);
        Console.WriteLine($"Executing query on connection {ConnectionId}: {query}");
        return new List<Product>();
    }
}

// Connection Pool (Flyweight Factory)
public class ConnectionPool
{
    private readonly Queue<DatabaseConnection> _availableConnections = new();
    private readonly HashSet<DatabaseConnection> _inUseConnections = new();
    private readonly string _connectionString;
    private readonly int _maxPoolSize;
    
    public ConnectionPool(string connectionString, int maxPoolSize = 10)
    {
        _connectionString = connectionString;
        _maxPoolSize = maxPoolSize;
    }
    
    public DatabaseConnection GetConnection()
    {
        lock (_availableConnections)
        {
            if (_availableConnections.Count > 0)
            {
                var connection = _availableConnections.Dequeue();
                _inUseConnections.Add(connection);
                Console.WriteLine($"Reusing connection {connection.ConnectionId}");
                return connection;
            }
            
            if (_inUseConnections.Count < _maxPoolSize)
            {
                var newConnection = new DatabaseConnection(_connectionString);
                _inUseConnections.Add(newConnection);
                return newConnection;
            }
            
            throw new InvalidOperationException("Connection pool exhausted");
        }
    }
    
    public void ReleaseConnection(DatabaseConnection connection)
    {
        lock (_availableConnections)
        {
            _inUseConnections.Remove(connection);
            _availableConnections.Enqueue(connection);
            Console.WriteLine($"Released connection {connection.ConnectionId}");
        }
    }
}

// Usage
var pool = new ConnectionPool("Server=localhost;Database=MyDB", maxPoolSize: 5);

// 100 requests but only 5 connections created
var tasks = new List<Task>();
for (int i = 0; i < 100; i++)
{
    var taskId = i;
    tasks.Add(Task.Run(async () =>
    {
        var connection = pool.GetConnection();
        try
        {
            await connection.ExecuteQueryAsync($"SELECT * FROM Products WHERE Id = {taskId}");
        }
        finally
        {
            pool.ReleaseConnection(connection);
        }
    }));
}

await Task.WhenAll(tasks);
```

---

## Caching Example (Flyweight Application)

```csharp
// Flyweight - Immutable product image
public class ProductImage
{
    public byte[] ImageData { get; }
    public string ImageHash { get; }
    
    public ProductImage(string imageUrl)
    {
        ImageData = DownloadImage(imageUrl);
        ImageHash = ComputeHash(ImageData);
        Console.WriteLine($"Downloaded image: {ImageHash}");
    }
    
    private byte[] DownloadImage(string url)
    {
        // Simulate downloading
        return new byte[500000];  // 500KB
    }
    
    private string ComputeHash(byte[] data)
    {
        using var sha = System.Security.Cryptography.SHA256.Create();
        var hash = sha.ComputeHash(data);
        return Convert.ToBase64String(hash);
    }
}

// Image Cache (Flyweight Factory)
public class ImageCache
{
    private readonly Dictionary<string, ProductImage> _cache = new();
    
    public ProductImage GetImage(string imageUrl)
    {
        if (_cache.ContainsKey(imageUrl))
        {
            Console.WriteLine($"Cache hit for {imageUrl}");
            return _cache[imageUrl];
        }
        
        Console.WriteLine($"Cache miss for {imageUrl}");
        var image = new ProductImage(imageUrl);
        _cache[imageUrl] = image;
        return image;
    }
    
    public int GetCacheSize() => _cache.Count;
}

// Product (uses flyweight for image)
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    private readonly ProductImage _image;  // Shared flyweight
    
    public Product(int id, string name, ProductImage image)
    {
        Id = id;
        Name = name;
        _image = image;
    }
}

// Usage
var imageCache = new ImageCache();

// 1000 products, but only 10 unique images
var products = new List<Product>();
for (int i = 0; i < 1000; i++)
{
    var imageUrl = $"https://example.com/images/product_{i % 10}.jpg";
    var image = imageCache.GetImage(imageUrl);
    products.Add(new Product(i, $"Product {i}", image));
}

Console.WriteLine($"\nTotal products: {products.Count}");
Console.WriteLine($"Unique images cached: {imageCache.GetCacheSize()}");
Console.WriteLine($"Memory saved: {(products.Count - imageCache.GetCacheSize()) * 500} KB");
```

---

## When to Use Flyweight Pattern

✅ **Use Flyweight when:**
- Application uses large numbers of objects
- Most object state can be made extrinsic (shared)
- Many objects can be replaced by few shared objects
- Object identity is not important
- Memory usage is a concern

❌ **Avoid when:**
- Few objects in the system
- Most state is extrinsic (nothing to share)
- Objects have unique state
- Performance overhead of looking up flyweights outweighs memory savings

---

## Advantages

✅ **Reduces memory usage** - Shares common state  
✅ **Improves performance** - Fewer objects means less GC pressure  
✅ **Centralized management** - Factory controls object creation  
✅ **Transparent to clients** - Clients don't need to know about sharing

---

## Disadvantages

❌ **Complexity** - Separating intrinsic/extrinsic state can be complex  
❌ **Runtime overhead** - Flyweight lookup and passing extrinsic state  
❌ **Thread safety** - Shared objects must be immutable or synchronized  
❌ **Harder to debug** - Shared state can make debugging tricky

---

## Intrinsic vs Extrinsic State

| Aspect | Intrinsic State | Extrinsic State |
|--------|----------------|-----------------|
| **Stored** | Inside flyweight | Passed by client |
| **Shared** | Yes, across objects | No, unique per object |
| **Immutability** | Must be immutable | Can change |
| **Examples** | Font, color, texture, icon | Position, size, unique ID |

---

## Related Patterns

- **Singleton** - Flyweight factory is often a singleton
- **Factory** - Uses factory to create and manage flyweights
- **Composite** - Can be combined with flyweight for shared leaf nodes
- **State/Strategy** - Can use flyweight for shared state/strategy objects

---

## Common Use Cases

- **Text editors** - Character formatting (fonts, styles)
- **Game development** - Trees, particles, sprites with shared assets
- **UI frameworks** - Icons, images used multiple times
- **Database connection pools** - Reuse expensive connections
- **String interning** - .NET strings are automatically interned
- **Object pools** - Reusable objects (threads, connections)

---

## .NET Built-in Flyweight

```csharp
// String interning (automatic flyweight)
string s1 = "hello";
string s2 = "hello";
Console.WriteLine(Object.ReferenceEquals(s1, s2));  // True - same object!

// Explicit interning
string s3 = new string(new[] { 'h', 'e', 'l', 'l', 'o' });
string s4 = string.Intern(s3);
Console.WriteLine(Object.ReferenceEquals(s1, s4));  // True - interned

// Integer boxing (small integers are cached)
int i1 = 100;
int i2 = 100;
object o1 = i1;
object o2 = i2;
// Small integers may be cached by runtime
```

---

## Best Practices

✅ Make flyweight objects **immutable**  
✅ Use **factory pattern** to control flyweight creation  
✅ Clearly separate **intrinsic** (shared) and **extrinsic** (unique) state  
✅ Use **weak references** if flyweights can be garbage collected  
✅ Consider **thread safety** for concurrent access  
✅ Profile **memory usage** to ensure pattern provides benefit
