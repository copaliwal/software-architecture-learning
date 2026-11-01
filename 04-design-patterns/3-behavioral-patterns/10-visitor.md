# Visitor Pattern

## Overview

**Category:** Behavioral Pattern  
**Intent:** Represent an operation to be performed on elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.

---

## Problem

You need to perform operations on a complex object structure (like a tree), but:
- Adding new operations requires modifying all element classes
- Operations are not related to the element's primary responsibility
- You want to keep related operations together
- Different operations need different data

Directly adding methods to each element violates Open/Closed Principle.

---

## Solution

Move operations into separate visitor classes. Elements accept a visitor and call the appropriate visitor method for their type (double dispatch).

**Key Concept:** Double dispatch - the operation executed depends on both the visitor type AND the element type.

---

## Structure

```
Visitor (interface)
├── VisitElementA(elementA)
└── VisitElementB(elementB)

ConcreteVisitor1
├── VisitElementA() - Operation for ElementA
└── VisitElementB() - Operation for ElementB

Element (interface)
└── Accept(visitor)

ConcreteElementA
└── Accept(visitor) - calls visitor.VisitElementA(this)

ConcreteElementB
└── Accept(visitor) - calls visitor.VisitElementB(this)
```

---

## C# Implementation

### Example: Document Structure

```csharp
// Visitor interface
public interface IDocumentVisitor
{
    void Visit(Paragraph paragraph);
    void Visit(Image image);
    void Visit(Table table);
}

// Element interface
public interface IDocumentElement
{
    void Accept(IDocumentVisitor visitor);
}

// Concrete Elements
public class Paragraph : IDocumentElement
{
    public string Text { get; set; }
    
    public Paragraph(string text)
    {
        Text = text;
    }
    
    public void Accept(IDocumentVisitor visitor)
    {
        visitor.Visit(this);  // Double dispatch
    }
}

public class Image : IDocumentElement
{
    public string FileName { get; set; }
    public int Width { get; set; }
    public int Height { get; set; }
    
    public Image(string fileName, int width, int height)
    {
        FileName = fileName;
        Width = width;
        Height = height;
    }
    
    public void Accept(IDocumentVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class Table : IDocumentElement
{
    public int Rows { get; set; }
    public int Columns { get; set; }
    
    public Table(int rows, int columns)
    {
        Rows = rows;
        Columns = columns;
    }
    
    public void Accept(IDocumentVisitor visitor)
    {
        visitor.Visit(this);
    }
}

// Concrete Visitor 1: HTML Exporter
public class HtmlExportVisitor : IDocumentVisitor
{
    private readonly StringBuilder _html = new();
    
    public void Visit(Paragraph paragraph)
    {
        _html.AppendLine($"<p>{paragraph.Text}</p>");
    }
    
    public void Visit(Image image)
    {
        _html.AppendLine($"<img src=\"{image.FileName}\" width=\"{image.Width}\" height=\"{image.Height}\" />");
    }
    
    public void Visit(Table table)
    {
        _html.AppendLine($"<table rows=\"{table.Rows}\" cols=\"{table.Columns}\"></table>");
    }
    
    public string GetHtml() => _html.ToString();
}

// Concrete Visitor 2: Markdown Exporter
public class MarkdownExportVisitor : IDocumentVisitor
{
    private readonly StringBuilder _markdown = new();
    
    public void Visit(Paragraph paragraph)
    {
        _markdown.AppendLine(paragraph.Text);
        _markdown.AppendLine();
    }
    
    public void Visit(Image image)
    {
        _markdown.AppendLine($"![Image]({image.FileName})");
    }
    
    public void Visit(Table table)
    {
        _markdown.AppendLine($"| Table {table.Rows}x{table.Columns} |");
    }
    
    public string GetMarkdown() => _markdown.ToString();
}

// Concrete Visitor 3: Word Counter
public class WordCountVisitor : IDocumentVisitor
{
    public int WordCount { get; private set; }
    
    public void Visit(Paragraph paragraph)
    {
        WordCount += paragraph.Text.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length;
    }
    
    public void Visit(Image image)
    {
        // Images don't contain words
    }
    
    public void Visit(Table table)
    {
        // Simplified: assume each cell has one word
        WordCount += table.Rows * table.Columns;
    }
}

// Document
public class Document
{
    private readonly List<IDocumentElement> _elements = new();
    
    public void Add(IDocumentElement element)
    {
        _elements.Add(element);
    }
    
    public void Accept(IDocumentVisitor visitor)
    {
        foreach (var element in _elements)
        {
            element.Accept(visitor);
        }
    }
}

// Usage
var document = new Document();
document.Add(new Paragraph("Welcome to my blog post!"));
document.Add(new Image("photo.jpg", 800, 600));
document.Add(new Paragraph("This is a very interesting article about design patterns."));
document.Add(new Table(3, 4));

// Export to HTML
var htmlVisitor = new HtmlExportVisitor();
document.Accept(htmlVisitor);
Console.WriteLine("=== HTML Export ===");
Console.WriteLine(htmlVisitor.GetHtml());

// Export to Markdown
var markdownVisitor = new MarkdownExportVisitor();
document.Accept(markdownVisitor);
Console.WriteLine("=== Markdown Export ===");
Console.WriteLine(markdownVisitor.GetMarkdown());

// Count words
var wordCountVisitor = new WordCountVisitor();
document.Accept(wordCountVisitor);
Console.WriteLine($"=== Word Count ===");
Console.WriteLine($"Total words: {wordCountVisitor.WordCount}");

// Output:
// === HTML Export ===
// <p>Welcome to my blog post!</p>
// <img src="photo.jpg" width="800" height="600" />
// <p>This is a very interesting article about design patterns.</p>
// <table rows="3" cols="4"></table>
// 
// === Markdown Export ===
// Welcome to my blog post!
// 
// ![Image](photo.jpg)
// This is a very interesting article about design patterns.
// 
// | Table 3x4 |
// 
// === Word Count ===
// Total words: 27
```

---

## File System Example

```csharp
// Visitor
public interface IFileSystemVisitor
{
    void Visit(FileNode file);
    void Visit(DirectoryNode directory);
}

// Elements
public interface IFileSystemNode
{
    string Name { get; }
    void Accept(IFileSystemVisitor visitor);
}

public class FileNode : IFileSystemNode
{
    public string Name { get; }
    public long Size { get; }
    public string Extension { get; }
    
    public FileNode(string name, long size, string extension)
    {
        Name = name;
        Size = size;
        Extension = extension;
    }
    
    public void Accept(IFileSystemVisitor visitor)
    {
        visitor.Visit(this);
    }
}

public class DirectoryNode : IFileSystemNode
{
    public string Name { get; }
    public List<IFileSystemNode> Children { get; } = new();
    
    public DirectoryNode(string name)
    {
        Name = name;
    }
    
    public void Add(IFileSystemNode node)
    {
        Children.Add(node);
    }
    
    public void Accept(IFileSystemVisitor visitor)
    {
        visitor.Visit(this);
        
        // Visit children
        foreach (var child in Children)
        {
            child.Accept(visitor);
        }
    }
}

// Concrete Visitor 1: Size Calculator
public class SizeCalculatorVisitor : IFileSystemVisitor
{
    public long TotalSize { get; private set; }
    
    public void Visit(FileNode file)
    {
        TotalSize += file.Size;
        Console.WriteLine($"File: {file.Name} ({file.Size} bytes)");
    }
    
    public void Visit(DirectoryNode directory)
    {
        Console.WriteLine($"Directory: {directory.Name}");
    }
}

// Concrete Visitor 2: Search by Extension
public class ExtensionSearchVisitor : IFileSystemVisitor
{
    private readonly string _targetExtension;
    public List<FileNode> MatchingFiles { get; } = new();
    
    public ExtensionSearchVisitor(string extension)
    {
        _targetExtension = extension;
    }
    
    public void Visit(FileNode file)
    {
        if (file.Extension == _targetExtension)
        {
            MatchingFiles.Add(file);
        }
    }
    
    public void Visit(DirectoryNode directory)
    {
        // Just traverse, don't collect directories
    }
}

// Concrete Visitor 3: Tree Printer
public class TreePrintVisitor : IFileSystemVisitor
{
    private int _indentLevel = 0;
    
    public void Visit(FileNode file)
    {
        Console.WriteLine($"{GetIndent()}📄 {file.Name} ({file.Size} bytes)");
    }
    
    public void Visit(DirectoryNode directory)
    {
        Console.WriteLine($"{GetIndent()}📁 {directory.Name}");
        _indentLevel++;
        
        foreach (var child in directory.Children)
        {
            child.Accept(this);
        }
        
        _indentLevel--;
    }
    
    private string GetIndent() => new string(' ', _indentLevel * 2);
}

// Usage
var root = new DirectoryNode("Projects");

var src = new DirectoryNode("src");
src.Add(new FileNode("Program.cs", 1500, ".cs"));
src.Add(new FileNode("Helper.cs", 800, ".cs"));

var docs = new DirectoryNode("docs");
docs.Add(new FileNode("README.md", 500, ".md"));
docs.Add(new FileNode("API.md", 1200, ".md"));

var images = new DirectoryNode("images");
images.Add(new FileNode("logo.png", 50000, ".png"));
images.Add(new FileNode("banner.jpg", 75000, ".jpg"));

root.Add(src);
root.Add(docs);
root.Add(images);

// Print tree
Console.WriteLine("=== Directory Tree ===\n");
var treePrinter = new TreePrintVisitor();
root.Accept(treePrinter);

// Calculate total size
Console.WriteLine("\n=== Size Calculation ===\n");
var sizeCalculator = new SizeCalculatorVisitor();
root.Accept(sizeCalculator);
Console.WriteLine($"\nTotal size: {sizeCalculator.TotalSize} bytes");

// Search for .cs files
Console.WriteLine("\n=== Search for .cs files ===\n");
var csSearch = new ExtensionSearchVisitor(".cs");
root.Accept(csSearch);
foreach (var file in csSearch.MatchingFiles)
{
    Console.WriteLine($"Found: {file.Name}");
}
```

---

## Expression Evaluator Example

```csharp
// Visitor
public interface IExpressionVisitor
{
    int Visit(NumberExpression number);
    int Visit(AdditionExpression addition);
    int Visit(MultiplicationExpression multiplication);
}

// Elements
public interface IExpression
{
    int Accept(IExpressionVisitor visitor);
}

public class NumberExpression : IExpression
{
    public int Value { get; }
    
    public NumberExpression(int value)
    {
        Value = value;
    }
    
    public int Accept(IExpressionVisitor visitor)
    {
        return visitor.Visit(this);
    }
}

public class AdditionExpression : IExpression
{
    public IExpression Left { get; }
    public IExpression Right { get; }
    
    public AdditionExpression(IExpression left, IExpression right)
    {
        Left = left;
        Right = right;
    }
    
    public int Accept(IExpressionVisitor visitor)
    {
        return visitor.Visit(this);
    }
}

public class MultiplicationExpression : IExpression
{
    public IExpression Left { get; }
    public IExpression Right { get; }
    
    public MultiplicationExpression(IExpression left, IExpression right)
    {
        Left = left;
        Right = right;
    }
    
    public int Accept(IExpressionVisitor visitor)
    {
        return visitor.Visit(this);
    }
}

// Concrete Visitor 1: Evaluator
public class EvaluatorVisitor : IExpressionVisitor
{
    public int Visit(NumberExpression number)
    {
        return number.Value;
    }
    
    public int Visit(AdditionExpression addition)
    {
        var left = addition.Left.Accept(this);
        var right = addition.Right.Accept(this);
        return left + right;
    }
    
    public int Visit(MultiplicationExpression multiplication)
    {
        var left = multiplication.Left.Accept(this);
        var right = multiplication.Right.Accept(this);
        return left * right;
    }
}

// Concrete Visitor 2: String Printer
public class PrintVisitor : IExpressionVisitor
{
    public int Visit(NumberExpression number)
    {
        Console.Write(number.Value);
        return 0;
    }
    
    public int Visit(AdditionExpression addition)
    {
        Console.Write("(");
        addition.Left.Accept(this);
        Console.Write(" + ");
        addition.Right.Accept(this);
        Console.Write(")");
        return 0;
    }
    
    public int Visit(MultiplicationExpression multiplication)
    {
        Console.Write("(");
        multiplication.Left.Accept(this);
        Console.Write(" * ");
        multiplication.Right.Accept(this);
        Console.Write(")");
        return 0;
    }
}

// Usage: (2 + 3) * (4 + 5)
var expression = new MultiplicationExpression(
    new AdditionExpression(
        new NumberExpression(2),
        new NumberExpression(3)
    ),
    new AdditionExpression(
        new NumberExpression(4),
        new NumberExpression(5)
    )
);

Console.Write("Expression: ");
var printer = new PrintVisitor();
expression.Accept(printer);
Console.WriteLine();

var evaluator = new EvaluatorVisitor();
var result = expression.Accept(evaluator);
Console.WriteLine($"Result: {result}");

// Output:
// Expression: ((2 + 3) * (4 + 5))
// Result: 45
```

---

## Shopping Cart Example

```csharp
public interface IItemVisitor
{
    decimal Visit(Book book);
    decimal Visit(Electronics electronics);
    decimal Visit(Clothing clothing);
}

public interface ICartItem
{
    string Name { get; }
    decimal BasePrice { get; }
    decimal Accept(IItemVisitor visitor);
}

public class Book : ICartItem
{
    public string Name { get; }
    public decimal BasePrice { get; }
    public string ISBN { get; }
    
    public Book(string name, decimal price, string isbn)
    {
        Name = name;
        BasePrice = price;
        ISBN = isbn;
    }
    
    public decimal Accept(IItemVisitor visitor) => visitor.Visit(this);
}

public class Electronics : ICartItem
{
    public string Name { get; }
    public decimal BasePrice { get; }
    public int WarrantyMonths { get; }
    
    public Electronics(string name, decimal price, int warrantyMonths)
    {
        Name = name;
        BasePrice = price;
        WarrantyMonths = warrantyMonths;
    }
    
    public decimal Accept(IItemVisitor visitor) => visitor.Visit(this);
}

public class Clothing : ICartItem
{
    public string Name { get; }
    public decimal BasePrice { get; }
    public string Size { get; }
    
    public Clothing(string name, decimal price, string size)
    {
        Name = name;
        BasePrice = price;
        Size = size;
    }
    
    public decimal Accept(IItemVisitor visitor) => visitor.Visit(this);
}

// Visitor 1: Tax Calculator (different tax rates per category)
public class TaxCalculatorVisitor : IItemVisitor
{
    public decimal Visit(Book book)
    {
        return book.BasePrice * 0.05m;  // 5% tax on books
    }
    
    public decimal Visit(Electronics electronics)
    {
        return electronics.BasePrice * 0.15m;  // 15% tax on electronics
    }
    
    public decimal Visit(Clothing clothing)
    {
        return clothing.BasePrice * 0.10m;  // 10% tax on clothing
    }
}

// Visitor 2: Discount Calculator
public class DiscountCalculatorVisitor : IItemVisitor
{
    public decimal Visit(Book book)
    {
        return book.BasePrice * 0.10m;  // 10% discount on books
    }
    
    public decimal Visit(Electronics electronics)
    {
        // No discount on electronics
        return 0;
    }
    
    public decimal Visit(Clothing clothing)
    {
        return clothing.BasePrice * 0.20m;  // 20% discount on clothing
    }
}

// Visitor 3: Shipping Cost Calculator
public class ShippingCostVisitor : IItemVisitor
{
    public decimal Visit(Book book)
    {
        return 5.00m;  // Flat rate for books
    }
    
    public decimal Visit(Electronics electronics)
    {
        return 15.00m;  // Higher shipping for electronics
    }
    
    public decimal Visit(Clothing clothing)
    {
        return 7.50m;  // Medium rate for clothing
    }
}

// Shopping Cart
public class ShoppingCart
{
    private readonly List<ICartItem> _items = new();
    
    public void Add(ICartItem item)
    {
        _items.Add(item);
    }
    
    public void CalculateTotals()
    {
        var taxVisitor = new TaxCalculatorVisitor();
        var discountVisitor = new DiscountCalculatorVisitor();
        var shippingVisitor = new ShippingCostVisitor();
        
        decimal subtotal = 0;
        decimal totalTax = 0;
        decimal totalDiscount = 0;
        decimal totalShipping = 0;
        
        Console.WriteLine("=== Shopping Cart ===\n");
        
        foreach (var item in _items)
        {
            var tax = item.Accept(taxVisitor);
            var discount = item.Accept(discountVisitor);
            var shipping = item.Accept(shippingVisitor);
            
            subtotal += item.BasePrice;
            totalTax += tax;
            totalDiscount += discount;
            totalShipping += shipping;
            
            Console.WriteLine($"{item.Name}:");
            Console.WriteLine($"  Price: ${item.BasePrice:F2}");
            Console.WriteLine($"  Tax: ${tax:F2}");
            Console.WriteLine($"  Discount: -${discount:F2}");
            Console.WriteLine($"  Shipping: ${shipping:F2}");
            Console.WriteLine();
        }
        
        var total = subtotal + totalTax - totalDiscount + totalShipping;
        
        Console.WriteLine($"Subtotal: ${subtotal:F2}");
        Console.WriteLine($"Tax: ${totalTax:F2}");
        Console.WriteLine($"Discount: -${totalDiscount:F2}");
        Console.WriteLine($"Shipping: ${totalShipping:F2}");
        Console.WriteLine($"TOTAL: ${total:F2}");
    }
}

// Usage
var cart = new ShoppingCart();
cart.Add(new Book("Design Patterns", 45.00m, "123-456"));
cart.Add(new Electronics("Laptop", 1200.00m, 24));
cart.Add(new Clothing("T-Shirt", 25.00m, "L"));

cart.CalculateTotals();
```

---

## When to Use Visitor Pattern

✅ **Use when:**
- Many distinct operations need to be performed on object structure
- Object structure is stable (classes rarely change)
- Operations are unrelated to element's primary responsibility
- Want to keep related operations together

❌ **Avoid when:**
- Object structure changes frequently (adding new elements requires updating all visitors)
- Few operations on structure
- Operations are core to element's purpose

---

## Advantages

✅ **Open/Closed** - Add new operations (visitors) without modifying elements  
✅ **Single Responsibility** - Related operations grouped in visitor  
✅ **Accumulate state** - Visitor can accumulate information  
✅ **Multiple operations** - Easy to add new visitors

---

## Disadvantages

❌ **Hard to add elements** - New element type requires updating all visitors  
❌ **Breaking encapsulation** - Visitors may need access to element internals  
❌ **Circular dependency** - Elements and visitors depend on each other

---

## Double Dispatch

```csharp
// Without Visitor (Single Dispatch - only visitor type matters)
public void Process(IElement element)
{
    element.DoSomething();  // Which method? Depends only on element type
}

// With Visitor (Double Dispatch - both types matter)
public void Process(IElement element, IVisitor visitor)
{
    element.Accept(visitor);  // Dispatches to element's Accept
    // Accept calls visitor.Visit(this) - dispatches to visitor's Visit
}
```

---

## Related Patterns

- **Composite** - Visitor often used with Composite structures
- **Iterator** - Visitor can use iterator to traverse structure
- **Interpreter** - Can use Visitor to evaluate AST

---

## Common Use Cases

- **Compilers** - AST traversal (type checking, code generation)
- **File systems** - Operations on file/directory trees
- **Document processing** - Export to multiple formats
- **Shopping carts** - Calculate taxes, discounts, shipping
- **Graphics** - Rendering, hit testing, bounding box calculation

---

## Best Practices

✅ Use when **object structure is stable**  
✅ Combine with **Composite pattern** for tree structures  
✅ Consider **generic visitors** to reduce boilerplate  
✅ Document **why visitor is needed** (multiple unrelated operations)  
✅ Use **interfaces** for both visitor and elements  
✅ Consider **extension methods** as alternative in C#