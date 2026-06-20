# 06 - Loops

## Learning Goal

Learn how to repeat work in Bash with `for`, `while`, `until`, arithmetic loops, `break`, and `continue`, while avoiding the file-name and input-reading mistakes that make shell loops fragile.

## Why Loops Matter

A loop runs the same commands more than once. In Bash, loops are often used to handle command-line arguments, files that match a pattern, lines from a file, retry checks, and numbered steps.

Bash loops all have the same basic idea:

1. Choose what controls the loop.
2. Put the repeated commands between `do` and `done`.
3. Make sure values that may contain spaces are quoted.

## For Loops Over Words

The beginner-friendly `for` loop has this shape:

```bash
for name in words; do
  commands
done
```

On each pass through the loop, Bash assigns the next word to `name`.

```bash
for tool in bash grep sed; do
  printf 'Tool: %s\n' "$tool"
done
```

Output:

```text
Tool: bash
Tool: grep
Tool: sed
```

Always quote the variable when you use it:

```bash
printf 'Tool: %s\n' "$tool"
```

Without quotes, Bash can split the value again and can also treat parts of it as glob patterns.

## Looping Over Script Arguments

`"$@"` means "all command-line arguments, kept as separate arguments." It is the safe way to loop over arguments.

```bash
#!/usr/bin/env bash

for arg in "$@"; do
  printf 'Argument: %s\n' "$arg"
done
```

If you run:

```bash
bash show_args.sh alpha "two words" gamma
```

The loop sees three arguments, not four:

```text
Argument: alpha
Argument: two words
Argument: gamma
```

## Glob Loops Over Files

Use globs when you want files in a directory:

```bash
for file in *.txt; do
  [[ -e "$file" ]] || continue
  printf 'Text file: %s\n' "$file"
done
```

The guard matters. If no `.txt` files exist, Bash normally leaves the unmatched pattern as the literal string `*.txt`. The `[[ -e "$file" ]] || continue` line skips that no-match case.

Do not write this:

```bash
for file in $(ls *.txt); do
  printf '%s\n' "$file"
done
```

`$(ls ...)` output is text. File names can contain spaces, tabs, and newlines, so parsing `ls` output breaks on real file names.

Also avoid this:

```bash
for file in $(find . -name '*.txt'); do
  printf '%s\n' "$file"
done
```

Use `find -exec ... {} \;` for simple actions, or use null-delimited output with `while read` for more advanced cases.

```bash
find . -name '*.txt' -exec printf 'Text file: %s\n' {} \;
```

## While Loops And Exit Status

A `while` loop runs as long as its test command returns exit status `0`, which means success.

```bash
count=1

while [[ "$count" -le 3 ]]; do
  printf 'Count: %s\n' "$count"
  ((count++))
done
```

Output:

```text
Count: 1
Count: 2
Count: 3
```

The command after `while` can be any command, not only `[[ ... ]]`. This loop keeps asking until `grep` finds the exact word `ready`:

```bash
while ! grep -qx 'ready' status.txt; do
  printf 'Waiting for ready...\n'
  sleep 1
done
```

Be careful with infinite loops. If the condition never becomes false, the loop never ends. When you intentionally write an infinite loop, include a clear stopping path such as `break`, a signal handler, or a maximum retry count.

## Reading A File Line By Line

Use `while IFS= read -r line` to read a file safely.

```bash
while IFS= read -r line || [[ -n "$line" ]]; do
  printf 'Line: %s\n' "$line"
done < "$file"
```

Why this shape matters:

- `IFS=` keeps leading and trailing whitespace.
- `read -r` prevents backslashes from being treated as escapes.
- `|| [[ -n "$line" ]]` still processes the last line if the file does not end with a newline.
- `< "$file"` redirects the file into the loop without using a pipeline.

Avoid piping into a `while` loop when the loop needs to change variables for later use:

```bash
count=0

printf '%s\n' one two | while IFS= read -r line; do
  ((count++))
done

printf 'Count: %s\n' "$count"
```

In many Bash setups, the loop on the right side of a pipeline runs in a subshell, so changes to `count` disappear when the loop ends. Prefer input redirection:

```bash
count=0

while IFS= read -r line; do
  ((count++))
done < words.txt

printf 'Count: %s\n' "$count"
```

## Until Loops

An `until` loop is the opposite of `while`: it runs as long as the test command fails.

```bash
until [[ -f report.txt ]]; do
  printf 'Waiting for report.txt\n'
  sleep 1
done
```

Use `until` when the sentence "until this becomes true" reads more clearly than "while this is not true."

## Arithmetic For Loops

Bash also has a C-style arithmetic loop:

```bash
for ((i = 1; i <= 5; i++)); do
  printf 'Number: %s\n' "$i"
done
```

This is useful when you need a counter. The three parts are:

1. Start: `i = 1`
2. Keep going while true: `i <= 5`
3. Update after each pass: `i++`

## Break And Continue

Use `break` to leave a loop early.

```bash
for word in build test STOP deploy; do
  if [[ "$word" == "STOP" ]]; then
    break
  fi
  printf '%s\n' "$word"
done
```

Output:

```text
build
test
```

Use `continue` to skip the rest of the current pass and move to the next item.

```bash
for word in build skip test; do
  if [[ "$word" == "skip" ]]; then
    continue
  fi
  printf '%s\n' "$word"
done
```

Output:

```text
build
test
```

## Common Mistakes

- Writing `for file in $(ls)`. Use globs, such as `for file in *; do ... done`, and quote `"$file"`.
- Writing `for file in $(find ...)`. Use `find -exec`, or use null-delimited `find -print0` with a matching reader when you need a loop.
- Forgetting quotes around variables: use `"$file"`, `"$line"`, and `"$arg"` unless you have a specific reason not to.
- Forgetting `do` or `done`. Every `for`, `while`, and `until` loop needs both.
- Creating an accidental infinite loop by never changing the value that controls the condition.
- Expecting variables changed inside a piped `while` loop to be available after the loop. Use input redirection instead when you need the updated value later.

## Exercise

Create a script named `task_report.sh`.

The script should:

1. Accept one task file path as its only argument.
2. Validate that the argument was supplied.
3. Validate that the path is a readable regular file.
4. Read the file line by line safely.
5. Ignore blank lines.
6. Ignore lines whose first non-space character is `#`.
7. Stop reading when the line is exactly `STOP`.
8. Number each printed task.
9. Print the total number of tasks at the end.

Use this sample file:

```text
# release checklist
update changelog

run tests
  # this comment is indented
tag release
STOP
deploy
```

Expected output:

```text
1. update changelog
2. run tests
3. tag release
Total: 3
```

## Worked Answer

```bash
#!/usr/bin/env bash

if [[ "$#" -ne 1 ]]; then
  printf 'Usage: %s TASK_FILE\n' "$0" >&2
  exit 1
fi

task_file=$1

if [[ ! -f "$task_file" || ! -r "$task_file" ]]; then
  printf 'Error: %s is not a readable regular file\n' "$task_file" >&2
  exit 1
fi

count=0

while IFS= read -r line || [[ -n "$line" ]]; do
  if [[ "$line" == "STOP" ]]; then
    break
  fi

  if [[ "$line" =~ ^[[:space:]]*$ ]]; then
    continue
  fi

  if [[ "$line" =~ ^[[:space:]]*# ]]; then
    continue
  fi

  ((count++))
  printf '%d. %s\n' "$count" "$line"
done < "$task_file"

printf 'Total: %d\n' "$count"
```

Run it like this:

```bash
chmod +x task_report.sh
./task_report.sh tasks.txt
```

The `break` stops at `STOP`, so `deploy` is never printed. The two `continue` commands skip blank lines and comments without ending the whole loop.

## Next Step

Next, learn arrays so you can store several values in one variable and loop over them with `"${array[@]}"`.

## Sources Used

- GNU Bash Manual: [Looping Constructs](https://www.gnu.org/software/bash/manual/html_node/Looping-Constructs.html)
- GNU Bash Manual: [Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html)
- ShellCheck Wiki: [SC2045 - Iterating over ls output is fragile](https://www.shellcheck.net/wiki/SC2045)
- ShellCheck Wiki: [SC2044 - For loops over find output are fragile](https://www.shellcheck.net/wiki/SC2044)
- Greg's Wiki: [BashFAQ/001 - Reading a file line by line](https://mywiki.wooledge.org/BashFAQ/001)
