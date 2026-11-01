The Bridge Design Pattern is used to decouple an abstraction from its implementation so that the two can vary independently. Here’s a simple example in C# to illustrate this pattern:

### Step 1: Define the Implementor Interface

```csharp
public interface IMessageSender
{
    void SendMessage(string message);
}

```


### Step 2: Create Concrete Implementors

```csharp
public class EmailSender : IMessageSender
{
    public void SendMessage(string message)
    {
        Console.WriteLine("Email: " + message);
    }
}

public class SmsSender : IMessageSender
{
    public void SendMessage(string message)
    {
        Console.WriteLine("SMS: " + message);
    }
}

```



### Step 3: Define the Abstraction


```csharp
public abstract class Message
{
    protected IMessageSender messageSender;

    protected Message(IMessageSender sender)
    {
        messageSender = sender;
    }

    public abstract void Send(string message);
}

```


### Step 4: Create Refined Abstractions


```csharp
public class LongMessage : Message
{
    public LongMessage(IMessageSender sender) : base(sender) { }

    public override void Send(string message)
    {
        messageSender.SendMessage("Long Message: " + message);
    }
}

public class ShortMessage : Message
{
    public ShortMessage(IMessageSender sender) : base(sender) { }

    public override void Send(string message)
    {
        messageSender.SendMessage("Short Message: " + message);
    }
}

```


### Step 5: Use the Bridge Pattern


```csharp
class Program
{
    static void Main(string[] args)
    {
        IMessageSender emailSender = new EmailSender();
        IMessageSender smsSender = new SmsSender();

        Message longMessage = new LongMessage(emailSender);
        longMessage.Send("This is a long message.");

        Message shortMessage = new ShortMessage(smsSender);
        shortMessage.Send("Short msg.");
    }
}

```

In this example:

-   `IMessageSender`  is the implementor interface.
-   `EmailSender`  and  `SmsSender`  are concrete implementors.
-   `Message`  is the abstraction.
-   `LongMessage`  and  `ShortMessage`  are refined abstractions.

[This setup allows you to change the message sending mechanism (email or SMS) independently of the message type (long or short) without modifying the client code](https://dotnettutorials.net/lesson/bridge-design-pattern/)[1](https://dotnettutorials.net/lesson/bridge-design-pattern/)[2](https://dotnettutorials.net/lesson/bridge-design-pattern-real-time-example/).
