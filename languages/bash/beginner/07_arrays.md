# 07 - Arrays

## Learning Goal

Learn how to store multiple values in a Bash array, read and update elements by index, loop over every element safely, and recognize the basic shape of associative arrays.

## Platform Note

This lesson uses Bash arrays.

- On Windows 10/11, run these examples inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- Indexed arrays are the focus of this lesson. The associative array preview uses `declare -A`, which requires a modern Bash; skip that preview if your `bash --version` is too old.

## Why Arrays Matter

A normal variable stores one value:

```bash
task="write notes"
```

An array stores a list of values under one name:

```bash
tasks=("write notes" "run tests" "commit changes")
```

Arrays are useful when a script needs to work through a group of files, tasks, names, options, commands, or results.

## Indexed Arrays

The most common beginner array in Bash is an indexed array. Indexed means each value has a numeric position.

Bash array indexes start at `0`, not `1`.

```bash
tasks=("write notes" "run tests" "commit changes")

echo "${tasks[0]}"
echo "${tasks[1]}"
echo "${tasks[2]}"
```

Output:

```text
write notes
run tests
commit changes
```

Use `${array[index]}` to read one element.

```bash
colors=("red" "green" "blue")

echo "First color: ${colors[0]}"
echo "Third color: ${colors[2]}"
```

## Count Elements

Use `${#array[@]}` to get the number of elements in an array.

```bash
tasks=("write notes" "run tests" "commit changes")

echo "Task count: ${#tasks[@]}"
```

Output:

```text
Task count: 3
```

The `#` asks for a length. With `[@]`, Bash counts the elements in the array.

## Loop Over Values Safely

Use `"${array[@]}"` when you want every element as its own value.

```bash
tasks=("write notes" "run tests" "commit changes")

for task in "${tasks[@]}"; do
  echo "- [ ] $task"
done
```

Output:

```text
- [ ] write notes
- [ ] run tests
- [ ] commit changes
```

The quotes are important. `"${tasks[@]}"` preserves each array element as a separate word, even when an element contains spaces.

Without quotes, Bash may split values on spaces:

```bash
tasks=("write notes" "run tests")

for task in ${tasks[@]}; do
  echo "$task"
done
```

Output:

```text
write
notes
run
tests
```

That is usually wrong because the two tasks became four loop values.

## `"${array[@]}"` vs `"${array[*]}"`

Inside double quotes, `[@]` and `[*]` are different.

`"${array[@]}"` expands each element separately:

```bash
tasks=("write notes" "run tests")

for task in "${tasks[@]}"; do
  echo "Task: $task"
done
```

Output:

```text
Task: write notes
Task: run tests
```

`"${array[*]}"` expands the whole array as one joined string. By default, the values are joined with spaces:

```bash
tasks=("write notes" "run tests")

for task in "${tasks[*]}"; do
  echo "Task: $task"
done
```

Output:

```text
Task: write notes run tests
```

Use `"${array[@]}"` for normal loops. Use `"${array[*]}"` only when you intentionally want one combined string.

## Append and Update Elements

Append a new value with `array+=("value")`.

```bash
tasks=("write notes" "run tests")

tasks+=("push changes")

printf '%s\n' "${tasks[@]}"
```

Output:

```text
write notes
run tests
push changes
```

Update an existing element by assigning to its index:

```bash
tasks=("write notes" "run tests" "push changes")

tasks[1]="run shellcheck"

printf '%s\n' "${tasks[@]}"
```

Output:

```text
write notes
run shellcheck
push changes
```

## Loop Over Indexes

Use `${!array[@]}` to get the indexes currently assigned in the array.

```bash
tasks=("write notes" "run tests" "commit changes")

for index in "${!tasks[@]}"; do
  echo "$index: ${tasks[$index]}"
done
```

Output:

```text
0: write notes
1: run tests
2: commit changes
```

This is useful when you need both the position and the value.

## Sparse Arrays

Bash arrays can be sparse, which means indexes do not have to be continuous.

```bash
items=()
items[0]="first"
items[5]="sixth"

echo "Count: ${#items[@]}"

for index in "${!items[@]}"; do
  echo "$index: ${items[$index]}"
done
```

Output:

```text
Count: 2
0: first
5: sixth
```

There are two elements, not six. Index `5` exists, but indexes `1` through `4` were never assigned.

## Associative Arrays Preview

Indexed arrays use number indexes. Associative arrays use string keys.

You create an associative array with `declare -A`.

```bash
declare -A capitals

capitals[france]="Paris"
capitals[japan]="Tokyo"

echo "${capitals[france]}"
```

Output:

```text
Paris
```

Associative arrays are useful for lookup tables, but this lesson focuses on indexed arrays.

If `declare -A capitals` prints an error on your machine, your Bash version does not support associative arrays. Continue with indexed arrays for now.

## Common Mistakes

- Using `$array` and expecting all values. `$array` means `${array[0]}`, only element `0`.
- Forgetting that indexes start at `0`.
- Looping with `${array[@]}` instead of `"${array[@]}"`, which breaks values that contain spaces.
- Writing values with spaces without quotes, such as `tasks=(write notes)`, which creates two elements instead of one.
- Expecting Bash arrays to be multidimensional. Bash arrays are one-dimensional. If you need nested data, use a clearer format such as lines in a file, CSV, JSON with a parser, or multiple arrays.
- Using `"${array[*]}"` in a loop when you meant `"${array[@]}"`.

## Exercise

Create a file named `checklist.sh`.

Your script should:

1. Create a `tasks` array with at least three tasks.
2. Print the first task.
3. Print the number of tasks.
4. Append one more task.
5. Update one existing task.
6. Loop over all tasks and print each one as a checklist item.
7. Optionally print each task with its index.

Example checklist format:

```text
- [ ] write notes
```

## Worked Answer

Create the script:

```bash
cat > checklist.sh <<'EOF'
#!/usr/bin/env bash

tasks=("write notes" "run tests" "commit changes")

echo "First task: ${tasks[0]}"
echo "Task count: ${#tasks[@]}"

tasks+=("push changes")
tasks[1]="run shellcheck"

echo
echo "Checklist:"
for task in "${tasks[@]}"; do
  echo "- [ ] $task"
done

echo
echo "Tasks with indexes:"
for index in "${!tasks[@]}"; do
  echo "$index: ${tasks[$index]}"
done
EOF
```

Run it with Bash:

```bash
bash checklist.sh
```

The script contents should be:

```bash
#!/usr/bin/env bash

tasks=("write notes" "run tests" "commit changes")

echo "First task: ${tasks[0]}"
echo "Task count: ${#tasks[@]}"

tasks+=("push changes")
tasks[1]="run shellcheck"

echo
echo "Checklist:"
for task in "${tasks[@]}"; do
  echo "- [ ] $task"
done

echo
echo "Tasks with indexes:"
for index in "${!tasks[@]}"; do
  echo "$index: ${tasks[$index]}"
done
```

## Expected Output

```text
First task: write notes
Task count: 3

Checklist:
- [ ] write notes
- [ ] run shellcheck
- [ ] commit changes
- [ ] push changes

Tasks with indexes:
0: write notes
1: run shellcheck
2: commit changes
3: push changes
```

## Next Step

Practice with values that contain spaces, such as file names or task descriptions. Then return to this level's README and continue with the next numbered lesson.

## Sources Used

- [GNU Bash Manual: Arrays](https://www.gnu.org/software/bash/manual/html_node/Arrays.html)
- [GNU Bash Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)
- [GNU Bash Manual: Double Quotes](https://www.gnu.org/software/bash/manual/html_node/Double-Quotes.html)
- [GNU Bash Manual: Word Splitting](https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html)
- [GNU Bash Manual: Bash Builtins](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html)
- [Microsoft WSL documentation](https://learn.microsoft.com/windows/wsl/)
- [Git for Windows](https://gitforwindows.org/)
- [Apple Terminal default shell documentation](https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac)
