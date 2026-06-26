# 08 - Async and Tasks

## Learning Goal

Use C# `async`, `await`, `Task`, and `Task<T>` to write asynchronous methods that wait without blocking, combine independent operations, handle exceptions, and support cooperative cancellation.

By the end of this lesson, you should be able to:

- Explain that `Task` represents an asynchronous operation and `Task<T>` represents an asynchronous operation that produces a value.
- Write `async Task` methods for work that completes without returning a value.
- Write `async Task<T>` methods for work that returns a value.
- Use `await` to suspend an async method until a task completes without blocking the current thread.
- Recognize when async is useful for I/O-bound waits.
- Use `Task.Run` only when you intentionally want to move CPU-bound work to the thread pool.
- Prefer `async Task` and `async Task<T>` over `async void`, except for event handlers.
- Follow the `Async` suffix convention for async method names.
- Start independent tasks before awaiting them and use `Task.WhenAll` to wait for all of them.
- Catch exceptions around awaited calls.
- Use `CancellationTokenSource`, `CancellationToken`, `CancelAfter`, and `OperationCanceledException` for cooperative cancellation.

## Why Async Matters

Many programs spend time waiting: reading a file, calling a service, querying a database, or waiting for a timer. Asynchronous code lets a method pause while that wait is in progress and resume when the operation completes. That pause is not the same as blocking a thread.

In C#, the common model is Task-based asynchronous programming:

- `Task` represents an operation that completes later and does not produce a result.
- `Task<T>` represents an operation that completes later and produces a `T`.
- `async` enables `await` inside a method.
- `await` waits for a task to complete, then continues the async method with the result or exception.

This lesson uses `Task.Delay` to simulate slow work. That keeps the examples deterministic and avoids depending on a live network service.

## A First Async Method

This is a complete runnable top-level program:

```csharp
using System;
using System.Threading.Tasks;

await PrintStatusAsync();

async Task PrintStatusAsync()
{
    Console.WriteLine("Checking status...");
    await Task.Delay(500);
    Console.WriteLine("Status: ready");
}
```

Possible output:

```text
Checking status...
Status: ready
```

`PrintStatusAsync` returns `Task` because callers need to know when the method completes, but the method does not return a value. The `Async` suffix tells readers that the method is asynchronous and should usually be awaited.

When execution reaches `await Task.Delay(500)`, the async method is suspended until the delay task completes. The current thread is not blocked during that wait. After the delay completes, the method resumes and prints the final line.

## Returning A Value With Task<T>

Use `Task<T>` when an async method produces a value.

```csharp
using System;
using System.Threading.Tasks;

int messageCount = await CountMessagesAsync();
Console.WriteLine($"Unread messages: {messageCount}");

async Task<int> CountMessagesAsync()
{
    await Task.Delay(500);
    return 3;
}
```

Expected output:

```text
Unread messages: 3
```

The method return type is `Task<int>`, but the `return` statement returns an `int`. The compiler wraps the result in the task that represents the async operation.

## Await Does Not Mean Start

Calling an async method starts the method and returns a task. Awaiting the task waits for its result.

```csharp
using System;
using System.Threading.Tasks;

Task<string> inventoryTask = CheckInventoryAsync();
Task<string> paymentTask = CheckPaymentAsync();

Console.WriteLine("Both checks have started.");

string inventory = await inventoryTask;
string payment = await paymentTask;

Console.WriteLine(inventory);
Console.WriteLine(payment);

async Task<string> CheckInventoryAsync()
{
    await Task.Delay(700);
    return "Inventory: available";
}

async Task<string> CheckPaymentAsync()
{
    await Task.Delay(300);
    return "Payment: approved";
}
```

Possible output:

```text
Both checks have started.
Inventory: available
Payment: approved
```

The two tasks are created before either one is awaited, so their delays can overlap. The output order is still controlled by the two `Console.WriteLine` calls after the awaits.

If the operations are independent, start them first and then wait for them. If the second operation needs the result of the first, await the first before starting the second.

## Waiting For Independent Work With Task.WhenAll

`Task.WhenAll` creates a task that completes when all supplied tasks complete. It is useful when independent operations can run during the same period of time.

```csharp
using System;
using System.Threading.Tasks;

Task<string> profileTask = LoadProfileAsync();
Task<string> settingsTask = LoadSettingsAsync();
Task<string> alertsTask = LoadAlertsAsync();

string[] results = await Task.WhenAll(profileTask, settingsTask, alertsTask);

foreach (string result in results)
{
    Console.WriteLine(result);
}

async Task<string> LoadProfileAsync()
{
    await Task.Delay(400);
    return "Profile loaded";
}

async Task<string> LoadSettingsAsync()
{
    await Task.Delay(200);
    return "Settings loaded";
}

async Task<string> LoadAlertsAsync()
{
    await Task.Delay(300);
    return "Alerts loaded";
}
```

Expected output:

```text
Profile loaded
Settings loaded
Alerts loaded
```

The results are returned in the same order as the tasks passed to `Task.WhenAll`, not in the order the simulated operations finish.

If one or more supplied tasks fault, the task returned by `Task.WhenAll` also faults. When you `await` it, the exception is observed by the awaiting code. If more than one task faults, the returned task contains the set of exceptions, even though an `await` expression rethrows an exception to the caller.

## Exceptions In Async Code

Put `try` and `catch` around the `await` that observes the task.

```csharp
using System;
using System.Threading.Tasks;

try
{
    string receipt = await SubmitOrderAsync();
    Console.WriteLine(receipt);
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Order failed: {ex.Message}");
}

async Task<string> SubmitOrderAsync()
{
    await Task.Delay(300);
    throw new InvalidOperationException("Payment method expired.");
}
```

Expected output:

```text
Order failed: Payment method expired.
```

The exception is raised from the async operation and observed when the task is awaited. This is one reason callers need a task. An `async void` method cannot be awaited by its caller, so normal async composition and exception handling are much harder.

## Cancellation Is Cooperative

Cancellation in async code is usually cooperative. The caller creates a `CancellationTokenSource`, passes its `CancellationToken` to async operations, and the operations check or pass along that token. `CancelAfter` schedules cancellation after a timeout.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

using CancellationTokenSource cts = new CancellationTokenSource();
cts.CancelAfter(500);

try
{
    string result = await LoadReportAsync(cts.Token);
    Console.WriteLine(result);
}
catch (OperationCanceledException)
{
    Console.WriteLine("The report timed out.");
}

async Task<string> LoadReportAsync(CancellationToken cancellationToken)
{
    await Task.Delay(1_000, cancellationToken);
    return "Report loaded";
}
```

Expected output:

```text
The report timed out.
```

`Task.Delay` observes the token. When the token is canceled before the delay finishes, the awaited operation throws `OperationCanceledException`. In real code, pass the token to async APIs that accept it and check it inside longer custom operations.

## Async And CPU-Bound Work

Async is most valuable when a program would otherwise block while waiting for I/O-bound work. `Task.Delay`, file APIs, database calls, and web service calls are common examples of asynchronous waiting.

Do not wrap every method in `Task.Run`. `Task.Run` schedules work on the thread pool. That can be appropriate when you intentionally need to move CPU-bound work away from the current thread, such as keeping a UI responsive during a calculation. It is not a way to make I/O faster, and it can waste thread-pool resources if used as a habit.

## Common Mistakes

- Forgetting to `await` a task, which can let a program continue before the operation finishes.
- Calling `.Result` or `.Wait()` in code that could use `await`, which blocks instead of asynchronously waiting.
- Writing `async void` for ordinary methods. Prefer `async Task` or `async Task<T>` so callers can await completion and handle exceptions. Use `async void` primarily for event handlers that require it.
- Leaving off the `Async` suffix from async method names, making call sites harder to read.
- Starting one independent operation, awaiting it immediately, and only then starting the next independent operation.
- Using `Task.Run` around I/O-bound operations instead of calling naturally asynchronous APIs.
- Catching exceptions before the awaited task is observed, or forgetting to catch exceptions around `await Task.WhenAll(...)`.
- Creating a `CancellationTokenSource` but not passing its token to the operations that need to observe cancellation.
- Catching all exceptions as cancellation. Cancellation is usually represented by `OperationCanceledException`.

## Practical Exercise

Build a service health checker.

Requirements:

1. Create a complete runnable top-level C# program.
2. Include `using System;`, `using System.Threading;`, and `using System.Threading.Tasks;`.
3. Write at least three simulated async operations.
4. Each simulated operation should return `Task<string>` or `Task<int>`.
5. Each simulated operation should call `Task.Delay` and pass along a `CancellationToken`.
6. Start all three tasks before awaiting any of them.
7. Use `Task.WhenAll` to await all three tasks.
8. Print clear output when all checks succeed.
9. Add a timeout with `CancellationTokenSource.CancelAfter`.
10. Catch `OperationCanceledException` and print a timeout message.
11. Include at least one `async Task` method and at least one `async Task<T>` method.

Use delays that make the timeout easy to observe. For example, set the timeout shorter than one of the simulated checks first, then increase it to see the successful path.

## Worked Answer

This is a complete runnable top-level program. It checks three simulated services. The timeout is intentionally short enough that the slowest check cancels.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

using CancellationTokenSource cts = new CancellationTokenSource();
cts.CancelAfter(900);

try
{
    await RunHealthCheckAsync(cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Health check timed out before every service responded.");
}

async Task RunHealthCheckAsync(CancellationToken cancellationToken)
{
    Console.WriteLine("Starting service health check...");

    Task<string> databaseTask = CheckDatabaseAsync(cancellationToken);
    Task<string> cacheTask = CheckCacheAsync(cancellationToken);
    Task<int> queueDepthTask = GetQueueDepthAsync(cancellationToken);

    await Task.WhenAll(databaseTask, cacheTask, queueDepthTask);

    Console.WriteLine(await databaseTask);
    Console.WriteLine(await cacheTask);
    Console.WriteLine($"Queue depth: {await queueDepthTask}");
    Console.WriteLine("Health check complete.");
}

async Task<string> CheckDatabaseAsync(CancellationToken cancellationToken)
{
    await Task.Delay(500, cancellationToken);
    return "Database: online";
}

async Task<string> CheckCacheAsync(CancellationToken cancellationToken)
{
    await Task.Delay(700, cancellationToken);
    return "Cache: online";
}

async Task<int> GetQueueDepthAsync(CancellationToken cancellationToken)
{
    await Task.Delay(1_200, cancellationToken);
    return 42;
}
```

Expected output with the timeout set to `900` milliseconds:

```text
Starting service health check...
Health check timed out before every service responded.
```

Possible output if you increase the timeout to `2_000` milliseconds:

```text
Starting service health check...
Database: online
Cache: online
Queue depth: 42
Health check complete.
```

Teaching notes:

- `RunHealthCheckAsync` is an `async Task` method because it performs asynchronous work but does not return a value.
- `CheckDatabaseAsync` and `CheckCacheAsync` return `Task<string>` because each check produces a status message.
- `GetQueueDepthAsync` returns `Task<int>` because it produces a number.
- The three tasks are created before `Task.WhenAll` is awaited, so the simulated waits can overlap.
- `Task.WhenAll(databaseTask, cacheTask, queueDepthTask)` completes after all supplied tasks have completed. If any task faults, the combined task is faulted; if cancellation is observed and no task faults, the combined task is canceled.
- The `CancellationTokenSource` owns the timeout. Its token is passed into each operation.
- `CancelAfter(900)` requests cancellation after 900 milliseconds. `GetQueueDepthAsync` is still waiting, so its `Task.Delay` observes the canceled token.
- The top-level `catch (OperationCanceledException)` handles the cooperative cancellation path.
- After `Task.WhenAll` completes successfully, awaiting the individual tasks reads their already-completed results.

## Sources

- Microsoft Learn: [Asynchronous programming with async and await](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- Microsoft Learn: [The Task asynchronous programming model](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/task-asynchronous-programming-model)
- Microsoft Learn: [Asynchronous programming scenarios](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-scenarios)
- Microsoft Learn: [async keyword](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async)
- Microsoft Learn: [await operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await)
- Microsoft Learn: [Async return types](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types)
- Microsoft Learn: [Task class](https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task)
- Microsoft Learn: [Task-based asynchronous programming](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-based-asynchronous-programming)
- Microsoft Learn: [Task-based Asynchronous Pattern overview](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
- Microsoft Learn: [Consuming the Task-based Asynchronous Pattern](https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/consuming-the-task-based-asynchronous-pattern)
- Microsoft Learn: [Cancel async tasks after a period of time](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/cancel-async-tasks-after-a-period-of-time)
