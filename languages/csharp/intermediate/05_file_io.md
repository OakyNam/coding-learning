# 05 - File I/O

## Learning Goal

Use `System.IO` to build paths, create directories, write text files, read text files, append text, and handle common file-system failures in C#.

By the end of this lesson, you should be able to:

- Use `Path.Combine` instead of hard-coded path separators.
- Create an output directory with `Directory.CreateDirectory` before writing files into it.
- Use simple `File` helpers such as `WriteAllText`, `ReadAllText`, `ReadAllLines`, and `AppendAllText`.
- Choose `StreamReader` or `StreamWriter` when line-by-line reading or writing is clearer.
- Use `using` declarations or `using` statements to dispose stream types.
- Catch common I/O exceptions such as `FileNotFoundException`, `DirectoryNotFoundException`, `UnauthorizedAccessException`, and `IOException`.
- Explain why `File.Exists` checks existence but does not validate whether a path is safe or usable.
- Recognize that async file I/O exists without needing it for small console programs.

## Why File I/O Matters

Many programs need data to survive after the program exits. A console app may write a daily log. A build tool may read a configuration file. A report generator may create an output folder and save a text summary.

In .NET, the `System.IO` namespace contains the common types for working with files, directories, paths, and streams. For small text files, the static `File` and `Directory` helper methods are usually enough. For larger files, streaming formats, or line-by-line processing, `StreamReader` and `StreamWriter` give you more control over how data is read or written.

This lesson focuses on text files in a console program. Binary files, compression, file watching, and deep async file I/O are useful topics, but they can wait until the basic file workflow feels natural.

## Build Paths With Path.Combine

Avoid building paths by typing `/` or `\` yourself. Different operating systems use different directory separators, and even on one operating system, hand-built paths are easy to mistype.

Use `Path.Combine` to join path parts:

```csharp
using System;
using System.IO;

string outputDirectory = "output";
string reportPath = Path.Combine(outputDirectory, "report.txt");

Console.WriteLine(reportPath);
```

Possible output on Windows:

```text
output\report.txt
```

Possible output on macOS or Linux:

```text
output/report.txt
```

The important idea is not which separator appears. The program asked .NET to combine path parts in a platform-aware way.

## Create The Directory Before Writing

Writing a file into a directory does not create the parent directory for you. If the directory is missing, writing the file can fail with `DirectoryNotFoundException`.

Call `Directory.CreateDirectory` before writing into an output folder:

```csharp
using System;
using System.IO;

string outputDirectory = "output";
Directory.CreateDirectory(outputDirectory);

string notePath = Path.Combine(outputDirectory, "note.txt");
File.WriteAllText(notePath, "File I/O starts with a valid path.");

Console.WriteLine(File.ReadAllText(notePath));
```

Expected output:

```text
File I/O starts with a valid path.
```

`Directory.CreateDirectory` is safe to call when the directory already exists. It creates the directory if needed and returns information about the directory.

## Simple File Helpers

The `File` class has static helper methods for common whole-file operations. They are excellent when the file is small enough to read or write all at once and the code does not need special streaming behavior.

```csharp
using System;
using System.IO;

string outputDirectory = "output";
Directory.CreateDirectory(outputDirectory);

string listPath = Path.Combine(outputDirectory, "tasks.txt");

File.WriteAllText(listPath, "Plan lesson\nWrite example\nReview output\n");

string fullText = File.ReadAllText(listPath);
Console.WriteLine("Full text:");
Console.WriteLine(fullText);

string[] lines = File.ReadAllLines(listPath);
Console.WriteLine("Numbered lines:");

for (int index = 0; index < lines.Length; index++)
{
    Console.WriteLine($"{index + 1}. {lines[index]}");
}
```

Expected output:

```text
Full text:
Plan lesson
Write example
Review output

Numbered lines:
1. Plan lesson
2. Write example
3. Review output
```

`File.WriteAllText` creates a new file, writes the text, and closes the file. If the file already exists, it overwrites it. `File.ReadAllText` returns one string containing the file contents. `File.ReadAllLines` returns an array where each element is one line.

Use `File.AppendAllText` when you want to add text to the end of a file:

```csharp
using System;
using System.IO;

string outputDirectory = "output";
Directory.CreateDirectory(outputDirectory);

string logPath = Path.Combine(outputDirectory, "app-log.txt");

File.WriteAllText(logPath, "Started\n");
File.AppendAllText(logPath, "Loaded settings\n");
File.AppendAllText(logPath, "Finished\n");

Console.WriteLine(File.ReadAllText(logPath));
```

Expected output:

```text
Started
Loaded settings
Finished
```

## When To Use StreamReader And StreamWriter

The simple `File` helpers read or write the whole text in one method call. That is clear and concise for small files.

Use `StreamReader` or `StreamWriter` when the program benefits from working line by line:

- Reading a large file one line at a time.
- Writing rows as they are produced.
- Keeping formatting code close to each written line.
- Using reader and writer APIs that work with other stream types too.

Stream types hold operating system resources while they are open. Dispose them when you are done. In C#, a `using` statement or `using` declaration disposes the object automatically.

```csharp
using System;
using System.IO;

string outputDirectory = "output";
Directory.CreateDirectory(outputDirectory);

string reportPath = Path.Combine(outputDirectory, "inventory-report.txt");

using (StreamWriter writer = new StreamWriter(reportPath))
{
    writer.WriteLine("Inventory Report");
    writer.WriteLine("Notebook: 12");
    writer.WriteLine("Pencil: 40");
}

using StreamReader reader = new StreamReader(reportPath);

string? line;
while ((line = reader.ReadLine()) is not null)
{
    Console.WriteLine(line);
}
```

Expected output:

```text
Inventory Report
Notebook: 12
Pencil: 40
```

The first `using` is a statement. It disposes `writer` when the block ends, which also flushes buffered text to the file. The second `using` is a declaration. It disposes `reader` when the current scope ends.

If you need to read a file immediately after writing it with a stream, make sure the writer has been disposed or flushed first. A `using` statement is a simple way to make that boundary obvious.

## Handling I/O Errors

File I/O reaches outside your program. A file can be missing, a directory can be missing, another process can be using a file, the user may not have permission, or the disk may be unavailable.

Catch exceptions when the program can recover or show a useful message:

```csharp
using System;
using System.IO;

string inputPath = Path.Combine("input", "settings.txt");

try
{
    string settings = File.ReadAllText(inputPath);
    Console.WriteLine(settings);
}
catch (FileNotFoundException)
{
    Console.WriteLine("The settings file was not found.");
}
catch (DirectoryNotFoundException)
{
    Console.WriteLine("The settings directory was not found.");
}
catch (UnauthorizedAccessException)
{
    Console.WriteLine("The program does not have permission to read the settings file.");
}
catch (IOException ex)
{
    Console.WriteLine($"The file could not be read: {ex.Message}");
}
```

Possible output:

```text
The settings directory was not found.
```

Order matters. `FileNotFoundException` and `DirectoryNotFoundException` are more specific I/O exception types, so catch them before the more general `IOException`.

Do not catch every exception just to continue. Catch only when you can give the user a clearer message, try a fallback, clean up, or let the application fail at a deliberate boundary.

## About File.Exists

`File.Exists(path)` answers one narrow question: does a file exist at that path, from the current program's point of view?

Do not use it as path validation. Microsoft Learn notes that `File.Exists` returns `false` for several cases, including `null`, invalid paths, zero-length strings, insufficient permissions, and other errors while checking. That means `false` does not tell you whether the path was valid, missing, inaccessible, or affected by another problem.

Also remember that a file system can change between checks. Another process may create, delete, or lock the file after your call to `File.Exists` but before your next operation.

Use `File.Exists` when it makes the user experience nicer:

```csharp
using System;
using System.IO;

string reportPath = Path.Combine("output", "report.txt");

if (File.Exists(reportPath))
{
    Console.WriteLine("A previous report exists.");
}
else
{
    Console.WriteLine("No previous report was found.");
}
```

Then still handle exceptions around the actual read or write operation.

## A Note About Async File I/O

.NET includes asynchronous file and stream methods such as `ReadAsync`, `WriteAsync`, and file helpers with `Async` in their names. Async I/O is useful when an application needs to stay responsive while waiting for slow file operations, especially in UI apps, servers, or larger workflows.

For the small console examples in this lesson, synchronous methods keep the main idea easier to see. Learn the same path, directory, disposal, and exception-handling habits first; they still matter when you later use async file I/O.

## Common Mistakes

- Hard-coding `C:\...` or `/home/...` paths in examples that should run on different machines. Use relative paths and `Path.Combine` while learning.
- Writing into a folder before creating it. Call `Directory.CreateDirectory` for output folders.
- Assuming `File.WriteAllText` appends. It overwrites an existing file; use `File.AppendAllText` or a `StreamWriter` opened for append when you want to add to a file.
- Forgetting to dispose `StreamReader` or `StreamWriter`. Use `using` so files are closed even when an exception happens.
- Reading a file before the writer has been disposed or flushed. End the writer's `using` block before reading.
- Catching `IOException` before `FileNotFoundException` or `DirectoryNotFoundException`. The general catch would handle the exception first.
- Treating `File.Exists` as path validation. It only checks existence and can return `false` for invalid paths, permission problems, and other errors.
- Building paths with string concatenation. `Path.Combine(outputDirectory, "daily-log.txt")` communicates the intent and handles separators.
- Swallowing I/O exceptions with an empty catch block. If a file operation failed, the program or user usually needs to know.

## Practical Exercise

Build a daily log console program.

Requirements:

1. Use a relative folder named `output`.
2. Create the output directory before writing into it.
3. Build the log file path with `Path.Combine(outputDirectory, "daily-log.txt")`.
4. Write at least three entries to `daily-log.txt`.
5. Read the entries back and print numbered lines.
6. Append one more entry.
7. Read and print the final contents.
8. Handle `IOException` and `UnauthorizedAccessException`.
9. Use `StreamWriter` or `StreamReader` with `using` in at least one place.
10. Use at least one simple `File` helper such as `WriteAllText`, `ReadAllText`, or `ReadAllLines`.

Before writing code, decide which API fits each part:

- Creating the folder is a `Directory.CreateDirectory` job.
- Building the path is a `Path.Combine` job.
- Writing or reading a small whole file can use `File` helpers.
- Appending or reading line by line is a good place to practice `StreamWriter` or `StreamReader`.
- The actual file operation still needs exception handling because the file system can fail for reasons outside your program.

## Worked Answer

This is a complete runnable top-level program. It overwrites the log at the start so the output stays deterministic, then appends one extra entry with a `StreamWriter`.

```csharp
using System;
using System.Collections.Generic;
using System.IO;

string outputDirectory = "output";
string logPath = Path.Combine(outputDirectory, "daily-log.txt");

List<string> entries = new List<string>
{
    "Opened the store",
    "Checked inventory",
    "Sent the afternoon report"
};

try
{
    Directory.CreateDirectory(outputDirectory);

    string startingText = string.Join(Environment.NewLine, entries) + Environment.NewLine;
    File.WriteAllText(logPath, startingText);

    Console.WriteLine("Initial log:");
    string[] initialLines = File.ReadAllLines(logPath);

    for (int index = 0; index < initialLines.Length; index++)
    {
        Console.WriteLine($"{index + 1}. {initialLines[index]}");
    }

    using (StreamWriter writer = new StreamWriter(logPath, append: true))
    {
        writer.WriteLine("Closed the store");
    }

    Console.WriteLine();
    Console.WriteLine("Final log:");

    using StreamReader reader = new StreamReader(logPath);

    string? line;
    while ((line = reader.ReadLine()) is not null)
    {
        Console.WriteLine(line);
    }
}
catch (UnauthorizedAccessException)
{
    Console.WriteLine("The program does not have permission to write or read the daily log.");
}
catch (IOException ex)
{
    Console.WriteLine($"The daily log could not be saved or read: {ex.Message}");
}
```

Expected output:

```text
Initial log:
1. Opened the store
2. Checked inventory
3. Sent the afternoon report

Final log:
Opened the store
Checked inventory
Sent the afternoon report
Closed the store
```

Teaching notes:

- `outputDirectory` is relative, so the folder is created under the program's current working directory.
- `Path.Combine(outputDirectory, "daily-log.txt")` avoids hard-coded operating system separators.
- `Directory.CreateDirectory(outputDirectory)` makes the output folder available before the program writes the file.
- `File.WriteAllText` is used for the initial small file and overwrites any previous log contents.
- `File.ReadAllLines` is used when the program wants an array of lines for numbering.
- `StreamWriter` is opened with `append: true`, so it adds one line instead of replacing the file.
- The writer's `using` statement disposes the writer at the end of the block. Disposal flushes buffered text and closes the file before the program reads it again.
- `StreamReader` is shown with a `using` declaration and reads the final file line by line.
- `UnauthorizedAccessException` is caught separately because permission problems deserve a clearer message. Other file-system failures are handled by `IOException`.

## Sources

- Microsoft Learn: [File and Stream I/O](https://learn.microsoft.com/en-us/dotnet/standard/io/)
- Microsoft Learn: [Common I/O Tasks](https://learn.microsoft.com/en-us/dotnet/standard/io/common-i-o-tasks)
- Microsoft Learn: [File Class](https://learn.microsoft.com/en-us/dotnet/api/system.io.file)
- Microsoft Learn: [File.ReadAllText Method](https://learn.microsoft.com/en-us/dotnet/api/system.io.file.readalltext)
- Microsoft Learn: [File.ReadAllLines Method](https://learn.microsoft.com/en-us/dotnet/api/system.io.file.readalllines)
- Microsoft Learn: [File.WriteAllText Method](https://learn.microsoft.com/en-us/dotnet/api/system.io.file.writealltext)
- Microsoft Learn: [File.AppendAllText Method](https://learn.microsoft.com/en-us/dotnet/api/system.io.file.appendalltext)
- Microsoft Learn: [StreamReader Class](https://learn.microsoft.com/en-us/dotnet/api/system.io.streamreader)
- Microsoft Learn: [StreamWriter Class](https://learn.microsoft.com/en-us/dotnet/api/system.io.streamwriter)
- Microsoft Learn: [Path Class](https://learn.microsoft.com/en-us/dotnet/api/system.io.path)
- Microsoft Learn: [Directory.CreateDirectory Method](https://learn.microsoft.com/en-us/dotnet/api/system.io.directory.createdirectory)
- Microsoft Learn: [`using` statement - ensure the correct use of disposable objects](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using)
- Microsoft Learn: [Handling I/O errors](https://learn.microsoft.com/en-us/dotnet/standard/io/handling-io-errors)
- Microsoft Learn: [Exception Handling](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/exception-handling)
- Microsoft Learn: [File.Exists Method](https://learn.microsoft.com/en-us/dotnet/api/system.io.file.exists)
