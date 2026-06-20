# 05 - Practice Project: Student Club Tracker

## Learning Goal

Build a small Python program that stores, searches, groups, counts, and summarizes signup records using multiple data structures.

By the end of this project, you should be able to choose a useful structure for each job instead of keeping every value in one flat list.

## What You Will Practice

- `list`: keep the signup records in order.
- `dict`: store each signup record with named fields like `student`, `club`, and `present`.
- `set`: collect unique clubs and unique student names.
- `tuple`: make simple summary rows such as `("Chess", 3)`.
- `Counter` or manual counting: count how many signups each club received.

## Project Scenario

A school has a club fair. Students can sign up for one or more clubs, and some students are marked present because they checked in at the table.

Your job is to write a tracker that answers practical questions:

- How many signup records are there?
- Which clubs received signups?
- Which clubs had the most signups?
- Which students joined more than one club?
- Which students were marked present?

## Starter Signup Data

Use this data in your program:

```python
signups = [
    {"student": "Ava", "club": "Robotics", "present": True},
    {"student": "Ben", "club": "Art", "present": False},
    {"student": "Mia", "club": "Robotics", "present": True},
    {"student": "Ava", "club": "Chess", "present": True},
    {"student": "Noah", "club": "Drama", "present": False},
    {"student": "Mia", "club": "Chess", "present": True},
    {"student": "Liam", "club": "Art", "present": True},
    {"student": "Zoe", "club": "Robotics", "present": False},
    {"student": "Liam", "club": "Drama", "present": True},
    {"student": "Sara", "club": "Chess", "present": False},
]
```

## Requirements

Your program should print:

1. Total number of signup records.
2. Unique club names in alphabetical order.
3. Signup counts per club, highest count first.
4. Students who signed up for more than one club.
5. Students who are marked present in at least one signup record.

## Build Steps

1. Store the starter records in a list of dictionaries.
2. Use `len(signups)` to count total signup records.
3. Build a set of club names, then sort it for display.
4. Count club names with `Counter`, or use a manual dictionary loop.
5. Convert the club counts into tuple summary rows like `("Chess", 3)`.
6. Build a dictionary that maps each student to a set of clubs.
7. Filter that dictionary to find students with more than one club.
8. Build a set of students whose `present` value is `True`.
9. Print the results in a readable format.

## Common Mistakes

- Counting unique students when the requirement asks for total signup records.
- Forgetting that one student can appear in more than one record.
- Printing a set directly and expecting alphabetical order. Sort it first.
- Using a list to collect unique values, then accidentally adding duplicates.
- Sorting counts alphabetically only, instead of sorting by highest count first.
- Treating `"True"` as a string instead of using the Boolean value `True`.

## Exercise

Create a file named `student_club_tracker.py` and write a program that uses the starter data to meet all five requirements.

Try to solve it in small checks:

1. Print only the total signup count.
2. Add unique clubs.
3. Add club counts.
4. Add students in more than one club.
5. Add present students.

When your output matches the expected output below, add one new signup record of your own and predict how the results should change before running the program again.

## Complete Worked Answer

```python
from collections import Counter


signups = [
    {"student": "Ava", "club": "Robotics", "present": True},
    {"student": "Ben", "club": "Art", "present": False},
    {"student": "Mia", "club": "Robotics", "present": True},
    {"student": "Ava", "club": "Chess", "present": True},
    {"student": "Noah", "club": "Drama", "present": False},
    {"student": "Mia", "club": "Chess", "present": True},
    {"student": "Liam", "club": "Art", "present": True},
    {"student": "Zoe", "club": "Robotics", "present": False},
    {"student": "Liam", "club": "Drama", "present": True},
    {"student": "Sara", "club": "Chess", "present": False},
]


total_signups = len(signups)

unique_clubs = sorted({signup["club"] for signup in signups})

club_counter = Counter(signup["club"] for signup in signups)

# Manual counting gives the same result as Counter and is useful to understand.
manual_club_counts = {}
for signup in signups:
    club = signup["club"]
    manual_club_counts[club] = manual_club_counts.get(club, 0) + 1

assert dict(club_counter) == manual_club_counts

club_summary_rows = sorted(
    club_counter.items(),
    key=lambda row: (-row[1], row[0]),
)

student_to_clubs = {}
for signup in signups:
    student = signup["student"]
    club = signup["club"]

    if student not in student_to_clubs:
        student_to_clubs[student] = set()

    student_to_clubs[student].add(club)

students_in_more_than_one_club = sorted(
    student
    for student, clubs in student_to_clubs.items()
    if len(clubs) > 1
)

present_students = sorted(
    {signup["student"] for signup in signups if signup["present"]}
)

print(f"Total signups: {total_signups}")
print(f"Unique clubs: {', '.join(unique_clubs)}")
print("Signups per club:")

for club, count in club_summary_rows:
    print(f"- {club}: {count}")

print(
    "Students in more than one club: "
    + ", ".join(students_in_more_than_one_club)
)
print("Students marked present: " + ", ".join(present_students))
```

## Expected Output

```text
Total signups: 10
Unique clubs: Art, Chess, Drama, Robotics
Signups per club:
- Chess: 3
- Robotics: 3
- Art: 2
- Drama: 2
Students in more than one club: Ava, Liam, Mia
Students marked present: Ava, Liam, Mia
```

## Reflection Prompts

- Which structure made it easiest to remove duplicates?
- Why is a list of dictionaries better here than several separate lists?
- What does the tuple row `("Chess", 3)` represent?
- Where did sorting make the output easier to read?
- When would you choose manual counting instead of `Counter`?

## Sources Used

- Python documentation: [Data Structures](https://docs.python.org/3/tutorial/datastructures.html)
- Python documentation: [`collections.Counter`](https://docs.python.org/3/library/collections.html#collections.Counter)
- Python for Everybody: [Chapter 8: Lists](https://www.py4e.com/html3/08-lists), [Chapter 9: Dictionaries](https://www.py4e.com/html3/09-dictionaries), and [Chapter 10: Tuples](https://www.py4e.com/html3/10-tuples)
