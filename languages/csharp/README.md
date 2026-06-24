# C#

C# is a modern, strongly typed language for the .NET platform. It is used for console apps, web APIs, ASP.NET Core sites and services, cloud services, desktop software, games, mobile and desktop apps, developer tooling, automation, and larger production systems.

This curriculum uses Microsoft Learn as the primary reference and focuses on writing real programs with the .NET SDK and the `dotnet` command-line interface.

## What You Will Learn

By the end of this path, you should be able to:

- Install the .NET SDK and run C# programs with the `dotnet` CLI.
- Use variables, built-in types, operators, control flow, methods, and collections.
- Explain the difference between value types and reference types.
- Define and use classes, records, interfaces, and generics.
- Handle errors with exceptions.
- Query and transform data with LINQ.
- Write asynchronous code with `async` and `await`.
- Organize code with projects and solutions.
- Add packages with NuGet.
- Write and run unit tests.
- Use Microsoft Learn and the official C# language reference when you need details.

## Setup

1. Install the [.NET SDK](https://learn.microsoft.com/en-us/dotnet/core/install/).
2. Use [Visual Studio](https://visualstudio.microsoft.com/vs/), [Visual Studio Code with C# Dev Kit](https://code.visualstudio.com/docs/csharp/get-started), or another editor that supports C#.
3. Open a terminal and verify the SDK:

   ```bash
   dotnet --version
   ```

4. Create and run a console app:

   ```bash
   dotnet new console
   dotnet run
   ```

## Levels

- [Beginner](beginner/README.md): Start with setup, basic syntax, variables, types, control flow, methods, and small console programs.
- [Intermediate](intermediate/README.md): Build larger programs with collections, files, JSON, exceptions, testing, LINQ, NuGet, and project organization.
- [Advanced](advanced/README.md): Go deeper into the type system, interfaces, generics, async code, architecture, performance, and maintainable .NET applications.

## Suggested Learning Order

1. Set up the .NET SDK, editor, and terminal workflow.
2. Learn C# syntax: variables, expressions, operators, strings, numbers, booleans, and control flow.
3. Practice methods and small console programs.
4. Work with arrays, lists, dictionaries, and other collections.
5. Study the type system, especially value types, reference types, nullable values, and generics.
6. Learn object-oriented C#: classes, records, structs, interfaces, encapsulation, inheritance, and polymorphism.
7. Handle errors with exceptions and learn when to throw, catch, or let failures surface.
8. Read and write files, then serialize and deserialize JSON.
9. Add unit tests and run them with `dotnet test`.
10. Use LINQ to filter, project, group, sort, and aggregate data.
11. Learn `async` and `await` for I/O-bound work.
12. Organize larger codebases with projects, solutions, namespaces, and NuGet packages.
13. Move into advanced topics such as design patterns, performance, concurrency, web APIs, desktop apps, games, cloud services, and deployment.

## How To Use This Curriculum

Run examples locally instead of only reading them. When code fails, read the compiler error carefully, fix one issue at a time, and rerun the program. Prefer small programs while learning new syntax, then combine those ideas into a small console app with input, branching, collections, file or JSON work, and tests. Move to the next level when you can build and explain that kind of app without copying the examples line by line.

## Official References

- [C# documentation](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [A tour of C#](https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/)
- [Create a .NET console application](https://learn.microsoft.com/en-us/dotnet/core/tutorials/create-console-app)
- [.NET CLI](https://learn.microsoft.com/en-us/dotnet/core/tools/)
- [C# type system](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/)
- [Object-oriented programming in C#](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/)
- [Language Integrated Query, LINQ](https://learn.microsoft.com/en-us/dotnet/csharp/linq/)
- [Asynchronous programming with async and await](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Exceptions and exception handling](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)
- [Testing in .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- [.NET coding conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [C# language reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/)
