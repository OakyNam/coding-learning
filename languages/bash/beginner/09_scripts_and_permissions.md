# 09 - Scripts and Permissions

## Learning Goal

Write a small Bash script, pass it arguments, understand how it exits, and use file permissions to run it both with `bash script.sh` and directly as `./script.sh`.

## Why It Matters

A shell script is a plain text file full of shell commands. Scripts let you save a command sequence, give it inputs, check for errors, and run it again later. Permissions decide whether the operating system will let you read, change, or execute that file.

These two ideas meet quickly: a script can be valid Bash, but `./script.sh` still fails if the file is not executable.

## What A Shell Script Is

A Bash script is a file that Bash can read and execute line by line.

Create a file named `hello.sh`:

```bash
#!/usr/bin/env bash

echo "Hello from $0"
echo "First argument: $1"
echo "Argument count: $#"
```

The first line is the shebang:

```bash
#!/usr/bin/env bash
```

When you run a script directly, such as `./hello.sh`, the operating system uses the shebang to find the interpreter. `env` looks up `bash` in your environment, which is more portable than hard-coding a path such as `/bin/bash`.

The shebang must be the first line of the file. If there is a blank line, comment, or invisible character before it, direct execution may not use it correctly.

## Running A Script Two Ways

You can run the same script in two common ways:

```bash
bash hello.sh Ada
```

or:

```bash
./hello.sh Ada
```

These are not exactly the same.

`bash hello.sh Ada` starts Bash and tells Bash to read `hello.sh`. The file needs to be readable, but it does not need execute permission.

`./hello.sh Ada` asks the operating system to execute the file. The file must have execute permission, and the shebang tells the operating system which interpreter to start.

The `./` matters because the current directory is usually not searched as a command location. Without it, `hello.sh` usually means "find a command named `hello.sh` somewhere in `PATH`."

## Script Arguments

Bash gives scripts special parameters:

```bash
#!/usr/bin/env bash

echo "Script name: $0"
echo "First argument: $1"
echo "Number of arguments: $#"

echo "All arguments:"
for arg in "$@"; do
  echo "- $arg"
done
```

Important parameters:

- `$0` is the script name or path used to start the script.
- `$1` is the first argument. `$2` is the second argument, and so on.
- `$#` is the number of arguments.
- `"$@"` expands to all arguments, preserving each argument as its own value.

Quote argument expansions unless you have a specific reason not to. `"$1"` preserves spaces in a path like `My Notes/file.txt`. Unquoted `$1` can split into multiple words and can also expand wildcard characters.

## Usage Checks And Exit Status

Commands finish with an exit status. By convention, `0` means success and a nonzero value means some kind of failure.

A script can stop with `exit`:

```bash
#!/usr/bin/env bash

if [ "$#" -ne 1 ]; then
  echo "Usage: $0 NAME" >&2
  exit 1
fi

echo "Hello, $1"
exit 0
```

The `>&2` sends the usage message to standard error. That is common for errors and diagnostics.

You can inspect the most recent exit status with:

```bash
echo "$?"
```

Example:

```bash
$ bash greet.sh
Usage: greet.sh NAME
$ echo "$?"
1
```

## Reading `ls -l` Permissions

Run:

```bash
ls -l hello.sh
```

You might see:

```text
-rw-r--r-- 1 learner staff 95 Jun 20 10:00 hello.sh
```

The first field is the permission mode:

```text
-rw-r--r--
```

Read it as:

```text
type owner group other
-    rw-   r--   r--
```

The first character is the file type:

- `-` means a regular file.
- `d` means a directory.
- `l` means a symbolic link.

The next nine characters are permissions for three classes:

- Owner, also called user: the account that owns the file.
- Group: users in the file's group.
- Other: everyone else.

Each class has three permission positions:

- `r` means read.
- `w` means write.
- `x` means execute.
- `-` means that permission is absent.

For regular files:

- `r` means you can read the file contents.
- `w` means you can modify the file contents.
- `x` means you can execute the file as a program or script.

For directories:

- `r` means you can list names in the directory.
- `w` means you can create, remove, or rename entries in the directory, usually only when `x` is also present.
- `x` means you can search or enter the directory and access entries by name.

Directory execute permission is often called search permission because it controls whether you can traverse the directory path.

## Adding Execute Permission

To let the owner run a script directly:

```bash
chmod u+x hello.sh
```

Then inspect it:

```bash
ls -l hello.sh
```

The mode changes shape from something like:

```text
-rw-r--r--
```

to:

```text
-rwxr--r--
```

Only the owner execute bit was added.

Avoid using `chmod 777` as a beginner default. It grants read, write, and execute permission to owner, group, and everyone else. That is rarely the smallest useful permission, and it can accidentally let other users modify or run files they should not control.

Prefer the smallest change that solves the problem, such as:

```bash
chmod u+x script.sh
```

## Common Mistakes

- Missing execute permission: `bash script.sh` may work while `./script.sh` says `Permission denied`.
- Forgetting `./`: `script.sh` may fail with `command not found` even when the file exists in the current directory.
- Putting anything before the shebang: `#!/usr/bin/env bash` must be the first line for direct execution.
- Leaving arguments unquoted: use `"$1"` and `"$@"` so paths with spaces stay intact.
- Saving with Windows line endings: a script with CRLF line endings may fail with errors involving `$'\r'` or `bad interpreter`.
- Assuming `bash script.sh` and `./script.sh` fail for the same reason: direct execution depends on execute permission and the shebang; `bash script.sh` mostly depends on Bash being able to read the file.

## Exercise

Create a script named `path_report.sh`.

Requirements:

1. Start with `#!/usr/bin/env bash`.
2. Accept exactly one path argument.
3. If the argument count is wrong, print `Usage: path_report.sh PATH` to standard error and exit with status `1`.
4. If the path does not exist, print `Missing path: PATH` to standard error and exit with status `2`.
5. Print whether the path is a file, directory, or other type.
6. Print whether the path is readable, writable, and executable.
7. For directories, also make it clear that execute means searchable.
8. Run it first with `bash path_report.sh PATH`.
9. Add owner execute permission with `chmod u+x path_report.sh`.
10. Inspect permissions with `ls -l path_report.sh`.
11. Run it directly with `./path_report.sh PATH`.

Test with at least one file and one directory.

## Worked Answer

```bash
#!/usr/bin/env bash

if [ "$#" -ne 1 ]; then
  echo "Usage: path_report.sh PATH" >&2
  exit 1
fi

path=$1

if [ ! -e "$path" ]; then
  echo "Missing path: $path" >&2
  exit 2
fi

if [ -f "$path" ]; then
  echo "Type: file"
elif [ -d "$path" ]; then
  echo "Type: directory"
else
  echo "Type: other"
fi

if [ -r "$path" ]; then
  echo "Readable: yes"
else
  echo "Readable: no"
fi

if [ -w "$path" ]; then
  echo "Writable: yes"
else
  echo "Writable: no"
fi

if [ -x "$path" ]; then
  if [ -d "$path" ]; then
    echo "Executable/searchable: yes"
  else
    echo "Executable: yes"
  fi
else
  if [ -d "$path" ]; then
    echo "Executable/searchable: no"
  else
    echo "Executable: no"
  fi
fi

exit 0
```

Run it with Bash:

```bash
bash path_report.sh path_report.sh
```

Expected output for a readable and writable script file before adding execute permission:

```text
Type: file
Readable: yes
Writable: yes
Executable: no
```

Check usage failure:

```bash
bash path_report.sh
echo "$?"
```

Expected output:

```text
Usage: path_report.sh PATH
1
```

Check missing path failure:

```bash
bash path_report.sh does-not-exist
echo "$?"
```

Expected output:

```text
Missing path: does-not-exist
2
```

Add execute permission for the owner:

```bash
chmod u+x path_report.sh
ls -l path_report.sh
```

The permission field should change shape like this:

```text
Before: -rw-r--r--
After:  -rwxr--r--
```

Your full `ls -l` output includes owner, group, size, and date too, so it may look like:

```text
-rwxr--r-- 1 learner staff 650 Jun 20 10:15 path_report.sh
```

Now run it directly:

```bash
./path_report.sh .
```

Expected output for a readable, writable, searchable current directory:

```text
Type: directory
Readable: yes
Writable: yes
Executable/searchable: yes
```

Your exact yes/no answers depend on your system permissions.

## Next Step

Next, learn how `PATH` lets you run reusable commands without typing `./` or a full path every time.

## Sources Used

- GNU Bash Manual: Shell Scripts.
- GNU Bash Manual: Special Parameters.
- GNU Bash Manual: Exit Status.
- GNU Coreutils Manual: File Permissions.
- GNU Coreutils Manual: Structure of File Modes.
- GNU Coreutils Manual: Symbolic Modes.
- GNU Coreutils Manual: Numeric Modes.
- Linux manual page `chmod(1)` from man7.org.
- Linux manual page `execve(2)` from man7.org.
