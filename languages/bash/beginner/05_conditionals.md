# 05 - Conditionals

## Learning Goal

Use Bash `if` statements to choose actions based on command success, strings, integers, and files.

## Core Idea

In Bash, an `if` statement does not check a special Boolean value. It runs a command or test and looks at that command's exit status.

- Status `0` means true, success, or yes.
- Any nonzero status means false, failure, or no.

That can feel backward if you come from some other languages, where `0` often means false. In shell scripts, `0` means the command succeeded.

You can see the last command's exit status with `$?`:

```bash
grep -q "root" /etc/passwd
echo "$?"
```

If `grep` found a match, it exits with status `0`. If it did not find a match, it exits with status `1`.

## Basic `if` Syntax

A basic `if` runs a command after `if`. If that command succeeds, Bash runs the commands between `then` and `fi`.

```bash
if command; then
  echo "The command succeeded."
fi
```

Add `else` when you want a fallback:

```bash
if command; then
  echo "The command succeeded."
else
  echo "The command failed."
fi
```

Use `elif` when there are several branches:

```bash
if first_test; then
  echo "First test passed."
elif second_test; then
  echo "Second test passed."
else
  echo "No tests passed."
fi
```

`fi` is `if` written backward. It closes the `if` block.

## Checking Command Success

Because `if` checks exit status, you can use normal commands directly.

```bash
if mkdir reports; then
  echo "Created reports directory."
else
  echo "Could not create reports directory."
fi
```

This works because `mkdir reports` exits with `0` when it creates the directory. It exits nonzero if it cannot create it, for example if `reports` already exists or permissions do not allow it.

Another common pattern is `grep -q`, where `-q` means quiet:

```bash
if grep -q "ERROR" app.log; then
  echo "The log contains an error."
else
  echo "No error found."
fi
```

The script does not need to read printed output from `grep`. It only needs to know whether `grep` succeeded.

## Using `[ ]` and `test`

The command `[` is another name for the `test` command. It checks expressions such as strings, numbers, and files.

These two examples mean the same thing:

```bash
if test "$name" = "Ada"; then
  echo "Hello, Ada."
fi
```

```bash
if [ "$name" = "Ada" ]; then
  echo "Hello, Ada."
fi
```

Spaces matter. The closing `]` is a separate argument, so it needs spaces around it.

Correct:

```bash
if [ "$name" = "Ada" ]; then
  echo "match"
fi
```

Incorrect:

```bash
if ["$name" = "Ada"]; then
  echo "match"
fi
```

When using `[ ]`, quote variables unless you have a specific reason not to:

```bash
if [ "$name" = "Ada Lovelace" ]; then
  echo "full name matched"
fi
```

Quoting protects values that are empty or contain spaces.

## Using `[[ ]]`

Bash also has `[[ ]]`, a Bash-specific conditional syntax. It is often safer and friendlier for beginner expressions because variables inside it do not undergo word splitting or pathname expansion.

```bash
if [[ $name = "Ada Lovelace" ]]; then
  echo "full name matched"
fi
```

`[[ ]]` is not POSIX `sh`. Use it when your script starts with Bash:

```bash
#!/usr/bin/env bash
```

If you need a script to run in plain `sh`, use `test` or `[ ]` instead.

## String Checks

Use string checks when you are comparing text.

```bash
name="Ada"

if [[ $name = "Ada" ]]; then
  echo "The name is Ada."
fi
```

Common string operators:

| Check | Meaning |
| --- | --- |
| `"$a" = "$b"` | strings are equal |
| `"$a" != "$b"` | strings are not equal |
| `-z "$a"` | string is empty |
| `-n "$a"` | string is not empty |

Examples:

```bash
if [[ -z $name ]]; then
  echo "Name is missing."
elif [[ $name = "Ada" ]]; then
  echo "Welcome, Ada."
else
  echo "Welcome, $name."
fi
```

With `[ ]`, keep the quotes:

```bash
if [ -n "$name" ]; then
  echo "Name was provided."
fi
```

## Integer Checks

Use integer checks when you are comparing whole numbers.

```bash
count=3

if [[ $count -gt 0 ]]; then
  echo "There are items to process."
fi
```

Common integer operators:

| Check | Meaning |
| --- | --- |
| `$a -eq $b` | equal |
| `$a -ne $b` | not equal |
| `$a -lt $b` | less than |
| `$a -le $b` | less than or equal |
| `$a -gt $b` | greater than |
| `$a -ge $b` | greater than or equal |

Example:

```bash
score=82

if [[ $score -ge 90 ]]; then
  echo "A"
elif [[ $score -ge 80 ]]; then
  echo "B"
elif [[ $score -ge 70 ]]; then
  echo "C"
else
  echo "Keep practicing."
fi
```

Do not use `>` and `<` for beginner integer checks. In `[ ]`, `>` can be treated as shell redirection unless you quote or escape it. The integer operators are clearer.

## File Checks

File checks ask questions about paths.

```bash
path="notes.txt"

if [[ -f $path ]]; then
  echo "$path is a regular file."
fi
```

Common file checks:

| Check | Meaning |
| --- | --- |
| `-e "$path"` | path exists |
| `-f "$path"` | path is a regular file |
| `-d "$path"` | path is a directory |
| `-r "$path"` | path is readable |
| `-w "$path"` | path is writable |
| `-x "$path"` | path is executable or searchable |
| `-s "$path"` | file exists and is not empty |

Example:

```bash
path="script.sh"

if [[ -f $path && -x $path ]]; then
  echo "$path is a runnable file."
elif [[ -f $path ]]; then
  echo "$path is a file, but it is not executable."
else
  echo "$path is not a regular file."
fi
```

## Combining Conditions

You can combine conditions lightly with `&&`, `||`, and `!`.

- `&&` means and.
- `||` means or.
- `!` means not.

Examples:

```bash
if [[ -f $path && -r $path ]]; then
  echo "The file exists and is readable."
fi
```

```bash
if [[ $answer = "yes" || $answer = "y" ]]; then
  echo "Continuing."
fi
```

```bash
if [[ ! -e $path ]]; then
  echo "That path does not exist."
fi
```

You can also combine commands directly:

```bash
if mkdir backup && cp notes.txt backup/; then
  echo "Backup created."
else
  echo "Backup failed."
fi
```

## Common Mistakes

Missing spaces around `[`, `]`, or operators:

```bash
# Wrong
if [$name = Ada]; then
  echo "match"
fi

# Right
if [ "$name" = "Ada" ]; then
  echo "match"
fi
```

Forgetting `then` or `fi`:

```bash
if [[ -f notes.txt ]]; then
  echo "notes.txt exists"
fi
```

Using string operators for numbers, or numeric operators for strings:

```bash
# String comparison
if [[ $word = "10" ]]; then
  echo "The text is 10."
fi

# Integer comparison
if [[ $count -eq 10 ]]; then
  echo "The number is 10."
fi
```

Leaving variables unquoted with `[ ]`:

```bash
# Risky with [ ] if name is empty or has spaces
if [ $name = "Ada Lovelace" ]; then
  echo "match"
fi

# Safer
if [ "$name" = "Ada Lovelace" ]; then
  echo "match"
fi
```

Confusing assignment and comparison:

```bash
name="Ada"          # assignment
if [[ $name = Ada ]]; then  # comparison
  echo "match"
fi
```

Confusing `>` with numeric comparison:

```bash
# Prefer this for integers
if [[ $count -gt 10 ]]; then
  echo "more than 10"
fi
```

Forgetting that true is status `0`:

```bash
if grep -q "ready" status.txt; then
  echo "grep found the text, so this branch runs."
else
  echo "grep did not find the text."
fi
```

## Exercise

Create a script named `check_path.sh`.

It should:

1. Accept one path as an argument.
2. Print usage and exit nonzero if the argument is missing.
3. Print whether the path is a directory.
4. Print whether the path is a regular file.
5. For regular files, print whether it is empty or non-empty.
6. Print a separate message for paths that exist but are neither regular files nor directories.
7. Print a separate message for paths that do not exist.

Example runs:

```bash
bash check_path.sh
bash check_path.sh .
bash check_path.sh notes.txt
bash check_path.sh missing.txt
```

## Worked Answer

```bash
#!/usr/bin/env bash

if [[ $# -ne 1 ]]; then
  echo "Usage: $0 PATH"
  exit 1
fi

path=$1

if [[ -d $path ]]; then
  echo "$path is a directory."
elif [[ -f $path ]]; then
  echo "$path is a regular file."

  if [[ -s $path ]]; then
    echo "$path is non-empty."
  else
    echo "$path is empty."
  fi
elif [[ -e $path ]]; then
  echo "$path exists, but it is not a regular file or directory."
else
  echo "$path does not exist."
fi
```

Try creating a file and an empty file to test both file branches:

```bash
echo "hello" > notes.txt
touch empty.txt

bash check_path.sh notes.txt
bash check_path.sh empty.txt
```

## Next Step

Next, practice reading `if` statements in scripts you already use. Look for the command or test after `if`, then ask: what exit status makes this branch run?

After that, continue to the next beginner Bash lesson.

## Sources Used

- GNU Bash Reference Manual, Conditional Constructs: https://www.gnu.org/software/bash/manual/html_node/Conditional-Constructs.html
- GNU Bash Reference Manual, Bash Conditional Expressions: https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html
- Linux manual page for `bash`, exit status and conditional behavior: https://man7.org/linux/man-pages/man1/bash.1.html
