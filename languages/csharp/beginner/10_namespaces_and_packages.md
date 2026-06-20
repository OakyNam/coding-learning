# 10 - Namespaces and Packages

Namespaces organize C# code.

```csharp
namespace MyApp.Tools;

public class Calculator
{
    public int Add(int a, int b) => a + b;
}
```

NuGet packages add reusable libraries.

```bash
dotnet add package Humanizer
```

Practice: add a package to a test console app and inspect the `.csproj` change.
