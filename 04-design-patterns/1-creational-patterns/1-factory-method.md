The **Factory Method** pattern defines an interface for creating an object but allows subclasses to alter the type of objects that will be created. Unlike the Abstract Factory pattern, which creates families of related objects, the Factory Method is about defining an interface for creating a single object and letting subclasses handle the instantiation.

Here’s an example in C# where the Factory Method is used to create different types of documents (e.g., **Word Document** and **PDF Document**) without specifying the exact class.

### 1. **Product Interface**

This defines the interface for the objects created by the factory method.

```csharp
// Abstract product interface
public interface IDocument
{
    void Open();
}
```

### 2. **Concrete Products**

These are specific implementations of the product interface.

```csharp
// Concrete product: Word document
public class WordDocument : IDocument
{
    public void Open()
    {
        Console.WriteLine("Opening Word document.");
    }
}

// Concrete product: PDF document
public class PDFDocument : IDocument
{
    public void Open()
    {
        Console.WriteLine("Opening PDF document.");
    }
}
```

### 3. **Creator (Abstract Factory Class)**

This defines the factory method which returns an object of type `IDocument`. Subclasses will override this method to provide the appropriate document type.

```csharp
// Abstract creator class
public abstract class DocumentCreator
{
    // Factory method
    public abstract IDocument CreateDocument();

    // A common method that uses the product
    public void OpenDocument()
    {
        IDocument document = CreateDocument();
        document.Open();
    }
}
```
 

### 4. **Concrete Creators**

These concrete classes override the factory method to create specific document objects (e.g., Word or PDF).

```csharp
// Concrete creator for Word documents
public class WordDocumentCreator : DocumentCreator
{
    public override IDocument CreateDocument()
    {
        return new WordDocument();
    }
}

// Concrete creator for PDF documents
public class PDFDocumentCreator : DocumentCreator
{
    public override IDocument CreateDocument()
    {
        return new PDFDocument();
    }
}
```

### 5. **Client Code**

The client code can use the `DocumentCreator` class to work with the `IDocument` products without knowing the concrete types.

 
```csharp
class Program
{
    static void Main(string[] args)
    {
        // Create a Word document
        DocumentCreator wordCreator = new WordDocumentCreator();
        Console.WriteLine("Word Document:");
        wordCreator.OpenDocument();

        // Create a PDF document
        DocumentCreator pdfCreator = new PDFDocumentCreator();
        Console.WriteLine("\nPDF Document:");
        pdfCreator.OpenDocument();
    }
}
```

### **Output**:

    Word Document:
    Opening Word document.
    
    PDF Document:
    Opening PDF document.


### **Class Diagram**

```text
                  ┌────────────────────┐
                  │    IDocument       │  <<interface>>
                  │────────────────────│
                  │ + Open(): void     │
                  └─────────▲──────────┘
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
┌────────────────────┐               ┌────────────────────┐
│   WordDocument     │               │   PDFDocument      │
│────────────────────│               │────────────────────│
│ + Open(): void     │               │ + Open(): void     │
└────────────────────┘               └────────────────────┘


                  ┌────────────────────────────┐
                  │     DocumentCreator        │  <<abstract class>>
                  │────────────────────────────│
                  │ + CreateDocument(): IDoc   │  <<abstract>>
                  │ + OpenDocument(): void     │
                  └─────────▲──────────────────┘
                            │
        ┌───────────────────┴────────────────────┐
        │                                        │
┌────────────────────────┐           ┌────────────────────────┐
│  WordDocumentCreator   │           │  PDFDocumentCreator    │
│────────────────────────│           │────────────────────────│
│ + CreateDocument(): ID │           │ + CreateDocument(): ID │
└────────────────────────┘           └────────────────────────┘
```

### Explanation:

1.  **Product (`IDocument`)**: Defines the interface for documents.
2.  **Concrete Products (`WordDocument` and `PDFDocument`)**: Implement the `IDocument` interface.
3.  **Creator (`DocumentCreator`)**: Defines the abstract factory method `CreateDocument()` which returns an `IDocument` object.
4.  **Concrete Creators (`WordDocumentCreator` and `PDFDocumentCreator`)**: Implement the `CreateDocument()` method to instantiate specific document types.
5.  **Client**: Uses the `DocumentCreator` to interact with documents without needing to know their concrete classes.

In this example, the **Factory Method** pattern allows you to instantiate different types of documents (Word and PDF) while ensuring that the client code only deals with the abstraction (`IDocument`). This makes the system flexible and extensible without modifying the client code.
