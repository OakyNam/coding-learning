# 08 - Functions

## Learning Goal

Write small Bash functions that accept arguments, keep temporary variables local, report success or failure with exit status, and can be reused from a script.

## Why It Matters

Functions let you give a name to a repeated task. Instead of copying the same test, message, or loop many times, you define it once and call it like a command.

Bash functions are especially useful in scripts because they make the main flow easier to read:

```bash
print_header
show_path_status "$path"
```

## Function Syntax

The common Bash function form is:

```bash
name() {
  commands
}
```

The function name comes first, then `()`, then a command group inside braces. The spaces around the braces matter. Write `{` after a space and put `}` where Bash can read it as its own token.

Good:

```bash
say_hello() {
  echo "Hello"
}
```

Bad:

```bash
say_hello(){echo "Hello"}
```

That bad version is hard for Bash to parse because `{`, `echo`, and `}` are crowded together. If you write a one-line function, use semicolons and spaces:

```bash
say_hello() { echo "Hello"; }
```

## Calling Functions

Call a function by writing its name like any other command:

```bash
say_hello() {
  echo "Hello"
}

say_hello
```

Bash reads and runs scripts from top to bottom, so define a function before the first line that calls it:

```bash
greet_user

greet_user() {
  echo "Hello"
}
```

That fails because `greet_user` does not exist yet when Bash reaches the call.

## Function Arguments

Inside a function, arguments are available as positional parameters:

- `$1` is the first argument.
- `$2` is the second argument.
- `$#` is the number of arguments.
- `"$@"` expands to all arguments, preserving each argument as a separate value.

Example:

```bash
greet_user() {
  local name="$1"
  local greeting="$2"

  echo "$greeting, $name"
}

greet_user "Ada" "Hello"
greet_user "Grace Hopper" "Welcome"
```

Output:

```text
Hello, Ada
Welcome, Grace Hopper
```

Quote arguments when passing them and quote variables when using them. Without quotes, a value such as `Grace Hopper` may be split into two words.

## Looping Over All Arguments

Use `"$@"` when a function should handle every argument it receives:

```bash
print_tasks() {
  local task

  echo "Task count: $#"

  for task in "$@"; do
    echo "- $task"
  done
}

print_tasks "write notes" "run tests" "commit changes"
```

Output:

```text
Task count: 3
- write notes
- run tests
- commit changes
```

Prefer `"$@"` over `$*` for loops. Quoted `"$@"` keeps each original argument separate. `$*` can merge arguments together in ways that surprise you.

## Local Variables

By default, variables assigned inside a Bash function are global to the shell. Use `local` for temporary variables that belong only to the function call:

```bash
greet_user() {
  local name="$1"
  echo "Hello, $name"
}
```

Using `local` prevents one function from accidentally changing a variable used somewhere else in the script. Bash uses dynamic scoping for local variables, which means a local variable can also be visible to functions called from that function. For beginner scripts, the practical rule is simple: use `local` for function-only variables and pass important data as arguments.

## Exit Status And Return

Bash functions act like commands. They finish with an exit status:

- `0` means success.
- Any non-zero number means failure.

Use `return n` to set a function's status:

```bash
file_exists() {
  local path="$1"

  if [[ -e "$path" ]]; then
    return 0
  else
    return 1
  fi
}

if file_exists "notes.txt"; then
  echo "notes.txt exists"
else
  echo "notes.txt is missing"
fi
```

`return` is not how functions send back string data. This is wrong:

```bash
get_name() {
  return "Ada"
}
```

`return` expects a numeric status. To produce text, print it with `echo` or `printf`:

```bash
get_name() {
  echo "Ada"
}

name="$(get_name)"
echo "Name: $name"
```

## Common Mistakes

- Calling a function before it has been defined.
- Writing bad brace spacing, such as `name(){echo "hi"}`.
- Expecting `return "text"` to send a string back.
- Forgetting to quote arguments and variables, especially `"$1"` and `"$@"`.
- Using global variables when function arguments would be clearer.
- Forgetting `local` for temporary variables inside a function.
- Using `$*` in loops instead of quoted `"$@"`.

## Exercise

Create a script named `check_paths.sh`.

Requirements:

1. Define a `print_header` function that prints a short title.
2. Define a `show_path_status` function that accepts one path.
3. `show_path_status` should print whether the path is a file, directory, or missing.
4. `show_path_status` should return `0` when the path exists and `1` when it is missing.
5. Define a `main` function.
6. `main` should print usage and return `1` when no arguments are provided.
7. `main` should loop over `"$@"` and call `show_path_status` for each path.
8. Call `main "$@"` at the end of the script.

Try it with paths that exist and paths that do not exist.

## Worked Answer

```bash
#!/usr/bin/env bash

print_header() {
  echo "Path status report"
  echo "------------------"
}

show_path_status() {
  local path="$1"

  if [[ -f "$path" ]]; then
    echo "file: $path"
    return 0
  elif [[ -d "$path" ]]; then
    echo "directory: $path"
    return 0
  else
    echo "missing: $path"
    return 1
  fi
}

main() {
  local path
  local missing_count=0

  if [[ $# -eq 0 ]]; then
    echo "Usage: $0 PATH..."
    return 1
  fi

  print_header

  for path in "$@"; do
    if ! show_path_status "$path"; then
      missing_count=$((missing_count + 1))
    fi
  done

  if [[ "$missing_count" -gt 0 ]]; then
    return 1
  fi

  return 0
}

main "$@"
```

Run it:

```bash
chmod +x check_paths.sh
./check_paths.sh check_paths.sh . missing.txt
echo "$?"
```

Expected behavior:

```text
Path status report
------------------
file: check_paths.sh
directory: .
missing: missing.txt
1
```

If every path exists, the final status printed by `echo "$?"` should be `0`. If one or more paths are missing, it should be `1`.

## Next Step

Continue to `09_scripts_and_permissions.md` and practice turning small Bash programs into executable scripts.

## Sources Used

- GNU Bash Reference Manual: Shell Functions: https://www.gnu.org/software/bash/manual/html_node/Shell-Functions.html
- GNU Bash Reference Manual: Positional Parameters: https://www.gnu.org/software/bash/manual/html_node/Positional-Parameters.html
- GNU Bash Reference Manual: Special Parameters: https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html
- GNU Bash Reference Manual: Bourne Shell Builtins (`return`): https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html
- man7.org bash(1), including `local` and function scoping details: https://man7.org/linux/man-pages/man1/bash.1.html
