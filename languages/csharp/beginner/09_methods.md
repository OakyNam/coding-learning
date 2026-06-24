# 09 - Methods

## Learning Goal

Learn how to write and call C# methods so you can break a program into small, named pieces.

By the end of this lesson, you should be able to:

- Declare a method with a return type, name, parameter list, and method body.
- Identify a method signature.
- Explain parameters versus arguments.
- Use `return` in methods that produce a value.
- Use `void` for methods that do work without producing a returned value.
- Understand beginner scope rules for variables inside and outside methods.
- Name methods, parameters, and local variables using common C# conventions.
- Write static helper methods in a top-level console app.

## Why Methods Matter

A method is a named block of code. Instead of writing every step in one long program, you can move a focused task into a method and call it when you need it.

Methods help you:

- Give a clear name to a piece of logic.
- Reuse code instead of copying it.
- Test one small part of a program at a time.
- Make longer programs easier to read.

For example, a receipt program might need to calculate a subtotal, calculate a total with tax, and print the receipt. Those are three separate jobs, so they are good candidates for three methods.

## Method Shape

Here is a small method:

```csharp
static decimal CalculateSubtotal(decimal price, int quantity)
{
    return price * quantity;
}
```

This method has several important parts:

- `static` means this helper method belongs to the program itself, not to an object you create.
- `decimal` before the name is the return type. This method returns a `decimal` value.
- `CalculateSubtotal` is the method name.
- `(decimal price, int quantity)` is the parameter list.
- `{ ... }` is the method body.
- `return price * quantity;` sends a value back to the code that called the method.

Call the method like this:

```csharp
decimal subtotal = CalculateSubtotal(12.50m, 2);
Console.WriteLine(subtotal);
```

Output:

```text
25.00
```

The complete method declaration is the code that defines the method. The method call is the code that runs it.

## Method Signatures

A method signature is the method name plus its parameter list. The return type is important when you write and use a method, but it is not part of the method signature for C# overload resolution.

For this beginner lesson, read the method like this:

```text
static decimal CalculateSubtotal(decimal price, int quantity)
```

- Method name: `CalculateSubtotal`
- Parameters: `decimal price` and `int quantity`
- Return type: `decimal`

The signature is:

```text
CalculateSubtotal(decimal, int)
```

C# uses the name and parameter types to know which method you mean when you call it.

## Parameters And Arguments

Parameters are the named inputs in the method declaration:

```text
static decimal CalculateSubtotal(decimal price, int quantity)
```

Here, `price` and `quantity` are parameters.

Arguments are the actual values you pass when you call the method:

```csharp
decimal subtotal = CalculateSubtotal(12.50m, 2);
```

Here, `12.50m` and `2` are arguments.

The argument values are copied into the matching parameters by position:

- `12.50m` becomes the value of `price`.
- `2` becomes the value of `quantity`.

The argument types must match what the method expects. Because `price` is a `decimal`, the money value uses the `m` suffix: `12.50m`.

## Methods That Return Values

Use a return-value method when the method should calculate or choose a value and give it back.

```csharp
static decimal CalculateTotal(decimal subtotal, decimal taxRate)
{
    decimal tax = subtotal * taxRate;
    return subtotal + tax;
}
```

Call it by storing or using the returned value:

```csharp
decimal subtotal = CalculateSubtotal(12.50m, 2);
decimal total = CalculateTotal(subtotal, 0.0825m);

Console.WriteLine($"Total: {total:C}");
```

Output:

```text
Total: $27.06
```

A method with a return type such as `decimal`, `int`, `string`, or `bool` must return a compatible value. In the example above, `CalculateTotal` promises to return a `decimal`, so it returns `subtotal + tax`.

## Void Methods

Use `void` when a method should do work but does not need to send a value back.

```csharp
static void PrintReceipt(string itemName, decimal subtotal, decimal total)
{
    Console.WriteLine("Receipt");
    Console.WriteLine($"Item: {itemName}");
    Console.WriteLine($"Subtotal: {subtotal:C}");
    Console.WriteLine($"Total: {total:C}");
}
```

Call it as a statement:

```csharp
PrintReceipt("Notebook", 25.00m, 27.06m);
```

Output:

```text
Receipt
Item: Notebook
Subtotal: $25.00
Total: $27.06
```

A `void` method does not produce a value that you can store in a variable. This is invalid:

```text
decimal result = PrintReceipt("Notebook", 25.00m, 27.06m);
```

`PrintReceipt` prints text, but it does not return a `decimal`.

## Scope Basics

Scope means where a name can be used.

A variable declared inside a method belongs to that method:

```csharp
static decimal CalculateTotal(decimal subtotal, decimal taxRate)
{
    decimal tax = subtotal * taxRate;
    return subtotal + tax;
}
```

The variable `tax` only exists inside `CalculateTotal`. Code outside the method cannot use it:

```text
Console.WriteLine(tax); // Error: tax is not in scope here.
```

Parameters also belong to their method:

```csharp
static decimal CalculateSubtotal(decimal price, int quantity)
{
    return price * quantity;
}
```

The names `price` and `quantity` can be used inside `CalculateSubtotal`, but not outside it.

In beginner programs, pass values into methods as parameters and return values you need later. Avoid trying to make one method reach into another method's local variables.

## Methods In Top-Level Console Apps

Modern C# console projects often use top-level statements. That means your program can start directly with statements like this:

```csharp
decimal subtotal = CalculateSubtotal(12.50m, 2);
Console.WriteLine($"Subtotal: {subtotal:C}");

static decimal CalculateSubtotal(decimal price, int quantity)
{
    return price * quantity;
}
```

Top-level statements must come before type declarations such as classes. Static local/helper methods can be placed below the top-level statements, which keeps the main program flow easy to read.

Older examples often show a `Program` class and a `Main` method:

```csharp
using System;

class Program
{
    static void Main()
    {
        decimal subtotal = CalculateSubtotal(12.50m, 2);
        Console.WriteLine($"Subtotal: {subtotal:C}");
    }

    static decimal CalculateSubtotal(decimal price, int quantity)
    {
        return price * quantity;
    }
}
```

Both styles are valid. In this beginner curriculum, most examples use top-level statements because new console apps commonly start that way.

## Naming Tips

Clear names make methods easier to use.

- Use `PascalCase` for method names and local functions: `CalculateSubtotal`, `PrintReceipt`.
- Use `camelCase` for parameters and local variables: `price`, `quantity`, `taxRate`.
- Prefer verb or verb phrase method names because methods do work: `CalculateTotal`, `PrintReceipt`, `ReadQuantity`.
- Use names that say what the method returns or does.
- Avoid vague names such as `DoStuff`, `Process`, or `Method1`.

Constants are often written with `PascalCase` in C#:

```csharp
const decimal TaxRate = 0.0825m;
```

## Common Mistakes

- Forgetting the return type before the method name, such as writing `CalculateTotal(...)` instead of `static decimal CalculateTotal(...)`.
- Returning a value from a `void` method.
- Forgetting to return a value from a method that promises one, such as `static decimal CalculateTotal(...)`.
- Calling a method with the wrong number of arguments.
- Passing arguments in the wrong order, such as `CalculateSubtotal(2, 12.50m)` when the method expects `decimal price, int quantity`.
- Forgetting the `m` suffix on decimal money values, such as writing `12.50` instead of `12.50m`.
- Trying to use a local variable outside the method where it was declared.
- Declaring helper methods above top-level statements in a way that breaks the top-level program structure.
- Mixing top-level statements with an extra `Main` method and creating more than one entry point.
- Naming methods with `camelCase` when the local style expects `PascalCase`.

## Exercise

Write a receipt calculator.

Requirements:

1. Create a `const decimal TaxRate` in the top-level statements.
2. Create variables for an item name, price, and quantity.
3. Write a method named `CalculateSubtotal` that accepts `decimal price` and `int quantity`, then returns `price * quantity`.
4. Write a method named `CalculateTotal` that accepts `decimal subtotal` and `decimal taxRate`, then returns the subtotal plus tax.
5. Write a `void` method named `PrintReceipt` that accepts the item name, price, quantity, subtotal, total, and tax rate.
6. In the top-level statements, call the calculation methods and store their return values.
7. Call `PrintReceipt` to display the receipt.
8. Use currency formatting for money values.

## Worked Answer

```csharp
using System;

const decimal TaxRate = 0.0825m;

string itemName = "Notebook";
decimal price = 12.50m;
int quantity = 2;

decimal subtotal = CalculateSubtotal(price, quantity);
decimal total = CalculateTotal(subtotal, TaxRate);

PrintReceipt(itemName, price, quantity, subtotal, total, TaxRate);

static decimal CalculateSubtotal(decimal price, int quantity)
{
    return price * quantity;
}

static decimal CalculateTotal(decimal subtotal, decimal taxRate)
{
    decimal tax = subtotal * taxRate;
    return subtotal + tax;
}

static void PrintReceipt(
    string itemName,
    decimal price,
    int quantity,
    decimal subtotal,
    decimal total,
    decimal taxRate)
{
    Console.WriteLine("Receipt");
    Console.WriteLine("-------");
    Console.WriteLine($"Item: {itemName}");
    Console.WriteLine($"Price: {price:C}");
    Console.WriteLine($"Quantity: {quantity}");
    Console.WriteLine($"Tax rate: {taxRate:P2}");
    Console.WriteLine($"Subtotal: {subtotal:C}");
    Console.WriteLine($"Total: {total:C}");
}
```

Expected output:

```text
Receipt
-------
Item: Notebook
Price: $12.50
Quantity: 2
Tax rate: 8.25%
Subtotal: $25.00
Total: $27.06
```

Teaching notes:

- `TaxRate` is a constant because the program should not change it while running.
- `CalculateSubtotal` and `CalculateTotal` return values, so their results are stored in variables.
- `PrintReceipt` is `void` because printing is the final action; the method does not need to send a value back.
- The helper methods are below the top-level statements. This keeps the main program flow at the top of the file.
- The money examples use `decimal` and the `m` suffix because `decimal` is the usual beginner choice for currency calculations.

## Sources

- Microsoft Learn: [Methods in C#](https://learn.microsoft.com/en-us/dotnet/csharp/methods)
- Microsoft Learn: [Methods - C# Programming Guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods)
- Microsoft Learn: [Method parameters and modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters)
- Microsoft Learn: [Top-level statements](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/program-structure/top-level-statements)
- Microsoft Learn: [Local functions](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions)
- Microsoft Learn: [Identifier names](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names)
- Microsoft Learn: [static modifier](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/static)

## Next Step

Return to this level's README and continue with the next numbered lesson.
