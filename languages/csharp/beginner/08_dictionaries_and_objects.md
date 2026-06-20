# 08 - Dictionaries and Objects

A dictionary stores key-value pairs.

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>();
ages["Ali"] = 25;
ages["Mona"] = 30;
```

Read safely:

```csharp
if (ages.TryGetValue("Ali", out int age))
{
    Console.WriteLine(age);
}
```

Practice: create a dictionary that maps product names to prices.
