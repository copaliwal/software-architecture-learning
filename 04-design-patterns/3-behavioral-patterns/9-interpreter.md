# Interpreter Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

---

## Problem

You need to evaluate or execute expressions written in a custom language/grammar, such as:
- Mathematical expressions
- Boolean logic
- Query languages
- Configuration DSLs
- Rule engines

Parsing and evaluating these expressions manually with if/switch statements becomes complex and unmaintainable.

---

## Solution

Define:
1. **Grammar** - Rules of the language
2. **Abstract expression** - Interface for all expressions
3. **Terminal expressions** - Leaf nodes (operands)
4. **Non-terminal expressions** - Composite nodes (operators)
5. **Context** - Holds global state
6. **Interpreter** - Evaluates expressions

Build an Abstract Syntax Tree (AST) from the expression and evaluate it.

---

## Structure

```
AbstractExpression
└── Interpret(context)

TerminalExpression
└── Interpret(context) - Leaf nodes (variables, constants)

NonTerminalExpression
├── LeftExpression
├── RightExpression
└── Interpret(context) - Composite nodes (operators)

Context
└── Holds variables and state

Client
└── Builds and interprets AST
```

---

## C# Implementation

### Example: Boolean Expression Interpreter

```csharp
// Context - Holds variable values
public class Context
{
    private readonly Dictionary<string, bool> _variables = new();
    
    public void SetVariable(string name, bool value)
    {
        _variables[name] = value;
    }
    
    public bool GetVariable(string name)
    {
        if (_variables.ContainsKey(name))
        {
            return _variables[name];
        }
        throw new Exception($"Variable '{name}' not found");
    }
}

// Abstract Expression
public interface IExpression
{
    bool Interpret(Context context);
}

// Terminal Expression - Variable
public class VariableExpression : IExpression
{
    private readonly string _name;
    
    public VariableExpression(string name)
    {
        _name = name;
    }
    
    public bool Interpret(Context context)
    {
        var value = context.GetVariable(_name);
        Console.WriteLine($"Variable '{_name}' = {value}");
        return value;
    }
}

// Terminal Expression - Constant
public class ConstantExpression : IExpression
{
    private readonly bool _value;
    
    public ConstantExpression(bool value)
    {
        _value = value;
    }
    
    public bool Interpret(Context context)
    {
        Console.WriteLine($"Constant = {_value}");
        return _value;
    }
}

// Non-Terminal Expression - AND
public class AndExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;
    
    public AndExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }
    
    public bool Interpret(Context context)
    {
        var leftResult = _left.Interpret(context);
        var rightResult = _right.Interpret(context);
        var result = leftResult && rightResult;
        Console.WriteLine($"AND({leftResult}, {rightResult}) = {result}");
        return result;
    }
}

// Non-Terminal Expression - OR
public class OrExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;
    
    public OrExpression(IExpression left, IExpression right)
    {
        _left = left;
        _right = right;
    }
    
    public bool Interpret(Context context)
    {
        var leftResult = _left.Interpret(context);
        var rightResult = _right.Interpret(context);
        var result = leftResult || rightResult;
        Console.WriteLine($"OR({leftResult}, {rightResult}) = {result}");
        return result;
    }
}

// Non-Terminal Expression - NOT
public class NotExpression : IExpression
{
    private readonly IExpression _expression;
    
    public NotExpression(IExpression expression)
    {
        _expression = expression;
    }
    
    public bool Interpret(Context context)
    {
        var expressionResult = _expression.Interpret(context);
        var result = !expressionResult;
        Console.WriteLine($"NOT({expressionResult}) = {result}");
        return result;
    }
}

// Usage
var context = new Context();
context.SetVariable("A", true);
context.SetVariable("B", false);
context.SetVariable("C", true);

// Expression: (A AND B) OR (NOT C)
// AST:
//        OR
//       /  \
//     AND  NOT
//     / \   |
//    A   B  C

var expression = new OrExpression(
    new AndExpression(
        new VariableExpression("A"),
        new VariableExpression("B")
    ),
    new NotExpression(
        new VariableExpression("C")
    )
);

Console.WriteLine("=== Evaluating: (A AND B) OR (NOT C) ===\n");
var result = expression.Interpret(context);
Console.WriteLine($"\nFinal Result: {result}");

// Output:
// === Evaluating: (A AND B) OR (NOT C) ===
// 
// Variable 'A' = True
// Variable 'B' = False
// AND(True, False) = False
// Variable 'C' = True
// NOT(True) = False
// OR(False, False) = False
// 
// Final Result: False
```

---

## Mathematical Expression Interpreter

```csharp
// Context for math expressions
public class MathContext
{
    private readonly Dictionary<string, int> _variables = new();
    
    public void SetVariable(string name, int value)
    {
        _variables[name] = value;
    }
    
    public int GetVariable(string name)
    {
        return _variables.ContainsKey(name) ? _variables[name] : 0;
    }
}

// Abstract expression
public interface IMathExpression
{
    int Interpret(MathContext context);
}

// Terminal - Number
public class NumberExpression : IMathExpression
{
    private readonly int _number;
    
    public NumberExpression(int number)
    {
        _number = number;
    }
    
    public int Interpret(MathContext context)
    {
        return _number;
    }
}

// Terminal - Variable
public class VariableMathExpression : IMathExpression
{
    private readonly string _name;
    
    public VariableMathExpression(string name)
    {
        _name = name;
    }
    
    public int Interpret(MathContext context)
    {
        return context.GetVariable(_name);
    }
}

// Non-Terminal - Add
public class AddExpression : IMathExpression
{
    private readonly IMathExpression _left;
    private readonly IMathExpression _right;
    
    public AddExpression(IMathExpression left, IMathExpression right)
    {
        _left = left;
        _right = right;
    }
    
    public int Interpret(MathContext context)
    {
        return _left.Interpret(context) + _right.Interpret(context);
    }
}

// Non-Terminal - Subtract
public class SubtractExpression : IMathExpression
{
    private readonly IMathExpression _left;
    private readonly IMathExpression _right;
    
    public SubtractExpression(IMathExpression left, IMathExpression right)
    {
        _left = left;
        _right = right;
    }
    
    public int Interpret(MathContext context)
    {
        return _left.Interpret(context) - _right.Interpret(context);
    }
}

// Non-Terminal - Multiply
public class MultiplyExpression : IMathExpression
{
    private readonly IMathExpression _left;
    private readonly IMathExpression _right;
    
    public MultiplyExpression(IMathExpression left, IMathExpression right)
    {
        _left = left;
        _right = right;
    }
    
    public int Interpret(MathContext context)
    {
        return _left.Interpret(context) * _right.Interpret(context);
    }
}

// Non-Terminal - Divide
public class DivideExpression : IMathExpression
{
    private readonly IMathExpression _left;
    private readonly IMathExpression _right;
    
    public DivideExpression(IMathExpression left, IMathExpression right)
    {
        _left = left;
        _right = right;
    }
    
    public int Interpret(MathContext context)
    {
        var divisor = _right.Interpret(context);
        if (divisor == 0)
            throw new DivideByZeroException();
        return _left.Interpret(context) / divisor;
    }
}

// Usage
var mathContext = new MathContext();
mathContext.SetVariable("x", 5);
mathContext.SetVariable("y", 10);

// Expression: (x + 3) * (y - 2)
// AST:
//         *
//        / \
//       +   -
//      / \ / \
//     x  3 y  2

var expr = new MultiplyExpression(
    new AddExpression(
        new VariableMathExpression("x"),
        new NumberExpression(3)
    ),
    new SubtractExpression(
        new VariableMathExpression("y"),
        new NumberExpression(2)
    )
);

Console.WriteLine("Expression: (x + 3) * (y - 2)");
Console.WriteLine($"Where x = 5, y = 10");
Console.WriteLine($"Result: {expr.Interpret(mathContext)}");  // (5 + 3) * (10 - 2) = 8 * 8 = 64
```

---

## Simple SQL-like Query Interpreter

```csharp
// Context - Database
public class QueryContext
{
    public List<Person> People { get; set; } = new()
    {
        new Person { Name = "Alice", Age = 25 },
        new Person { Name = "Bob", Age = 30 },
        new Person { Name = "Charlie", Age = 25 },
        new Person { Name = "David", Age = 35 }
    };
}

public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// Abstract expression
public interface IQueryExpression
{
    List<Person> Interpret(QueryContext context);
}

// Terminal - Select all
public class SelectAllExpression : IQueryExpression
{
    public List<Person> Interpret(QueryContext context)
    {
        Console.WriteLine("[Query] SELECT * FROM People");
        return context.People;
    }
}

// Non-Terminal - Where condition
public class WhereExpression : IQueryExpression
{
    private readonly IQueryExpression _source;
    private readonly Func<Person, bool> _predicate;
    private readonly string _condition;
    
    public WhereExpression(IQueryExpression source, Func<Person, bool> predicate, string condition)
    {
        _source = source;
        _predicate = predicate;
        _condition = condition;
    }
    
    public List<Person> Interpret(QueryContext context)
    {
        var sourceData = _source.Interpret(context);
        var filtered = sourceData.Where(_predicate).ToList();
        Console.WriteLine($"[Query] WHERE {_condition} (Found {filtered.Count} results)");
        return filtered;
    }
}

// Non-Terminal - Order by
public class OrderByExpression : IQueryExpression
{
    private readonly IQueryExpression _source;
    private readonly Func<Person, object> _keySelector;
    private readonly string _field;
    
    public OrderByExpression(IQueryExpression source, Func<Person, object> keySelector, string field)
    {
        _source = source;
        _keySelector = keySelector;
        _field = field;
    }
    
    public List<Person> Interpret(QueryContext context)
    {
        var sourceData = _source.Interpret(context);
        var ordered = sourceData.OrderBy(_keySelector).ToList();
        Console.WriteLine($"[Query] ORDER BY {_field}");
        return ordered;
    }
}

// Usage
var queryContext = new QueryContext();

// Query: SELECT * FROM People WHERE Age = 25 ORDER BY Name
var query = new OrderByExpression(
    new WhereExpression(
        new SelectAllExpression(),
        p => p.Age == 25,
        "Age = 25"
    ),
    p => p.Name,
    "Name"
);

Console.WriteLine("=== Executing Query ===\n");
var results = query.Interpret(queryContext);

Console.WriteLine("\n=== Results ===");
foreach (var person in results)
{
    Console.WriteLine($"Name: {person.Name}, Age: {person.Age}");
}

// Output:
// === Executing Query ===
// 
// [Query] SELECT * FROM People
// [Query] WHERE Age = 25 (Found 2 results)
// [Query] ORDER BY Name
// 
// === Results ===
// Name: Alice, Age: 25
// Name: Charlie, Age: 25
```

---

## Configuration DSL Interpreter

```csharp
// Simple configuration language
// Example: SET timeout = 30; SET retry = 3; GET timeout

public class ConfigContext
{
    private readonly Dictionary<string, string> _settings = new();
    
    public void Set(string key, string value)
    {
        _settings[key] = value;
        Console.WriteLine($"[Config] {key} = {value}");
    }
    
    public string Get(string key)
    {
        return _settings.ContainsKey(key) ? _settings[key] : null;
    }
}

public interface IConfigExpression
{
    void Execute(ConfigContext context);
}

public class SetExpression : IConfigExpression
{
    private readonly string _key;
    private readonly string _value;
    
    public SetExpression(string key, string value)
    {
        _key = key;
        _value = value;
    }
    
    public void Execute(ConfigContext context)
    {
        context.Set(_key, _value);
    }
}

public class GetExpression : IConfigExpression
{
    private readonly string _key;
    
    public GetExpression(string key)
    {
        _key = key;
    }
    
    public void Execute(ConfigContext context)
    {
        var value = context.Get(_key);
        Console.WriteLine($"[Config] GET {_key} = {value ?? "NOT FOUND"}");
    }
}

// Simple parser
public class ConfigParser
{
    public static IConfigExpression Parse(string command)
    {
        var parts = command.Trim().Split(' ', StringSplitOptions.RemoveEmptyEntries);
        
        if (parts[0].ToUpper() == "SET" && parts.Length == 4 && parts[2] == "=")
        {
            return new SetExpression(parts[1], parts[3]);
        }
        
        if (parts[0].ToUpper() == "GET" && parts.Length == 2)
        {
            return new GetExpression(parts[1]);
        }
        
        throw new Exception($"Invalid command: {command}");
    }
}

// Usage
var configContext = new ConfigContext();

var commands = new[]
{
    "SET timeout = 30",
    "SET retry = 3",
    "SET host = localhost",
    "GET timeout",
    "GET retry",
    "GET unknown"
};

Console.WriteLine("=== Executing Configuration Commands ===\n");

foreach (var command in commands)
{
    var expression = ConfigParser.Parse(command);
    expression.Execute(configContext);
}

// Output:
// === Executing Configuration Commands ===
// 
// [Config] timeout = 30
// [Config] retry = 3
// [Config] host = localhost
// [Config] GET timeout = 30
// [Config] GET retry = 3
// [Config] GET unknown = NOT FOUND
```

---

## Roman Numeral Interpreter

```csharp
public class RomanContext
{
    public string Input { get; set; }
    public int Output { get; set; }
    
    public RomanContext(string input)
    {
        Input = input;
    }
}

public interface IRomanExpression
{
    void Interpret(RomanContext context);
}

public class RomanNumberExpression : IRomanExpression
{
    private readonly int _value;
    private readonly string _roman;
    
    public RomanNumberExpression(int value, string roman)
    {
        _value = value;
        _roman = roman;
    }
    
    public void Interpret(RomanContext context)
    {
        while (context.Input.StartsWith(_roman))
        {
            context.Output += _value;
            context.Input = context.Input.Substring(_roman.Length);
        }
    }
}

// Usage
var romanExpressions = new List<IRomanExpression>
{
    new RomanNumberExpression(1000, "M"),
    new RomanNumberExpression(900, "CM"),
    new RomanNumberExpression(500, "D"),
    new RomanNumberExpression(400, "CD"),
    new RomanNumberExpression(100, "C"),
    new RomanNumberExpression(90, "XC"),
    new RomanNumberExpression(50, "L"),
    new RomanNumberExpression(40, "XL"),
    new RomanNumberExpression(10, "X"),
    new RomanNumberExpression(9, "IX"),
    new RomanNumberExpression(5, "V"),
    new RomanNumberExpression(4, "IV"),
    new RomanNumberExpression(1, "I")
};

var testCases = new[] { "IX", "MCMXC", "MMXXIV", "CDXLIV" };

foreach (var roman in testCases)
{
    var context = new RomanContext(roman);
    
    foreach (var expression in romanExpressions)
    {
        expression.Interpret(context);
    }
    
    Console.WriteLine($"{roman} = {context.Output}");
}

// Output:
// IX = 9
// MCMXC = 1990
// MMXXIV = 2024
// CDXLIV = 444
```

---

## When to Use Interpreter Pattern

✅ **Use when:**
- Grammar is simple and stable
- Efficiency is not critical
- Need to interpret custom DSL
- Want to represent grammar as classes

❌ **Avoid when:**
- Grammar is complex (use parser generators like ANTLR)
- Performance is critical
- Grammar changes frequently
- Better tools exist (regex, LINQ, expression trees)

---

## Advantages

✅ **Easy to extend** - Add new expressions easily  
✅ **Grammar representation** - Each rule is a class  
✅ **Flexibility** - Change interpretation without changing grammar

---

## Disadvantages

❌ **Complex grammars** - Many classes for complex languages  
❌ **Performance** - Slower than compiled approaches  
❌ **Maintenance** - Large class hierarchies

---

## Interpreter vs Other Approaches

| Approach | Use Case | Example |
|----------|----------|---------|
| **Interpreter Pattern** | Simple custom DSL | Boolean expressions, math |
| **Regex** | Pattern matching | Text validation |
| **Parser Generator** | Complex languages | Programming languages (ANTLR) |
| **Expression Trees** | Dynamic queries | LINQ, Entity Framework |

---

## Related Patterns

- **Composite** - Grammar structure is often composite
- **Visitor** - Can be used to interpret AST
- **Flyweight** - Share terminal symbols

---

## Common Use Cases

- **Boolean expressions** - Rule engines
- **Mathematical expressions** - Calculators
- **Query languages** - Simple SQL-like DSLs
- **Configuration languages** - Custom config formats
- **Regular expressions** - Pattern matching
- **Roman numerals** - Number conversion

---

## Best Practices

✅ Keep **grammar simple** - complex grammars need parser generators  
✅ Use **Composite pattern** for tree structure  
✅ Consider **Flyweight** for shared terminals  
✅ Build **AST first**, then interpret  
✅ Use **Visitor** for complex operations on AST  
✅ Document **grammar** (BNF notation)  
✅ For .NET, consider **Expression Trees** or **Roslyn** for complex scenarios