# 09 - Dependency Injection

## Learning Goal

Use dependency injection in a C# console app to remove hard-coded dependencies, register services, choose appropriate lifetimes, and make classes easier to test and maintain.

By the end of this lesson, you should be able to:

- Explain what a dependency is.
- Explain what a service is in .NET dependency injection terminology.
- Replace a hard-coded dependency with an interface dependency.
- Create an interface, an implementation class, and a consumer class.
- Use constructor injection as the default pattern for required dependencies.
- Register services in an `IServiceCollection`.
- Build a generic host and resolve an application service from its service provider.
- Choose between transient, scoped, and singleton service lifetimes.
- Avoid common dependency injection mistakes such as service locator overuse, missing registrations, and lifetime mismatches.

## Why Dependency Injection Matters

A dependency is an object that another object needs in order to do its work. For example, a report runner might need something that sends notifications. Without dependency injection, the report runner may create that notification object directly:

```csharp
public sealed class ReportRunner
{
    private readonly ConsoleNotifier _notifier = new ConsoleNotifier();

    public void Run()
    {
        _notifier.Send("Report complete.");
    }
}
```

That works for a tiny program, but the dependency is now hard-coded. If you want to send email instead of writing to the console, you must edit `ReportRunner`. If `ConsoleNotifier` later needs configuration or logging, `ReportRunner` may start collecting setup code that is not part of running reports. Unit tests are also harder because the class creates the real notifier instead of accepting a test double.

Dependency injection changes the direction of control. A class asks for what it needs, usually through its constructor, and the application supplies that dependency from a service container.

## Core Terms

In .NET dependency injection:

- A dependency is any object a class depends on.
- A service is an object registered with the dependency injection container. The word does not mean a web service.
- An abstraction is commonly an interface or base class that represents what the consumer needs.
- An implementation is the concrete class that does the work.
- A consumer is the class that receives and uses the dependency.
- The service provider is the built container that resolves registered services.

The built-in .NET container uses `IServiceCollection` for registration and `IServiceProvider` for resolution. Apps that use the generic host get a service provider from the built host.

## A Small Console Example

Create a console app and add the hosting package:

```powershell
dotnet new console -o DiReports
cd DiReports
dotnet add package Microsoft.Extensions.Hosting
```

Replace `Program.cs` with this complete program:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);

builder.Services.AddTransient<INotifier, ConsoleNotifier>();
builder.Services.AddTransient<ReportRunner>();

using IHost host = builder.Build();

ReportRunner runner = host.Services.GetRequiredService<ReportRunner>();
runner.Run();

public interface INotifier
{
    void Send(string message);
}

public sealed class ConsoleNotifier : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine($"Notification: {message}");
    }
}

public sealed class ReportRunner
{
    private readonly INotifier _notifier;

    public ReportRunner(INotifier notifier)
    {
        _notifier = notifier;
    }

    public void Run()
    {
        Console.WriteLine("Running report...");
        _notifier.Send("Report complete.");
    }
}
```

Expected output:

```text
Running report...
Notification: Report complete.
```

`ReportRunner` does not create `ConsoleNotifier`. It depends on `INotifier`, the interface that represents the behavior it needs. The container sees that `ReportRunner` requires an `INotifier`, uses the registration to create a `ConsoleNotifier`, and passes it into the `ReportRunner` constructor.

This is the main benefit: `ReportRunner` is no longer tied to one concrete notification mechanism. You can change the notifier implementation or provide a fake notifier in a unit test without changing the consumer class.

## Constructor Injection

Constructor injection means required dependencies are listed as constructor parameters:

```csharp
public sealed class ReportRunner
{
    private readonly INotifier _notifier;

    public ReportRunner(INotifier notifier)
    {
        _notifier = notifier;
    }
}
```

Use constructor injection as the default pattern for required services because it makes dependencies visible. A reader can look at the constructor and see what the class needs before it can run.

Avoid creating required dependencies inside the class:

```text
public sealed class ReportRunner
{
    private readonly ConsoleNotifier _notifier = new ConsoleNotifier();
}
```

That version hides the dependency and forces `ReportRunner` to know exactly which notifier implementation to use.

## Registering Services

Services are registered before the host is built:

```csharp
HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);

builder.Services.AddTransient<INotifier, ConsoleNotifier>();
builder.Services.AddTransient<ReportRunner>();

using IHost host = builder.Build();
```

The first registration says: when something asks for `INotifier`, create a `ConsoleNotifier`. The second registration says `ReportRunner` itself can be created by the container.

You can register a concrete type without an interface when no abstraction is useful:

```csharp
builder.Services.AddTransient<ReportRunner>();
```

You can also register several framework services by using extension methods such as `AddOptions`, `AddLogging`, or framework-specific methods. The same idea applies: registration describes what the container can create.

## Resolving Services

In a small console app, it is common to resolve one top-level application service and call it:

```csharp
ReportRunner runner = host.Services.GetRequiredService<ReportRunner>();
runner.Run();
```

Keep that kind of direct resolution near the application boundary, such as `Program.cs`. Inside ordinary classes, prefer constructor injection over repeatedly calling `GetRequiredService`. When a class reaches into the container to find its own dependencies, it is using the service locator pattern. That hides dependencies and makes the class harder to test.

ASP.NET Core also uses the same dependency injection system, but web apps usually resolve controllers, endpoints, middleware, and other framework types for you. This lesson uses a console app so the container mechanics are visible without adding web-specific concepts.

## Service Lifetimes

Every registration has a lifetime. Choose the lifetime based on how long the same service instance should be reused.

### Transient

Transient services are created each time they are requested:

```csharp
builder.Services.AddTransient<INotifier, ConsoleNotifier>();
```

Use transient for lightweight, stateless services. The notifier in this lesson is transient because it has no shared state and is cheap to create.

### Scoped

Scoped services are created once per scope:

```csharp
builder.Services.AddScoped<IReportStore, ReportStore>();
```

In ASP.NET Core request-processing apps, a scope is commonly created for each client request. In a console or worker app, there is no automatic web request scope. If you need scoped behavior, create a scope explicitly with `IServiceScopeFactory` or `host.Services.CreateScope()`.

Scoped services are useful when several operations inside the same unit of work should share one instance, but a different unit of work should get a different instance.

### Singleton

Singleton services are created once and reused:

```csharp
builder.Services.AddSingleton<IClock, SystemClock>();
```

Use singleton for services that are safe to share for the whole application lifetime. Singleton services should be thread-safe if they can be used concurrently. Do not store per-user, per-request, or frequently changing operation state in a singleton.

## Lifetime Rule Of Thumb

Shorter-lived services can depend on longer-lived services:

```text
Transient -> Scoped -> Singleton can be okay depending on the design.
```

Longer-lived services should not directly depend on shorter-lived services:

```text
Singleton -> Scoped is a lifetime mismatch.
```

A singleton that directly receives a scoped service can accidentally keep that scoped service alive too long. In development, the default service provider performs scope validation for common cases and can throw an exception when a scoped service is resolved from the root provider or injected into a singleton.

If a singleton truly needs scoped work, inject `IServiceScopeFactory`, create a scope for the operation, resolve the scoped service from that scope, and dispose the scope when the operation ends.

## Changing The Implementation

Because `ReportRunner` depends on `INotifier`, you can change the concrete notifier without changing `ReportRunner`.

```csharp
public sealed class TimestampedConsoleNotifier : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine($"[{DateTimeOffset.Now:HH:mm:ss}] Notification: {message}");
    }
}
```

Update the registration:

```csharp
builder.Services.AddTransient<INotifier, TimestampedConsoleNotifier>();
```

`ReportRunner` still receives an `INotifier`. It does not know or care which implementation was registered.

## Common Mistakes

- Forgetting to register a service. If `ReportRunner` asks for `INotifier` but no `INotifier` registration exists, resolving `ReportRunner` fails at runtime.
- Injecting concrete types unnecessarily. If a consumer only needs `Send`, depend on `INotifier` instead of `ConsoleNotifier`.
- Using the service locator pattern inside ordinary classes. Prefer constructor parameters over calling `GetService` or `GetRequiredService` throughout the codebase.
- Registering everything as singleton. Singletons live for the container lifetime and must be safe to share.
- Injecting a scoped service into a singleton. Create an explicit scope when a singleton needs scoped work.
- Manually disposing services created by the container. The container is responsible for disposing container-created services according to their lifetime.
- Building extra service providers inside registration code or feature classes. In most apps, build the host once and use the host's provider.
- Hiding slow work or failures in constructors. Constructors should capture dependencies and simple setup; perform real work in methods.

## Practical Exercise

Build a notification report console app.

Requirements:

1. Create a console project.
2. Add the `Microsoft.Extensions.Hosting` package if the project does not already reference it.
3. Define an `INotifier` interface with `void Send(string message)`.
4. Implement `ConsoleNotifier`.
5. Define a `ReportRunner` class that receives `INotifier` through its constructor.
6. Register `INotifier` with `ConsoleNotifier`.
7. Register `ReportRunner`.
8. Build a generic host with `Host.CreateApplicationBuilder(args)`.
9. Resolve `ReportRunner` from the host service provider and call `Run`.
10. Change `ConsoleNotifier` so it adds a timestamp or prefix to each message without changing `ReportRunner`.

Keep the app small. The goal is to see how the interface, implementation, consumer, registration, and host work together.

## Worked Answer

Create the project:

```powershell
dotnet new console -o NotificationReport
cd NotificationReport
dotnet add package Microsoft.Extensions.Hosting
```

Replace `Program.cs` with this complete program:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

HostApplicationBuilder builder = Host.CreateApplicationBuilder(args);

builder.Services.AddTransient<INotifier, ConsoleNotifier>();
builder.Services.AddTransient<ReportRunner>();

using IHost host = builder.Build();

ReportRunner runner = host.Services.GetRequiredService<ReportRunner>();
runner.Run();

public interface INotifier
{
    void Send(string message);
}

public sealed class ConsoleNotifier : INotifier
{
    public void Send(string message)
    {
        string timestamp = DateTimeOffset.Now.ToString("HH:mm:ss");
        Console.WriteLine($"[{timestamp}] REPORT: {message}");
    }
}

public sealed class ReportRunner
{
    private readonly INotifier _notifier;

    public ReportRunner(INotifier notifier)
    {
        _notifier = notifier;
    }

    public void Run()
    {
        Console.WriteLine("Preparing report...");
        Console.WriteLine("Calculating totals...");
        _notifier.Send("Daily report finished.");
    }
}
```

Run the app:

```powershell
dotnet run
```

Possible output:

```text
Preparing report...
Calculating totals...
[14:30:05] REPORT: Daily report finished.
```

The exact timestamp will vary.

Teaching notes:

- `INotifier` is the abstraction. `ReportRunner` depends on this interface because it only needs notification behavior.
- `ConsoleNotifier` is the implementation. It can change its formatting without forcing `ReportRunner` to change.
- `ReportRunner` is the consumer. Its constructor makes the required dependency visible.
- `AddTransient<INotifier, ConsoleNotifier>()` maps the interface to the implementation.
- `AddTransient<ReportRunner>()` allows the container to construct the top-level application service.
- `GetRequiredService<ReportRunner>()` is used once at the application boundary. After that, dependencies are supplied through constructors.
- Transient is a reasonable lifetime here because both classes are lightweight and stateless.

## Sources

- Microsoft Learn: [.NET dependency injection](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/overview)
- Microsoft Learn: [Quickstart: Dependency injection basics in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/basics)
- Microsoft Learn: [Tutorial: Use dependency injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/usage)
- Microsoft Learn: [Service registration](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-registration)
- Microsoft Learn: [Service lifetimes](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/service-lifetimes)
- Microsoft Learn: [Dependency injection guidelines](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection/guidelines)
- Microsoft Learn: [Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
