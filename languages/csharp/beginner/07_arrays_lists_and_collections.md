# 07 - Arrays, Lists, and Collections

## Learning Goal

Learn how to store multiple related values in C# by using arrays and `List<T>`. By the end of this lesson, you should be able to:

- Create arrays and lists.
- Read items by index.
- Check collection size with `Length` and `Count`.
- Loop through items with `foreach`.
- Add and remove items from a list.
- Choose between an array and a list for beginner programs.

## Why Collections Matter

Many programs need to work with more than one related value. A shopping list has several items. A week has several days. A game might store several scores.

Without a collection, you might write this:

```csharp
string day1 = "Monday";
string day2 = "Tuesday";
string day3 = "Wednesday";
```

That becomes hard to manage quickly. Collections let you keep related values together under one name.

The two most useful collection types for beginners are:

- An **array**, which is a fixed-size ordered collection.
- A **`List<T>`**, which is a growable ordered collection.

Both arrays and lists keep items in order. Each item has an index, and indexes start at `0`, not `1`.

## Arrays

An array stores multiple values of the same type. Its size is fixed after the array is created.

```csharp
string[] days = { "Monday", "Tuesday", "Wednesday" };
int[] scores = new int[] { 85, 92, 78 };
```

The `string[]` array stores strings. The `int[]` array stores integers.

You can read an item by using its index:

```csharp
Console.WriteLine(days[0]);
Console.WriteLine(days[2]);
```

Output:

```text
Monday
Wednesday
```

Because indexes start at `0`, `days[0]` is the first item and `days[2]` is the third item.

You can also update an item by index:

```csharp
scores[1] = 95;
Console.WriteLine(scores[1]);
```

Output:

```text
95
```

The array still has the same number of items. You changed one value inside it.

## Array Length

Use the `Length` property to find out how many items are in an array.

```csharp
string[] days = { "Monday", "Tuesday", "Wednesday" };

Console.WriteLine(days.Length);
```

Output:

```text
3
```

Since indexes start at `0`, the last valid index is one less than the length:

```csharp
Console.WriteLine(days[days.Length - 1]);
```

Output:

```text
Wednesday
```

This is invalid:

```csharp
Console.WriteLine(days[days.Length]);
```

`days.Length` is `3`, but the valid indexes are `0`, `1`, and `2`. Trying to read `days[3]` causes an error.

## Looping With foreach

A `foreach` loop is the easiest way to read every item in a collection.

```csharp
string[] days = { "Monday", "Tuesday", "Wednesday" };

foreach (string day in days)
{
    Console.WriteLine(day);
}
```

Output:

```text
Monday
Tuesday
Wednesday
```

For a regular single-dimensional array like this one, `foreach` reads the items in order from first to last.

Use `foreach` when you want every item and do not need to change the index yourself.

## Lists

A `List<T>` is like an array in that it stores ordered values of one type. The difference is that a list can grow and shrink.

To use `List<T>`, include this line at the top of the file in older project styles:

```csharp
using System.Collections.Generic;
```

Then create a list:

```csharp
List<string> tasks = new List<string> { "Wash dishes", "Fold laundry" };
```

`List<string>` means "a list of strings." The `T` in `List<T>` stands for the item type.

Modern C# also supports collection expressions in many contexts, such as:

```csharp
List<string> tasks = ["Wash dishes", "Fold laundry"];
```

For beginner lessons, the `new List<string> { ... }` form is still a clear and widely recognized starting point.

## List Count, Add, Remove, and RemoveAt

Use `Count` to find out how many items are in a list.

```csharp
List<string> tasks = new List<string> { "Wash dishes", "Fold laundry" };

Console.WriteLine(tasks.Count);
```

Output:

```text
2
```

Use `Add` to add an item to the end of a list:

```csharp
tasks.Add("Take out trash");
```

Use `Remove` to remove an item by value:

```csharp
tasks.Remove("Fold laundry");
```

Use `RemoveAt` to remove an item by index:

```csharp
tasks.RemoveAt(0);
```

The difference matters:

- `Remove("Fold laundry")` looks for that value and removes it.
- `RemoveAt(0)` removes whatever item is currently at index `0`.

After an item is removed, later items move down to fill the gap.

## Choosing an Array or a List

Use an array when:

- You know the number of items ahead of time.
- The number of items should stay stable.
- You want a simple fixed-size collection.

Use a list when:

- You need to add items.
- You need to remove items.
- The number of items can change while the program runs.

For example, the days of the week are a good fit for an array. A packing list that changes while you remember more items is a good fit for a list.

## Common Mistakes

- Thinking index `1` means the first item. The first item is index `0`.
- Using `Length` or `Count` as an index. The last valid index is `Length - 1` or `Count - 1`.
- Writing `myList.Length` in beginner examples. Lists use `Count`.
- Writing `myArray.Count` in beginner examples. Arrays use `Length`.
- Trying to call `Add` on an array. Arrays have a fixed size.
- Removing items from a list inside a `foreach` loop over that same list. This can cause errors or confusing behavior.
- Forgetting `using System.Collections.Generic;` in older project styles before using `List<T>`.
- Mixing item types, such as trying to put a string into an `int[]` or an `int` into a `List<string>`.

## Exercise

Write a packing-list program.

Requirements:

1. Create a `string[]` named `starterItems` with `"shirt"`, `"socks"`, and `"toothbrush"`.
2. Print the number of starter items using `starterItems.Length`.
3. Print the first starter item by index.
4. Print the last starter item using `starterItems[starterItems.Length - 1]`.
5. Use `foreach` to print every starter item.
6. Create a `List<string>` named `packingList` with the same three items.
7. Add `"charger"` and `"snacks"` to the list.
8. Remove `"shirt"` by value.
9. Print the number of items using `packingList.Count`.
10. Use a final `foreach` loop to print every item in the packing list.

## Worked Answer

```csharp
using System;
using System.Collections.Generic;

string[] starterItems = { "shirt", "socks", "toothbrush" };

Console.WriteLine($"Starter item count: {starterItems.Length}");
Console.WriteLine($"First starter item: {starterItems[0]}");
Console.WriteLine($"Last starter item: {starterItems[starterItems.Length - 1]}");

Console.WriteLine("Starter items:");
foreach (string item in starterItems)
{
    Console.WriteLine(item);
}

List<string> packingList = new List<string> { "shirt", "socks", "toothbrush" };

packingList.Add("charger");
packingList.Add("snacks");
packingList.Remove("shirt");

Console.WriteLine($"Packing list count: {packingList.Count}");
Console.WriteLine("Final packing list:");
foreach (string item in packingList)
{
    Console.WriteLine(item);
}
```

Expected output:

```text
Starter item count: 3
First starter item: shirt
Last starter item: toothbrush
Starter items:
shirt
socks
toothbrush
Packing list count: 4
Final packing list:
socks
toothbrush
charger
snacks
```

## Sources

- [Microsoft Learn: Arrays](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/arrays)
- [Microsoft Learn: Store and iterate through sequences of data using arrays and foreach](https://learn.microsoft.com/en-us/training/modules/csharp-arrays/)
- [Microsoft Learn: List&lt;T&gt; class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1)
- [Microsoft Learn: List&lt;T&gt;.Add(T)](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.add)
- [Microsoft Learn: List&lt;T&gt;.Remove(T)](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.remove)
- [Microsoft Learn: List&lt;T&gt;.RemoveAt(Int32)](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.removeat)
- [Microsoft Learn: List&lt;T&gt;.Count](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.count)
- [Microsoft Learn: Object and collection initializers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers)
- [Microsoft Learn: Collection expressions](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/collection-expressions)
