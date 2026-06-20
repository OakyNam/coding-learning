# 01 - Data Structures Foundations

## Learning Goal

Choose an appropriate beginner Python data structure for a small problem, explain why it fits, and use it to store, look up, update, remove, count, and iterate over data.

## What Is a Data Structure?

A data structure is a way to organize data so your program can work with it clearly and efficiently. Different structures make different jobs easier:

- Store: keep values for later.
- Look up: find a value when you need it.
- Update: change a value.
- Remove: delete a value.
- Count: know how many values there are, or how many times something appears.
- Iterate: go through values one at a time.

For example, a coffee shop program might need to remember orders in order, count how many lattes were sold, and keep a unique list of drink names. Those are related tasks, but they are not all best solved with the same structure.

## Abstract Ideas vs Python Implementations

Some data structures are abstract ideas. They describe behavior, not one specific Python type.

A stack means "last in, first out." The most recent item added is the first item removed. In Python, a list can implement a stack with `append()` and `pop()`.

A queue means "first in, first out." The oldest item added is the first item removed. In Python, `collections.deque` is a better queue implementation than a list because it is designed for fast appends and pops from both ends.

So ask two questions:

1. What behavior do I need?
2. Which Python type gives me that behavior cleanly?

## Choosing a Structure

Use a `list` when order matters and the collection can change. Lists are good for orders, task lists, search results, and steps in a process.

Use a `tuple` when the values belong together and should not change. Tuples are good for fixed records like a menu item code, name, and price.

Use a `set` when uniqueness matters more than order. Sets are good for removing duplicates or checking membership, such as whether a drink name has appeared before.

Use a `dict` when you need to connect keys to values. Dictionaries are good for counts, lookups by name or ID, settings, profiles, and grouped information.

Use a stack when you need to undo, backtrack, or process the newest item first. A Python list is often enough for this.

Use a queue or `deque` when you need to process the oldest item first. This fits waiting lines, order processing, print jobs, and message handling.

## Comparison Table

| Structure | Best for | Order | Duplicates | Common operations |
| --- | --- | --- | --- | --- |
| `list` | Changeable ordered data | Keeps insertion order | Allows duplicates | `append`, `pop`, index lookup, loop |
| `tuple` | Fixed grouped data | Keeps insertion order | Allows duplicates | index lookup, unpacking, loop |
| `set` | Unique values and membership tests | Display order is not guaranteed | Removes duplicates | `add`, `remove`, `in`, loop |
| `dict` | Key-value lookup and counting | Keeps insertion order in modern Python | Keys are unique | assign by key, `get`, `items`, loop |
| stack | Newest item first | Depends on implementation | Usually allows duplicates | push/add, pop/remove newest |
| queue / `deque` | Oldest item first | Keeps queue order | Usually allows duplicates | append, `popleft`, loop |

## Example: Coffee Orders

```python
from collections import deque

orders = ["latte", "tea", "latte", "mocha", "tea"]

menu_item = ("L1", "latte", 4.50)
unique_drinks = set()
drink_counts = {}
order_queue = deque()

for drink in orders:
    unique_drinks.add(drink)
    drink_counts[drink] = drink_counts.get(drink, 0) + 1
    order_queue.append(drink)

print("Fixed menu item:", menu_item)
print("Unique drinks:", unique_drinks)
print("Drink counts:", drink_counts)

print("Preparing orders:")
while order_queue:
    next_drink = order_queue.popleft()
    print("-", next_drink)
```

Expected output:

```text
Fixed menu item: ('L1', 'latte', 4.5)
Unique drinks: {'latte', 'tea', 'mocha'}
Drink counts: {'latte': 2, 'tea': 2, 'mocha': 1}
Preparing orders:
- latte
- tea
- latte
- mocha
- tea
```

The set display order may vary. For example, Python might print `{'tea', 'latte', 'mocha'}` instead. The important idea is that each drink appears once.

## Common Mistakes

- Using a list for everything, even when a set or dict would make the code clearer.
- Expecting a set to keep the same display order as the original data.
- Forgetting that dict keys must be unique, so assigning the same key again replaces the old value.
- Using a list as a queue with `pop(0)` for larger workloads instead of using `deque.popleft()`.
- Mutating a list while looping over it, which can skip items or make the loop harder to reason about.
- Confusing a tuple with "a list that uses parentheses." A tuple communicates that the grouped values are fixed.
- Calling methods like `list.sort()` or `list.append()` and expecting them to return a new list. Many mutating methods change the object and return `None`.

## Exercise

You are tracking drink orders for a small cafe.

Given this starter data:

```python
orders = ["tea", "latte", "tea", "espresso", "latte", "tea"]
```

Write a program that:

1. Stores the orders in a queue.
2. Builds a set of unique drinks.
3. Builds a dictionary that counts each drink.
4. Uses a tuple for one fixed menu item, such as `("E1", "espresso", 3.00)`.
5. Prints the fixed menu item, unique drinks, drink counts, and the orders as they are prepared.

## Worked Answer

```python
from collections import deque

orders = ["tea", "latte", "tea", "espresso", "latte", "tea"]

menu_item = ("E1", "espresso", 3.00)
order_queue = deque()
unique_drinks = set()
drink_counts = {}

for drink in orders:
    order_queue.append(drink)
    unique_drinks.add(drink)
    drink_counts[drink] = drink_counts.get(drink, 0) + 1

print("Fixed menu item:", menu_item)
print("Unique drinks:", unique_drinks)
print("Drink counts:", drink_counts)

print("Preparing orders:")
while order_queue:
    print("-", order_queue.popleft())
```

Expected output:

```text
Fixed menu item: ('E1', 'espresso', 3.0)
Unique drinks: {'tea', 'latte', 'espresso'}
Drink counts: {'tea': 3, 'latte': 2, 'espresso': 1}
Preparing orders:
- tea
- latte
- tea
- espresso
- latte
- tea
```

The set display order may vary, so a different order for `Unique drinks` is still correct.

## Sources Used

- Python Tutorial: Data Structures - https://docs.python.org/3/tutorial/datastructures.html
- Python `collections.deque` documentation - https://docs.python.org/3/library/collections.html#collections.deque
- Python Built-in Types documentation - https://docs.python.org/3/library/stdtypes.html
