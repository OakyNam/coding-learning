# 05 - Control Flow

## Learning Goal

Learn how to make decisions in C# with `if`, `else if`, `else`, and `switch`, using comparison, equality, logical operators, and a light introduction to pattern matching.

By the end, you should be able to:

- Choose which block of code runs based on a condition.
- Write clear `if`, `else if`, and `else` chains.
- Use `switch` statements and simple `switch` expressions.
- Combine conditions with `&&`, `||`, and `!`.
- Recognize beginner pattern matching such as `is not null`, relational patterns, `_`, `and`, and `or`.

## Core Idea

Control flow means your program decides which block of code should run.

An `if` statement checks a condition. If the condition is `true`, the block under the `if` runs. If the condition is `false`, C# skips that block.

```csharp
int score = 82;

if (score >= 70)
{
    Console.WriteLine("You passed.");
}
```

The condition `score >= 70` is true, so this prints:

```text
You passed.
```

Use braces `{ }` around the block. C# allows single-line `if` statements without braces, but braces make beginner code safer and easier to read.

## If And Else

Use `else` as the fallback when the `if` condition is false.

```csharp
int score = 64;

if (score >= 70)
{
    Console.WriteLine("You passed.");
}
else
{
    Console.WriteLine("Try again.");
}
```

Output:

```text
Try again.
```

Only one of these two blocks runs.

## Else If Order

Use `else if` when you have several possible decisions. C# checks the conditions in order and runs the first block whose condition is true.

```csharp
int score = 86;

if (score >= 90)
{
    Console.WriteLine("Grade: A");
}
else if (score >= 80)
{
    Console.WriteLine("Grade: B");
}
else if (score >= 70)
{
    Console.WriteLine("Grade: C");
}
else
{
    Console.WriteLine("Grade: Not passing yet");
}
```

Output:

```text
Grade: B
```

The order matters. Because `86 >= 80` is true, C# runs the `Grade: B` block and does not check the later conditions.

## Operators For Conditions

Conditions usually produce a `bool`: either `true` or `false`.

Common comparison and equality operators:

| Operator | Meaning |
| --- | --- |
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal to |
| `>=` | Greater than or equal to |

Common logical operators:

| Operator | Meaning |
| --- | --- |
| `&&` | AND: both sides must be true |
| `||` | OR: at least one side must be true |
| `!` | NOT: reverses true and false |

Example:

```csharp
bool hasTicket = true;
int age = 17;

if (hasTicket && age >= 13)
{
    Console.WriteLine("You can enter the concert.");
}

if (!hasTicket || age < 13)
{
    Console.WriteLine("You need help from the ticket desk.");
}
```

`&&` and `||` short-circuit:

- For `left && right`, if `left` is false, C# does not need to check `right`.
- For `left || right`, if `left` is true, C# does not need to check `right`.

That helps avoid unnecessary work and can avoid errors when the right side depends on the left side.

```csharp
string? name = "Maya";

if (name is not null && name.Length > 0)
{
    Console.WriteLine("Name was provided.");
}
```

The `name.Length` check is safe because it only runs after `name is not null` is true.

## Switch Statements

A `switch` statement is useful when one value can match several named cases.

```csharp
string command = "start";

switch (command)
{
    case "start":
        Console.WriteLine("Starting...");
        break;

    case "stop":
        Console.WriteLine("Stopping...");
        break;

    default:
        Console.WriteLine("Unknown command.");
        break;
}
```

Output:

```text
Starting...
```

Important parts:

- `switch (command)` chooses the value to inspect.
- `case "start":` runs when `command` equals `"start"`.
- `default:` runs when no case matches.
- `break;` exits the switch after a matching case runs.

## Beginner Pattern Matching

Pattern matching lets C# ask whether a value has a certain shape or fits a pattern.

A useful beginner pattern is `is not null`:

```csharp
string? message = "Hello";

if (message is not null)
{
    Console.WriteLine(message);
}
```

Switch statements can also use relational patterns:

```csharp
int temperature = 32;

switch (temperature)
{
    case < 32:
        Console.WriteLine("Below freezing");
        break;

    case >= 32 and < 70:
        Console.WriteLine("Cool or mild");
        break;

    case >= 70:
        Console.WriteLine("Warm");
        break;
}
```

Output:

```text
Cool or mild
```

Inside patterns, use the words `and` and `or`:

```csharp
case >= 32 and < 70:
```

For ordinary Boolean conditions, use `&&` and `||`:

```csharp
if (temperature >= 32 && temperature < 70)
```

The discard pattern `_` means "anything else." It is often used in switch expressions.

```csharp
int score = 91;

string grade = score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _ => "Not passing yet"
};

Console.WriteLine(grade);
```

Output:

```text
A
```

This is a switch expression. It returns a value, so it is a good fit when each branch should produce one result.

## Common Mistakes

- Using `=` instead of `==`.

```csharp
// Wrong for comparison:
// if (score = 70)

// Correct:
if (score == 70)
{
    Console.WriteLine("Exactly 70");
}
```

- Putting broad ranges before narrow ranges.

```csharp
// Problem: 95 matches the first condition, so "A" never runs.
if (score >= 70)
{
    Console.WriteLine("Passing");
}
else if (score >= 90)
{
    Console.WriteLine("A");
}
```

- Forgetting braces and accidentally making code look grouped when it is not.

```csharp
if (hasTicket)
{
    Console.WriteLine("Ticket found.");
    Console.WriteLine("Enjoy the show.");
}
```

- Forgetting `break;` in a `switch` statement.

```csharp
switch (command)
{
    case "start":
        Console.WriteLine("Starting...");
        break;
}
```

- Using `||` when you mean `&&`.

```csharp
// Allows anyone who has a ticket OR is old enough.
if (hasTicket || age >= 18)
{
    Console.WriteLine("Entry allowed.");
}

// Requires both.
if (hasTicket && age >= 18)
{
    Console.WriteLine("Entry allowed.");
}
```

- Confusing ordinary logical operators with pattern keywords.

```csharp
// Ordinary Boolean condition:
if (age >= 13 && age <= 19)
{
    Console.WriteLine("Teen");
}

// Pattern:
string label = age switch
{
    >= 13 and <= 19 => "Teen",
    _ => "Not teen"
};
```

## Practice

Write a movie ticket program.

Use these variables:

```csharp
int age = 16;
bool hasAdult = true;
string day = "Saturday";
```

Your program should:

1. Print whether the person is a `child`, `teen`, or `adult`.
2. Print whether the person is allowed into a late show. A late show is allowed when the person is at least 17 or has an adult with them.
3. Print the ticket price category based on `day`:
   - `"Monday"`, `"Tuesday"`, `"Wednesday"`, or `"Thursday"`: `Weekday price`
   - `"Friday"`, `"Saturday"`, or `"Sunday"`: `Weekend price`
   - Anything else: `Unknown day`

Use `if`, `else if`, `else`, logical operators, and either a `switch` expression or a `switch` statement.

## Worked Answer

```csharp
int age = 16;
bool hasAdult = true;
string day = "Saturday";

if (age < 13)
{
    Console.WriteLine("child");
}
else if (age <= 19)
{
    Console.WriteLine("teen");
}
else
{
    Console.WriteLine("adult");
}

if (age >= 17 || hasAdult)
{
    Console.WriteLine("Late show allowed");
}
else
{
    Console.WriteLine("Late show not allowed");
}

string priceCategory = day switch
{
    "Monday" or "Tuesday" or "Wednesday" or "Thursday" => "Weekday price",
    "Friday" or "Saturday" or "Sunday" => "Weekend price",
    _ => "Unknown day"
};

Console.WriteLine(priceCategory);
```

Expected output:

```text
teen
Late show allowed
Weekend price
```

## Sources

- Microsoft Learn: [Selection statements - `if` and `switch`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/selection-statements)
- Microsoft Learn: [Comparison operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/comparison-operators)
- Microsoft Learn: [Equality operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/equality-operators)
- Microsoft Learn: [Boolean logical operators](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/boolean-logical-operators)
- Microsoft Learn: [Pattern matching](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns)
- Microsoft Learn: [Switch expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression)
