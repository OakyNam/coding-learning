# 04 - Input and Output

## Learning Goal

Write a C# console program that prints output, asks the user for input, stores that input in variables, converts numeric text safely with `int.TryParse`, and handles invalid input without crashing.

## Why It Matters

Console programs talk to the user in two directions:

- **Output** is text your program prints for the user to read.
- **Input** is text the user types for your program to read.

Most beginner console programs follow this flow:

1. **Prompt** the user with a question.
2. **Read** what the user typed.
3. **Process** the value in a variable.
4. **Print** a result.

That pattern shows up in quizzes, calculators, menu programs, games, and many small tools.

## Printing Output

Use `Console.WriteLine` to print text and then move to the next line.

```csharp
Console.WriteLine("Welcome to the level planner!");
Console.WriteLine("This line appears below the first one.");
```

Use `Console.Write` to print text without moving to the next line. This is useful for prompts because the user's answer can appear on the same line as the question.

```csharp
Console.Write("What is your name? ");
Console.Write("This text continues on the same line. ");
Console.WriteLine("Now this ends the line.");
```

## Reading Text Input

Use `Console.ReadLine()` to wait until the user types text and presses Enter.

```csharp
Console.Write("What is your name? ");
string? name = Console.ReadLine();
Console.WriteLine($"Hello, {name}!");
```

Important details:

- `Console.Write("What is your name? ")` prints a prompt and leaves the cursor on the same line.
- `Console.ReadLine()` waits for the user to press Enter.
- `ReadLine` returns the text the user typed.
- The type is `string?` because `ReadLine` can return `null` in some situations, such as when input is redirected and ends.
- `Console.WriteLine($"Hello, {name}!")` prints the result on its own line.

## Variables Store Input

Input is useful after you store it in a variable.

```csharp
Console.Write("Favorite city: ");
string? city = Console.ReadLine();

Console.WriteLine($"You entered {city}.");
```

`city` stores the text returned by `Console.ReadLine()`. The program can then use that value later.

## String Interpolation

String interpolation lets you place variable values and expressions inside a string. Start the string with `$`, then put values or expressions inside `{}`.

```csharp
string name = "Maya";
int level = 4;

Console.WriteLine($"Player: {name}");
Console.WriteLine($"Current level: {level}");
Console.WriteLine($"Next level: {level + 1}");
```

Without the `$`, C# prints the braces as normal text:

```csharp
Console.WriteLine("Hello, {name}!"); // prints Hello, {name}!
Console.WriteLine($"Hello, {name}!"); // prints Hello, Maya!
```

You can also use basic formatting inside interpolation. For example, `:C` formats a number as currency using the computer's current culture settings.

```csharp
decimal price = 2.50m;
Console.WriteLine($"Snack price: {price:C}");
```

Do not worry about advanced formatting yet. For now, recognize that interpolation can insert values, expressions, and simple display formats.

## Converting Input To Numbers

`Console.ReadLine()` always gives you text, even when the user types digits.

```csharp
Console.Write("Age: ");
string? ageText = Console.ReadLine();
```

At this point, `ageText` is text. C# will not automatically treat it as an `int`. Use `int.TryParse` when the user might type something invalid.

```csharp
Console.Write("Age: ");
string? ageText = Console.ReadLine();

if (int.TryParse(ageText, out int age))
{
    Console.WriteLine($"Next year you will be {age + 1}.");
}
else
{
    Console.WriteLine("Please enter a whole number for age.");
}
```

`int.TryParse(ageText, out int age)` tries to convert `ageText` into an integer.

- If conversion succeeds, it returns `true` and stores the number in `age`.
- If conversion fails, it returns `false` and the `else` block runs.

This is safer than using `int.Parse` too early, because `int.Parse` throws an error when the text is not a valid number.

## Full Example: Level Planner

```csharp
Console.WriteLine("Level Planner");

Console.Write("Player name: ");
string? name = Console.ReadLine();

Console.Write("Current level: ");
string? levelText = Console.ReadLine();

if (int.TryParse(levelText, out int level))
{
    int nextLevel = level + 1;
    Console.WriteLine($"{name}, your next level is {nextLevel}.");
}
else
{
    Console.WriteLine("Current level must be a whole number.");
}
```

### Trace The Program

Example run with valid input:

```text
Level Planner
Player name: Mina
Current level: 7
Mina, your next level is 8.
```

What happened:

1. The program printed `Level Planner`.
2. It prompted for the player's name.
3. `Console.ReadLine()` read `Mina` as text and stored it in `name`.
4. It prompted for the current level.
5. `Console.ReadLine()` read `7` as text and stored it in `levelText`.
6. `int.TryParse(levelText, out int level)` converted `"7"` into the integer `7`.
7. The program calculated `level + 1`.
8. It printed the result with string interpolation.

Example run with invalid input:

```text
Level Planner
Player name: Mina
Current level: seven
Current level must be a whole number.
```

This time, `TryParse` returned `false`, so the program printed an error message instead of trying to calculate the next level.

## Common Mistakes

- Missing parentheses: write `Console.ReadLine()` and `Console.WriteLine("Hi")`, not `Console.ReadLine` or `Console.WriteLine "Hi"`.
- Forgetting that `ReadLine` returns text, not a number.
- Using `int.Parse` before learning how to handle invalid user input.
- Forgetting the `$` before an interpolated string.
- Ignoring the `false` result from `TryParse`.
- Testing only valid input and never checking what happens when the user types words, blanks, or symbols.

## Practice: Snack Budget

Write a program that:

1. Asks for the user's name.
2. Asks how many snacks they want to buy.
3. Asks how many whole dollars each snack costs.
4. Uses `int.TryParse` for both numbers.
5. If both numbers are valid, calculates the total cost.
6. If either number is invalid, prints a helpful error message.

Expected valid run:

```text
Snack Budget
Name: Jordan
Snack count: 3
Dollars per snack: 4
Jordan, your snack total is $12.
```

Expected invalid run:

```text
Snack Budget
Name: Jordan
Snack count: three
Dollars per snack: 4
Please enter whole numbers for snack count and dollars per snack.
```

## Worked Answer

```csharp
Console.WriteLine("Snack Budget");

Console.Write("Name: ");
string? name = Console.ReadLine();

Console.Write("Snack count: ");
string? snackCountText = Console.ReadLine();

Console.Write("Dollars per snack: ");
string? dollarsPerSnackText = Console.ReadLine();

bool snackCountIsValid = int.TryParse(snackCountText, out int snackCount);
bool dollarsPerSnackIsValid = int.TryParse(dollarsPerSnackText, out int dollarsPerSnack);

if (snackCountIsValid && dollarsPerSnackIsValid)
{
    int total = snackCount * dollarsPerSnack;
    Console.WriteLine($"{name}, your snack total is ${total}.");
}
else
{
    Console.WriteLine("Please enter whole numbers for snack count and dollars per snack.");
}
```

The program reads all input as text first. Then it uses `int.TryParse` twice: once for the snack count and once for the dollars per snack. The `if` statement checks that both conversions worked before doing multiplication. If either conversion fails, the program prints a clear error instead of crashing.

## Sources

- Microsoft Learn: [`Console.WriteLine` Method](https://learn.microsoft.com/en-us/dotnet/api/system.console.writeline)
- Microsoft Learn: [`Console.Write` Method](https://learn.microsoft.com/en-us/dotnet/api/system.console.write)
- Microsoft Learn: [`Console.ReadLine` Method](https://learn.microsoft.com/en-us/dotnet/api/system.console.readline)
- Microsoft Learn: [String interpolation - C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated)
- Microsoft Learn: [How to convert a string to a number - C#](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/types/how-to-convert-a-string-to-a-number)
- Microsoft Learn: [`Int32.TryParse` Method](https://learn.microsoft.com/en-us/dotnet/api/system.int32.tryparse)
