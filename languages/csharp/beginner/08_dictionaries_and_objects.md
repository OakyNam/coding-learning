# 08 - Dictionaries and Objects

## Learning Goal

Use `Dictionary<TKey, TValue>` to look up values by key, then store simple objects as dictionary values.

By the end of this lesson, you should be able to:

- Explain key/value pairs.
- Create and initialize a dictionary.
- Read and update dictionary values by key.
- Avoid missing-key and duplicate-key errors.
- Use `ContainsKey` and `TryGetValue`.
- Loop through dictionary entries with `.Key` and `.Value`.
- Define a small class and create objects with object initializers.
- Store objects in a dictionary, such as students looked up by ID.

## Why Dictionaries Matter

Arrays and lists are ordered collections. You read their values by index:

```csharp
string[] names = { "Mina", "Leo" };
Console.WriteLine(names[0]);
```

A dictionary is different. It stores **key/value pairs**, so you can look up a value by a meaningful key:

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>();

scores.Add("Mina", 90);
scores["Leo"] = 85;

Console.WriteLine(scores["Mina"]);
```

Output:

```text
90
```

In `Dictionary<string, int>`:

- `string` is the key type.
- `int` is the value type.
- `"Mina"` and `"Leo"` are keys.
- `90` and `85` are values.

Each key in a dictionary must be unique. Values do not have to be unique. Two students can both have a score of `90`, but the same key cannot be added twice.

## Creating A Dictionary

To use `Dictionary<TKey, TValue>`, include this line at the top of the file in older project styles:

```csharp
using System.Collections.Generic;
```

Then create an empty dictionary:

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>();

scores.Add("Mina", 90);
scores["Leo"] = 85;
```

`Add` creates a new key/value pair. The indexer syntax, `scores["Leo"]`, can also create a new pair when the key is not already present.

You can also create a dictionary with initial values:

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>
{
    { "Mina", 90 },
    { "Leo", 85 },
    { "Ava", 90 }
};
```

This is called a collection initializer. Notice that `"Mina"` and `"Ava"` have the same value, `90`. That is allowed because the keys are different.

C# also supports dictionary index initializers:

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>
{
    ["Mina"] = 90,
    ["Leo"] = 85,
    ["Ava"] = 90
};
```

Both initializer styles are common. For beginner code, use whichever one your class or team uses most often.

## Reading And Updating Values

Use square brackets with a key to read a value:

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>
{
    { "Mina", 90 },
    { "Leo", 85 }
};

Console.WriteLine(scores["Mina"]);
```

Output:

```text
90
```

Use the same indexer to update an existing value:

```csharp
scores["Leo"] = 88;
Console.WriteLine(scores["Leo"]);
```

Output:

```text
88
```

Important difference:

- `scores.Add("Mina", 95)` throws an error if `"Mina"` is already a key.
- `scores["Mina"] = 95` overwrites the value if `"Mina"` already exists.
- `scores["Nora"] = 82` adds a new key/value pair if `"Nora"` does not exist.
- `scores["MissingName"]` throws an error when reading if the key does not exist.

The indexer is convenient, but it is not always safe for reading. Check first when a key might be missing.

## ContainsKey Before Add

Use `ContainsKey` when you want to add a value only if the key is not already present.

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>
{
    { "Mina", 90 },
    { "Leo", 85 }
};

string name = "Mina";

if (!scores.ContainsKey(name))
{
    scores.Add(name, 95);
}
else
{
    Console.WriteLine($"{name} already has a score.");
}
```

Output:

```text
Mina already has a score.
```

This avoids the duplicate-key error that `Add` would throw.

## TryGetValue For Safe Reads

Use `TryGetValue` when you want to read a value only if the key exists.

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>
{
    { "Mina", 90 },
    { "Leo", 85 }
};

string name = "Ava";

if (scores.TryGetValue(name, out int score))
{
    Console.WriteLine($"{name}: {score}");
}
else
{
    Console.WriteLine($"{name} was not found.");
}
```

Output:

```text
Ava was not found.
```

`TryGetValue` returns a `bool`:

- `true` means the key was found, and the value was placed in the `out` variable.
- `false` means the key was not found.

This is the safest beginner pattern for user input, because users often type keys that are not in the dictionary.

## Looping Through A Dictionary

Use `foreach` to loop through the entries in a dictionary.

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>
{
    { "Mina", 90 },
    { "Leo", 85 },
    { "Ava", 90 }
};

foreach (KeyValuePair<string, int> entry in scores)
{
    Console.WriteLine($"{entry.Key}: {entry.Value}");
}
```

Possible output:

```text
Mina: 90
Leo: 85
Ava: 90
```

Each loop item is a key/value pair. Use `.Key` to read the key and `.Value` to read the value.

Do not write code that depends on a beginner dictionary printing in a specific order. Use a dictionary when lookup by key matters most.

## Beginner Objects And Classes

A dictionary is great for lookup, but it only connects one key to one value. Sometimes the value needs several related pieces of data.

For example, a student has a name and a grade. You can define a small class:

```csharp
public class Student
{
    public string Name { get; set; } = "";
    public int Grade { get; set; }
}
```

A class is a blueprint. An object is one actual value created from that blueprint.

Create a `Student` object with an object initializer:

```csharp
Student student = new Student
{
    Name = "Mina",
    Grade = 90
};

Console.WriteLine($"{student.Name}: {student.Grade}");
```

Output:

```text
Mina: 90
```

Use dot syntax for object properties, such as `student.Name` and `student.Grade`.

Use bracket syntax for dictionary keys, such as `scores["Mina"]`.

That difference is small, but it prevents many beginner mistakes.

## Dictionary With Object Values

The value in a dictionary can be an object. For example, use a student ID as the key and a `Student` object as the value:

```csharp
Dictionary<string, Student> studentsById = new Dictionary<string, Student>
{
    ["S100"] = new Student { Name = "Mina", Grade = 90 },
    ["S101"] = new Student { Name = "Leo", Grade = 85 },
    ["S102"] = new Student { Name = "Ava", Grade = 90 }
};

string id = "S101";

if (studentsById.TryGetValue(id, out Student? student))
{
    Console.WriteLine($"{id}: {student.Name}, grade {student.Grade}");
}
else
{
    Console.WriteLine($"{id} was not found.");
}
```

Output:

```text
S101: Leo, grade 85
```

The key is the lookup value: `"S101"`.

The value is the object, such as `new Student { Name = "Leo", Grade = 85 }`.

This is a common pattern: use a short, unique key for lookup and store a richer object as the value.

## Dictionary vs Object

A **dictionary** is a collection of key/value pairs. Use it when:

- You have many entries of the same general kind.
- You need to find an entry by key.
- The keys are data, such as student IDs, usernames, product codes, or commands.
- Entries may be added, removed, or looked up while the program runs.

An **object** is one thing with named properties. Use it when:

- You want to model one real concept, such as one student.
- The property names are part of the design, such as `Name` and `Grade`.
- You want dot syntax, such as `student.Name`.

For this lesson:

- `Dictionary<string, Student>` is the roster.
- `"S101"` is a dictionary key.
- `Student` is the object type.
- `Name` and `Grade` are object properties.

Do not make every value a string just because user input is text. Use the type that matches the data. A grade should usually be an `int`, while a name or ID should usually be a `string`.

## Common Mistakes

- Reading a missing key with the indexer: `scores["Ava"]` throws an error if `"Ava"` is not a key.
- Calling `Add` with a duplicate key. Use `ContainsKey` first, or use the indexer when overwriting is intended.
- Forgetting that keys must be unique, but values may repeat.
- Confusing the dictionary key with an object property. In `studentsById["S101"].Name`, `"S101"` is the key and `Name` is a property on the `Student` object.
- Making all values strings. Store numbers as `int` or `double` when you need numeric behavior.
- Using dot syntax for dictionary keys, such as `scores.Mina`. Dictionaries use brackets: `scores["Mina"]`.
- Using bracket syntax for object properties, such as `student["Name"]`. Regular C# objects use dots: `student.Name`.
- Depending on dictionary loop order for beginner programs. If order matters, learn an ordered collection or sort the data before printing.

## Exercise

Write a student roster program.

Requirements:

1. Define a `Student` class with `Name` and `Grade` properties.
2. Create a `Dictionary<string, Student>` named `studentsById`.
3. Add at least three students. Use student IDs as keys, such as `"S100"`.
4. Ask the user to enter a student ID.
5. Use `TryGetValue` to print the student's name and grade when the ID is found.
6. Print a not-found message when the ID is missing.
7. Loop through the full roster and print each ID, name, and grade.
8. Optional: before adding a new student, use `ContainsKey` to avoid adding a duplicate ID.

Example interaction:

```text
Enter student ID: S101
Found S101: Leo, grade 85

Roster:
S100 - Mina - 90
S101 - Leo - 85
S102 - Ava - 90
```

## Worked Answer

```csharp
using System;
using System.Collections.Generic;

Dictionary<string, Student> studentsById = new Dictionary<string, Student>
{
    ["S100"] = new Student { Name = "Mina", Grade = 90 },
    ["S101"] = new Student { Name = "Leo", Grade = 85 },
    ["S102"] = new Student { Name = "Ava", Grade = 90 }
};

string newId = "S101";

if (!studentsById.ContainsKey(newId))
{
    studentsById.Add(newId, new Student { Name = "Nora", Grade = 88 });
}
else
{
    Console.WriteLine($"{newId} is already in the roster.");
}

Console.Write("Enter student ID: ");
string requestedId = Console.ReadLine() ?? "";

if (studentsById.TryGetValue(requestedId, out Student? foundStudent))
{
    Console.WriteLine($"Found {requestedId}: {foundStudent.Name}, grade {foundStudent.Grade}");
}
else
{
    Console.WriteLine($"{requestedId} was not found.");
}

Console.WriteLine();
Console.WriteLine("Roster:");

foreach (KeyValuePair<string, Student> entry in studentsById)
{
    Console.WriteLine($"{entry.Key} - {entry.Value.Name} - {entry.Value.Grade}");
}

public class Student
{
    public string Name { get; set; } = "";
    public int Grade { get; set; }
}
```

Expected output if the user enters `S101`:

```text
S101 is already in the roster.
Enter student ID: S101
Found S101: Leo, grade 85

Roster:
S100 - Mina - 90
S101 - Leo - 85
S102 - Ava - 90
```

Teaching notes:

- `studentsById` is the dictionary.
- Each key is a student ID, such as `"S101"`.
- Each value is a `Student` object.
- `ContainsKey` prevents a duplicate `Add`.
- `TryGetValue` prevents a missing-key read error.
- `entry.Key` prints the ID.
- `entry.Value.Name` and `entry.Value.Grade` read properties from the object stored as the value.
- The `Student` class is placed after the top-level statements. That keeps the beginner program easy to read while still defining the object type.

## Sources

- Microsoft Learn: [Dictionary&lt;TKey,TValue&gt; class](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2)
- Microsoft Learn: [Dictionary&lt;TKey,TValue&gt;.ContainsKey(TKey)](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2.containskey)
- Microsoft Learn: [Dictionary&lt;TKey,TValue&gt;.TryGetValue(TKey, TValue)](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2.trygetvalue)
- Microsoft Learn: [Collections - C# reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/collections)
- Microsoft Learn: [Commonly Used Collection Types](https://learn.microsoft.com/en-us/dotnet/standard/collections/commonly-used-collection-types)
- Microsoft Learn: [C# classes](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes)
- Microsoft Learn: [Objects - create instances of types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/objects)
- Microsoft Learn: [Object and collection initializers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/object-and-collection-initializers)
- Microsoft Learn: [How to initialize a dictionary with a collection initializer](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-initialize-a-dictionary-with-a-collection-initializer)
