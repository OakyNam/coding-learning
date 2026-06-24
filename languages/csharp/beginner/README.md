# C# Beginner

C# is a strongly typed programming language commonly used with .NET to build console apps, web apps, desktop apps, services, games, and cloud software. This beginner track focuses on console programs and the core language fundamentals you need before moving into larger .NET projects.

Use this map as a suggested learning order. Work through the lessons in sequence, run small console programs as you go, and keep notes on syntax, terminology, and errors you learn to fix.

## Who This Is For

This track is for new programmers and for programmers who are new to C#. No prior C# experience is required.

## What You Need

- The [.NET SDK](https://dotnet.microsoft.com/download), which includes the `dotnet` command-line tools.
- A terminal or command prompt.
- An editor such as [Visual Studio Code](https://code.visualstudio.com/) with [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit), or [Visual Studio](https://visualstudio.microsoft.com/).
- Microsoft's [Create a .NET console application](https://learn.microsoft.com/en-us/dotnet/core/tutorials/create-console-app?pivots=vscode) tutorial for first-project guidance.
- Microsoft's [.NET CLI overview](https://learn.microsoft.com/en-us/dotnet/core/tools/) for the commands used to create, build, run, and manage projects.

## Learning Goals

By the end of this level, you should be able to:

- Create and run a console project with `dotnet new console` and `dotnet run`.
- Explain the role of `Program.cs`, `.csproj` files, namespaces, and top-level statements.
- Use variables, common types, `var`, strings, and string interpolation.
- Read from and write to the console.
- Use conditionals, loops, arrays, lists, and dictionaries.
- Write and call methods with parameters and return values.
- Define simple classes and records.
- Explain the basic difference between value types and reference types.
- Recognize and handle basic exceptions.
- Add and use NuGet dependencies in a project.

## Lessons

1. [Setup and Install](01_setup_and_install.md) - Install the tools you need and confirm that C# and .NET commands work on your machine.
2. [Project and File Structure](02_project_and_file_structure.md) - Learn how a small C# console project is organized and what each generated file does.
3. [Variables and Data Types](03_variables_and_data_types.md) - Practice declaring values with explicit types and `var`, then use strings and interpolation.
4. [Input and Output](04_input_and_output.md) - Use `Console.WriteLine` and `Console.ReadLine` to communicate with the user.
5. [Control Flow](05_control_flow.md) - Make decisions with `if`, `else`, comparison operators, and related branching syntax.
6. [Loops](06_loops.md) - Repeat work with loop structures such as `while`, `for`, and `foreach`.
7. [Arrays Lists and Collections](07_arrays_lists_and_collections.md) - Store multiple values with arrays and lists, then practice adding, reading, and iterating over items.
8. [Dictionaries and Objects](08_dictionaries_and_objects.md) - Work with keyed data in dictionaries and begin thinking about simple object-shaped data.
9. [Methods](09_methods.md) - Organize code into reusable methods with clear inputs and outputs.
10. [Namespaces and Packages](10_namespaces_and_packages.md) - Understand namespaces, project references, and the basics of bringing in packages.

## Microsoft References

- [Introduction to C# tutorials](https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/)
- [A tour of C#](https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/)
- [Create a .NET console application](https://learn.microsoft.com/en-us/dotnet/core/tutorials/create-console-app?pivots=vscode)
- [General structure of a C# program](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/program-structure/)
- [The C# type system](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/)
- [Classes, structs, and records](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/)
- [Exceptions and exception handling](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)
- [.NET CLI overview](https://learn.microsoft.com/en-us/dotnet/core/tools/)
- [What is NuGet?](https://learn.microsoft.com/en-us/nuget/what-is-nuget)

## Topics to Add Later

If this track grows, add fuller coverage of classes and records, plus a focused introduction to basic error handling with `try`, `catch`, and common exception types.

## Completion Goal

You are ready to complete this level when you can build a small console app from scratch that asks for input, stores several values in collections, uses conditionals and loops, calls your own methods, defines at least one simple class or record, and explains how the project runs through .NET.
