# 06 - Nullable Reference Types

## Learning Goal

Use nullable reference types to express when a C# reference is expected to contain `null`, respond to nullable warnings, and write small programs that handle missing values deliberately.

By the end of this lesson, you should be able to:

- Explain that nullable reference types are compile-time analysis, not a runtime safety system.
- Enable nullable analysis with `<Nullable>enable</Nullable>` or `#nullable enable`.
- Use `string` for values that should not be `null` and `string?` for values that may be `null`.
- Read the compiler's null-state warnings as design feedback.
- Handle nullable values with `is null`, `is not null`, `??`, and `?.`.
- Recognize common warnings such as assigning maybe-null values, dereferencing possibly-null values, and leaving non-nullable properties uninitialized.
- Use the null-forgiving operator `!` only as an explicit warning suppression when you have information the compiler cannot see.

## Why Nullable Reference Types Matter

`NullReferenceException` happens when code dereferences a reference that is actually `null`. Nullable reference types help you find many of those mistakes earlier by making your intent visible in the type annotations.

In a nullable-aware context, `string` means "this reference is intended to be non-null." `string?` means "this reference may be null." The compiler then tracks the null-state of expressions and warns when your code assigns, returns, or dereferences values in ways that do not match those annotations.

Nullable reference types do not create new runtime types. They do not add automatic null checks to every dereference. They also do not remove the possibility of `NullReferenceException` by themselves. They are a compile-time feature that helps you write clearer contracts and notice risky code before the program runs.

## Enable A Nullable-Aware Context

Nullable annotations only have their full meaning in a nullable-aware context. Most production projects enable nullable analysis at the project level:

```xml
<PropertyGroup>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

You can also enable it for a single source file or region with `#nullable enable`:

```csharp
#nullable enable

string name = "Ada";
string? email = null;
```

This lesson uses `#nullable enable` in runnable examples so the examples behave the same way even if you paste them into a project that has not enabled nullable analysis in the project file.

Existing projects often adopt nullable reference types gradually. A large older codebase may produce many warnings when nullable is enabled all at once, so teams commonly migrate file by file or area by area before converging on project-level `<Nullable>enable</Nullable>`.

## Non-Nullable And Nullable References

The annotation describes the reference contract:

```csharp
#nullable enable

string customerName = "Mina";
string? emailAddress = null;

Console.WriteLine(customerName.ToUpper());

if (emailAddress is null)
{
    Console.WriteLine("No email on file.");
}
```

Expected output:

```text
MINA
No email on file.
```

`customerName` is declared as `string`, so the code is saying it should always contain a string object. `emailAddress` is declared as `string?`, so the code is saying `null` is an expected value.

The compiler uses those annotations in two directions:

- It warns when a value that may be `null` is assigned to a non-nullable variable.
- It warns when code dereferences a value that may be `null`.

These warnings are not just noise. They usually mean the code has not proved that a value exists.

## Check Null Before Dereferencing

Use an `is null` check when the `null` case needs its own behavior:

```csharp
#nullable enable

PrintEmail(null);
PrintEmail("support@example.com");

void PrintEmail(string? email)
{
    if (email is null)
    {
        Console.WriteLine("Email: not provided");
        return;
    }

    Console.WriteLine($"Email: {email.ToLower()}");
}
```

Expected output:

```text
Email: not provided
Email: support@example.com
```

After the `if (email is null)` block returns, the compiler can treat `email` as not-null in the remaining code. That is null-state analysis at work.

Use `is not null` when the non-null case is the branch you want to enter:

```csharp
#nullable enable

string? phone = "555-0100";

if (phone is not null)
{
    Console.WriteLine($"Phone digits: {phone.Length}");
}
```

Expected output:

```text
Phone digits: 8
```

Inside the `if` block, `phone` is known to be non-null, so accessing `phone.Length` is safe.

## Use Null Operators For Simple Cases

The null-coalescing operator `??` gives a fallback value when the left side is `null`:

```csharp
#nullable enable

string? nickname = null;
string displayName = nickname ?? "Guest";

Console.WriteLine(displayName);
```

Expected output:

```text
Guest
```

The null-conditional operator `?.` calls a member only when the receiver is not `null`. If the receiver is `null`, the whole expression evaluates to `null`:

```csharp
#nullable enable

string? message = null;
int? length = message?.Length;

Console.WriteLine(length ?? 0);
```

Expected output:

```text
0
```

Use these operators when they keep the intent obvious. For richer behavior, an `if` statement is often easier to read.

## Common Nullable Warnings

Nullable warnings are compiler diagnostics produced by static analysis. The exact warning code depends on the situation, but the core problems are usually familiar.

Assigning a maybe-null value to a non-nullable reference:

```text
#nullable enable

string? possibleName = GetName();
string name = possibleName; // Warning: possibleName may be null.
```

Fix it by proving the value is not `null`, choosing a fallback, or changing the receiving type to nullable:

```csharp
#nullable enable

string? possibleName = GetName();
string name = possibleName ?? "Unknown customer";

Console.WriteLine(name);

string? GetName()
{
    return null;
}
```

Expected output:

```text
Unknown customer
```

Dereferencing a possibly-null reference:

```text
#nullable enable

string? email = FindEmail();
Console.WriteLine(email.ToLower()); // Warning: email may be null.
```

Fix it by checking the value first:

```csharp
#nullable enable

string? email = FindEmail();

if (email is not null)
{
    Console.WriteLine(email.ToLower());
}
else
{
    Console.WriteLine("No email found.");
}

string? FindEmail()
{
    return null;
}
```

Expected output:

```text
No email found.
```

Leaving a non-nullable property uninitialized:

```text
#nullable enable

public class Customer
{
    public string Name { get; set; } // CS8618: not initialized when the constructor exits.
}
```

The warning commonly appears as CS8618. It means the class promises `Name` is non-null, but an object can be created without assigning it. Fix it by requiring the value in a constructor, giving the property a real default, or making the property nullable if `null` is a valid state.

```csharp
#nullable enable

public class Customer
{
    public Customer(string name)
    {
        Name = name;
    }

    public string Name { get; }
}
```

Do not make a property nullable just to silence the warning. Make it nullable only when the domain actually allows the value to be missing.

## Null-Forgiving Operator

The null-forgiving operator `!` tells the compiler to treat an expression as not-null. It has no runtime effect.

```csharp
#nullable enable

string? value = GetValueFromTrustedSource();
Console.WriteLine(value!.Length);

string? GetValueFromTrustedSource()
{
    return "ready";
}
```

Expected output:

```text
5
```

Use `!` rarely. It is useful when you know something the compiler cannot infer, such as a test intentionally passing `null` to validation code or a framework assigning a property before it is used. It is not a normal fix for nullable warnings. If `value` is actually `null` at runtime, `value!.Length` can still throw `NullReferenceException`.

Prefer one of these fixes first:

- Add an `is null` or `is not null` check.
- Use `??` to choose a fallback.
- Change the API annotation to match the real contract.
- Initialize non-nullable members in a constructor.

## Common Mistakes

- Assuming `string` can never be `null` at runtime. Nullable analysis warns about risk, but it does not add universal runtime enforcement.
- Writing `string?` everywhere to make warnings disappear. Nullable annotations should describe the domain, not silence the compiler.
- Forgetting to enable nullable analysis. Without a nullable-aware context, `string?` does not give the same warning behavior.
- Ignoring CS8618 instead of deciding how a non-nullable property gets initialized.
- Using `!` as a habit. It suppresses a warning without changing the runtime value.
- Dereferencing a nullable value before the null check. Check first, then use the value in the branch where the compiler knows it is safe.
- Treating nullable reference types as a replacement for input validation. Public methods may still need runtime checks such as `ArgumentNullException.ThrowIfNull`.

## Practical Exercise

Build a customer contact formatter.

Requirements:

1. Enable nullable analysis with `#nullable enable` or project-level `<Nullable>enable</Nullable>`.
2. Create a `Customer` type.
3. Give `Customer` a non-nullable `Name`.
4. Give `Customer` nullable `Email` and `Phone` values.
5. Write `FormatContact(Customer customer)` so it returns:
   - `"Name: email"` when `Email` is available.
   - `"Name: phone"` when `Email` is missing but `Phone` is available.
   - `"Name: no contact method"` when both are missing.
6. Include sample customers for all three cases.
7. Make the code compile with nullable enabled and no nullable warnings.

Before writing code, decide which values are required by the domain. A customer must have a name for this exercise, so `Name` should be `string`. Email and phone are optional contact methods, so they should be `string?`.

## Worked Answer

This is a complete runnable top-level program. It uses `#nullable enable` so nullable annotations and warnings are active even if the project file does not set `<Nullable>enable</Nullable>`.

```csharp
#nullable enable

using System;
using System.Collections.Generic;

List<Customer> customers = new List<Customer>
{
    new Customer("Avery Stone", "avery@example.com", "555-0101"),
    new Customer("Blair Chen", null, "555-0102"),
    new Customer("Casey Morgan", null, null)
};

foreach (Customer customer in customers)
{
    Console.WriteLine(FormatContact(customer));
}

string FormatContact(Customer customer)
{
    if (customer.Email is not null)
    {
        return $"{customer.Name}: {customer.Email}";
    }

    if (customer.Phone is not null)
    {
        return $"{customer.Name}: {customer.Phone}";
    }

    return $"{customer.Name}: no contact method";
}

public class Customer
{
    public Customer(string name, string? email, string? phone)
    {
        Name = name;
        Email = email;
        Phone = phone;
    }

    public string Name { get; }
    public string? Email { get; }
    public string? Phone { get; }
}
```

Expected output:

```text
Avery Stone: avery@example.com
Blair Chen: 555-0102
Casey Morgan: no contact method
```

Teaching notes:

- `Name` is `string` because the exercise says every customer must have a name.
- `Email` and `Phone` are `string?` because either contact method may be missing.
- The constructor initializes `Name`, so the class does not need to suppress CS8618.
- `FormatContact` checks `customer.Email is not null` before using `customer.Email` as an available contact method.
- The phone branch uses the same pattern with `customer.Phone is not null`.
- The final return handles the case where both nullable properties are `null`.
- The code does not use `!` because the checks give the compiler enough information.

## Sources

- Microsoft Learn: [Nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/nullable-reference-types)
- Microsoft Learn: [Nullable reference types language reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-reference-types)
- Microsoft Learn: [Nullable reference type warnings](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/nullable-warnings)
- Microsoft Learn: [Null-forgiving operator](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving)
- Microsoft Learn: [Nullable migration strategies](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/update-applications/nullable-migration-strategies)
