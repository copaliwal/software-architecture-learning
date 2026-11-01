The **Prototype** design pattern is used when creating a new instance of a class is either costly or complex, and instead, we can clone an existing instance to obtain a new object. This pattern allows you to create new objects by copying or cloning a prototype rather than instantiating new objects directly.

Here’s an example of the **Prototype** pattern in C# where we can clone different types of shapes (like **Circle** and **Rectangle**) to create new objects.

### 1. **Prototype Interface**

This interface declares the `Clone()` method, which will be implemented by concrete classes.

    // Prototype interface
    public interface IShape
    {
        IShape Clone();
        void GetInfo();
    }

 

### 2. **Concrete Prototypes**

These are concrete implementations of the prototype interface. They implement the `Clone()` method to return a copy of the current object.

    // Concrete prototype: Circle
    public class Circle : IShape
    {
        public int Radius { get; set; }
    
        public Circle(int radius)
        {
            Radius = radius;
        }
    
        // Clone method to create a copy of the Circle object
        public IShape Clone()
        {
            return new Circle(this.Radius);
        }
    
        public void GetInfo()
        {
            Console.WriteLine($"Circle with radius: {Radius}");
        }
    }
    
    // Concrete prototype: Rectangle
    public class Rectangle : IShape
    {
        public int Width { get; set; }
        public int Height { get; set; }
    
        public Rectangle(int width, int height)
        {
            Width = width;
            Height = height;
        }
    
        // Clone method to create a copy of the Rectangle object
        public IShape Clone()
        {
            return new Rectangle(this.Width, this.Height);
        }
    
        public void GetInfo()
        {
            Console.WriteLine($"Rectangle with width: {Width} and height: {Height}");
        }
    } 

### 3. **Client Code**

The client can create new objects by cloning existing ones instead of creating them from scratch.


    class Program
    {
        static void Main(string[] args)
        {
            // Create an instance of Circle and clone it
            Circle circle1 = new Circle(10);
            Circle circle2 = (Circle)circle1.Clone();
    
            // Modify the cloned object
            circle2.Radius = 20;
    
            // Display information about both circles
            circle1.GetInfo();  // Circle with radius: 10
            circle2.GetInfo();  // Circle with radius: 20
    
            // Create an instance of Rectangle and clone it
            Rectangle rectangle1 = new Rectangle(15, 25);
            Rectangle rectangle2 = (Rectangle)rectangle1.Clone();
    
            // Modify the cloned object
            rectangle2.Width = 30;
    
            // Display information about both rectangles
            rectangle1.GetInfo();  // Rectangle with width: 15 and height: 25
            rectangle2.GetInfo();  // Rectangle with width: 30 and height: 25
        }
    } 

### **Output**:

arduino

Copy code

    Circle with radius: 10
    Circle with radius: 20
    Rectangle with width: 15 and height: 25
    Rectangle with width: 30 and height: 25

### Explanation:

1.  **Prototype Interface (`IShape`)**: Declares the `Clone()` method that allows creating a copy of the current object.
2.  **Concrete Prototypes (`Circle`, `Rectangle`)**: Implement the `Clone()` method to clone themselves.
3.  **Client**: Uses the `Clone()` method to create new objects (clones) from existing objects instead of creating new ones from scratch.

In this example:

-   `Circle` and `Rectangle` objects are cloned using the `Clone()` method, allowing the creation of new objects that are copies of the original ones.
-   The client code does not need to know the exact type of the object being cloned; it can work with the `IShape` interface. This makes the system extensible as new shapes can be added without changing the client code.
