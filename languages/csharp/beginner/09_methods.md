# 09 - Methods

Methods are reusable blocks of code.

```csharp
static string Greet(string name)
{
    return $"Hello, {name}!";
}

Console.WriteLine(Greet("Mona"));
```

Methods can return values:

```csharp
static int Add(int a, int b)
{
    return a + b;
}
```

Practice: write a method that returns the square of a number.
