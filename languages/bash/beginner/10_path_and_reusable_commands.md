# 10 - PATH and Reusable Commands

## Learning Goal

Learn how Bash finds commands with `PATH`, how to inspect command lookup, and how to turn a small shell script into a reusable command.

## Why It Matters

Typing a full path like `/home/you/bin/today-note` works, but it gets old quickly. `PATH` is the shell feature that lets you type short command names such as `ls`, `python`, or your own `today-note`.

Understanding `PATH` also helps you debug confusing problems:

- "command not found"
- "permission denied"
- Bash running a different command than the one you expected
- a command working in one terminal but not another

## What PATH Is

`PATH` is an environment variable containing a list of directories. On Unix-like systems, the directories are separated with colons:

```bash
printf '%s\n' "$PATH"
```

You might see output shaped like this:

```text
/home/you/bin:/usr/local/bin:/usr/bin:/bin
```

When you type a command name with no slash in it, Bash searches those directories from left to right. The first matching executable file wins.

For example, if `PATH` is:

```text
/home/you/bin:/usr/local/bin:/usr/bin:/bin
```

and both `/home/you/bin/report` and `/usr/bin/report` exist, this runs the one in your home directory:

```bash
report
```

Bash searches `PATH` only when the command name contains no `/`.

These do use `PATH`:

```bash
ls
python3
today-note
```

These do not use `PATH`, because each command contains a slash:

```bash
./today-note
/home/you/bin/today-note
scripts/build
```

Those paths are used directly.

## Inspecting Command Lookup

Use `command -v` when you want a simple answer about what Bash would run:

```bash
command -v ls
command -v today-note
```

Example output:

```text
/usr/bin/ls
/home/you/bin/today-note
```

Use `type` when you want more detail:

```bash
type ls
type cd
type today-note
```

`type` can tell you whether a name is an executable file, shell builtin, alias, function, or keyword.

Use `type -a` when you want to see all matches Bash knows about:

```bash
type -a python3
type -a today-note
```

This is useful when the wrong command seems to run. If several matches exist, the first one shown is usually the one Bash will choose.

Bash may remember command locations in a hash table for speed. If you moved or replaced a command and Bash still seems to use old information, clear that cache:

```bash
hash -r
```

## Make A Reusable Command

A reusable command is often just a script stored in a directory that appears in `PATH`.

Create a personal command directory:

```bash
mkdir -p "$HOME/bin"
```

Create a script:

```bash
nano "$HOME/bin/hello-name"
```

Put this in the file:

```bash
#!/usr/bin/env bash

name=${1:-friend}
printf 'Hello, %s!\n' "$name"
```

The first line is the shebang. It tells the system which interpreter should run the script when the file is executed directly. `#!/usr/bin/env bash` asks `env` to find `bash` through the current environment.

Make the script executable for your user:

```bash
chmod u+x "$HOME/bin/hello-name"
```

Run it by absolute path first:

```bash
"$HOME/bin/hello-name" Sam
```

Expected output:

```text
Hello, Sam!
```

Now temporarily put `$HOME/bin` at the front of `PATH` for the current shell:

```bash
export PATH="$HOME/bin:$PATH"
```

The quotes matter. They preserve the old `PATH` exactly while building the new value.

Verify lookup:

```bash
command -v hello-name
type hello-name
```

Run the command by name:

```bash
hello-name Ada
```

Expected output:

```text
Hello, Ada!
```

## Make PATH Persistent

The temporary `export PATH=...` lasts only for the current shell. To make it available in future interactive Bash sessions, add a duplicate-safe block to your `~/.bashrc`:

```bash
case ":$PATH:" in
  *":$HOME/bin:"*) ;;
  *) export PATH="$HOME/bin:$PATH" ;;
esac
```

Then reload `~/.bashrc` in the current shell:

```bash
source "$HOME/.bashrc"
```

Or close and reopen your terminal.

The `case` block wraps `PATH` in colons so it can match complete directory entries. That avoids adding `$HOME/bin` again and again.

## PATH Safety

Small `PATH` mistakes can create confusing or unsafe behavior.

- Preserve the old value with `"$PATH"` when editing it.
- Avoid putting `.` in `PATH`. It means "the current directory", so a random local file can shadow a normal command.
- Avoid empty entries in `PATH`. Empty entries can also mean the current directory.
- Be careful when prepending a directory. Earlier directories win.
- Use unique command names for your own scripts. A name like `test`, `ls`, or `python` is likely to collide with something important.

Prefer this:

```bash
export PATH="$HOME/bin:$PATH"
```

Avoid this:

```bash
export PATH="$HOME/bin:"
```

That loses the old `PATH`.

Also avoid this:

```bash
export PATH=".:$PATH"
```

That makes the current directory win before normal system command directories.

## Troubleshooting

### command not found

Check whether the script exists:

```bash
ls -l "$HOME/bin/today-note"
```

Check whether `$HOME/bin` is in `PATH`:

```bash
printf '%s\n' "$PATH"
command -v today-note
```

If `command -v` prints nothing, Bash did not find it through `PATH`.

### permission denied

The file may not be executable:

```bash
chmod u+x "$HOME/bin/today-note"
```

Then try again:

```bash
today-note "learned PATH"
```

### The wrong command runs

See every match:

```bash
type -a today-note
```

If another directory appears first, adjust `PATH` carefully or choose a more unique command name.

### .bashrc was edited but nothing changed

Reload it:

```bash
source "$HOME/.bashrc"
```

Or open a new interactive Bash terminal. Some shells or terminal setups may read different startup files, so confirm you are actually using Bash:

```bash
printf '%s\n' "$BASH_VERSION"
```

### Bash remembers an old command location

Clear Bash's command hash table:

```bash
hash -r
```

Then inspect lookup again:

```bash
command -v today-note
type today-note
```

## Exercise

Create a reusable command named `today-note`.

Requirements:

1. Store it at `$HOME/bin/today-note`.
2. Make it executable.
3. Make it available through `PATH`.
4. It must accept note text as command arguments.
5. It must append one line to `$HOME/today-notes.txt`.
6. Each line must have this format:

```text
YYYY-MM-DD - note text
```

Example use:

```bash
today-note "practiced Bash PATH"
```

Example line added to `$HOME/today-notes.txt`:

```text
2026-06-20 - practiced Bash PATH
```

After creating it, move to another directory and verify Bash still finds it:

```bash
cd /tmp
command -v today-note
type today-note
today-note "verified from another directory"
```

## Worked Answer

Create the personal command directory:

```bash
mkdir -p "$HOME/bin"
```

Create the script:

```bash
nano "$HOME/bin/today-note"
```

Put this in the file:

```bash
#!/usr/bin/env bash

if [ "$#" -eq 0 ]; then
  printf 'Usage: today-note NOTE TEXT...\n' >&2
  exit 1
fi

note=$*
today=$(date +%F)
printf '%s - %s\n' "$today" "$note" >> "$HOME/today-notes.txt"
```

Make it executable:

```bash
chmod u+x "$HOME/bin/today-note"
```

Test it by absolute path:

```bash
"$HOME/bin/today-note" "created the command"
```

Add `$HOME/bin` to `PATH` for the current shell:

```bash
export PATH="$HOME/bin:$PATH"
```

Add the persistent version to `~/.bashrc` if it is not already there:

```bash
case ":$PATH:" in
  *":$HOME/bin:"*) ;;
  *) export PATH="$HOME/bin:$PATH" ;;
esac
```

Reload `~/.bashrc`:

```bash
source "$HOME/.bashrc"
```

Verify command lookup:

```bash
command -v today-note
type today-note
```

Expected lookup output should point to your script:

```text
/home/you/bin/today-note
today-note is /home/you/bin/today-note
```

Verify from another directory:

```bash
cd /tmp
today-note "verified from another directory"
tail -n 2 "$HOME/today-notes.txt"
```

Expected behavior:

- `today-note` runs without typing `$HOME/bin/today-note`.
- Each run appends a new dated line to `$HOME/today-notes.txt`.
- Running it with no note text prints a usage message and exits without adding a blank note.

## Common Mistakes

- Writing `export PATH="$HOME/bin"` and accidentally removing the rest of `PATH`.
- Forgetting `chmod u+x`, then getting `Permission denied`.
- Naming a script after an existing command and accidentally shadowing it.
- Testing only from `$HOME/bin`; use another directory to prove `PATH` lookup works.
- Editing `~/.bashrc` but forgetting to `source "$HOME/.bashrc"` or open a new shell.
- Assuming `./script-name` and `script-name` are the same. `./script-name` uses a relative path directly; `script-name` searches `PATH`.

## Next Step

Return to the [beginner README](README.md) and continue with the next numbered lesson.

## Sources Used

- GNU Bash Reference Manual, "Command Search and Execution": https://www.gnu.org/software/bash/manual/bash.html#Command-Search-and-Execution
- GNU Bash Reference Manual, "Bash Variables" (`PATH`): https://www.gnu.org/software/bash/manual/bash.html#Bash-Variables
- GNU Bash Reference Manual, "Bourne Shell Builtins" (`type`): https://www.gnu.org/software/bash/manual/bash.html#Bourne-Shell-Builtins
- GNU Bash Reference Manual, "Bash Builtin Commands" (`hash`): https://www.gnu.org/software/bash/manual/bash.html#Bash-Builtins
- GNU Bash Reference Manual, "Bash Startup Files": https://www.gnu.org/software/bash/manual/bash.html#Bash-Startup-Files
- Linux `execve(2)` manual page: https://man7.org/linux/man-pages/man2/execve.2.html
- GNU Coreutils manual, "`env` invocation": https://www.gnu.org/software/coreutils/manual/html_node/env-invocation.html
- GNU Coreutils manual, "File permissions": https://www.gnu.org/software/coreutils/manual/html_node/File-permissions.html
