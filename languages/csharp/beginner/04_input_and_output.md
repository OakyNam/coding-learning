# 04 - Input and Output

Use `Console.WriteLine` for output and `Console.ReadLine` for input.

```csharp
Console.Write("Name: ");
string? name = Console.ReadLine();

Console.WriteLine($"Hello, {name}!");
```

Convert text input when you need numbers:

```csharp
int age = Convert.ToInt32(Console.ReadLine());
Console.WriteLine(age + 1);
```

Practice: ask for two numbers and print their sum.
