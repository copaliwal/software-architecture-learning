The **Decorator** Design Pattern allows you to dynamically add behavior to an object at runtime without altering its structure. Here’s a simple example in C# to illustrate this pattern:

### Step 1: Define the Component Interface


```csharp
public interface ICoffee
{
    string GetDescription();
    double GetCost();
}

```


### Step 2: Create Concrete Components

```csharp
public class SimpleCoffee : ICoffee
{
    public string GetDescription()
    {
        return "Simple Coffee";
    }

    public double GetCost()
    {
        return 5.0;
    }
}

```

### Step 3: Create the Decorator Base Class

```csharp
public abstract class CoffeeDecorator : ICoffee
{
    protected ICoffee _coffee;

    public CoffeeDecorator(ICoffee coffee)
    {
        _coffee = coffee;
    }

    public virtual string GetDescription()
    {
        return _coffee.GetDescription();
    }

    public virtual double GetCost()
    {
        return _coffee.GetCost();
    }
}

```



### Step 4: Create Concrete Decorators


```csharp
public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription()
    {
        return _coffee.GetDescription() + ", Milk";
    }

    public override double GetCost()
    {
        return _coffee.GetCost() + 1.5;
    }
}


public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(ICoffee coffee) : base(coffee) { }

    public override string GetDescription()
    {
        return _coffee.GetDescription() + ", Sugar";
    }

    public override double GetCost()
    {
        return _coffee.GetCost() + 0.5;
    }
}

```


### Step 5: Use the Decorator Pattern


```csharp
class Program
{
    static void Main(string[] args)
    {
        ICoffee coffee = new SimpleCoffee();
        Console.WriteLine($"{coffee.GetDescription()} : ${coffee.GetCost()}");

        coffee = new MilkDecorator(coffee);
        Console.WriteLine($"{coffee.GetDescription()} : ${coffee.GetCost()}");

        coffee = new SugarDecorator(coffee);
        Console.WriteLine($"{coffee.GetDescription()} : ${coffee.GetCost()}");
    }
}

```
Output

    Simple Coffee Cost: $2.00 
    Simple Coffee, Milk Cost: $2.50 
    Simple Coffee, Milk, Sugar Cost: $2.75 

In this example:

-   `ICoffee`  is the component interface.
-   `SimpleCoffee`  is a concrete component.
-   `CoffeeDecorator`  is the base decorator class.
-   `MilkDecorator`  and  `SugarDecorator`  are concrete decorators that add new behavior.

[This setup allows you to add new functionalities (like milk and sugar) to the coffee dynamically without modifying the existing classes](https://dotnettutorials.net/lesson/decorator-design-pattern/)[1](https://dotnettutorials.net/lesson/decorator-design-pattern/)[2](https://www.csharptutorial.net/csharp-design-patterns/csharp-decorator-pattern/).
