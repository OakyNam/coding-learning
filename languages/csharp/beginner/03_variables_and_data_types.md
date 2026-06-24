# 03 - Variables and Data Types

## Learning Goal

Learn how to declare variables, choose common C# data types, update values, use `var` carefully, create constants, work with strings, and represent simple missing values with nullable types such as `int?`.

By the end, you should be able to:

- Declare variables with explicit types.
- Assign and reassign values.
- Choose from common built-in types: `int`, `double`, `decimal`, `bool`, `char`, and `string`.
- Use `var` when the initializer makes the type obvious.
- Use `const` for values that do not change.
- Build strings with interpolation.
- Use nullable value types such as `int?`.

## Core Idea

A variable is named storage for a value. In C#, every variable has a type, and that type controls what kind of value the variable can hold.

C# is type-safe. If a variable is declared as an `int`, C# will not let you store text in it later.

The usual declaration pattern is:

```csharp
type name = value;
```

Example:

```csharp
int age = 30;
string name = "Amina";
bool isActive = true;
```

Read `int age = 30;` as: "Create an integer variable named `age` and store the value `30` in it."

## Assignment and Reassignment

Assignment stores a value in a variable. Reassignment replaces the old value with a new value of the same compatible type.

```csharp
int score = 10;
score = 15;
score = score + 5;

Console.WriteLine(score); // 20
```

C# checks the type each time:

```csharp
int score = 10;

// score = "ten";     // Invalid: "ten" is a string, not an int.
// score = 10.5;      // Invalid: 10.5 is not an int.
// int score = 20;    // Invalid in the same scope: score is already declared.
```

If you need a different kind of value, create a different variable with the right type:

```csharp
int ticketCount = 3;
string ticketLabel = "General Admission";
```

## Common Built-In Types

| Type | Use it for | Example | Notes |
| --- | --- | --- | --- |
| `int` | Whole numbers | `int quantity = 4;` | No decimal point. |
| `double` | General decimal numbers | `double temperature = 98.6;` | Useful for measurements and scientific-style values. |
| `decimal` | Money-like values | `decimal price = 19.99m;` | Use the `m` suffix. Prefer `decimal` for prices, totals, and money-like calculations. |
| `bool` | True/false values | `bool isOpen = true;` | Only `true` or `false`. |
| `char` | One character | `char grade = 'A';` | Uses single quotes. |
| `string` | Text | `string city = "Seattle";` | Uses double quotes. |

Examples:

```csharp
int quantity = 2;
double distanceMiles = 4.75;
decimal unitPrice = 12.50m;
bool isMember = false;
char section = 'B';
string productName = "Notebook";
```

## Strings

Use `string` for text:

```csharp
string firstName = "Maya";
string lastName = "Chen";
```

`string` is the C# keyword for `System.String`. These two declarations mean the same thing:

```csharp
string city = "Chicago";
System.String state = "Illinois";
```

Most C# code uses the shorter `string` keyword.

Strings use double quotes:

```csharp
string message = "Hello";

// char letter = "A";    // Invalid: "A" is a string.
// string letter = 'A';  // Invalid: 'A' is a char.
```

String interpolation lets you place variables inside a string with `$` and `{}`:

```csharp
string name = "Sam";
int points = 42;

Console.WriteLine($"{name} has {points} points.");
```

Output:

```text
Sam has 42 points.
```

Strings are immutable, which means a string value is not changed in place. When you build a new string, C# creates a new string value:

```csharp
string label = "Order";
label = label + " #1042";

Console.WriteLine(label); // Order #1042
```

An empty string is text with zero characters:

```csharp
string note = "";
Console.WriteLine($"Note length: {note.Length}"); // 0
```

## Using `var`

`var` asks the compiler to infer the variable's type from the initializer.

```csharp
var count = 5;          // Compiler infers int.
var name = "Jordan";    // Compiler infers string.
var price = 9.99m;      // Compiler infers decimal.
```

The variable is still strongly typed:

```csharp
var count = 5;

count = 6;
// count = "six"; // Invalid: count was inferred as int.
```

Rules for `var`:

- `var` is for local variables.
- `var` requires an initializer.
- `var` does not mean "any type."
- While learning, prefer explicit types unless the initializer makes the type obvious.

Examples:

```csharp
var today = DateTime.Today; // Obvious from the initializer.

// var total;              // Invalid: initializer required.
// var value = null;       // Invalid: compiler cannot infer a useful type from null alone.
```

## Constants with `const`

Use `const` for a value that is known at compile time and should not change.

```csharp
const decimal SalesTaxRate = 0.0825m;
const int DaysInWeek = 7;
const string StoreName = "Northwind Market";
```

Constants cannot be reassigned:

```csharp
const decimal SalesTaxRate = 0.0825m;

// SalesTaxRate = 0.09m; // Invalid: constants cannot be modified.
```

Use `const` for stable values such as fixed rates in an exercise, mathematical constants, limits, labels, or messages. Do not use `const` for information that is expected to change while the program is running.

## Value Types, Reference Types, and `null`

At a beginner level, a useful first distinction is:

- Simple value types such as `int`, `double`, `decimal`, `bool`, and `char` directly hold a value.
- `string` is a reference type, but C# gives it friendly syntax because text is so common.
- `null` means "no value" or "does not refer to an object."

With nullable reference types enabled, you can show whether a string is allowed to be missing:

```csharp
#nullable enable

string name = "Riley";
string? nickname = null;

// name = null; // Warning with nullable reference types enabled.
```

A normal `int` cannot be `null`:

```csharp
int ticketCount = 3;

// ticketCount = null; // Invalid: int is not nullable.
```

## Nullable Value Types

Use `?` after a value type when the value might be missing.

```csharp
int? couponPercent = null;
bool? surveyComplete = null;
```

`int?` means "an `int` value, or `null`."

The null-coalescing operator `??` provides a fallback when a nullable value is `null`:

```csharp
int? couponPercent = null;
int discountPercent = couponPercent ?? 0;

Console.WriteLine(discountPercent); // 0
```

You can also check `HasValue`:

```csharp
int? bonusPoints = 25;

if (bonusPoints.HasValue)
{
    Console.WriteLine($"Bonus points: {bonusPoints.Value}");
}
else
{
    Console.WriteLine("No bonus points.");
}
```

For most beginner programs, `??` is the simplest way to use a fallback value.

## Mini Program

This complete program calculates a small receipt.

```csharp
const decimal TaxRate = 0.0825m;

string productName = "Travel Mug";
int quantity = 2;
decimal unitPrice = 14.99m;
bool isMember = true;
int? couponPercent = null;

decimal subtotal = quantity * unitPrice;
int appliedCouponPercent = couponPercent ?? 0;
decimal couponDiscount = subtotal * appliedCouponPercent / 100;
decimal memberDiscount = isMember ? subtotal * 0.10m : 0m;
decimal taxableAmount = subtotal - couponDiscount - memberDiscount;
decimal tax = taxableAmount * TaxRate;
decimal total = taxableAmount + tax;

Console.WriteLine($"Product: {productName}");
Console.WriteLine($"Quantity: {quantity}");
Console.WriteLine($"Unit price: {unitPrice:C}");
Console.WriteLine($"Subtotal: {subtotal:C}");
Console.WriteLine($"Coupon discount: {couponDiscount:C}");
Console.WriteLine($"Member discount: {memberDiscount:C}");
Console.WriteLine($"Tax: {tax:C}");
Console.WriteLine($"Total: {total:C}");
```

Possible output:

```text
Product: Travel Mug
Quantity: 2
Unit price: $14.99
Subtotal: $29.98
Coupon discount: $0.00
Member discount: $3.00
Tax: $2.23
Total: $29.21
```

## Common Mistakes

- Using double quotes for a `char`: use `'A'`, not `"A"`.
- Forgetting the `m` suffix on decimal literals: use `19.99m`, not `19.99`.
- Trying to store decimal values in an `int`.
- Thinking `var` means the variable can change type later.
- Declaring `var total;` without an initializer.
- Assigning `null` to a normal value type such as `int`.
- Using `double` for money-like calculations when `decimal` is the better beginner choice.
- Forgetting that strings use double quotes.
- Redeclaring a variable with the same name in the same scope.
- Dividing integers when you meant to keep a decimal result.

## Practice

Create an event ticket receipt program.

Requirements:

1. Create a constant named `ServiceFeeRate`.
2. Create variables named `eventName`, `ticketCount`, `ticketPrice`, `isStudent`, and `studentDiscountPercent`.
3. Use `decimal` for money-like values.
4. Make `studentDiscountPercent` nullable with `int?`.
5. Calculate `subtotal`, `serviceFee`, `discount`, and `total`.
6. Use `??` so a missing student discount counts as `0`.
7. Print a receipt with string interpolation.

Starter idea:

```csharp
const decimal ServiceFeeRate = 0.05m;

string eventName = "Community Coding Night";
int ticketCount = 2;
decimal ticketPrice = 18.50m;
bool isStudent = true;
int? studentDiscountPercent = 15;

// Calculate the receipt values here.
// Print the receipt here.
```

## Worked Answer

```csharp
const decimal ServiceFeeRate = 0.05m;

string eventName = "Community Coding Night";
int ticketCount = 2;
decimal ticketPrice = 18.50m;
bool isStudent = true;
int? studentDiscountPercent = 15;

decimal subtotal = ticketCount * ticketPrice;
decimal serviceFee = subtotal * ServiceFeeRate;
int appliedStudentDiscountPercent = isStudent ? studentDiscountPercent ?? 0 : 0;
decimal discount = subtotal * appliedStudentDiscountPercent / 100;
decimal total = subtotal + serviceFee - discount;

Console.WriteLine($"Event: {eventName}");
Console.WriteLine($"Tickets: {ticketCount}");
Console.WriteLine($"Ticket price: {ticketPrice:C}");
Console.WriteLine($"Subtotal: {subtotal:C}");
Console.WriteLine($"Service fee: {serviceFee:C}");
Console.WriteLine($"Student discount: {discount:C}");
Console.WriteLine($"Total: {total:C}");
```

Possible output:

```text
Event: Community Coding Night
Tickets: 2
Ticket price: $18.50
Subtotal: $37.00
Service fee: $1.85
Student discount: $5.55
Total: $33.30
```

Try changing `studentDiscountPercent` to `null`. The program should still run, and the discount should become `$0.00`.

## Sources

- Microsoft Learn: [The C# type system](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/)
- Microsoft Learn: [Variables](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables)
- Microsoft Learn: [Declaration statements](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/declarations)
- Microsoft Learn: [Built-in types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types)
- Microsoft Learn: [Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)
- Microsoft Learn: [Floating-point numeric types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types)
- Microsoft Learn: [Implicitly typed local variables](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/implicitly-typed-local-variables)
- Microsoft Learn: [Strings](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/strings/)
- Microsoft Learn: [String interpolation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated)
- Microsoft Learn: [Built-in reference types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/reference-types)
- Microsoft Learn: [Nullable value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types)
- Microsoft Learn: [Constants](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constants)
- Microsoft Learn: [The `const` keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/const)
