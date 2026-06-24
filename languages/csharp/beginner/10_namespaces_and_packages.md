# 10 - Namespaces and Packages

## Learning Goal

Learn the difference between C# namespaces and NuGet packages so you can organize your own code, use code from other files, and add external libraries to a project.

By the end of this lesson, you should be able to:

- Explain that a namespace organizes code names.
- Explain that a NuGet package distributes and installs external dependencies.
- Use a type with its fully qualified name.
- Add `using` directives to shorten code.
- Understand that child namespaces are not imported automatically.
- Create a class in a file-scoped namespace.
- Recognize global and implicit `using` directives.
- Add a NuGet package to a project with the .NET CLI.
- Find a `<PackageReference>` in a `.csproj` file.
- Restore packages before building or running a project.

## Namespace vs Package

A **namespace** is a C# naming tool. It organizes types such as classes, records, structs, and interfaces so their names do not collide.

A **package** is a distribution tool. In .NET, packages usually come from NuGet. A package contains compiled code and metadata in a `.nupkg` file, and you add it to a project when you want to use an external dependency.

These ideas are related, but they are not the same:

- A namespace answers: "What is this type's full name?"
- A package answers: "Where does this external code come from, and how is it installed?"

For example, `System.Collections.Generic` is a namespace. It contains types such as `List<T>`.

`Humanizer` is a NuGet package. After you add the package to your project, it gives you extension methods and namespaces that your code can use.

Adding a `using` directive does not install a package. Installing a package does not automatically teach you every namespace inside it. You usually need both:

1. A package reference in the project when the code comes from an external dependency.
2. A `using` directive in a C# file when you want shorter type names.

## Fully Qualified Names

A fully qualified name includes the namespace and the type name.

You can write `Console` with its namespace:

```csharp
System.Console.WriteLine("Hello from the full name.");
```

You can also write `List<string>` with its namespace:

```csharp
System.Collections.Generic.List<string> names = new System.Collections.Generic.List<string>();

names.Add("Mina");
names.Add("Leo");

foreach (string name in names)
{
    System.Console.WriteLine(name);
}
```

Output:

```text
Mina
Leo
```

Fully qualified names are useful when you want to be exact. They can also make beginner code noisy. That is why C# has `using` directives.

## using Directives

A `using` directive imports names from a namespace into the current file, so you can write shorter code.

```csharp
using System;
using System.Collections.Generic;

List<string> names = new List<string>();

names.Add("Mina");
names.Add("Leo");

foreach (string name in names)
{
    Console.WriteLine(name);
}
```

This code uses:

- `Console` instead of `System.Console`.
- `List<string>` instead of `System.Collections.Generic.List<string>`.

A `using` directive does not install anything. This line:

```csharp
using Humanizer;
```

only works after the project has access to the Humanizer package. If the package is missing, the compiler cannot find the namespace.

Child namespaces are also not imported automatically. For example:

```csharp
using System.Collections;
```

does not import:

```csharp
System.Collections.Generic
```

If you want `List<T>` in an older project style, use the specific namespace:

```csharp
using System.Collections.Generic;
```

## Creating Your Own Namespace

When your app grows beyond one file, namespaces help keep related code together.

Imagine a contact book app. A `Contact` class belongs in a `Models` namespace because it describes one kind of data in the app.

Create `Models/Contact.cs`:

```csharp
namespace ContactBook.Models;

public class Contact
{
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
}
```

This is a **file-scoped namespace**. The line:

```csharp
namespace ContactBook.Models;
```

means the rest of the file belongs to the `ContactBook.Models` namespace.

You may also see block-scoped namespace syntax in older examples:

```csharp
namespace ContactBook.Models
{
    public class Contact
    {
        public string Name { get; set; } = "";
        public string Email { get; set; } = "";
    }
}
```

Both styles are valid. File-scoped namespaces are common in modern C# because they reduce indentation.

Use the namespace from `Program.cs`:

```csharp
using ContactBook.Models;

Contact contact = new Contact
{
    Name = "Mina",
    Email = "mina@example.com"
};

Console.WriteLine($"{contact.Name}: {contact.Email}");
```

Output:

```text
Mina: mina@example.com
```

Without `using ContactBook.Models;`, you can still use the fully qualified name:

```csharp
ContactBook.Models.Contact contact = new ContactBook.Models.Contact
{
    Name = "Mina",
    Email = "mina@example.com"
};
```

That is valid, but it is usually too wordy for normal app code.

## Global And Implicit Usings

C# also supports **global using directives**. A global using applies across the whole project:

```csharp
global using System.Collections.Generic;
```

Teams often put global usings in a shared file such as `GlobalUsings.cs`.

Modern SDK-style console projects can also enable **implicit usings** in the `.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```

With implicit usings enabled, the .NET SDK automatically adds common namespaces for the project type. That is why many new console apps can use `Console.WriteLine` and `List<T>` without writing `using System;` or `using System.Collections.Generic;`.

Implicit usings are helpful, but they can hide what is happening. When learning, it is still useful to know the namespace a type comes from.

## NuGet Packages

NuGet is the package manager for .NET. A NuGet package is a `.nupkg` file that can contain compiled code, supporting files, and package metadata.

Packages are added to a **project**, not to one C# file. The project file records the dependency so everyone building the app can restore the same package.

From inside a project folder, add Humanizer with the current noun-first command in .NET 10:

```bash
dotnet package add Humanizer
```

In .NET 9 or earlier, use the older verb-first command:

```bash
dotnet add package Humanizer
```

Both commands add a package reference to the project file. The project file might contain an entry like this:

```xml
<ItemGroup>
  <PackageReference Include="Humanizer" Version="2.14.1" />
</ItemGroup>
```

The exact version may be different when you run the command.

To restore packages listed in the project file, run:

```bash
dotnet restore
```

Many commands, including `dotnet build` and `dotnet run`, restore automatically when needed. Still, `dotnet restore` is useful when you want to make dependency download explicit.

## Worked Example: Contact Book

Create a new console app:

```bash
dotnet new console -n ContactBook
cd ContactBook
```

For .NET 10, add Humanizer:

```bash
dotnet package add Humanizer
```

For .NET 9 or earlier, use:

```bash
dotnet add package Humanizer
```

Create a `Models` folder, then create `Models/Contact.cs`:

```csharp
namespace ContactBook.Models;

public class Contact
{
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public DateTime LastContacted { get; set; }
}
```

Update `Program.cs`:

```csharp
using ContactBook.Models;
using Humanizer;

List<Contact> contacts = new List<Contact>
{
    new Contact
    {
        Name = "Mina",
        Email = "mina@example.com",
        LastContacted = DateTime.Now.AddDays(-1)
    },
    new Contact
    {
        Name = "Leo",
        Email = "leo@example.com",
        LastContacted = DateTime.Now.AddDays(-12)
    }
};

Console.WriteLine("Contact Book");
Console.WriteLine("------------");

foreach (Contact contact in contacts)
{
    Console.WriteLine($"{contact.Name} <{contact.Email}>");
    Console.WriteLine($"Last contacted: {contact.LastContacted.Humanize()}");
    Console.WriteLine();
}
```

Possible output:

```text
Contact Book
------------
Mina <mina@example.com>
Last contacted: yesterday

Leo <leo@example.com>
Last contacted: 12 days ago
```

Teaching notes:

- `ContactBook.Models` is your namespace. It organizes the `Contact` class name.
- `using ContactBook.Models;` lets `Program.cs` write `Contact` instead of `ContactBook.Models.Contact`.
- `Humanizer` is a NuGet package. It is installed through the project file.
- `using Humanizer;` imports Humanizer's extension methods so `Humanize()` is available.
- `Humanize()` is called on the `DateTime` value `contact.LastContacted`.

## Common Mistakes

- Thinking a namespace installs code. A namespace only organizes names.
- Thinking a package is the same as a namespace. A package is installed into a project; a namespace is used in code.
- Adding `using Humanizer;` but forgetting to add the Humanizer NuGet package.
- Adding a NuGet package but forgetting the `using` directive needed for the example code.
- Adding a package command while inside the wrong folder. Run package commands from the folder that contains the `.csproj` file, or pass the project path.
- Expecting `using System.Collections;` to import `System.Collections.Generic`. Child namespaces are not imported automatically.
- Forgetting that packages are project-level dependencies, not file-level dependencies.
- Looking for `<PackageReference>` in `Program.cs`. It belongs in the `.csproj` file.
- Forgetting to restore packages after cloning or changing dependencies.
- Mixing up .NET CLI command forms: use `dotnet package add Humanizer` for .NET 10, and `dotnet add package Humanizer` for .NET 9 or earlier.

## Exercise

Create a small contact book app.

Requirements:

1. Create a console project named `ContactBook`.
2. Create a `Models` folder.
3. Create a `Contact` class in `Models/Contact.cs`.
4. Put the class in the `ContactBook.Models` namespace.
5. Give `Contact` a `Name`, `Email`, and `LastContacted` property.
6. In `Program.cs`, use the `ContactBook.Models` namespace.
7. Create a `List<Contact>` with at least three contacts.
8. Print each contact's name and email.
9. Add the Humanizer NuGet package.
10. Use `Humanize()` to print the last contacted date in friendly text.
11. Inspect the `.csproj` file and find the `<PackageReference>` for Humanizer.
12. Write one sentence explaining what the namespace did.
13. Write one sentence explaining what the package did.

## Worked Answer

Commands:

```bash
dotnet new console -n ContactBook
cd ContactBook
mkdir Models
dotnet package add Humanizer
```

If you are using .NET 9 or earlier, use this package command instead:

```bash
dotnet add package Humanizer
```

`Models/Contact.cs`:

```csharp
namespace ContactBook.Models;

public class Contact
{
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public DateTime LastContacted { get; set; }
}
```

`Program.cs`:

```csharp
using ContactBook.Models;
using Humanizer;

List<Contact> contacts = new List<Contact>
{
    new Contact
    {
        Name = "Mina",
        Email = "mina@example.com",
        LastContacted = DateTime.Now.AddDays(-1)
    },
    new Contact
    {
        Name = "Leo",
        Email = "leo@example.com",
        LastContacted = DateTime.Now.AddDays(-3)
    },
    new Contact
    {
        Name = "Ava",
        Email = "ava@example.com",
        LastContacted = DateTime.Now.AddMonths(-1)
    }
};

Console.WriteLine("Contact Book");
Console.WriteLine("------------");

foreach (Contact contact in contacts)
{
    Console.WriteLine($"{contact.Name} <{contact.Email}>");
    Console.WriteLine($"Last contacted: {contact.LastContacted.Humanize()}");
    Console.WriteLine();
}
```

Run the app:

```bash
dotnet run
```

Possible output:

```text
Contact Book
------------
Mina <mina@example.com>
Last contacted: yesterday

Leo <leo@example.com>
Last contacted: 3 days ago

Ava <ava@example.com>
Last contacted: one month ago
```

The `.csproj` file should include a package reference similar to this:

```xml
<ItemGroup>
  <PackageReference Include="Humanizer" Version="2.14.1" />
</ItemGroup>
```

The version number can be different.

Concept answers:

- The namespace `ContactBook.Models` organized the `Contact` class and let `Program.cs` import that name with `using ContactBook.Models;`.
- The Humanizer package added external code to the project so the program could call `Humanize()`.

Teaching notes:

- If `Contact` cannot be found, check the namespace in `Models/Contact.cs` and the `using ContactBook.Models;` line in `Program.cs`.
- If `Humanize()` cannot be found, check both pieces: the Humanizer `<PackageReference>` in the `.csproj` file and `using Humanizer;` in `Program.cs`.
- If restore fails, run `dotnet restore` from the project folder and read the first package-related error.

## Sources

- Microsoft Learn: [namespace keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/namespace)
- Microsoft Learn: [Namespaces - C# language specification](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/namespaces)
- Microsoft Learn: [using directive](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-directive)
- Microsoft Learn: [.NET project SDK overview](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/overview)
- Microsoft Learn: [MSBuild reference for .NET SDK projects](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props)
- Microsoft Learn: [C# console app template changes in .NET 6+](https://learn.microsoft.com/en-us/dotnet/core/tutorials/top-level-templates)
- Microsoft Learn: [What is NuGet?](https://learn.microsoft.com/en-us/nuget/what-is-nuget)
- Microsoft Learn: [Install and manage NuGet packages with the dotnet CLI](https://learn.microsoft.com/en-us/nuget/consume-packages/install-use-packages-dotnet-cli)
- Microsoft Learn: [dotnet package add command](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-package-add)
- Microsoft Learn: [PackageReference in project files](https://learn.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files)
- Microsoft Learn: [dotnet restore command](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-restore)

## Next Step

Return to this level's README and review how namespaces and packages connect to larger beginner projects.
