# 02 - Core Concepts

## Learning Goal

Choose between Python's core data structures, list, tuple, dict, and set, by deciding whether your data needs order, mutation, index access, key lookup, duplicates, or uniqueness.

## Why It Matters

Most beginner Python programs collect more than one value: names in a class, prices in a cart, IDs already seen, or settings for a user. The data structure you choose controls how easy the next step is.

If you choose well, your code reads naturally:

- "Keep these items in order" becomes a list.
- "This record should not change" becomes a tuple.
- "Find the value for this name" becomes a dict.
- "Keep each value only once" becomes a set.

## The Four Big Questions

Before choosing a structure, ask these questions.

| Question | Meaning | Structures to think about |
| --- | --- | --- |
| Ordered or unordered? | Do positions matter? | list and tuple keep sequence order; dict keeps insertion order; set should not be used for position-based output |
| Mutable or immutable? | Should the collection change after creation? | list, dict, and set are mutable; tuple is immutable |
| Indexed values or key-value mappings? | Do you look up by position or by a meaningful key? | list and tuple use indexes; dict uses keys; set has membership checks |
| Duplicates or uniqueness? | Can the same value appear more than once? | list and tuple allow duplicates; set keeps unique values; dict keeps unique keys |

## List

A list is an ordered, mutable collection. Use it when you need a sequence that can grow, shrink, or be rearranged.

```python
students = ["Maya", "Ibrahim", "Maya"]

students.append("Lina")
students[1] = "Omar"

print(students)
print(students[0])
print(len(students))
```

Expected output:

```text
['Maya', 'Omar', 'Maya', 'Lina']
Maya
4
```

Notice that the duplicate `"Maya"` values stay in the list. Lists are good when every item matters, including repeated items.

## Tuple

A tuple is an ordered, immutable collection. Use it when the values belong together as one fixed record.

```python
registration = ("Maya", "Python Basics")

student_name = registration[0]
course_name = registration[1]

print(registration)
print(student_name)
print(course_name)
```

Expected output:

```text
('Maya', 'Python Basics')
Maya
Python Basics
```

Tuples can contain duplicates, but they cannot be changed in place. That makes them useful for small records such as coordinates, dates, or one registration entry.

## Dict

A dict is a mutable key-value mapping. Use it when you want to look up a value by a meaningful key instead of by a numbered position.

```python
course_counts = {
    "Python Basics": 2,
    "Data Structures": 1,
}

course_counts["Python Basics"] += 1
course_counts["Web Basics"] = 1

print(course_counts)
print(course_counts["Python Basics"])
```

Expected output:

```text
{'Python Basics': 3, 'Data Structures': 1, 'Web Basics': 1}
3
```

Dict keys are unique. If you assign a value to an existing key, the old value is replaced.

## Set

A set is a mutable collection of unique values. Use it when you care whether something is present, but not where it appears.

```python
students = {"Maya", "Ibrahim", "Maya"}

students.add("Lina")

print("Maya" in students)
print(len(students))
print(sorted(students))
```

Expected output:

```text
True
3
['Ibrahim', 'Lina', 'Maya']
```

The duplicate `"Maya"` is stored only once. The example uses `sorted(students)` because sets should not be used when display order matters.

## Decision Table

| Need | Choose | Why |
| --- | --- | --- |
| Keep items in sequence and edit them | list | Ordered, mutable, allows duplicates |
| Store one fixed record | tuple | Ordered, immutable, simple to unpack |
| Look up values by name, ID, or label | dict | Key-value mapping with unique keys |
| Remove duplicates or test membership quickly | set | Unique values and direct membership checks |
| Count repeated values by category | dict | Keys can be categories and values can be counts |
| Preserve every repeated event | list | A set would erase duplicates |
| Return several values that should travel together | tuple | A compact fixed grouping |

## Common Mistakes

### `{}` creates an empty dict, not an empty set

```python
empty = {}
empty_set = set()

print(type(empty))
print(type(empty_set))
```

Expected output:

```text
<class 'dict'>
<class 'set'>
```

### `.append()` changes the list and returns `None`

```python
names = ["Maya"]
result = names.append("Omar")

print(names)
print(result)
```

Expected output:

```text
['Maya', 'Omar']
None
```

Do not write `names = names.append("Omar")`. That replaces your list with `None`.

### Missing dict keys raise `KeyError`

```python
scores = {"Maya": 10}

print(scores.get("Omar", 0))
```

Expected output:

```text
0
```

Use `.get()` when a key might be missing and you want a default value.

### Sets are not for predictable display order

```python
courses = {"Python Basics", "Web Basics", "Data Structures"}

print(sorted(courses))
```

Expected output:

```text
['Data Structures', 'Python Basics', 'Web Basics']
```

Sort a set when humans need stable output.

### Do not choose a list when a dict or set fits better

```python
student_ids = [101, 102, 101, 103]

unique_ids = set(student_ids)
student_names = {
    101: "Maya",
    102: "Omar",
    103: "Lina",
}

print(unique_ids)
print(student_names[102])
```

A list is fine for raw events, but a set is better for uniqueness and a dict is better for lookup by ID.

## Exercise

You are given workshop registrations as a list of tuples. Each tuple contains a student name and a course name.

```python
registrations = [
    ("Maya", "Python Basics"),
    ("Omar", "Data Structures"),
    ("Maya", "Data Structures"),
    ("Lina", "Python Basics"),
    ("Omar", "Python Basics"),
]
```

Write a program that:

1. Builds a set of unique students.
2. Builds a dict where each course maps to a list of students registered for that course.
3. Builds a tuple summary containing the number of registrations, number of unique students, and number of courses.
4. Prints the unique students, course roster, and summary.

Expected output:

```text
Students: ['Lina', 'Maya', 'Omar']
Roster:
Data Structures: ['Omar', 'Maya']
Python Basics: ['Maya', 'Lina', 'Omar']
Summary: (5, 3, 2)
```

## Worked Answer

```python
registrations = [
    ("Maya", "Python Basics"),
    ("Omar", "Data Structures"),
    ("Maya", "Data Structures"),
    ("Lina", "Python Basics"),
    ("Omar", "Python Basics"),
]

students = set()
course_roster = {}

for student, course in registrations:
    students.add(student)

    if course not in course_roster:
        course_roster[course] = []

    course_roster[course].append(student)

summary = (len(registrations), len(students), len(course_roster))

print("Students:", sorted(students))
print("Roster:")

for course in sorted(course_roster):
    print(f"{course}: {course_roster[course]}")

print("Summary:", summary)
```

Expected output:

```text
Students: ['Lina', 'Maya', 'Omar']
Roster:
Data Structures: ['Omar', 'Maya']
Python Basics: ['Maya', 'Lina', 'Omar']
Summary: (5, 3, 2)
```

Why each structure fits:

- The original registrations are a list because the input is an ordered collection of events.
- Each registration is a tuple because the student and course form one fixed pair.
- The students collection is a set because each student should appear once.
- The course roster is a dict because each course name maps to its registered students.
- The summary is a tuple because the three final counts belong together and do not need to change.

## Sources Used

- Python tutorial: Data Structures - https://docs.python.org/3/tutorial/datastructures.html
- Python standard library: `collections` - https://docs.python.org/3/library/collections.html
- Python standard library: Built-in Types - https://docs.python.org/3/library/stdtypes.html
