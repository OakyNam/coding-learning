# 02 - Interfaces

## Learning Goal

Use C# interfaces to model shared behavior across different types, then write code that depends on the interface instead of one specific class.

By the end of this lesson, you should be able to:

- Define an interface with method and property members.
- Implement an interface in a class, record, or struct.
- Use an interface variable or parameter for polymorphism.
- Explain why interface names usually start with `I`, such as `INotifier`.
- Choose between an interface and an abstract class for common design problems.
- Recognize when explicit interface implementation is useful.

## Why Interfaces Matter

An interface is a contract. It says what members a type must provide, but it does not say which concrete type must provide them.

That is useful when unrelated types share a capability:

- An `EmailNotifier` and an `SmsNotifier` can both send notifications.
- A `MarkdownExporter` and a `PlainTextExporter` can both export reports.
- A `FileCache`, `MemoryCache`, and `DatabaseCache` can all retrieve stored values.

Those types might not belong in the same inheritance family. They may not share fields, constructors, or base behavior. They simply promise to support the same operations. Interfaces let the rest of your program rely on that promise.

By convention, C# interface names usually begin with `I`. The prefix is not required by the compiler, but it is common in .NET code and makes contracts easy to recognize.

## Basic Interface Syntax

An interface uses the `interface` keyword. Members are declarations of what implementing types must provide.

```csharp
public interface INotifier
{
    string Channel { get; }
    void Send(string recipient, string message);
}
```

This interface says:

- Any `INotifier` has a readable `Channel` property.
- Any `INotifier` has a `Send` method that accepts a recipient and message.

Interfaces commonly contain methods and properties. They can also contain events and indexers, but those are less common in small application code and are not the focus of this lesson.

## Implementing An Interface

A class, record, or struct implements an interface by listing it after a colon and providing every required member.

```csharp
public class EmailNotifier : INotifier
{
    public string Channel => "Email";

    public void Send(string recipient, string message)
    {
        Console.WriteLine($"Email to {recipient}: {message}");
    }
}

public class SmsNotifier : INotifier
{
    public string Channel => "SMS";

    public void Send(string recipient, string message)
    {
        Console.WriteLine($"Text to {recipient}: {message}");
    }
}
```

This is implicit implementation. The members are public members of the class, and they also satisfy the interface contract.

Now any method can accept `INotifier` instead of a specific notifier class:

```csharp
void SendWelcomeMessage(INotifier notifier, string recipient)
{
    Console.WriteLine($"Using {notifier.Channel}");
    notifier.Send(recipient, "Welcome to your new account.");
}

INotifier email = new EmailNotifier();
INotifier sms = new SmsNotifier();

SendWelcomeMessage(email, "ada@example.com");
SendWelcomeMessage(sms, "555-0100");
```

Output:

```text
Using Email
Email to ada@example.com: Welcome to your new account.
Using SMS
Text to 555-0100: Welcome to your new account.
```

This is polymorphism. The method works with the interface, and the runtime object decides what actually happens.

## Interfaces Across Type Kinds

Interfaces are not limited to classes. Records and structs can implement them too.

```csharp
public interface IDescribable
{
    string Describe();
}

public record Product(string Name, decimal Price) : IDescribable
{
    public string Describe()
    {
        return $"{Name} costs {Price:C}.";
    }
}

public readonly struct TemperatureReading : IDescribable
{
    public TemperatureReading(decimal celsius)
    {
        Celsius = celsius;
    }

    public decimal Celsius { get; }

    public string Describe()
    {
        return $"{Celsius:0.0} C";
    }
}
```

The important idea is the shared behavior, not whether the implementing type is a class, record, or struct.

## Explicit Interface Implementation

Most of the time, use implicit implementation because it keeps the public API easy to see and call.

Explicit interface implementation is useful when two interfaces have members with the same signature but different meanings, or when you want a member to be available only through the interface.

```csharp
Invoice invoice = new Invoice("INV-100");

// invoice.Print(); // This does not compile.

IPrintable printable = invoice;
printable.Print();

public interface IPrintable
{
    void Print();
}

public class Invoice : IPrintable
{
    public string Number { get; }

    public Invoice(string number)
    {
        Number = number;
    }

    void IPrintable.Print()
    {
        Console.WriteLine($"Printing invoice {Number}");
    }
}
```

Output:

```text
Printing invoice INV-100
```

Because `Print` is implemented explicitly, it is callable through an `IPrintable` reference, not directly from an `Invoice` variable.

## Multiple Interfaces

A type can implement more than one interface. Use this when a type has more than one independent capability.

```csharp
public interface IValidatable
{
    bool IsValid();
}

public interface ISavable
{
    void Save();
}

public class CustomerForm : IValidatable, ISavable
{
    public string Email { get; set; } = "";

    public bool IsValid()
    {
        return Email.Contains("@");
    }

    public void Save()
    {
        Console.WriteLine($"Saved customer with email {Email}");
    }
}
```

The two contracts stay separate. A method that only needs validation can accept `IValidatable`. A method that only needs saving can accept `ISavable`.

## Interface Inheritance

An interface can inherit from another interface. Use this when one contract naturally builds on another.

```csharp
public interface IReadOnlyReport
{
    string Title { get; }
    string GetBody();
}

public interface IEditableReport : IReadOnlyReport
{
    void Rename(string newTitle);
    void ReplaceBody(string newBody);
}
```

Any type that implements `IEditableReport` must provide the members from both `IEditableReport` and `IReadOnlyReport`.

Keep interface inheritance practical. If every method needs a different combination of members, smaller separate interfaces are often clearer than a deep interface hierarchy.

## Interface Or Abstract Class?

Interfaces and abstract classes both let you write code against a general type, but they solve different design problems.

| Use an interface when | Use an abstract class when |
| --- | --- |
| You are modeling a capability or contract. | You are modeling a shared base kind of object. |
| Unrelated types should be able to implement the same behavior. | Related types share state, constructors, or common base behavior. |
| A type may need to implement multiple contracts. | A type should inherit from one base implementation. |
| You want consumers to depend on what can be done, not what something is. | You want derived classes to reuse code from the base class. |

For example, `INotifier` is a good interface because email, SMS, and push notifications are different implementations of the same capability. An `Account` abstract class might make sense if `CheckingAccount` and `SavingsAccount` share account numbers, balances, constructors, and common validation logic.

## Common Mistakes

- Trying to instantiate an interface:

```text
INotifier notifier = new INotifier(); // Interfaces describe a contract; they are not concrete objects.
```

- Forgetting to implement every interface member. If `INotifier` requires `Channel` and `Send`, the implementing type must provide both.
- Adding instance fields or constructors to an interface. Interfaces define members that implementing types must provide; store state and initialize objects in the concrete type.
- Expecting explicitly implemented members to be callable from the concrete type. Cast or assign to the interface first.
- Creating an interface for every class automatically. Interfaces help most when there are multiple implementations, tests need a replaceable dependency, or the contract is meaningful on its own.
- Making interfaces too large. A small focused contract is easier to implement and reuse than one interface that tries to describe an entire application workflow.

## Practical Exercise

Create a report export feature using an interface.

Requirements:

1. Define an `IReportExporter` interface.
2. Add a readable `Format` property.
3. Add an `Export(string title, string body)` method.
4. Implement `MarkdownExporter`.
5. Implement `PlainTextExporter`.
6. Write a method that accepts `IReportExporter`, a title, and a body, then prints which format is being used and calls `Export`.
7. Call your method once with each exporter.

Before writing code, decide what belongs in the interface and what belongs in each concrete exporter. The interface should describe the shared contract. Each exporter should decide its own output format.

## Worked Answer

This is a complete runnable top-level program.

```csharp
using System;

IReportExporter markdown = new MarkdownExporter();
IReportExporter plainText = new PlainTextExporter();

ExportReport(markdown, "Weekly Status", "Interfaces let unrelated types share a contract.");
Console.WriteLine();
ExportReport(plainText, "Weekly Status", "Interfaces let unrelated types share a contract.");

void ExportReport(IReportExporter exporter, string title, string body)
{
    Console.WriteLine($"Exporting as {exporter.Format}");
    exporter.Export(title, body);
}

public interface IReportExporter
{
    string Format { get; }
    void Export(string title, string body);
}

public class MarkdownExporter : IReportExporter
{
    public string Format => "Markdown";

    public void Export(string title, string body)
    {
        Console.WriteLine($"# {title}");
        Console.WriteLine();
        Console.WriteLine(body);
    }
}

public class PlainTextExporter : IReportExporter
{
    public string Format => "Plain text";

    public void Export(string title, string body)
    {
        Console.WriteLine(title.ToUpperInvariant());
        Console.WriteLine(new string('=', title.Length));
        Console.WriteLine(body);
    }
}
```

Expected output:

```text
Exporting as Markdown
# Weekly Status

Interfaces let unrelated types share a contract.

Exporting as Plain text
WEEKLY STATUS
=============
Interfaces let unrelated types share a contract.
```

Teaching notes:

- `ExportReport` depends on `IReportExporter`, so it works with any current or future exporter.
- `MarkdownExporter` and `PlainTextExporter` do not need a shared base class because they do not share state or reusable implementation.
- The `Format` property belongs in the interface because consumers can use it without knowing the concrete exporter type.
- Each `Export` method belongs in the concrete class because the formatting rules are different.

## Sources

- Microsoft Learn: [Interfaces - define behavior for multiple types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces)
- Microsoft Learn: [interface keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface)
- Microsoft Learn: [Explicit Interface Implementation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/explicit-interface-implementation)
