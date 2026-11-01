## Abstract Factory vs Factory Method

Both **Abstract Factory** and **Factory Method** are creational design patterns that deal with object creation, but they serve different purposes and have distinct structures.

### **1. Factory Method Pattern**

-   **Purpose**: The Factory Method pattern is used to define an interface for creating an object, but it allows subclasses to alter the type of objects that will be created.
-   **Focus**: The focus is on a **single product**. It provides a method in a class that is responsible for instantiating an object, but it delegates the actual object creation to subclasses.
-   **Implementation**: Each subclass overrides the factory method to specify which class to instantiate.

#### Example:

Let's assume you have a simple factory method to create buttons in different operating systems:


```csharp
// Product interface
public interface IButton
{
    void Render();
}

// Concrete Products
public class WindowsButton : IButton
{
    public void Render()
    {
        Console.WriteLine("Rendering Windows Button");
    }
}

public class MacButton : IButton
{
    public void Render()
    {
        Console.WriteLine("Rendering Mac Button");
    }
}

// Creator (with the factory method)
public abstract class Dialog
{
    public abstract IButton CreateButton(); // Factory Method

    public void RenderWindow()
    {
        IButton button = CreateButton();
        button.Render();
    }
}

// Concrete Creators
public class WindowsDialog : Dialog
{
    public override IButton CreateButton()
    {
        return new WindowsButton();
    }
}

public class MacDialog : Dialog
{
    public override IButton CreateButton()
    {
        return new MacButton();
    }
}
``` 

**Key Points**:

-   **Single product** creation (e.g., one type of `Button` per dialog).
-   Each subclass decides which object to create by overriding the `CreateButton()` method.

### **2. Abstract Factory Pattern**

-   **Purpose**: The Abstract Factory pattern provides an interface for creating **families of related or dependent objects** without specifying their concrete classes.
-   **Focus**: The focus is on creating **multiple related products** (a family of products). It's a super-factory that can create related objects, not just one type.
-   **Implementation**: The pattern uses a set of abstract methods to create multiple products, and each concrete factory produces a specific variant of these products.

#### Example:

Let's extend the button example to also include checkboxes:

```csharp
// Abstract Products
public interface IButton
{
    void Render();
}

public interface ICheckbox
{
    void Check();
}

// Concrete Products for Windows
public class WindowsButton : IButton
{
    public void Render()
    {
        Console.WriteLine("Rendering Windows Button");
    }
}

public class WindowsCheckbox : ICheckbox
{
    public void Check()
    {
        Console.WriteLine("Checking Windows Checkbox");
    }
}

// Concrete Products for Mac
public class MacButton : IButton
{
    public void Render()
    {
        Console.WriteLine("Rendering Mac Button");
    }
}

public class MacCheckbox : ICheckbox
{
    public void Check()
    {
        Console.WriteLine("Checking Mac Checkbox");
    }
}

// Abstract Factory
public interface IGUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

// Concrete Factories
public class WindowsFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new WindowsButton();
    }

    public ICheckbox CreateCheckbox()
    {
        return new WindowsCheckbox();
    }
}

public class MacFactory : IGUIFactory
{
    public IButton CreateButton()
    {
        return new MacButton();
    }

    public ICheckbox CreateCheckbox()
    {
        return new MacCheckbox();
    }
}

// Client Code
public class Application
{
    private IButton _button;
    private ICheckbox _checkbox;

    public Application(IGUIFactory factory)
    {
        _button = factory.CreateButton();
        _checkbox = factory.CreateCheckbox();
    }

    public void Render()
    {
        _button.Render();
        _checkbox.Check();
    }
}
``` 

**Key Points**:

-   **Multiple related products** are created (both `Button` and `Checkbox` in this example).
-   The factory interface is responsible for creating families of objects.
-   The specific family of products (Windows or Mac) is decided by the concrete factory (`WindowsFactory`, `MacFactory`).

### **Key Differences**

|Aspect  | Factory Method | Abstract Factory |
|--|--|--|
| **Purpose** | To define a method for creating a single object, allowing subclasses to alter the type of object created. | To provide an interface for creating families of related or dependent objects. |
| **Focus** | Focuses on a single product creation. | Focuses on creating multiple related products. |
| **Example** | A factory method for creating a `Button` object. | A factory for creating both `Button` and `Checkbox` objects, possibly more. |
| **Number of Products** | Usually creates one type of product. | Creates multiple related products (a family).|
| **Implementation** | Subclasses override a method to specify which product to instantiate. | Uses a concrete factory class to instantiate families of related products. |
| **Level of Abstraction** | Less abstract: deals with the creation of one product. | More abstract: deals with the creation of multiple related products. |
| **Usage** | When you need a single product with varying implementations. |When you need to create a set of related products that must be used together. |

### When to Use:

-   **Factory Method**: Use it when you need to create one product but want to defer the decision of which class to instantiate to subclasses.
-   **Abstract Factory**: Use it when you need to create a family of related objects that should work together (e.g., different UI components for different platforms).
