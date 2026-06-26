# 01 - Setup and Install

## Learning Goal

By the end of this lesson, you will be able to set up C# on your computer and run your first console program.

You will learn that:

- C# programs run on .NET.
- The .NET SDK is different from the .NET runtime.
- Beginners should install and verify the .NET SDK.
- The `dotnet` command is the main terminal tool for .NET projects.
- `dotnet new console` creates a C# console app.
- `dotnet run` builds and runs the app.
- You can edit `Program.cs` and rerun the program to see your changes.

## C#, .NET, SDK, and Runtime

C# is the programming language. You write C# code in files that usually end with `.cs`.

.NET is the developer platform that runs C# programs. It includes tools, libraries, and runtimes for building console apps, web apps, desktop apps, services, and more.

The .NET runtime runs apps that are already built. If you only install a runtime, you can run some .NET apps, but you usually cannot create new projects from the terminal.

The .NET SDK, short for Software Development Kit, includes the runtime plus the developer tools. The SDK gives you the `dotnet` command, project templates, build tools, and everything else you need for this course.

For this course, install the latest supported .NET SDK shown on the official [.NET download page](https://dotnet.microsoft.com/download/dotnet). Support dates change, so use the [.NET support policy](https://dotnet.microsoft.com/platform/support/policy/dotnet-core) to confirm whether a version is still supported. Use a specific SDK only when your class, job, or project requires it.

You also need:

- A terminal, such as PowerShell, Windows Terminal, Terminal on macOS, or a Linux shell.
- A code editor, such as Visual Studio Code with the C# Dev Kit extension.
- Or Visual Studio with a .NET workload installed.

## Install the .NET SDK

On Windows:

1. Open the official .NET download page: <https://dotnet.microsoft.com/download/dotnet>
2. Choose the latest supported .NET version.
3. Download the **SDK**, not just the runtime.
4. Choose the `x64` installer unless you know you need `Arm64` or `x86`.
5. Run the installer and follow the prompts.
6. Close and reopen your terminal after installation.

Closing and reopening the terminal matters because installers often update your `PATH`. The `PATH` is how your terminal finds commands like `dotnet`.

On macOS with Apple Silicon:

1. Follow Microsoft's [Install .NET on macOS](https://learn.microsoft.com/en-us/dotnet/core/install/macos) instructions.
2. Choose the current supported **Arm64** SDK installer for Apple Silicon Macs.
3. Install the SDK, then close and reopen Terminal so `zsh` can find `dotnet`.

The important beginner rule is the same on every operating system: install the SDK, not only the runtime.

## Verify the Install

Open a new terminal and run:

```shell
dotnet --version
```

This prints the default SDK version your terminal will use, such as:

```text
10.0.100
```

The exact number can be different. That is okay as long as it is a supported SDK version.

Run:

```shell
dotnet --info
```

This prints more detail, including the installed SDK, runtimes, operating system, architecture, and install location. Use this when you need to check what .NET sees on your machine.

Run:

```shell
dotnet --list-sdks
```

This lists every .NET SDK installed on your machine. It is normal to have more than one SDK installed side by side.

If `dotnet` is not recognized:

- Close and reopen the terminal.
- Restart your computer if the command still does not appear.
- Confirm that you installed the SDK, not only the runtime.
- Check the install instructions for your operating system and rerun the installer if necessary.
- Use `dotnet --info` in a new terminal after the command is available to confirm the SDK, architecture, and install location that .NET detects.
- If you installed Visual Studio or VS Code tooling, confirm it installed or can find a .NET SDK.

## Create Your First C# Project

Choose a folder where you keep practice projects. Then run these commands:

```shell
mkdir csharp-playground
cd csharp-playground
dotnet new console -n HelloCSharp
cd HelloCSharp
```

Here is what each command does:

- `mkdir csharp-playground` creates a folder for your C# practice.
- `cd csharp-playground` moves into that folder.
- `dotnet new console -n HelloCSharp` creates a new console app project named `HelloCSharp`.
- `cd HelloCSharp` moves into the project folder.

Now run the app:

```shell
dotnet run
```

You should see:

```text
Hello, World!
```

`dotnet run` builds the project and runs it. For beginner work, this is the command you will use most often.

## Edit `Program.cs`

Open `Program.cs` in your editor. A new console project usually starts with a small program like this:

```csharp
Console.WriteLine("Hello, World!");
```

This is a top-level statement. Older C# programs often showed a `class`, a `Main` method, and several braces before the first useful line. Modern C# lets small programs start directly with statements such as `Console.WriteLine(...)`. Behind the scenes, .NET still creates the normal program entry point for you.

Replace the code in `Program.cs` with:

```csharp
Console.WriteLine("Hello, C#!");
Console.WriteLine("My setup works.");
```

Save the file, then run:

```shell
dotnet run
```

Expected output:

```text
Hello, C#!
My setup works.
```

## Basic Project Files

A beginner console project contains a few important files and folders:

- `Program.cs` contains your C# code. This is the first file you edit.
- `HelloCSharp.csproj` is the project file. It tells .NET which SDK style to use, which target framework to build for, and which project settings apply.
- `bin/` contains built output. It is created when you build or run the project.
- `obj/` contains temporary build files. It is also created by .NET build commands.

You normally edit `Program.cs` often. You edit the `.csproj` file only when you need to change project settings or add packages. You usually do not edit `bin/` or `obj/` by hand.

## Common Setup Problems

- Installing the runtime instead of the SDK. Fix this by installing the SDK from the official .NET download page.
- Running `dotnet run` from the wrong folder. Run it from the folder that contains the `.csproj` file, such as `HelloCSharp`.
- Forgetting to reopen the terminal after installation. Open a new terminal so it can find the updated `dotnet` command.
- Creating the project but not moving into it. After `dotnet new console -n HelloCSharp`, run `cd HelloCSharp`.
- Seeing a different SDK version than expected. Run `dotnet --list-sdks` to see all installed SDKs.
- Editing `Program.cs` but seeing old output. Save the file before running again.
- Deleting or editing generated build folders. If `bin/` or `obj/` cause confusion, you can usually delete them and let `dotnet run` recreate them.

## Exercise

Create and run a new console app named `SetupCheck`.

1. Verify that the SDK is installed.
2. Create a new folder named `csharp-playground` if you do not already have one.
3. Create a console project named `SetupCheck`.
4. Run the default app and confirm it prints `Hello, World!`.
5. Edit `Program.cs` so it prints exactly these three lines:

```text
C# runs on .NET.
I installed the SDK.
The dotnet command works.
```

6. Run the app again.
7. Write down the command that created the project.
8. Write down the command that ran the project.

## Worked Answer

One complete command sequence is:

```shell
dotnet --version
dotnet --info
dotnet --list-sdks

mkdir csharp-playground
cd csharp-playground
dotnet new console -n SetupCheck
cd SetupCheck
dotnet run
```

The first run should print:

```text
Hello, World!
```

Then replace the contents of `Program.cs` with:

```csharp
Console.WriteLine("C# runs on .NET.");
Console.WriteLine("I installed the SDK.");
Console.WriteLine("The dotnet command works.");
```

Run the project again:

```shell
dotnet run
```

Expected output:

```text
C# runs on .NET.
I installed the SDK.
The dotnet command works.
```

The command that created the project was:

```shell
dotnet new console -n SetupCheck
```

The command that ran the project was:

```shell
dotnet run
```

## Sources

- Microsoft Learn: Install .NET on Windows - <https://learn.microsoft.com/en-us/dotnet/core/install/windows>
- Microsoft Learn: Install .NET on macOS - <https://learn.microsoft.com/en-us/dotnet/core/install/macos>
- .NET: Download .NET - <https://dotnet.microsoft.com/download/dotnet>
- .NET: Support policy - <https://dotnet.microsoft.com/platform/support/policy/dotnet-core>
- Microsoft Learn: .NET CLI overview - <https://learn.microsoft.com/en-us/dotnet/core/tools/>
- Microsoft Learn: `dotnet` command - <https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet>
- Microsoft Learn: `dotnet new` - <https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-new>
- Microsoft Learn: `dotnet run` - <https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-run>
- Microsoft Learn: Create a .NET console application - <https://learn.microsoft.com/en-us/dotnet/core/tutorials/create-console-app>
- Microsoft Learn: Hello World tutorial - <https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/hello-world>
