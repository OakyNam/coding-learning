# 04 - Data Structures In C

## Learning Goal

Build data structures in C from arrays, `struct`, pointers, and heap allocation. By the end of this lesson, you should be able to:

- Implement a small dynamic collection in C.
- Explain why arrays are contiguous but linked structures connect nodes with pointers.
- Use `malloc`, `calloc`, `realloc`, and `free` safely.
- Trace ownership: who allocates, who frees, and when.

## Why It Matters

C has arrays and `struct`, but it does not provide built-in `list`, `vector`, or `map` types, and it does not have a garbage collector. When a program needs a collection that grows at run time, you usually build the storage policy yourself or use a project-specific library.

Many real C APIs expose memory buffers, nodes, tables, and callback-based sorting or searching. Good data-structure code is mostly clear invariants plus disciplined memory management: every successful allocation needs a cleanup path, and every pointer needs an owner.

## Platform Notes

Create a scratch directory for this lesson and save the worked answer as `task_list.c`.

On Windows 10/11 PowerShell, these commands assume GCC or Clang is installed and available on `PATH`:

```powershell
gcc -std=c17 -Wall -Wextra -pedantic .\task_list.c -o .\task_list.exe
.\task_list.exe
```

On macOS Apple Silicon with `zsh`, use Apple Clang:

```bash
clang -std=c17 -Wall -Wextra -pedantic ./task_list.c -o ./task_list
./task_list
```

If you are using MSVC from a Visual Studio Developer PowerShell, use MSVC's warning and language-version options instead of GCC-style flags:

```powershell
cl /std:c17 /W4 task_list.c
.\task_list.exe
```

Warnings are part of the lesson. Treat pointer conversion warnings, possible uninitialized values, and signed/unsigned comparison warnings as bugs to investigate.

## Core Concepts

A `struct` groups fields into one record. This `Task` record stores all fields that describe one task:

```c
typedef struct {
    int id;
    char title[64];
    int done;
} Task;
```

An array stores elements contiguously. If `items` points at the first `Task`, then `items[0]`, `items[1]`, and `items[2]` live next to each other in one block of memory. That gives direct indexing: `items[index]` is an O(1) operation when `index` is valid.

A dynamic array keeps three facts together:

```c
typedef struct {
    Task *items;
    size_t length;
    size_t capacity;
} TaskList;
```

- `items` points to the heap buffer.
- `length` is the number of initialized elements.
- `capacity` is the number of elements the current buffer can hold.

The dynamic-array invariant is:

- `0 <= length <= capacity`.
- `items == NULL` only when `capacity` is zero or allocation failed.

Use `malloc` when you need an uninitialized block, `calloc` when you need a zero-initialized block, `realloc` when resizing an existing allocation, and `free` exactly once for each live allocation you own. Check the result of every allocation call before storing through the returned pointer.

A linked list stores nodes separately. Each node points to the next node:

```c
typedef struct Node {
    Task task;
    struct Node *next;
} Node;
```

Traversal starts at `head` and follows `next` pointers until `NULL`. The linked-list invariant is:

- `head == NULL` means the list is empty.
- The final node has `next == NULL`.

## Complexity Snapshot

| Operation | Typical Cost |
| --- | --- |
| Array index | O(1) |
| Dynamic array append | Amortized O(1) |
| Dynamic array insert/delete in the middle | O(n) |
| Linked-list prepend | O(1) |
| Linked-list search | O(n) |
| Linked-list append without a tail pointer | O(n) |

Binary search over a sorted array is O(log n) conceptually. The C standard library provides `bsearch`, but avoid claiming a specific implementation complexity from the C standard itself.

## Dynamic Array Flow

```mermaid
flowchart TD
    A[User adds task] --> B[task_list_push]
    B --> C{length == capacity?}
    C -- yes --> D[realloc larger items buffer]
    D --> E{allocation succeeded?}
    E -- no --> F[return failure, keep old list valid]
    E -- yes --> G[store new pointer and capacity]
    C -- no --> H[write task at items[length]]
    G --> H
    H --> I[increment length]
    I --> J[User prints/searches tasks]
    J --> K[task_list_free releases heap memory]
```

## Main Example: A TaskList Dynamic Array

The important part of a growable array is not the assignment into `items[length]`. It is the growth path. `realloc` can move a block, and it can fail. If you assign directly back into `list->items`, a failed growth loses the only pointer to the old allocation:

```text
list->items = realloc(list->items, new_capacity * sizeof list->items[0]);
```

Use a temporary pointer instead. If `realloc` fails, the old allocation remains valid and still belongs to the list.

```c
static int task_list_push(TaskList *list, Task task) {
    if (list->length == list->capacity) {
        size_t new_capacity = list->capacity == 0 ? 4 : list->capacity * 2;

        if (new_capacity < list->capacity ||
            new_capacity > (size_t)-1 / sizeof list->items[0]) {
            return 0;
        }

        Task *grown = realloc(list->items, new_capacity * sizeof *grown);

        if (grown == NULL) {
            return 0;
        }

        list->items = grown;
        list->capacity = new_capacity;
    }

    list->items[list->length] = task;
    list->length++;
    return 1;
}
```

Ownership rule: `task_list_push` may allocate or grow the `items` buffer, but the `TaskList` owns that buffer. The caller must eventually call `task_list_free` for every initialized list.

```c
static void task_list_init(TaskList *list) {
    list->items = NULL;
    list->length = 0;
    list->capacity = 0;
}

static Task *task_list_find(TaskList *list, int id) {
    for (size_t i = 0; i < list->length; i++) {
        if (list->items[i].id == id) {
            return &list->items[i];
        }
    }

    return NULL;
}

static void task_list_free(TaskList *list) {
    free(list->items);
    list->items = NULL;
    list->length = 0;
    list->capacity = 0;
}
```

After `free`, do not read through the old pointer. Setting `items` to `NULL` prevents accidental reuse through the `TaskList` object.

## Linked-List Comparison

A linked list has a different shape. It does not grow one contiguous buffer. Each node is allocated separately, and each node links to the next one.

```c
typedef struct Node {
    Task task;
    struct Node *next;
} Node;

static Node *node_prepend(Node *head, Task task) {
    Node *node = malloc(sizeof *node);

    if (node == NULL) {
        return head;
    }

    node->task = task;
    node->next = head;
    return node;
}

static void node_print_all(const Node *head) {
    for (const Node *current = head; current != NULL; current = current->next) {
        printf("%d: %s\n", current->task.id, current->task.title);
    }
}

static void node_free_all(Node *head) {
    while (head != NULL) {
        Node *next = head->next;
        free(head);
        head = next;
    }
}
```

This is a comparison, not the project for this lesson. Notice the tradeoff: prepend is O(1), but finding the last node or searching by id requires traversal unless you store extra pointers or indexes.

## Standard-Library Connection

`<stdlib.h>` provides generic array tools such as `qsort` and `bsearch`. They work with raw element sizes and comparator callbacks, so the compiler cannot check as much for you as it can with typed functions.

```c
static int compare_tasks_by_id(const void *left, const void *right) {
    const Task *left_task = left;
    const Task *right_task = right;

    if (left_task->id < right_task->id) {
        return -1;
    }

    if (left_task->id > right_task->id) {
        return 1;
    }

    return 0;
}
```

Avoid this comparator shortcut:

```text
return left_task->id - right_task->id;
```

If that subtraction overflows a signed `int`, the behavior is undefined. Use explicit comparisons.

## Common Mistakes

- Forgetting the invariant and allowing `length` to grow beyond `capacity`.
- Assigning `realloc` directly to the only live pointer and leaking the old allocation on failure.
- Returning a pointer into a dynamic array, then calling `task_list_push` again and assuming the old pointer still points into the current buffer.
- Freeing only the linked-list head and leaking the rest of the nodes.
- Traversing a linked list with `current = current->next` after freeing `current`.
- Forgetting that `bsearch` only works correctly when the array is sorted according to the same comparator.
- Writing a comparator with subtraction and risking signed overflow.

## Exercise

Build a small command-line task inventory using a dynamic array.

Requirements:

- Store tasks with `id`, `title`, and `done`.
- Start with an empty `TaskList`.
- Append at least five tasks.
- Print all tasks.
- Mark one task complete by `id`.
- Search for one existing and one missing task.
- Free all heap memory before exit.
- Compile with warnings enabled.

Optional extension: sort tasks by `id` with `qsort`, then search with `bsearch`. If you do this, sort before calling `bsearch`, cast from `const void *` in the comparator, and compare without subtraction.

## Worked Answer

Save this as `task_list.c`:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int id;
    char title[64];
    int done;
} Task;

typedef struct {
    Task *items;
    size_t length;
    size_t capacity;
} TaskList;

static void task_list_init(TaskList *list) {
    list->items = NULL;
    list->length = 0;
    list->capacity = 0;
}

static int task_make(Task *task, int id, const char *title) {
    int written = snprintf(task->title, sizeof task->title, "%s", title);

    if (written < 0 || (size_t)written >= sizeof task->title) {
        return 0;
    }

    task->id = id;
    task->done = 0;
    return 1;
}

static int task_list_push(TaskList *list, Task task) {
    if (list->length == list->capacity) {
        size_t new_capacity = list->capacity == 0 ? 4 : list->capacity * 2;

        if (new_capacity < list->capacity) {
            return 0;
        }

        if (new_capacity > (size_t)-1 / sizeof list->items[0]) {
            return 0;
        }

        Task *grown = realloc(list->items, new_capacity * sizeof *grown);

        if (grown == NULL) {
            return 0;
        }

        list->items = grown;
        list->capacity = new_capacity;
    }

    list->items[list->length] = task;
    list->length++;
    return 1;
}

static Task *task_list_find(TaskList *list, int id) {
    for (size_t i = 0; i < list->length; i++) {
        if (list->items[i].id == id) {
            return &list->items[i];
        }
    }

    return NULL;
}

static void task_list_print(const TaskList *list) {
    for (size_t i = 0; i < list->length; i++) {
        const Task *task = &list->items[i];
        printf("[%c] %d: %s\n", task->done ? 'x' : ' ', task->id, task->title);
    }
}

static void task_list_free(TaskList *list) {
    free(list->items);
    list->items = NULL;
    list->length = 0;
    list->capacity = 0;
}

static int add_task(TaskList *list, int id, const char *title) {
    Task task;

    if (!task_make(&task, id, title)) {
        return 0;
    }

    return task_list_push(list, task);
}

int main(void) {
    TaskList tasks;
    task_list_init(&tasks);

    if (!add_task(&tasks, 101, "Map input files") ||
        !add_task(&tasks, 102, "Parse task records") ||
        !add_task(&tasks, 103, "Check memory paths") ||
        !add_task(&tasks, 104, "Print inventory") ||
        !add_task(&tasks, 105, "Release storage")) {
        fprintf(stderr, "Could not build task list.\n");
        task_list_free(&tasks);
        return 1;
    }

    printf("Initial tasks:\n");
    task_list_print(&tasks);

    Task *task = task_list_find(&tasks, 103);
    if (task != NULL) {
        task->done = 1;
    }

    printf("\nAfter completing task 103:\n");
    task_list_print(&tasks);

    task = task_list_find(&tasks, 104);
    if (task != NULL) {
        printf("\nFound task %d: %s\n", task->id, task->title);
    }

    task = task_list_find(&tasks, 999);
    if (task == NULL) {
        printf("Task 999 was not found.\n");
    }

    task_list_free(&tasks);
    return 0;
}
```

Compile and run on Windows 10/11 PowerShell with GCC or Clang on `PATH`:

```powershell
gcc -std=c17 -Wall -Wextra -pedantic .\task_list.c -o .\task_list.exe
.\task_list.exe
```

Compile and run on macOS Apple Silicon with `zsh`:

```bash
clang -std=c17 -Wall -Wextra -pedantic ./task_list.c -o ./task_list
./task_list
```

Expected output:

```text
Initial tasks:
[ ] 101: Map input files
[ ] 102: Parse task records
[ ] 103: Check memory paths
[ ] 104: Print inventory
[ ] 105: Release storage

After completing task 103:
[ ] 101: Map input files
[ ] 102: Parse task records
[x] 103: Check memory paths
[ ] 104: Print inventory
[ ] 105: Release storage

Found task 104: Print inventory
Task 999 was not found.
```

## Memory Checking

AddressSanitizer is a good first check for leaks and invalid memory access when your compiler supports it.

Windows PowerShell with GCC or Clang-style options:

```powershell
gcc -std=c17 -Wall -Wextra -pedantic -fsanitize=address -g .\task_list.c -o .\task_list_asan.exe
$env:ASAN_OPTIONS = "detect_leaks=1"
.\task_list_asan.exe
```

macOS Apple Silicon with `zsh`:

```bash
clang -std=c17 -Wall -Wextra -pedantic -fsanitize=address -g ./task_list.c -o ./task_list_asan
ASAN_OPTIONS=detect_leaks=1 ./task_list_asan
```

Do not require Valgrind for macOS Apple Silicon for this lesson. AddressSanitizer is the preferred memory-checking path here.

## Next Step

Continue with `05_concurrency_basics.md`. Data-structure invariants matter even more once multiple parts of a program can observe or modify shared state.

## Sources Used

- ISO C working draft N1570, especially library memory management in `7.22.3`, `qsort` in `7.22.5.2`, and `bsearch` in `7.22.5.1`: https://www.iso-9899.info/n1570.html
- cppreference, C `struct`: https://en.cppreference.com/c/language/struct
- cppreference, `malloc`: https://www.cppreference.com/c/memory/malloc
- SEI CERT C MEM31-C, free dynamically allocated memory when no longer needed: https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/memory-management-mem/mem31-c/
- SEI CERT C MEM34-C, only free memory allocated dynamically: https://cmu-sei.github.io/secure-coding-standards/sei-cert-c-coding-standard/rules/memory-management-mem/mem34-c/
- cppreference, `qsort`: https://en.cppreference.com/c/algorithm/qsort
- cppreference, `bsearch`: https://en.cppreference.com/c/algorithm/bsearch
- GCC C dialect options: https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html
- GCC warning options: https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html
- Clang AddressSanitizer documentation: https://clang.llvm.org/docs/AddressSanitizer.html
- Microsoft `/std` language version documentation: https://learn.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version?view=msvc-170
