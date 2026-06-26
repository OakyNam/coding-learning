# 03 - Collections And LINQ

## Learning Goal

Use strongly typed C# collections and LINQ to store, find, filter, sort, project, and summarize in-memory data.

By the end of this lesson, you should be able to:

- Choose `List<T>` when order and index access matter.
- Choose `Dictionary<TKey, TValue>` when key lookup matters.
- Use `IEnumerable<T>` when code only needs to iterate through values.
- Explain why generic collections are strongly typed and usually preferred in normal C# application code.
- Add to a list, read by index, check `Count`, and iterate with `foreach`.
- Add or assign dictionary values by key, use `TryGetValue`, and iterate key/value pairs.
- Write LINQ queries with `Where`, `Select`, `OrderBy`, `ThenBy`, `GroupBy`, `Count`, `Sum`, `Average`, and `ToList`.
- Recognize when LINQ query syntax and method syntax express the same query.
- Explain deferred execution and name common operations that force a query to run.

## Why Collections Matter

Most programs work with groups of values, not just one value at a time. A store has many products. A report has many rows. A search result has many matches.

The first design question is: how will the program use the group?

| Access pattern | Common type | Use it when |
| --- | --- | --- |
| Ordered items and index access | `List<T>` | You need a resizable sequence where position matters. |
| Key lookup | `Dictionary<TKey, TValue>` | You need to find a value by a unique key such as name, SKU, email, or ID. |
| Iteration only | `IEnumerable<T>` | Code only needs to loop through values or describe a query. |

Generic collection types such as `List<Product>` and `Dictionary<string, Product>` are strongly typed. A `List<Product>` stores products, so the compiler helps prevent adding unrelated values by mistake. That is why generic collections are usually preferred for normal C# application code.

This lesson uses a small product inventory domain:

```text
public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

The same collection ideas apply to invoices, books, users, orders, messages, or any other domain where you manage a group of typed values.

## Lists For Ordered Values

Use `List<T>` when you want a resizable collection of values in a meaningful order. A list can be initialized with values, extended with `Add`, read by index, counted, and iterated.

```csharp
using System;
using System.Collections.Generic;

List<Product> products = new List<Product>
{
    new Product("Notebook", "Stationery", 4.50m, 12),
    new Product("Desk Lamp", "Office", 29.99m, 5)
};

products.Add(new Product("Coffee Mug", "Kitchen", 8.25m, 0));

Console.WriteLine($"First product: {products[0].Name}");
Console.WriteLine($"Product count: {products.Count}");

foreach (Product product in products)
{
    Console.WriteLine($"{product.Name}: {product.UnitsInStock} in stock");
}

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Expected output:

```text
First product: Notebook
Product count: 3
Notebook: 12 in stock
Desk Lamp: 5 in stock
Coffee Mug: 0 in stock
```

List indexes start at `0`. In the example, `products[0]` is the first product. Use index access only when you know the index is valid. If you mainly need to process every item, `foreach` is clearer and avoids index mistakes.

## Dictionaries For Key Lookup

Use `Dictionary<TKey, TValue>` when each value should be found by a key. In an inventory program, a product name or SKU can be the key.

```csharp
using System;
using System.Collections.Generic;

Dictionary<string, Product> inventoryBySku = new Dictionary<string, Product>
{
    ["NB-100"] = new Product("Notebook", "Stationery", 4.50m, 12),
    ["DL-200"] = new Product("Desk Lamp", "Office", 29.99m, 5)
};

inventoryBySku["MG-300"] = new Product("Coffee Mug", "Kitchen", 8.25m, 0);
inventoryBySku["NB-100"] = new Product("Notebook", "Stationery", 4.25m, 20);

if (inventoryBySku.TryGetValue("DL-200", out Product? lamp))
{
    Console.WriteLine($"Found {lamp.Name} at {lamp.Price:0.00}");
}

if (!inventoryBySku.TryGetValue("XX-999", out Product? missing))
{
    Console.WriteLine("SKU XX-999 was not found.");
}

foreach (KeyValuePair<string, Product> entry in inventoryBySku)
{
    Console.WriteLine($"{entry.Key}: {entry.Value.Name}");
}

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Possible output:

```text
Found Desk Lamp at 29.99
SKU XX-999 was not found.
NB-100: Notebook
DL-200: Desk Lamp
MG-300: Coffee Mug
```

The dictionary indexer has two common uses:

- `inventoryBySku["MG-300"] = product` adds the key if it is missing.
- `inventoryBySku["NB-100"] = product` replaces the value if the key already exists.

Keys must be unique. Calling `Add` with an existing key throws an exception; assigning through the indexer replaces the value. Also, do not use dictionary iteration order for sorted reports. If order matters, sort the entries before printing them.

Reading with the indexer is different:

```text
Product product = inventoryBySku["XX-999"]; // Throws if the key is missing.
```

Use `TryGetValue` when the key might not exist. It keeps lookup code explicit and avoids using exceptions for normal missing-key cases.

## IEnumerable For Iteration

`IEnumerable<T>` represents a sequence that can be enumerated. Many collection types implement it, including `List<T>` and dictionary key/value collections. LINQ also returns `IEnumerable<T>` for many queries.

Use `IEnumerable<T>` as a parameter when a method only needs to read and loop through values:

```csharp
using System;
using System.Collections.Generic;

List<Product> products = new List<Product>
{
    new Product("Notebook", "Stationery", 4.50m, 12),
    new Product("Desk Lamp", "Office", 29.99m, 5),
    new Product("Coffee Mug", "Kitchen", 8.25m, 0)
};

PrintInventory(products);

void PrintInventory(IEnumerable<Product> inventory)
{
    foreach (Product product in inventory)
    {
        Console.WriteLine($"{product.Name}: {product.UnitsInStock}");
    }
}

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Expected output:

```text
Notebook: 12
Desk Lamp: 5
Coffee Mug: 0
```

The method does not need list-specific operations such as `Add`, `Count`, or index access, so `IEnumerable<Product>` communicates exactly what it needs: a sequence of products to iterate.

## LINQ Over In-Memory Collections

LINQ lets you query objects with C# syntax. For in-memory collections such as `List<Product>`, the most common LINQ operators come from `System.Linq`.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

List<Product> products = new List<Product>
{
    new Product("Notebook", "Stationery", 4.50m, 12),
    new Product("Desk Lamp", "Office", 29.99m, 5),
    new Product("Coffee Mug", "Kitchen", 8.25m, 0),
    new Product("Pencil", "Stationery", 0.75m, 40),
    new Product("Monitor Stand", "Office", 42.00m, 3)
};

List<string> affordableInStockNames = products
    .Where(product => product.UnitsInStock > 0)
    .Where(product => product.Price < 10.00m)
    .OrderBy(product => product.Category)
    .ThenBy(product => product.Name)
    .Select(product => product.Name)
    .ToList();

foreach (string name in affordableInStockNames)
{
    Console.WriteLine(name);
}

int inStockCount = products.Count(product => product.UnitsInStock > 0);
decimal inventoryValue = products.Sum(product => product.Price * product.UnitsInStock);
decimal averagePrice = products.Average(product => product.Price);

Console.WriteLine($"In-stock product types: {inStockCount}");
Console.WriteLine($"Inventory value: {inventoryValue:0.00}");
Console.WriteLine($"Average price: {averagePrice:0.00}");

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Expected output:

```text
Notebook
Pencil
In-stock product types: 4
Inventory value: 359.95
Average price: 17.10
```

Read the method chain from top to bottom:

- `Where` keeps only matching products.
- `OrderBy` starts the sort.
- `ThenBy` adds a secondary sort.
- `Select` projects each product into a different shape, in this case just the product name.
- `ToList` creates a concrete `List<string>` from the query results.
- `Count`, `Sum`, and `Average` calculate aggregate values.

## Query Syntax And Method Syntax

C# supports two LINQ styles: query syntax and method syntax. They are semantically equivalent for many queries. Choose the one your team finds clearer for the query at hand.

These two queries produce the same output:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

List<Product> products = new List<Product>
{
    new Product("Notebook", "Stationery", 4.50m, 12),
    new Product("Desk Lamp", "Office", 29.99m, 5),
    new Product("Coffee Mug", "Kitchen", 8.25m, 0),
    new Product("Pencil", "Stationery", 0.75m, 40),
    new Product("Monitor Stand", "Office", 42.00m, 3)
};

IEnumerable<string> methodSyntax = products
    .Where(product => product.UnitsInStock > 0 && product.Price < 10.00m)
    .OrderBy(product => product.Category)
    .ThenBy(product => product.Name)
    .Select(product => product.Name);

IEnumerable<string> querySyntax =
    from product in products
    where product.UnitsInStock > 0 && product.Price < 10.00m
    orderby product.Category, product.Name
    select product.Name;

Console.WriteLine("Method syntax:");
foreach (string name in methodSyntax)
{
    Console.WriteLine(name);
}

Console.WriteLine();
Console.WriteLine("Query syntax:");
foreach (string name in querySyntax)
{
    Console.WriteLine(name);
}

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Expected output:

```text
Method syntax:
Notebook
Pencil

Query syntax:
Notebook
Pencil
```

Method syntax is required for some operations because not every standard query operator has a query syntax keyword. For example, `Count`, `Sum`, `Average`, `ToList`, and `ToDictionary` are written as method calls.

## Grouping And Summaries

Use `GroupBy` when a report should collect items under a shared key, such as category.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

List<Product> products = new List<Product>
{
    new Product("Notebook", "Stationery", 4.50m, 12),
    new Product("Desk Lamp", "Office", 29.99m, 5),
    new Product("Coffee Mug", "Kitchen", 8.25m, 0),
    new Product("Pencil", "Stationery", 0.75m, 40),
    new Product("Monitor Stand", "Office", 42.00m, 3)
};

IEnumerable<IGrouping<string, Product>> groups = products
    .GroupBy(product => product.Category)
    .OrderBy(group => group.Key);

foreach (IGrouping<string, Product> group in groups)
{
    int units = group.Sum(product => product.UnitsInStock);
    decimal value = group.Sum(product => product.Price * product.UnitsInStock);

    Console.WriteLine($"{group.Key}: {group.Count()} products, {units} units, {value:0.00} value");
}

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Expected output:

```text
Kitchen: 1 products, 0 units, 0.00 value
Office: 2 products, 8 units, 275.95 value
Stationery: 2 products, 52 units, 84.00 value
```

Each group has a `Key`, which is the category in this example. The group itself is also a sequence of `Product` values, so you can use `Count`, `Sum`, `Average`, `Where`, and other LINQ operations on each group.

## Deferred Execution

Many LINQ queries return `IEnumerable<T>`. That usually means the query describes work to do later. The query does not run when you create it. It runs when you enumerate it.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

List<Product> products = new List<Product>
{
    new Product("Notebook", "Stationery", 4.50m, 12),
    new Product("Desk Lamp", "Office", 29.99m, 5)
};

IEnumerable<Product> inStock = products.Where(product => product.UnitsInStock > 0);

products.Add(new Product("Pencil", "Stationery", 0.75m, 40));

foreach (Product product in inStock)
{
    Console.WriteLine(product.Name);
}

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Expected output:

```text
Notebook
Desk Lamp
Pencil
```

`Pencil` appears because the `Where` query did not run until the `foreach` loop enumerated it.

Operations such as `Count`, `First`, `Max`, `ToList`, and `ToArray` force execution. Use `ToList` when you want to capture the results now:

```text
List<Product> snapshot = products
    .Where(product => product.UnitsInStock > 0)
    .ToList();
```

After `ToList`, `snapshot` is a separate list of the results from that moment.

## Common Mistakes

- Choosing a list when every operation is "find by key." A dictionary usually better communicates key-based lookup.
- Choosing a dictionary when duplicate keys are expected. A dictionary maps each key to one value.
- Reading a dictionary with `inventoryBySku[key]` when the key might be missing. Use `TryGetValue` for unknown keys.
- Treating `IEnumerable<T>` like a list. It supports iteration, but it does not promise index access or `Add`.
- Forgetting that list indexes start at `0`.
- Forgetting `using System.Linq;` before calling LINQ extension methods such as `Where` and `Select`.
- Assuming a LINQ query ran when it was assigned to a variable. Many `IEnumerable<T>` queries run later when enumerated.
- Forgetting that `ToList` creates a snapshot. Later changes to the original list are not automatically added to that snapshot.
- Overusing LINQ when a simple `foreach` loop would be easier to read. Clear code is the goal.

## Practical Exercise

Build an inventory report program.

Requirements:

1. Create a `Product` record with `Name`, `Category`, `Price`, and `UnitsInStock`.
2. Create a `List<Product>` with at least six products across at least three categories.
3. Print every product in the list with a `foreach` loop.
4. Use LINQ to report products that are in stock.
5. Use LINQ to report products under a chosen price.
6. Sort products by category, then name.
7. Use `GroupBy` to print category summaries.
8. Create a `Dictionary<string, Product>` keyed by product name or SKU.
9. Use `TryGetValue` for a safe lookup that succeeds.
10. Use `TryGetValue` for a safe lookup that fails.
11. Print clear report output so a reader can tell which section produced each line.

Before writing code, choose the access pattern for each part:

- The full inventory is a list because report order and iteration matter.
- The lookup table is a dictionary because one product should be found by one key.
- LINQ queries can stay as `IEnumerable<Product>` until you need a concrete list or aggregate value.

## Worked Answer

This is a complete runnable top-level program.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

List<Product> products = new List<Product>
{
    new Product("Notebook", "Stationery", 4.50m, 12),
    new Product("Pencil", "Stationery", 0.75m, 40),
    new Product("Desk Lamp", "Office", 29.99m, 5),
    new Product("Monitor Stand", "Office", 42.00m, 3),
    new Product("Coffee Mug", "Kitchen", 8.25m, 0),
    new Product("Water Bottle", "Kitchen", 14.00m, 9)
};

Console.WriteLine("Full inventory:");
foreach (Product product in products)
{
    Console.WriteLine($"- {product.Name} ({product.Category}): {product.Price:0.00}, {product.UnitsInStock} units");
}

Console.WriteLine();
Console.WriteLine("In-stock products sorted by category and name:");
IEnumerable<string> inStockProductNames = products
    .Where(product => product.UnitsInStock > 0)
    .OrderBy(product => product.Category)
    .ThenBy(product => product.Name)
    .Select(product => $"{product.Category}: {product.Name}");

foreach (string line in inStockProductNames)
{
    Console.WriteLine($"- {line}");
}

Console.WriteLine();
decimal chosenPrice = 10.00m;
Console.WriteLine($"Products under {chosenPrice:0.00}:");
IEnumerable<Product> productsUnderPrice =
    from product in products
    where product.Price < chosenPrice
    orderby product.Category, product.Name
    select product;

foreach (Product product in productsUnderPrice)
{
    Console.WriteLine($"- {product.Name}: {product.Price:0.00}");
}

Console.WriteLine();
Console.WriteLine("Equivalent method-syntax query:");
IEnumerable<Product> productsUnderPriceMethodSyntax = products
    .Where(product => product.Price < chosenPrice)
    .OrderBy(product => product.Category)
    .ThenBy(product => product.Name);

foreach (Product product in productsUnderPriceMethodSyntax)
{
    Console.WriteLine($"- {product.Name}: {product.Price:0.00}");
}

Console.WriteLine();
Console.WriteLine("Category summaries:");
foreach (IGrouping<string, Product> group in products.GroupBy(product => product.Category).OrderBy(group => group.Key))
{
    int productCount = group.Count();
    int totalUnits = group.Sum(product => product.UnitsInStock);
    decimal totalValue = group.Sum(product => product.Price * product.UnitsInStock);
    decimal averagePrice = group.Average(product => product.Price);

    Console.WriteLine($"- {group.Key}: {productCount} products, {totalUnits} units, {totalValue:0.00} value, {averagePrice:0.00} average price");
}

Console.WriteLine();
int totalUnitsInStock = products.Sum(product => product.UnitsInStock);
decimal totalInventoryValue = products.Sum(product => product.Price * product.UnitsInStock);
int outOfStockCount = products.Count(product => product.UnitsInStock == 0);

Console.WriteLine("Inventory totals:");
Console.WriteLine($"- Total units: {totalUnitsInStock}");
Console.WriteLine($"- Inventory value: {totalInventoryValue:0.00}");
Console.WriteLine($"- Out-of-stock product types: {outOfStockCount}");

Console.WriteLine();
Dictionary<string, Product> productsByName = products.ToDictionary(product => product.Name);

Console.WriteLine("Safe dictionary lookup:");
if (productsByName.TryGetValue("Desk Lamp", out Product? deskLamp))
{
    Console.WriteLine($"Found Desk Lamp: {deskLamp.UnitsInStock} units at {deskLamp.Price:0.00}");
}

if (!productsByName.TryGetValue("Stapler", out Product? stapler))
{
    Console.WriteLine("Stapler was not found.");
}

List<Product> currentInStockSnapshot = products
    .Where(product => product.UnitsInStock > 0)
    .ToList();

Console.WriteLine();
Console.WriteLine($"Snapshot count after ToList: {currentInStockSnapshot.Count}");

public record Product(string Name, string Category, decimal Price, int UnitsInStock);
```

Expected output:

```text
Full inventory:
- Notebook (Stationery): 4.50, 12 units
- Pencil (Stationery): 0.75, 40 units
- Desk Lamp (Office): 29.99, 5 units
- Monitor Stand (Office): 42.00, 3 units
- Coffee Mug (Kitchen): 8.25, 0 units
- Water Bottle (Kitchen): 14.00, 9 units

In-stock products sorted by category and name:
- Kitchen: Water Bottle
- Office: Desk Lamp
- Office: Monitor Stand
- Stationery: Notebook
- Stationery: Pencil

Products under 10.00:
- Coffee Mug: 8.25
- Notebook: 4.50
- Pencil: 0.75

Equivalent method-syntax query:
- Coffee Mug: 8.25
- Notebook: 4.50
- Pencil: 0.75

Category summaries:
- Kitchen: 2 products, 9 units, 126.00 value, 11.13 average price
- Office: 2 products, 8 units, 275.95 value, 36.00 average price
- Stationery: 2 products, 52 units, 84.00 value, 2.63 average price

Inventory totals:
- Total units: 69
- Inventory value: 485.95
- Out-of-stock product types: 1

Safe dictionary lookup:
Found Desk Lamp: 5 units at 29.99
Stapler was not found.

Snapshot count after ToList: 5
```

Teaching notes:

- The first report uses the list directly because it wants every product in insertion order.
- `inStockProductNames` is an `IEnumerable<string>` query. It runs when the `foreach` loop enumerates it.
- The query-expression version and method-syntax version of "products under 10.00" produce the same product order and output.
- `GroupBy` creates one group per category. Each group can be summarized with `Count`, `Sum`, and `Average`.
- `ToDictionary` builds a lookup table keyed by product name. This works because every product name in the list is unique.
- `TryGetValue` handles both found and missing keys without throwing for the missing key.
- `ToList` forces the in-stock query to run and stores a snapshot of the current results.

## Sources

- Microsoft Learn: [Collections - C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/collections)
- Microsoft Learn: [Collections and Data Structures](https://learn.microsoft.com/en-us/dotnet/standard/collections/)
- Microsoft Learn: [Language Integrated Query (LINQ)](https://learn.microsoft.com/en-us/dotnet/csharp/linq/)
- Microsoft Learn: [LINQ overview](https://learn.microsoft.com/en-us/dotnet/standard/linq/)
- Microsoft Learn: [Introduction to LINQ Queries](https://learn.microsoft.com/en-us/dotnet/csharp/linq/get-started/introduction-to-linq-queries)
- Microsoft Learn: [Write LINQ queries](https://learn.microsoft.com/en-us/dotnet/csharp/linq/get-started/write-linq-queries)
- Microsoft Learn: [Standard query operators overview](https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/)
- Microsoft Learn: [Dictionary<TKey,TValue> Class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2)
- Microsoft Learn: [Dictionary<TKey,TValue>.TryGetValue Method](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2.trygetvalue)
- Microsoft Learn: [List<T> Class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)
