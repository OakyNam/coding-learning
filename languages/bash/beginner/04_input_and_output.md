# 04 - Input and Output

## Learning Goal

Learn how Bash programs receive input and produce output. By the end, you should be able to:

- Read keyboard input with `read`.
- Print output with `echo` and `printf`.
- Redirect output and errors to files.
- Connect simple commands with pipes.

## Platform Note

This lesson uses Bash syntax.

- On Windows 10/11, run the examples inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- The examples use files in the current practice directory, not hard-coded platform paths.

## Standard Streams

Every command starts with three standard streams:

- `stdin` is standard input. It is file descriptor `0`.
- `stdout` is standard output. It is file descriptor `1`.
- `stderr` is standard error. It is file descriptor `2`.

Most commands read from `stdin`, write normal results to `stdout`, and write error messages to `stderr`.

For example:

```bash
ls
```

`ls` writes the list of files to `stdout`.

```bash
ls missing-file
```

Because `missing-file` does not exist, `ls` writes an error message to `stderr`.

This split matters because Bash can redirect `stdout` and `stderr` separately.

## Printing Output

You will often see `echo` used for quick output:

```bash
name="Amina"
echo "Hello, $name"
```

`echo` is convenient for simple messages, but it is not always predictable across shells. Some versions treat options or escape sequences differently, such as `-n` or `\n`.

For scripts, prefer `printf` when you want reliable formatting:

```bash
name="Amina"
printf 'Hello, %s\n' "$name"
```

The `%s` is a placeholder for a string. The `\n` prints a newline. Unlike `echo`, `printf` does not automatically add a newline.

More examples:

```bash
printf 'One line\n'
printf 'Name: %s\n' "Sam"
printf 'Score: %s/%s\n' "8" "10"
```

Use double quotes around variables so Bash keeps the value together:

```bash
message="hello there"
printf 'Message: %s\n' "$message"
```

## Reading Input

Use `read` to store input in a variable.

```bash
read -r -p "What is your name? " name
printf 'Hello, %s\n' "$name"
```

The options are important:

- `-p` prints a prompt before reading.
- `-r` tells Bash not to treat backslashes as special escape characters.

Without `-r`, input containing backslashes can be changed in surprising ways. For beginner scripts, `read -r` is the safer habit.

You can read more than one value, but one variable is usually easiest while learning:

```bash
read -r -p "Favorite shell? " shell_name
printf 'You chose %s\n' "$shell_name"
```

## Redirecting Output

Redirection sends a stream somewhere else, often to a file.

Use `>` to write `stdout` to a file:

```bash
printf 'first line\n' > notes.txt
```

Be careful: `>` overwrites the file if it already exists.

Use `>>` to append instead:

```bash
printf 'second line\n' >> notes.txt
```

Use `<` to read `stdin` from a file:

```bash
wc -l < notes.txt
```

This counts the number of lines in `notes.txt`. The file is connected to `wc` as input.

## Redirecting Errors

Remember: normal output is `stdout` fd `1`, and errors are `stderr` fd `2`.

This redirects only `stdout`:

```bash
ls > files.txt
```

This redirects only `stderr`:

```bash
ls missing-file 2> errors.txt
```

This sends `stdout` and `stderr` to different files:

```bash
ls missing-file . > output.txt 2> errors.txt
```

The `.` directory listing goes to `output.txt`. The missing file error goes to `errors.txt`.

## Redirecting stdout and stderr Together

Use `2>&1` to send `stderr` to the same place as `stdout`.

Ordering matters.

This sends both `stdout` and `stderr` to `all.txt`:

```bash
ls missing-file . > all.txt 2>&1
```

Read it left to right:

1. `> all.txt` sends `stdout` to `all.txt`.
2. `2>&1` sends `stderr` to wherever `stdout` is currently going.

This is different:

```bash
ls missing-file . 2>&1 > all.txt
```

Here, `stderr` is first connected to the current `stdout`, usually the terminal. Then `stdout` is sent to `all.txt`. The error still appears on the terminal.

Many Bash scripts also use this shorter form:

```bash
ls missing-file . &> all.txt
```

For learning, `> all.txt 2>&1` makes the stream ordering easier to see.

## Pipelines

A pipeline connects the `stdout` of one command to the `stdin` of the next command.

```bash
printf 'apple\nbanana\ncarrot\n' | grep 'a'
```

The `printf` command writes three lines to `stdout`. The pipe `|` sends those lines into `grep` as `stdin`. `grep 'a'` prints the lines containing `a`.

Another example:

```bash
printf 'red\nblue\ngreen\n' | grep 'e' | wc -l
```

This pipeline:

1. Prints three lines.
2. Keeps only lines containing `e`.
3. Counts the remaining lines.

A pipe passes a stream of text between commands. It does not pass a filename. If a command expects a file path argument, a pipe may not be enough.

## Common Mistakes

- Using `>` when you meant `>>`, which overwrites the file instead of appending.
- Expecting `>` to capture errors. It redirects `stdout`, not `stderr`.
- Forgetting to quote variables, such as writing `$message` instead of `"$message"`.
- Forgetting that `printf` does not add a newline unless you include `\n`.
- Using `read` without `-r`, which can change backslashes in user input.
- Thinking a pipe passes files. A pipe passes a stream from `stdout` to `stdin`.
- Getting redirection order wrong with `2>&1`.

## Exercise

Create a script named `guestbook.sh`.

The script should:

1. Ask for a guest name.
2. Ask for a message.
3. Append one line to `guestbook.txt` in this format:

```text
Name: Message
```

4. Print a saved message to the terminal.
5. Show guestbook entries containing the letter `a` by using a pipeline.

Try writing it yourself before looking at the worked answer.

## Worked Answer

Create the script:

```bash
cat > guestbook.sh <<'EOF'
#!/usr/bin/env bash

read -r -p "Name: " name
read -r -p "Message: " message

printf '%s: %s\n' "$name" "$message" >> guestbook.txt

printf 'Saved entry for %s\n' "$name"
printf 'Entries containing a:\n'
cat guestbook.txt | grep 'a'
EOF
```

Inspect it before running:

```bash
cat guestbook.sh
```

Run it with Bash:

```bash
bash guestbook.sh
```

The script contents should be:

```bash
#!/usr/bin/env bash

read -r -p "Name: " name
read -r -p "Message: " message

printf '%s: %s\n' "$name" "$message" >> guestbook.txt

printf 'Saved entry for %s\n' "$name"
printf 'Entries containing a:\n'
cat guestbook.txt | grep 'a'
```

The append line uses `>>` so old guestbook entries are not erased.

The final line uses a pipeline: `cat guestbook.txt` writes the file contents to `stdout`, and `grep 'a'` reads that stream from `stdin`.

An optional run might look like this:

```text
$ bash guestbook.sh
Name: Amina
Message: Bash is fun
Saved entry for Amina
Entries containing a:
Amina: Bash is fun
```

You can inspect the file afterward:

```bash
cat guestbook.txt
```

## Next Step

Practice changing where streams go. Try sending normal output to one file and errors to another:

```bash
ls missing-file . > output.txt 2> errors.txt
```

Then continue to the next beginner Bash lesson.

## Sources Used

- GNU Bash Manual, redirections: https://www.gnu.org/software/bash/manual/html_node/Redirections.html
- GNU Bash Manual, pipelines: https://www.gnu.org/software/bash/manual/html_node/Pipelines.html
- GNU Bash Manual, Bash builtins including `read`: https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html
- POSIX `printf` utility documentation: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/printf.html
- Microsoft WSL documentation: https://learn.microsoft.com/windows/wsl/
- Git for Windows: https://gitforwindows.org/
- Apple Terminal default shell documentation: https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
- GNU Coreutils documentation for `cat`, `ls`, and `wc`.
