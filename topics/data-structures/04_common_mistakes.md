# 04 - Common Data Structure Mistakes

## Learning Goal

Recognize and fix common Python mistakes with lists, dictionaries, sets, tuples, and copies.

## Why This Matters

Most data structure bugs are not syntax errors. They are logic errors caused by aliasing, mutating something in place, using the wrong kind of key, or assuming a collection behaves like another collection. These bugs can look small, but they often leak into larger programs as missing students, duplicated courses, changed originals, or confusing output.

## 1. In-Place Methods Returning `None`

Many list methods change the existing list instead of creating a new one. Methods such as `sort()`, `append()`, `extend()`, and `reverse()` return `None`.

Broken:

```python
scores = [88, 72, 95]
sorted_scores = scores.sort()

print(sorted_scores)  # None
print(scores)         # [72, 88, 95]
```

Fixed when you want to change the original list:

```python
scores = [88, 72, 95]
scores.sort()

print(scores)  # [72, 88, 95]
```

Fixed when you want a new sorted list:

```python
scores = [88, 72, 95]
sorted_scores = sorted(scores)

print(sorted_scores)  # [72, 88, 95]
print(scores)         # [88, 72, 95]
```

## 2. Shared Mutable Lists From Repetition

The `*` operator repeats references. If the repeated item is a mutable list, every row can point to the same inner list.

Broken:

```python
rosters = [[]] * 3
rosters[0].append("Maya")

print(rosters)
# [['Maya'], ['Maya'], ['Maya']]
```

Fixed:

```python
rosters = [[] for _ in range(3)]
rosters[0].append("Maya")

print(rosters)
# [['Maya'], [], []]
```

Use a comprehension when each slot needs its own independent mutable object.

## 3. Assignment vs Shallow Copy vs Deep Copy

Assignment does not copy a collection. It creates another name for the same object.

Broken:

```python
original = [["Maya", "Python"], ["Omar", "SQL"]]
backup = original

backup[0].append("Git")

print(original)
# [['Maya', 'Python', 'Git'], ['Omar', 'SQL']]
```

Fixed with a shallow copy when only the outer collection needs to be separate:

```python
original = [["Maya", "Python"], ["Omar", "SQL"]]
backup = original.copy()

backup.append(["Lina", "HTML"])

print(original)
# [['Maya', 'Python'], ['Omar', 'SQL']]
```

Important: a shallow copy still shares nested mutable objects.

```python
original = [["Maya", "Python"], ["Omar", "SQL"]]
backup = original.copy()

backup[0].append("Git")

print(original)
# [['Maya', 'Python', 'Git'], ['Omar', 'SQL']]
```

Fixed with a deep copy when nested mutable data must be independent:

```python
import copy

original = [["Maya", "Python"], ["Omar", "SQL"]]
backup = copy.deepcopy(original)

backup[0].append("Git")

print(original)
# [['Maya', 'Python'], ['Omar', 'SQL']]
print(backup)
# [['Maya', 'Python', 'Git'], ['Omar', 'SQL']]
```

## 4. Modifying Collections While Iterating

Changing the size or contents of a collection while looping over it can skip values, process values twice, or raise an error.

Broken:

```python
students = ["Maya", "", "Omar", "", "Lina"]

for student in students:
    if student == "":
        students.remove(student)

print(students)
# This can leave empty values behind in other examples.
```

Fixed by building a new list:

```python
students = ["Maya", "", "Omar", "", "Lina"]
students = [student for student in students if student != ""]

print(students)
# ['Maya', 'Omar', 'Lina']
```

Broken dictionary example:

```python
status = {"Maya": "active", "Omar": "inactive", "Lina": "active"}

for name, state in status.items():
    if state == "inactive":
        del status[name]

# RuntimeError: dictionary changed size during iteration
```

Fixed by looping over a copy of the items:

```python
status = {"Maya": "active", "Omar": "inactive", "Lina": "active"}

for name, state in list(status.items()):
    if state == "inactive":
        del status[name]

print(status)
# {'Maya': 'active', 'Lina': 'active'}
```

## 5. Dictionary `KeyError` and Invalid Keys

A dictionary raises `KeyError` when you request a missing key with square brackets.

Broken:

```python
grades = {"Maya": 95}

print(grades["Omar"])
# KeyError: 'Omar'
```

Fixed with `get()` when a default value is acceptable:

```python
grades = {"Maya": 95}

print(grades.get("Omar", "missing"))
# missing
```

Fixed with an explicit check when missing data needs special handling:

```python
grades = {"Maya": 95}

name = "Omar"
if name in grades:
    print(grades[name])
else:
    print(f"{name} has no grade yet")
```

Dictionary keys must be hashable. Mutable objects such as lists and dictionaries cannot be dictionary keys.

Broken:

```python
course_grades = {}
course = ["Python", "morning"]

course_grades[course] = 95
# TypeError: unhashable type: 'list'
```

Fixed with an immutable tuple key:

```python
course_grades = {}
course = ("Python", "morning")

course_grades[course] = 95

print(course_grades)
# {('Python', 'morning'): 95}
```

## 6. Set Misunderstandings

Sets store unique values and do not preserve duplicates. They are useful for membership tests and removing duplicates, but they are not a good choice when order or repeated values matter.

Broken:

```python
attendance = ["Maya", "Omar", "Maya", "Lina"]
unique_attendance = set(attendance)

print(unique_attendance[0])
# TypeError: 'set' object is not subscriptable
```

Fixed when you only need membership:

```python
attendance = ["Maya", "Omar", "Maya", "Lina"]
unique_attendance = set(attendance)

print("Maya" in unique_attendance)
# True
```

Fixed when you need a sorted display:

```python
attendance = ["Maya", "Omar", "Maya", "Lina"]
unique_attendance = set(attendance)

print(sorted(unique_attendance))
# ['Lina', 'Maya', 'Omar']
```

Set output order may vary, so do not write code that depends on the printed order of a set.

## 7. Tuple Shallow Immutability

A tuple cannot be resized or reassigned by index, but it can contain mutable objects. The tuple is immutable only at the top level.

Broken assumption:

```python
student_record = ("Maya", ["Python"])
student_record[1].append("SQL")

print(student_record)
# ('Maya', ['Python', 'SQL'])
```

Fixed by storing immutable nested data:

```python
student_record = ("Maya", ("Python",))

updated_record = (student_record[0], student_record[1] + ("SQL",))

print(student_record)
# ('Maya', ('Python',))
print(updated_record)
# ('Maya', ('Python', 'SQL'))
```

## 8. Mutable Default Arguments

Default argument values are created when the function is defined, not each time the function is called. A mutable default can accidentally remember data across calls.

Broken:

```python
def add_course(student, course, courses=[]):
    courses.append(course)
    return {"student": student, "courses": courses}

print(add_course("Maya", "Python"))
print(add_course("Omar", "SQL"))
# {'student': 'Maya', 'courses': ['Python']}
# {'student': 'Omar', 'courses': ['Python', 'SQL']}
```

Fixed:

```python
def add_course(student, course, courses=None):
    if courses is None:
        courses = []
    courses.append(course)
    return {"student": student, "courses": courses}

print(add_course("Maya", "Python"))
print(add_course("Omar", "SQL"))
# {'student': 'Maya', 'courses': ['Python']}
# {'student': 'Omar', 'courses': ['SQL']}
```

## Practical Debugging Checklist

- If a variable is `None`, check whether you assigned the result of an in-place method such as `sort()` or `append()`.
- If changing one list changes another list, check whether you used assignment, repetition, or a shallow copy.
- If nested data changes unexpectedly, decide whether you need `copy.deepcopy()`.
- If a loop skips values or crashes, check whether the code mutates the collection being iterated.
- If a dictionary lookup crashes, check whether the key exists before using square brackets.
- If a dictionary key raises `TypeError`, check whether the key is hashable.
- If a set loses values, remember that sets remove duplicates.
- If set output looks shuffled, remember that set display order is not something to rely on.
- If a tuple changes through a nested list, remember that tuple immutability is shallow.
- If a function remembers old values, check for mutable default arguments.

## Exercise

Fix this broken rosters/students/courses/add_course program from audit.

The program should:

- Keep each course roster independent.
- Keep each student's course list independent.
- Avoid mutating collections while iterating over them.
- Avoid `KeyError` for missing students.
- Use valid dictionary keys.
- Use a set only for unique course names.
- Avoid mutable default arguments.

Broken program:

```python
rosters = [[]] * 3
courses = ["Python", "SQL", "Git"]
students = {
    "Maya": {"courses": []},
    "Omar": {"courses": []},
}

def add_course(student_name, course, student_courses=[]):
    student_courses.append(course)
    students[student_name]["courses"] = student_courses
    return student_courses

rosters[0].append("Maya")
rosters[1].append("Omar")

add_course("Maya", "Python")
add_course("Omar", "SQL")

students["Lina"]["courses"].append("Git")

course_lookup = {}
course_lookup[["Python", "morning"]] = "Room 101"

for name, info in students.items():
    if not info["courses"]:
        del students[name]

unique_courses = set(["Python", "SQL", "Python", "Git"])

print("Rosters:", rosters)
print("Students:", students)
print("Unique courses:", unique_courses)
print("Lookup:", course_lookup)
```

Expected output:

```python
Rosters: [['Maya'], ['Omar'], []]
Students: {'Maya': {'courses': ['Python']}, 'Omar': {'courses': ['SQL']}, 'Lina': {'courses': ['Git']}}
Unique courses: {'Python', 'SQL', 'Git'}
Lookup: {('Python', 'morning'): 'Room 101'}
```

Note: set output order may vary. For example, `{'Git', 'Python', 'SQL'}` is also valid.

## Worked Answer

```python
rosters = [[] for _ in range(3)]
courses = ["Python", "SQL", "Git"]
students = {
    "Maya": {"courses": []},
    "Omar": {"courses": []},
}

def add_course(student_name, course, student_courses=None):
    if student_courses is None:
        student_courses = []

    if student_name not in students:
        students[student_name] = {"courses": []}

    student_courses.append(course)
    students[student_name]["courses"] = student_courses
    return student_courses

rosters[0].append("Maya")
rosters[1].append("Omar")

add_course("Maya", "Python")
add_course("Omar", "SQL")
add_course("Lina", "Git")

course_lookup = {}
course_lookup[("Python", "morning")] = "Room 101"

for name, info in list(students.items()):
    if not info["courses"]:
        del students[name]

unique_courses = set(["Python", "SQL", "Python", "Git"])

print("Rosters:", rosters)
print("Students:", students)
print("Unique courses:", unique_courses)
print("Lookup:", course_lookup)
```

One possible output:

```python
Rosters: [['Maya'], ['Omar'], []]
Students: {'Maya': {'courses': ['Python']}, 'Omar': {'courses': ['SQL']}, 'Lina': {'courses': ['Git']}}
Unique courses: {'Python', 'SQL', 'Git'}
Lookup: {('Python', 'morning'): 'Room 101'}
```

Set output order may vary.

## Sources Used

- Python Tutorial: Data Structures - https://docs.python.org/3/tutorial/datastructures.html
- Python Tutorial: More Control Flow Tools, including mutating while iterating and default argument values - https://docs.python.org/3/tutorial/controlflow.html
- Python Library Reference: `copy`, shallow copy, and deep copy - https://docs.python.org/3/library/copy.html
- Python Library Reference: Built-in Types - https://docs.python.org/3/library/stdtypes.html
- Python Language Reference: Function Definitions - https://docs.python.org/3/reference/compound_stmts.html#function-definitions
