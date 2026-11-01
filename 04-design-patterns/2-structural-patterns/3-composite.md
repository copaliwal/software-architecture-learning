The **Composite** design pattern is a structural pattern that allows you to compose objects into tree-like structures to represent part-whole hierarchies. This pattern lets clients treat individual objects and compositions of objects uniformly.

### Scenario:

Consider a graphic system where you can group shapes like **Circles** and **Squares** into a **Composite Shape**. This allows you to treat both individual shapes and groups of shapes uniformly.

### 1. **Component Interface (IShape)**

This interface defines common operations for both simple and composite shapes.

```csharp
// Component interface
public interface IShape
{
    void Draw();
}
``` 

### 2. **Leaf Classes**

These are the basic shapes that implement the `IShape` interface.


```csharp
// Leaf class: Circle
public class Circle : IShape
{
    private readonly string _name;

    public Circle(string name)
    {
        _name = name;
    }

    public void Draw()
    {
        Console.WriteLine($"Drawing Circle: {_name}");
    }
}

// Leaf class: Square
public class Square : IShape
{
    private readonly string _name;

    public Square(string name)
    {
        _name = name;
    }

    public void Draw()
    {
        Console.WriteLine($"Drawing Square: {_name}");
    }
}
``` 

### 3. **Composite Class**

This class can hold multiple `IShape` objects (both leaves and composites).

```csharp
// Composite class
public class CompositeShape : IShape
{
    private readonly List<IShape> _shapes = new List<IShape>();

    // Method to add shapes to the composite
    public void Add(IShape shape)
    {
        _shapes.Add(shape);
    }

    // Method to remove shapes from the composite
    public void Remove(IShape shape)
    {
        _shapes.Remove(shape);
    }

    // Draw method to draw all shapes in the composite
    public void Draw()
    {
        Console.WriteLine("Drawing Composite Shape:");
        foreach (var shape in _shapes)
        {
            shape.Draw();
        }
    }
}
``` 

### 4. **Client Code**

The client can create a composite shape and add different shapes to it.

```csharp 
class Program
{
    static void Main(string[] args)
    {
        // Create leaf shapes
        Circle circle1 = new Circle("Circle 1");
        Circle circle2 = new Circle("Circle 2");
        Square square1 = new Square("Square 1");
        
        // Create a composite shape
        CompositeShape compositeShape = new CompositeShape();
        
        // Add shapes to the composite
        compositeShape.Add(circle1);
        compositeShape.Add(circle2);
        compositeShape.Add(square1);
        
        // Draw the composite shape
        compositeShape.Draw();
        
        // Create another composite shape
        CompositeShape anotherComposite = new CompositeShape();
        
        // Add composite to another composite
        anotherComposite.Add(compositeShape);
        anotherComposite.Add(new Square("Square 2"));
        
        // Draw the second composite shape
        anotherComposite.Draw();
    }
}
``` 

### **Output**:

mathematica

Copy code

    Drawing Composite Shape:
    Drawing Circle: Circle 1
    Drawing Circle: Circle 2
    Drawing Square: Square 1
    Drawing Composite Shape:
    Drawing Composite Shape:
    Drawing Circle: Circle 1
    Drawing Circle: Circle 2
    Drawing Square: Square 1
    Drawing Square: Square 2

### Explanation:

1.  **Component Interface (`IShape`)**: This interface defines the common operations for all shapes, both simple and composite.
2.  **Leaf Classes (`Circle`, `Square`)**: These classes implement the `IShape` interface and define how to draw individual shapes.
3.  **Composite Class (`CompositeShape`)**: This class can hold multiple shapes (both leaf and composite) and implements the `Draw()` method to draw all contained shapes.
4.  **Client**: The client creates individual shapes and composite shapes. It can call the `Draw()` method on the composite shape, which in turn calls the `Draw()` method on each contained shape.

### Benefits of the Composite Pattern:

-   **Uniformity**: Clients can treat individual objects and compositions of objects uniformly. They can interact with the structure as if it is a single entity.
-   **Flexibility**: You can easily add new types of components without affecting existing code.
-   **Simplicity**: It simplifies the client code by allowing it to work with hierarchical structures without needing to understand the details of each individual object.
