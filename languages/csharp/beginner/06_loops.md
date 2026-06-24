# 06 - Loops

## Learning Goal

Use C# loops to repeat work, stop at the right time, and control loop flow with `break` and `continue`.

## Why Loops Matter

Programs often need to do the same kind of work more than once. Loops help when you want to:

- Ask for input repeatedly until it is valid.
- Count through a known range of values.
- Retry an operation until it succeeds or reaches a limit.
- Run the same code for every item in a collection.

Without loops, you would copy and paste the same lines over and over. With loops, you describe the repeated pattern once.

## Core Idea

A loop repeats a block of code while a condition is `true`, or once for each item in a collection. Microsoft calls these C# `iteration statements`: `while`, `do`, `for`, and `foreach`.

Most loops need three ideas:

- A starting state.
- A condition that decides whether to keep going.
- A change that moves the loop toward stopping.

## `while`

Use `while` when you do not know ahead of time how many times the loop should run.

A `while` loop checks its condition before the body runs. If the condition is already `false`, the body runs zero times.

```csharp
int attempts = 0;
bool signedIn = false;

while (!signedIn && attempts < 3)
{
    attempts++;
    Console.WriteLine($"Attempt {attempts}");

    // Imagine this checks a password.
    if (attempts == 2)
    {
        signedIn = true;
    }
}

Console.WriteLine($"Signed in: {signedIn}");
```

Output:

```text
Attempt 1
Attempt 2
Signed in: True
```

The `attempts++` line matters. It changes the value used by the condition, so the loop can eventually stop.

## `do`

Use `do` when the body must run at least once.

A `do` loop checks its condition after the body runs. This is useful for prompts because the program must ask the question before it can check the answer.

```csharp
string name;

do
{
    Console.Write("Enter your name: ");
    name = Console.ReadLine() ?? "";
}
while (name == "");

Console.WriteLine($"Hello, {name}!");
```

This loop always asks at least once. It keeps asking while the user enters an empty string.

## `for`

Use `for` when you know the count or range before the loop starts.

A `for` loop has three parts in its header:

- Initializer: runs once before the loop starts.
- Condition: checked before each iteration.
- Iterator: runs after each iteration.

```csharp
for (int i = 1; i <= 5; i++)
{
    Console.WriteLine(i);
}
```

Output:

```text
1
2
3
4
5
```

Read the header like this:

- `int i = 1` starts the counter at `1`.
- `i <= 5` keeps looping while `i` is less than or equal to `5`.
- `i++` adds `1` after each loop body finishes.

When you use an index to read an array, the usual condition is `i < items.Length`, not `i <= items.Length`.

```csharp
string[] items = { "sword", "potion", "map" };

for (int i = 0; i < items.Length; i++)
{
    Console.WriteLine(items[i]);
}
```

Array indexes start at `0`, so the last valid index is `items.Length - 1`.

## Trace Table

Trace tables help you slow down and see how a loop changes.

Code:

```csharp
for (int i = 1; i <= 3; i++)
{
    Console.WriteLine(i);
}
```

| Step | `i` before condition | Condition `i <= 3` | Body output | Iterator after body |
| --- | ---: | --- | --- | --- |
| 1 | 1 | `true` | `1` | `i` becomes 2 |
| 2 | 2 | `true` | `2` | `i` becomes 3 |
| 3 | 3 | `true` | `3` | `i` becomes 4 |
| 4 | 4 | `false` | none | loop stops |

## `foreach`

Use `foreach` when you want to run code once for every item in a collection.

```csharp
string[] names = { "Maya", "Ibrahim", "Sofia" };

foreach (string name in names)
{
    Console.WriteLine($"Hello, {name}");
}
```

Output:

```text
Hello, Maya
Hello, Ibrahim
Hello, Sofia
```

The loop variable, `name`, is the current item. On each iteration, it refers to the next value from the array.

Do not add or remove items from the same collection while a `foreach` loop is reading it. Many collection types throw an error when they are modified during enumeration. Also, the `foreach` iteration variable itself is read-only; assign a new local variable if you need a changed value.

## `break` and `continue`

Use `break` to exit the nearest loop immediately.

Use `continue` to skip the rest of the current iteration and move to the next iteration of the nearest loop.

```csharp
int[] scores = { 10, -1, 8, 0, 7 };

foreach (int score in scores)
{
    if (score == -1)
    {
        continue;
    }

    if (score == 0)
    {
        break;
    }

    Console.WriteLine(score);
}
```

Output:

```text
10
8
```

The `-1` value is skipped because `continue` jumps to the next score. The `0` value stops the loop because `break` exits the loop. The `7` is never printed because it comes after the `0`.

## How To Choose A Loop

- Use `while` when the number of repetitions is unknown and the loop might run zero times.
- Use `do` when the body must run at least once, such as prompting for input.
- Use `for` when you have a known count, range, or array index.
- Use `foreach` when you want every item from a collection and do not need the index.
- Use `break` when the loop has found a stopping point early.
- Use `continue` when one item should be skipped but the loop should keep going.

## Common Mistakes

- Infinite `while` loop: forgetting to change a value used by the condition.
- `<=` vs `<` with indexes: `i <= items.Length` goes one past the end of an array.
- Semicolon after a loop header: `while (x < 3);` creates an empty loop body.
- Confusing `break` and `continue`: `break` exits the loop, while `continue` skips to the next iteration.
- Expecting `break` to exit every loop: it exits only the closest loop around it.
- Modifying a collection while using `foreach` on that same collection.
- Assigning to a `foreach` iteration variable, which is not allowed.

## Exercise

You are given daily step counts:

```csharp
int[] steps = { 4200, 8000, -1, 10500, 0, 7300 };
```

Write a program that:

1. Uses `foreach`.
2. Skips negative values with `continue`.
3. Stops at `0` with `break`.
4. Tracks the total steps.
5. Tracks the number of valid days.
6. Prints the integer average.

Expected output:

```text
Total steps: 22700
Valid days: 3
Average steps: 7566
```

## Worked Answer

```csharp
int[] steps = { 4200, 8000, -1, 10500, 0, 7300 };

int total = 0;
int validDays = 0;

foreach (int daySteps in steps)
{
    if (daySteps < 0)
    {
        continue;
    }

    if (daySteps == 0)
    {
        break;
    }

    total += daySteps;
    validDays++;
}

int average = total / validDays;

Console.WriteLine($"Total steps: {total}");
Console.WriteLine($"Valid days: {validDays}");
Console.WriteLine($"Average steps: {average}");
```

The loop reads each value in order. `4200` and `8000` are counted. `-1` is skipped because it is negative. `10500` is counted. `0` stops the loop, so `7300` is ignored. The total is `22700`, the valid day count is `3`, and integer division gives an average of `7566`.

## Sources

- Microsoft Learn: [Iteration statements - `for`, `foreach`, `do`, and `while`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/iteration-statements)
- Microsoft Learn: [Jump statements - `break`, `continue`, `return`, and `goto`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/jump-statements)
- Microsoft Learn: [Branches and loops - introductory C# tutorial](https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/branches-and-loops)
- Microsoft Learn: [Statements - C# programming guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/statements)
- Microsoft Learn: [C# language specification - statements](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/statements)
