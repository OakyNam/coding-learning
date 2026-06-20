# 07 - Arrays, Lists, and Collections

Arrays have a fixed size.

```csharp
int[] scores = { 90, 85, 100 };
Console.WriteLine(scores[0]);
```

Lists can grow and shrink.

```csharp
List<string> names = new List<string>();
names.Add("Ali");
names.Add("Mona");
```

Loop through a list:

```csharp
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

Practice: create a list of cities and print each city.
