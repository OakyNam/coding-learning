# 03 - Practical Data Structure Patterns

## Learning Goal

Use the right Python data structure pattern for everyday collection work: lookup, counting, grouping, deduplication, sorting, and queue-style processing.

By the end, you should be able to look at a small dataset and choose between `list`, `tuple`, `dict`, `set`, `Counter`, `defaultdict(list)`, `deque`, and `sorted(..., key=...)` without guessing.

## Why It Matters

Most real programs spend time reshaping records:

- Find one record by ID.
- Remove duplicate values.
- Count how many times each value appears.
- Group records by a field.
- Sort records for display without destroying the original order.
- Process work in first-in, first-out order.

These patterns are small, but they show up everywhere: support tickets, shopping carts, API responses, log files, student records, and task queues.

## Decision Table

| Need | Use | Why |
| --- | --- | --- |
| Keep ordered records | `list` | Preserves order and is easy to loop over. |
| Store one fixed record | `tuple` | Good for small values that belong together and should not change. |
| Find a record by key | `dict` | Fast lookup by ID, name, email, or other unique key. |
| Keep only unique values | `set` | Automatically removes duplicates. |
| Count repeated values | `collections.Counter` | Designed for frequency counts. |
| Group many values under one key | `collections.defaultdict(list)` | Creates each group list when first needed. |
| Process oldest item first | `collections.deque` | Efficient queue operations with `append()` and `popleft()`. |
| Return sorted records | `sorted(..., key=...)` | Builds a new sorted list and leaves the original unchanged. |

## Example Dataset

Each ticket is a `dict`. The whole dataset is a `list`. A small fixed pair like `("high", 1)` is a `tuple`.

```python
from collections import Counter, defaultdict, deque

tickets = [
    {"id": 101, "status": "open", "priority": "high", "owner": "Maya"},
    {"id": 102, "status": "closed", "priority": "low", "owner": "Noah"},
    {"id": 103, "status": "open", "priority": "medium", "owner": "Maya"},
    {"id": 104, "status": "waiting", "priority": "high", "owner": "Lina"},
    {"id": 105, "status": "open", "priority": "low", "owner": "Noah"},
]

priority_rank = ("high", "medium", "low")
```

## Pattern 1: Look Up Records With `dict`

When each record has a unique ID, build a dictionary once and look up records by ID.

```python
ticket_by_id = {ticket["id"]: ticket for ticket in tickets}

print(ticket_by_id[103])
```

Expected output:

```text
{'id': 103, 'status': 'open', 'priority': 'medium', 'owner': 'Maya'}
```

Use this when you will ask "give me ticket 103" more than once. It avoids scanning the whole list every time.

## Pattern 2: Unique Values With `set`

Use a set when duplicates do not matter.

```python
owners = {ticket["owner"] for ticket in tickets}

print(owners)
```

Possible output:

```text
{'Maya', 'Noah', 'Lina'}
```

Sets are unordered, so do not use them when display order matters.

## Pattern 3: Count With `Counter`

Use `Counter` to count repeated values.

```python
status_counts = Counter(ticket["status"] for ticket in tickets)

print(status_counts)
print(status_counts["open"])
```

Expected output:

```text
Counter({'open': 3, 'closed': 1, 'waiting': 1})
3
```

This is clearer than manually writing:

```python
counts = {}
for ticket in tickets:
    status = ticket["status"]
    counts[status] = counts.get(status, 0) + 1
```

## Pattern 4: Group With `defaultdict(list)`

Use `defaultdict(list)` when each key should collect many values.

```python
ids_by_priority = defaultdict(list)

for ticket in tickets:
    ids_by_priority[ticket["priority"]].append(ticket["id"])

print(dict(ids_by_priority))
```

Expected output:

```text
{'high': [101, 104], 'low': [102, 105], 'medium': [103]}
```

This pattern is different from `itertools.groupby()`. `groupby()` only groups neighboring records with the same key, so the data usually must be sorted first. `defaultdict(list)` is often simpler for beginner-friendly grouping across the whole dataset.

## Pattern 5: Sort Without Losing the Original

Use `sorted()` when you want a new sorted list and want to keep the original list unchanged.

```python
priority_order = {"high": 0, "medium": 1, "low": 2}

sorted_tickets = sorted(
    tickets,
    key=lambda ticket: priority_order[ticket["priority"]],
)

print([ticket["id"] for ticket in sorted_tickets])
print([ticket["id"] for ticket in tickets])
```

Expected output:

```text
[101, 104, 103, 102, 105]
[101, 102, 103, 104, 105]
```

The first list is priority-sorted. The second list proves the original arrival order is still available.

## Pattern 6: Process a FIFO Queue With `deque`

Use `deque` when you repeatedly process the oldest remaining item.

```python
open_queue = deque(ticket for ticket in tickets if ticket["status"] == "open")

while open_queue:
    ticket = open_queue.popleft()
    print(f"Process ticket {ticket['id']} for {ticket['owner']}")
```

Expected output:

```text
Process ticket 101 for Maya
Process ticket 103 for Maya
Process ticket 105 for Noah
```

`append()` adds to the right side. `popleft()` removes from the left side. Together, they make a first-in, first-out queue.

## Common Mistakes

- Using a `list` for repeated ID lookups. Build a `dict` once when lookup is the main operation.
- Expecting a `set` to keep insertion order. Sets are for uniqueness, not ordered display.
- Rewriting manual counting loops when `Counter` communicates the intent directly.
- Using a normal `dict` for grouping and forgetting to create the empty list before `.append()`.
- Sorting with `.sort()` when you still need the original order later. Prefer `sorted()` for a new list.
- Sorting priorities alphabetically. `"high"`, `"low"`, `"medium"` is not the business order; define a rank dictionary.
- Using `list.pop(0)` for queue work. It works on tiny examples, but `deque.popleft()` is the queue pattern.
- Assuming `itertools.groupby()` groups all matching values everywhere in the list. It groups consecutive matching values.

## Exercise

Use this support tickets dataset:

```python
from collections import Counter, defaultdict, deque

tickets = [
    {"id": 201, "status": "open", "priority": "medium", "owner": "Ari"},
    {"id": 202, "status": "closed", "priority": "low", "owner": "Bea"},
    {"id": 203, "status": "open", "priority": "high", "owner": "Ari"},
    {"id": 204, "status": "waiting", "priority": "high", "owner": "Chen"},
    {"id": 205, "status": "open", "priority": "low", "owner": "Bea"},
    {"id": 206, "status": "closed", "priority": "medium", "owner": "Dana"},
]
```

Write code that:

1. Counts tickets by `status`.
2. Finds the unique owners.
3. Groups ticket IDs by `priority`.
4. Sorts tickets by priority order: high, medium, low.
5. Processes only open tickets as a FIFO queue in arrival order.

Expected output:

```text
Status counts: Counter({'open': 3, 'closed': 2, 'waiting': 1})
Unique owners: ['Ari', 'Bea', 'Chen', 'Dana']
IDs by priority: {'medium': [201, 206], 'low': [202, 205], 'high': [203, 204]}
Sorted IDs: [203, 204, 201, 206, 202, 205]
Processing open ticket 201
Processing open ticket 203
Processing open ticket 205
```

## Worked Answer

```python
from collections import Counter, defaultdict, deque

tickets = [
    {"id": 201, "status": "open", "priority": "medium", "owner": "Ari"},
    {"id": 202, "status": "closed", "priority": "low", "owner": "Bea"},
    {"id": 203, "status": "open", "priority": "high", "owner": "Ari"},
    {"id": 204, "status": "waiting", "priority": "high", "owner": "Chen"},
    {"id": 205, "status": "open", "priority": "low", "owner": "Bea"},
    {"id": 206, "status": "closed", "priority": "medium", "owner": "Dana"},
]

status_counts = Counter(ticket["status"] for ticket in tickets)
print("Status counts:", status_counts)

unique_owners = sorted({ticket["owner"] for ticket in tickets})
print("Unique owners:", unique_owners)

ids_by_priority = defaultdict(list)
for ticket in tickets:
    ids_by_priority[ticket["priority"]].append(ticket["id"])
print("IDs by priority:", dict(ids_by_priority))

priority_order = {"high": 0, "medium": 1, "low": 2}
sorted_tickets = sorted(
    tickets,
    key=lambda ticket: priority_order[ticket["priority"]],
)
print("Sorted IDs:", [ticket["id"] for ticket in sorted_tickets])

open_queue = deque(ticket for ticket in tickets if ticket["status"] == "open")
while open_queue:
    ticket = open_queue.popleft()
    print(f"Processing open ticket {ticket['id']}")
```

## Sources Used

- Python tutorial: Data Structures - https://docs.python.org/3/tutorial/datastructures.html
- Python Standard Library: `collections` - https://docs.python.org/3/library/collections.html
- Python Sorting HOWTO - https://docs.python.org/3/howto/sorting.html
- Python Standard Library: `itertools.groupby()` - https://docs.python.org/3/library/itertools.html#itertools.groupby
