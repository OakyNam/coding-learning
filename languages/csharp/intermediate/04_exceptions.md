# 04 - Exceptions

## Learning Goal

Use C# exceptions to represent failures, handle recoverable problems with `try`/`catch`/`finally`, and throw appropriate exception types from your own methods.

By the end of this lesson, you should be able to:

- Explain that C# exceptions are objects derived from `System.Exception`.
- Describe how an unhandled exception moves up the call stack until it is caught or terminates the program.
- Use `try`, specific `catch` blocks, and `finally` for cleanup.
- Order `catch` blocks from most specific exception type to least specific exception type.
- Decide when catching an exception is useful and when it only hides a bug.
- Throw predefined exception types such as `ArgumentNullException`, `ArgumentException`, `ArgumentOutOfRangeException`, and `InvalidOperationException`.
- Rethrow with `throw;` when you need to preserve the original stack trace.
- Avoid using exceptions for normal control flow when a check or `TryParse`-style API is clearer.

## Why Exceptions Matter

Programs fail for different reasons. A caller may pass an invalid value. An object may be in the wrong state for the requested operation. A network request may fail. A file may be missing. Exceptions give C# and .NET a consistent way to report failures that interrupt normal execution.

An exception is an object. All exception types ultimately derive from `System.Exception`, which means exception objects can carry a type, a message, a stack trace, and sometimes extra details about what went wrong.

When code throws an exception, the runtime stops the current flow and looks for a matching `catch` block. If the current method does not handle the exception, the exception propagates to the method that called it, then to that method's caller, and so on up the call stack. If no matching handler is found, the program terminates with an unhandled exception.

Use exceptions for exceptional failures, not expected choices. A customer entering `abc` instead of a quantity is normal invalid input in a console app; use `int.TryParse`. A method being asked to add `0` items to an order is a bad argument; throwing `ArgumentOutOfRangeException` can be appropriate.

## Throwing An Exception

Throw an exception when a method cannot complete the operation it promised to perform. Choose an exception type that explains the problem.

```csharp
using System;

try
{
    AddItem("Notebook", 0);
}
catch (ArgumentOutOfRangeException ex)
{
    Console.WriteLine($"Could not add item: {ex.Message}");
}

void AddItem(string itemName, int quantity)
{
    if (quantity <= 0)
    {
        throw new ArgumentOutOfRangeException(nameof(quantity), "Quantity must be greater than zero.");
    }

    Console.WriteLine($"Added {quantity} {itemName}");
}
```

Expected output:

```text
Could not add item: Quantity must be greater than zero. (Parameter 'quantity')
```

The method throws because adding zero items would break the method's contract. The caller catches the specific exception and prints a friendly message.

## Common Predefined Exception Types

Use predefined exception types when they accurately describe the failure.

| Exception type | Use it when |
| --- | --- |
| `ArgumentNullException` | A required argument is `null`. |
| `ArgumentException` | An argument is the right type but has an invalid value, such as an empty name. |
| `ArgumentOutOfRangeException` | A numeric or comparable argument is outside the allowed range. |
| `InvalidOperationException` | The object is in a state where the requested operation is not valid. |

```csharp
using System;
using System.Collections.Generic;

Order order = new Order();

try
{
    order.AddItem("", 2);
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Item was not added: {ex.Message}");
}

public class Order
{
    private readonly List<string> _items = new List<string>();

    public void AddItem(string itemName, int quantity)
    {
        if (itemName is null)
        {
            throw new ArgumentNullException(nameof(itemName));
        }

        if (string.IsNullOrWhiteSpace(itemName))
        {
            throw new ArgumentException("Item name is required.", nameof(itemName));
        }

        if (quantity <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(quantity), "Quantity must be greater than zero.");
        }

        _items.Add(itemName);
    }
}
```

Expected output:

```text
Item was not added: Item name is required. (Parameter 'itemName')
```

Do not intentionally throw runtime-reserved exceptions such as `NullReferenceException` or `IndexOutOfRangeException` to describe your own validation rules. Those exception types usually mean code attempted an invalid operation at runtime. Prefer the argument and operation exceptions that tell the caller what contract was violated.

## Try, Catch, And Finally

Use a `try` block for code that might fail. Use `catch` blocks to handle specific recoverable failures. Use `finally` for cleanup that should run when control leaves the `try`, whether the operation succeeded or failed.

```csharp
using System;

bool checkoutStarted = false;

try
{
    checkoutStarted = true;
    Checkout(0);
    Console.WriteLine("Checkout complete.");
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Checkout failed: {ex.Message}");
}
finally
{
    if (checkoutStarted)
    {
        Console.WriteLine("Checkout attempt finished.");
    }
}

void Checkout(int itemCount)
{
    if (itemCount == 0)
    {
        throw new InvalidOperationException("Cannot check out an empty order.");
    }
}
```

Expected output:

```text
Checkout failed: Cannot check out an empty order.
Checkout attempt finished.
```

`finally` is often used for cleanup. For resources that implement `IDisposable`, such as many streams and database connections, a `using` statement or declaration is usually the cleaner way to guarantee disposal. Use `finally` when you have cleanup work that is not already handled by `using`.

## Catch Specific Exceptions First

When there are multiple `catch` blocks, C# checks them from top to bottom. Put more specific exception types before more general exception types.

```csharp
using System;

try
{
    ReserveSeats(-2);
}
catch (ArgumentOutOfRangeException ex)
{
    Console.WriteLine($"Seat count problem: {ex.Message}");
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Reservation argument problem: {ex.Message}");
}

void ReserveSeats(int seatCount)
{
    if (seatCount <= 0)
    {
        throw new ArgumentOutOfRangeException(nameof(seatCount), "Seat count must be greater than zero.");
    }
}
```

Expected output:

```text
Seat count problem: Seat count must be greater than zero. (Parameter 'seatCount')
```

This order matters because `ArgumentOutOfRangeException` derives from `ArgumentException`. If the `ArgumentException` catch came first, it would handle the more specific exception before the more specific block had a chance to run.

Avoid a broad `catch (Exception)` in normal application code. Catch only when you can recover, add useful context, clean up, translate the error for the caller, or rethrow. If you catch a broad exception only to log it and continue, you may hide a bug and leave the program in an unknown state.

## Rethrowing Correctly

Sometimes a method catches an exception to add context, log details, or clean up, but it cannot actually recover. In that case, rethrow the same exception with `throw;`.

```text
try
{
    ProcessOrder(order);
}
catch (Exception ex)
{
    LogFailure(ex);
    throw;
}
```

Use `throw;`, not `throw ex;`. The `throw;` statement preserves the original stack trace, which helps the next handler or debugger find where the failure began. The `throw ex;` form resets part of that stack information and makes troubleshooting harder.

Many small programs do not need to catch and rethrow. If a method cannot handle the exception meaningfully, let it propagate naturally.

## Avoid Exceptions For Normal Control Flow

If invalid input is expected during normal use, prefer a check or a `TryParse`-style API.

```csharp
using System;

Console.Write("Quantity: ");
string? input = Console.ReadLine();

if (int.TryParse(input, out int quantity) && quantity > 0)
{
    Console.WriteLine($"Quantity accepted: {quantity}");
}
else
{
    Console.WriteLine("Please enter a whole number greater than zero.");
}
```

Sample interaction:

```text
Quantity: three
Please enter a whole number greater than zero.
```

The user typing `three` is not an exceptional program failure. It is expected invalid input, so the program checks it and asks for a better value.

## Common Mistakes

- Catching `Exception` everywhere. Broad catches make real bugs harder to find unless they immediately rethrow or are at a deliberate application boundary.
- Putting a general catch before a specific catch. More-derived exception types must come first.
- Using `throw ex;` when rethrowing. Use `throw;` to preserve the original stack trace.
- Throwing `NullReferenceException`, `IndexOutOfRangeException`, or similar runtime exceptions intentionally. Use argument or operation exceptions for your own validation.
- Catching an exception and continuing when the program is not in a known valid state.
- Using exceptions for expected invalid input, such as a user typing text where a number is needed. Use `TryParse`.
- Swallowing exceptions with an empty catch block. If nothing happens, future readers cannot tell whether the failure was understood or accidentally hidden.
- Throwing from a `finally` block. A new exception from cleanup can hide the original failure.

## Practical Exercise

Build a small order total console program.

Requirements:

1. Create an `Order` class with a private list of order lines.
2. Add an `AddItem(string? itemName, decimal price, int quantity)` method.
3. Throw `ArgumentNullException` when the item name is `null`.
4. Throw `ArgumentException` when the item name is empty or whitespace.
5. Throw `ArgumentOutOfRangeException` when the price is negative or the quantity is less than or equal to zero.
6. Add a `Checkout()` method that throws `InvalidOperationException` if the order is empty.
7. In the caller, catch specific exception types and print friendly messages.
8. Read normal user input with `decimal.TryParse` or `int.TryParse` instead of throwing for bad console input.
9. Include a successful checkout path and at least one validation failure path.

Before writing code, decide which layer owns each problem:

- User typing an invalid number is normal input handling in the caller.
- A method receiving a bad argument is the method's contract problem, so it can throw an argument exception.
- Checking out an empty order is an invalid object state, so it can throw `InvalidOperationException`.

## Worked Answer

This is a complete runnable top-level program. It uses `TryParse` for normal console input and exceptions for invalid method arguments or invalid order state.

```csharp
using System;
using System.Collections.Generic;

Order order = new Order("A100");

Console.Write("Item name: ");
string? itemName = Console.ReadLine();

decimal price = ReadPrice();
int quantity = ReadQuantity();

try
{
    order.AddItem(itemName, price, quantity);
    order.PrintSummary();
    order.Checkout();
}
catch (ArgumentNullException ex)
{
    Console.WriteLine($"Missing value: {ex.ParamName} is required.");
}
catch (ArgumentOutOfRangeException ex)
{
    Console.WriteLine($"Value out of range: {ex.Message}");
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Invalid item: {ex.Message}");
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Order problem: {ex.Message}");
}

Console.WriteLine();
Console.WriteLine("Trying to check out an empty order:");

Order emptyOrder = new Order("A101");

try
{
    emptyOrder.Checkout();
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Order problem: {ex.Message}");
}

decimal ReadPrice()
{
    Console.Write("Price: ");
    string? input = Console.ReadLine();

    if (decimal.TryParse(input, out decimal price))
    {
        return price;
    }

    Console.WriteLine("That was not a valid price, so the program will use 0.00 for this demo.");
    return 0.00m;
}

int ReadQuantity()
{
    Console.Write("Quantity: ");
    string? input = Console.ReadLine();

    if (int.TryParse(input, out int quantity))
    {
        return quantity;
    }

    Console.WriteLine("That was not a valid whole number, so the program will use 1 for this demo.");
    return 1;
}

public class Order
{
    private readonly List<OrderLine> _lines = new List<OrderLine>();

    public Order(string orderNumber)
    {
        OrderNumber = orderNumber;
    }

    public string OrderNumber { get; }

    public void AddItem(string? itemName, decimal price, int quantity)
    {
        if (itemName is null)
        {
            throw new ArgumentNullException(nameof(itemName));
        }

        if (string.IsNullOrWhiteSpace(itemName))
        {
            throw new ArgumentException("Item name is required.", nameof(itemName));
        }

        if (price < 0)
        {
            throw new ArgumentOutOfRangeException(nameof(price), "Price cannot be negative.");
        }

        if (quantity <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(quantity), "Quantity must be greater than zero.");
        }

        _lines.Add(new OrderLine(itemName, price, quantity));
    }

    public decimal CalculateTotal()
    {
        decimal total = 0.00m;

        foreach (OrderLine line in _lines)
        {
            total += line.LineTotal;
        }

        return total;
    }

    public void Checkout()
    {
        if (_lines.Count == 0)
        {
            throw new InvalidOperationException("Cannot check out an empty order.");
        }

        Console.WriteLine($"Order {OrderNumber} checked out for {CalculateTotal():0.00}.");
    }

    public void PrintSummary()
    {
        Console.WriteLine($"Order {OrderNumber}");

        foreach (OrderLine line in _lines)
        {
            Console.WriteLine($"- {line.ItemName}: {line.Quantity} at {line.Price:0.00} = {line.LineTotal:0.00}");
        }

        Console.WriteLine($"Total: {CalculateTotal():0.00}");
    }
}

public record OrderLine(string ItemName, decimal Price, int Quantity)
{
    public decimal LineTotal => Price * Quantity;
}
```

Sample interaction:

```text
Item name: Notebook
Price: 4.50
Quantity: 2
Order A100
- Notebook: 2 at 4.50 = 9.00
Total: 9.00
Order A100 checked out for 9.00.

Trying to check out an empty order:
Order problem: Cannot check out an empty order.
```

Sample interaction with invalid user input:

```text
Item name: Pen
Price: free
That was not a valid price, so the program will use 0.00 for this demo.
Quantity: two
That was not a valid whole number, so the program will use 1 for this demo.
Order A100
- Pen: 1 at 0.00 = 0.00
Total: 0.00
Order A100 checked out for 0.00.

Trying to check out an empty order:
Order problem: Cannot check out an empty order.
```

Teaching notes:

- `ReadPrice` and `ReadQuantity` use `TryParse` because invalid console input is expected during normal use.
- `AddItem` throws argument exceptions because callers should not pass invalid values into the method.
- `Checkout` throws `InvalidOperationException` because an empty order is not in a valid state for checkout.
- The `catch` blocks are ordered from more specific argument exceptions to the more general `ArgumentException`.
- There is no broad `catch (Exception)` because the program only handles failures it can explain to the user.
- The worked answer does not rethrow because the caller handles the recoverable failures directly.

## Sources

- Microsoft Learn: [Exceptions and Exception Handling](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)
- Microsoft Learn: [Exception Handling](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/exception-handling)
- Microsoft Learn: [Exception-handling statements - `throw` and `try`, `catch`, `finally`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/exception-handling-statements)
- Microsoft Learn: [Creating and Throwing Exceptions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/creating-and-throwing-exceptions)
- Microsoft Learn: [Best practices for exceptions](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
- Microsoft Learn: [Handling and throwing exceptions in .NET](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/)
