The **Builder** design pattern is used to construct a complex object step by step. It separates the construction process from the representation so that the same construction process can create different representations of the object.

Here’s a C# example where the **Builder** pattern is used to create a custom **House** object with various components (like walls, doors, windows, and a roof).

### 1. **Product Class (House)**

This class represents the complex object that will be built.

```csharp
public class House
{
    public string Walls { get; set; }
    public string Doors { get; set; }
    public string Windows { get; set; }
    public string Roof { get; set; }

    public override string ToString()
    {
        return $"House with {Walls}, {Doors}, {Windows}, and {Roof}.";
    }
}
```

### 2. **Builder Interface**

This interface declares the steps to build the various parts of the product.

```csharp
// Abstract builder interface
public interface IHouseBuilder
{
    void BuildWalls();
    void BuildDoors();
    void BuildWindows();
    void BuildRoof();
    House GetHouse();
}
```

### 3. **Concrete Builders**

These classes implement the steps defined in the `IHouseBuilder` interface to construct specific types of houses.

```csharp
// Concrete builder for a wooden house
public class WoodenHouseBuilder : IHouseBuilder
{
    private House _house = new House();

    public void BuildWalls()
    {
        _house.Walls = "Wooden Walls";
    }

    public void BuildDoors()
    {
        _house.Doors = "Wooden Doors";
    }

    public void BuildWindows()
    {
        _house.Windows = "Wooden Windows";
    }

    public void BuildRoof()
    {
        _house.Roof = "Wooden Roof";
    }

    public House GetHouse()
    {
        return _house;
    }
}

// Concrete builder for a stone house
public class StoneHouseBuilder : IHouseBuilder
{
    private House _house = new House();

    public void BuildWalls()
    {
        _house.Walls = "Stone Walls";
    }

    public void BuildDoors()
    {
        _house.Doors = "Steel Doors";
    }

    public void BuildWindows()
    {
        _house.Windows = "Glass Windows";
    }

    public void BuildRoof()
    {
        _house.Roof = "Concrete Roof";
    }

    public House GetHouse()
    {
        return _house;
    }
} 
```

### 4. **Director**

The **Director** class is responsible for executing the building steps in a particular sequence to construct the final product.

```csharp
// Director class that constructs the house using the builder
public class HouseDirector
{
    private IHouseBuilder _houseBuilder;

    public HouseDirector(IHouseBuilder houseBuilder)
    {
        _houseBuilder = houseBuilder;
    }

    // Builds the house step by step
    public void ConstructHouse()
    {
        _houseBuilder.BuildWalls();
        _houseBuilder.BuildDoors();
        _houseBuilder.BuildWindows();
        _houseBuilder.BuildRoof();
    }

    // Returns the finished house
    public House GetHouse()
    {
        return _houseBuilder.GetHouse();
    }
}
```

### 5. **Client Code**

The client can use the **Director** and the builders to create different types of houses.

```csharp
class Program
{
    static void Main(string[] args)
    {
        // Create a Wooden House
        IHouseBuilder woodenHouseBuilder = new WoodenHouseBuilder();
        HouseDirector director = new HouseDirector(woodenHouseBuilder);
        director.ConstructHouse();
        House woodenHouse = director.GetHouse();
        Console.WriteLine(woodenHouse);

        // Create a Stone House
        IHouseBuilder stoneHouseBuilder = new StoneHouseBuilder();
        director = new HouseDirector(stoneHouseBuilder);
        director.ConstructHouse();
        House stoneHouse = director.GetHouse();
        Console.WriteLine(stoneHouse);
    }
} 
```

### **Output**:

    House with Wooden Walls, Wooden Doors, Wooden Windows, and Wooden Roof.
    House with Stone Walls, Steel Doors, Glass Windows, and Concrete Roof.

### Explanation:

-   **Product (`House`)**: The complex object that we want to build.
-   **Builder Interface (`IHouseBuilder`)**: Specifies the steps to build a house.
-   **Concrete Builders (`WoodenHouseBuilder` and `StoneHouseBuilder`)**: Implement the steps for building specific types of houses (wooden and stone).
-   **Director (`HouseDirector`)**: Orchestrates the construction process by calling the builder's methods in the right sequence.
-   **Client**: Chooses which builder to use and interacts with the `HouseDirector` to construct the desired type of house.

In this example, the **Builder** pattern allows flexibility in the construction process, and new types of houses can easily be added by creating new builders without changing the client code.
