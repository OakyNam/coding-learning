# 01 - Classes, Records, And Structs

## Learning Goal

Design small custom C# types and choose between `class`, `record class`, `struct`, and `record struct` based on the kind of data and behavior your program needs.

By the end of this lesson, you should be able to:

- Explain the difference between reference types and value types.
- Use a `class` for mutable state, behavior, identity, and long-lived objects.
- Use a `record class` for data-focused reference types with value equality.
- Use a `struct` for small value types with copy semantics.
- Use a `record struct` for small data values that benefit from generated equality and printing.
- Compare class assignment, struct assignment, and record equality in running code.
- Use `public`, `private`, `get`, `set`, and `init` in simple custom types.

## Why Custom Types Matter

Intermediate C# programs are built from types. Strings, arrays, lists, and dictionaries are useful, but real programs usually need names from the problem domain:

- `Order`
- `Product`
- `Money`
- `LibraryCard`
- `Checkout`

A good custom type makes code easier to read because the type name explains what the value represents. It also gives you one place to protect rules. For example, an `Order` can decide when items may be added, when the status changes, and how totals are calculated.

The important design question is not "Which C# feature is newest?" It is "What kind of thing am I modeling?"

## The Four Main Choices

| Type kind | Use it when | Assignment behavior | Equality behavior |
| --- | --- | --- | --- |
| `class` | You need mutable state, behavior, identity, inheritance, or a large/long-lived object. | Copies the reference. Two variables can point to the same object. | Usually compares object identity unless you write custom equality. |
| `record class` | You need a data-focused reference type with generated value equality, generated `ToString`, and `with` copies. | Copies the reference. The object still lives separately from the variable. | Compares values by default. |
| `struct` | You need a small value type with copy semantics. | Copies the whole value. | Compares values only if you provide or get appropriate equality behavior. |
| `record struct` | You need a small data value with copy semantics plus generated equality and readable printing. | Copies the whole value. | Compares values by default. |

Most application objects should start as classes. Reach for structs only when the value is small, self-contained, and naturally copied, such as a coordinate, measurement, or money amount. Reach for records when the type is mostly data and value-based equality is useful.

## Reference Types And Value Types

A `class` is a reference type. A variable of a class type stores a reference to an object. If you assign one class variable to another, both variables point to the same object.

```text
Order first = new Order("A100");
Order second = first;

second.AddItem(new Product("Notebook", new Money(4.50m, "USD")));
second.MarkPaid();

Console.WriteLine(first.Status);
Console.WriteLine(first.ItemCount);
```

Output:

```text
Paid
1
```

`first` and `second` are two variables, but they refer to one `Order` object. Changing the order through `second` is visible through `first`.

A `struct` is a value type. A variable of a struct type stores the value directly. If you assign one struct variable to another, C# copies the value.

```text
Money original = new Money(12.00m, "USD");
Money copy = original;
copy = copy with { Amount = 15.00m };

Console.WriteLine(original);
Console.WriteLine(copy);
```

Output:

```text
12.00 USD
15.00 USD
```

Changing `copy` does not change `original` because `Money` is a small value type.

## Records Compare Values

Records are designed for data-focused types. A `record class` is still a reference type, but it gets generated value equality. Two separate record objects can compare as equal when their stored values match.

```text
Product mug1 = new Product("Travel Mug", new Money(18.00m, "USD"));
Product mug2 = new Product("Travel Mug", new Money(18.00m, "USD"));

Console.WriteLine(mug1 == mug2);
Console.WriteLine(ReferenceEquals(mug1, mug2));
Console.WriteLine(mug1);
```

Output:

```text
True
False
Product { Name = Travel Mug, Price = 18.00 USD }
```

`mug1 == mug2` is `True` because record equality compares the data. `ReferenceEquals(mug1, mug2)` is `False` because they are still two different objects.

Records also support `with` expressions, which create a copy with selected changes:

```text
Product saleMug = mug1 with { Price = new Money(14.00m, "USD") };

Console.WriteLine(mug1);
Console.WriteLine(saleMug);
```

Output:

```text
Product { Name = Travel Mug, Price = 18.00 USD }
Product { Name = Travel Mug, Price = 14.00 USD }
```

## A Store Mini-Domain

This small domain uses three type choices:

- `Order` is a `class` because it has mutable state, behavior, and identity.
- `Product` is a `record class` because it is data-focused and two products with the same name and price should compare as equal.
- `Money` is a `readonly record struct` because it is a small value that is safe to copy and useful to print.

```csharp
using System;
using System.Collections.Generic;

Order order = new Order("A100");

Product notebook = new Product("Notebook", new Money(4.50m, "USD"));
Product pen = new Product("Pen", new Money(1.25m, "USD"));

order.AddItem(notebook);
order.AddItem(pen);
order.AddItem(pen);

order.PrintReceipt();
order.MarkPaid();

Console.WriteLine($"Status: {order.Status}");

public class Order
{
    private readonly List<Product> _items = new List<Product>();

    public Order(string orderNumber)
    {
        OrderNumber = orderNumber;
        Status = "Open";
    }

    public string OrderNumber { get; }
    public string Status { get; private set; }
    public int ItemCount => _items.Count;

    public void AddItem(Product product)
    {
        if (Status != "Open")
        {
            Console.WriteLine($"Cannot add {product.Name}; order {OrderNumber} is {Status}.");
            return;
        }

        _items.Add(product);
    }

    public void MarkPaid()
    {
        if (_items.Count == 0)
        {
            Console.WriteLine("Cannot pay for an empty order.");
            return;
        }

        Status = "Paid";
    }

    public Money CalculateTotal()
    {
        Money total = new Money(0.00m, "USD");

        foreach (Product item in _items)
        {
            total = total.Add(item.Price);
        }

        return total;
    }

    public void PrintReceipt()
    {
        Console.WriteLine($"Order {OrderNumber}");

        foreach (Product item in _items)
        {
            Console.WriteLine($"- {item.Name}: {item.Price}");
        }

        Console.WriteLine($"Total: {CalculateTotal()}");
    }
}

public record class Product(string Name, Money Price);

public readonly record struct Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
        {
            throw new InvalidOperationException("Cannot add money values with different currencies.");
        }

        return new Money(Amount + other.Amount, Currency);
    }

    public override string ToString()
    {
        return $"{Amount:0.00} {Currency}";
    }
}
```

Expected output:

```text
Order A100
- Notebook: 4.50 USD
- Pen: 1.25 USD
- Pen: 1.25 USD
Total: 7.00 USD
Status: Paid
```

## Access And Properties

Use access modifiers to show what other code is allowed to touch:

- `public` means code outside the type can use the member.
- `private` means only code inside the same type can use the member.

In the `Order` class:

```text
private readonly List<Product> _items = new List<Product>();
public string Status { get; private set; }
```

`_items` is private because outside code should not bypass the order's rules by editing the list directly. `Status` has a public getter and a private setter, so other code can read the status but only `Order` can change it.

Properties describe controlled access to data:

- `get` lets code read the property.
- `set` lets code change the property after the object is created.
- `init` lets code set the property during object creation, then keeps it stable.

For example:

```csharp
public class Customer
{
    public string Name { get; init; } = "";
    public string Email { get; set; } = "";
}
```

`Name` uses `init` because the name should be provided when the customer is created. `Email` uses `set` because a customer might update it later.

Records often use positional syntax:

```text
public record class Product(string Name, Money Price);
```

That one line creates public properties, a constructor, value equality, a readable `ToString`, and support for `with` expressions.

## How To Choose

Ask these questions in order:

1. Does this thing have identity and a lifetime? Use a `class`.
2. Does it need to change over time through methods? Use a `class`.
3. Is it mainly data, and should two instances with the same values compare as equal? Use a `record class`.
4. Is it small, self-contained, and naturally copied? Consider a `struct`.
5. Is it a small data value where generated equality and printing would help? Use a `record struct`.
6. Would copying it be expensive or surprising? Use a reference type instead of a struct.

When in doubt for business objects, use a class. When in doubt for small immutable values, consider a `readonly record struct`.

## Common Mistakes

- Using a normal `class` when value equality is expected. Two separate class instances with the same property values are not equal by default.
- Making large mutable structs. Copying a large struct can be expensive, and mutating copies can be confusing.
- Forgetting that class variables can point to the same object. Assignment copies the reference, not the object.
- Assuming records make nested mutable objects immutable. A record containing a mutable list can still expose changes to that list.
- Mutating struct copies unintentionally. If a struct is copied into another variable, changes to the copy do not change the original.
- Using public fields instead of properties. Prefer properties so your type can control access and evolve safely.
- Choosing records only because they are shorter. Records are best when value equality and data-focused behavior match the model.

## Practical Exercise

Build a small library checkout or store cart model. Use exactly these three custom type roles:

1. One `class` for the object with mutable state and behavior.
2. One `record class` for a data item.
3. One `record struct` for a small value.

Store cart option:

1. Create a `Cart` class with a private list of items.
2. Create a `Product` record class with `Name` and `Price`.
3. Create a `Money` readonly record struct with `Amount` and `Currency`.
4. Add methods to `Cart` for `AddItem`, `Checkout`, `CalculateTotal`, and `PrintSummary`.
5. Prevent adding new items after checkout.
6. Show that assigning a cart variable copies the reference.
7. Show that assigning a money variable copies the value.
8. Show that two matching products compare as equal.

Library checkout option:

1. Create a `Checkout` class with a private list of books and a mutable status.
2. Create a `Book` record class with `Title` and `Author`.
3. Create a `LoanPeriod` readonly record struct with `Days`.
4. Add methods to check out books, close the checkout, and print a summary.
5. Demonstrate class assignment, record equality, and struct copy behavior.

## Worked Answer

This answer uses the store cart option. It is a complete runnable top-level program.

```csharp
using System;
using System.Collections.Generic;

Cart cart = new Cart("C100");

Product tea = new Product("Mint Tea", new Money(3.50m, "USD"));
Product matchingTea = new Product("Mint Tea", new Money(3.50m, "USD"));
Product bread = new Product("Bread", new Money(4.25m, "USD"));

Console.WriteLine($"Products equal by value: {tea == matchingTea}");
Console.WriteLine($"Same product object: {ReferenceEquals(tea, matchingTea)}");
Console.WriteLine();

Cart sameCart = cart;
sameCart.AddItem(tea);
sameCart.AddItem(bread);

Console.WriteLine($"Original cart item count: {cart.ItemCount}");
Console.WriteLine($"Same cart item count: {sameCart.ItemCount}");
Console.WriteLine();

Money price = new Money(10.00m, "USD");
Money copiedPrice = price;
copiedPrice = copiedPrice with { Amount = 8.00m };

Console.WriteLine($"Original price: {price}");
Console.WriteLine($"Copied price: {copiedPrice}");
Console.WriteLine();

cart.PrintSummary();
cart.Checkout();
cart.AddItem(new Product("Coffee", new Money(6.00m, "USD")));

public class Cart
{
    private readonly List<Product> _items = new List<Product>();

    public Cart(string cartId)
    {
        CartId = cartId;
        Status = "Open";
    }

    public string CartId { get; }
    public string Status { get; private set; }
    public int ItemCount => _items.Count;

    public void AddItem(Product product)
    {
        if (Status != "Open")
        {
            Console.WriteLine($"Cannot add {product.Name}; cart {CartId} is {Status}.");
            return;
        }

        _items.Add(product);
    }

    public Money CalculateTotal()
    {
        Money total = new Money(0.00m, "USD");

        foreach (Product item in _items)
        {
            total = total.Add(item.Price);
        }

        return total;
    }

    public void Checkout()
    {
        if (_items.Count == 0)
        {
            Console.WriteLine("Cannot check out an empty cart.");
            return;
        }

        Status = "Checked out";
        Console.WriteLine($"Cart {CartId} checked out.");
    }

    public void PrintSummary()
    {
        Console.WriteLine($"Cart {CartId} ({Status})");

        foreach (Product item in _items)
        {
            Console.WriteLine($"- {item.Name}: {item.Price}");
        }

        Console.WriteLine($"Total: {CalculateTotal()}");
    }
}

public record class Product(string Name, Money Price);

public readonly record struct Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
        {
            throw new InvalidOperationException("Cannot add money values with different currencies.");
        }

        return new Money(Amount + other.Amount, Currency);
    }

    public override string ToString()
    {
        return $"{Amount:0.00} {Currency}";
    }
}
```

Expected output:

```text
Products equal by value: True
Same product object: False

Original cart item count: 2
Same cart item count: 2

Original price: 10.00 USD
Copied price: 8.00 USD

Cart C100 (Open)
- Mint Tea: 3.50 USD
- Bread: 4.25 USD
Total: 7.75 USD
Cart C100 checked out.
Cannot add Coffee; cart C100 is Checked out.
```

Teaching notes:

- `Cart` is a class because the cart has changing state and behavior.
- `sameCart = cart` copies the reference, so both variables point to the same cart.
- `Product` is a record class, so `tea == matchingTea` compares property values.
- `ReferenceEquals(tea, matchingTea)` is false because the two products are different objects.
- `Money` is a readonly record struct, so assigning it copies the value.
- `Cart` keeps `_items` private so all changes go through `AddItem`.
- `Status` uses `private set` so only the `Cart` class can change its own status.

## Sources

- Microsoft Learn: [Classes, structs, and records overview](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/)
- Microsoft Learn: [C# classes](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes)
- Microsoft Learn: [Structure types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct)
- Microsoft Learn: [Record types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)
- Microsoft Learn: [C# record types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/records)
- Microsoft Learn: [Choose between tuples, records, structs, and classes](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/choosing-types)
