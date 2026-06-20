# 01 - Setup and Shell Basics

## Learning Goal

Learn what Bash is, how to open and verify a Bash shell, how to read simple command examples, how to move around the filesystem, and how to run your first small Bash script.

## What Bash Is

Bash is a shell. A shell is a program that reads commands you type, asks the operating system to run them, and shows the result. Bash is common on Linux, available on macOS, and available on Windows through tools such as WSL or Git Bash.

You will use Bash in two main ways:

- Interactive shell: you type one command at a time.
- Script: you save commands in a file and run them together.

When you see examples in this lesson, type only the command after the prompt. Do not type the `$`.

```bash
$ pwd
```

In that example, type this:

```bash
pwd
```

## Opening Bash

On macOS:

1. Open Terminal.
2. Run `bash --version`.
3. If your default shell is not Bash, you can start Bash by typing `bash`.

On Linux:

1. Open your terminal application.
2. Run `bash --version`.
3. Most Linux systems already include Bash.

On Windows with WSL:

1. Open your installed Linux distribution, such as Ubuntu, from the Start menu.
2. You can also open PowerShell or Command Prompt and run `wsl`.
3. After WSL opens, run `bash --version`.

On Windows with Git Bash:

1. Open Git Bash from the Start menu.
2. Run `bash --version`.
3. Git Bash is useful for learning many Bash basics, but WSL is closer to a real Linux environment.

## Verify Your Shell

Run these commands:

```bash
bash --version
echo "$SHELL"
```

Expected output shape:

```text
GNU bash, version ...
/bin/bash
```

Your exact version and path may be different. On some systems, `echo "$SHELL"` may show another shell such as `/bin/zsh`, even if you are currently running Bash inside it. `bash --version` is the more direct check that Bash is installed and working.

## The Prompt

Your terminal shows a prompt before each command. It may look like this:

```text
$
```

or this:

```text
user@computer:~/projects$
```

The prompt is not the command. It is Bash saying, "I am ready." Many prompts show your user name, computer name, current directory, or whether the previous command succeeded. Prompt styles vary by system.

## Simple Commands

Try these commands one at a time:

```bash
pwd
ls
echo "Hello from Bash"
date
whoami
history
```

What they do:

- `pwd` prints the current working directory.
- `ls` lists files and directories.
- `echo` prints text.
- `date` prints the current date and time.
- `whoami` prints your current user name.
- `history` shows commands you have typed in this shell.

Expected output shape:

```text
/home/your-name
Desktop  Documents  Downloads
Hello from Bash
Sat Jun 20 ...
your-name
1  pwd
2  ls
...
```

Your filenames, date format, user name, and history numbers will be different.

## Command Shape

Most shell commands have this shape:

```text
command options arguments
```

Examples:

```bash
ls
ls -l
echo "Bash is running"
mkdir bash-practice
touch notes.txt
cat notes.txt
```

In these examples:

- `ls` is a command with no extra details.
- `-l` is an option that changes how `ls` behaves.
- `"Bash is running"` is an argument passed to `echo`.
- `bash-practice` is an argument passed to `mkdir`.

Options often begin with `-` or `--`. Arguments are the things the command works with, such as text, filenames, or directories.

## Current Directory And Paths

Bash always has a current directory. Commands like `ls`, `touch`, and `cat` use that location unless you give them a different path.

```bash
pwd
ls
cd ~
pwd
```

Important path symbols:

- `~` means your home directory.
- `/` means the filesystem root on macOS, Linux, WSL, and Git Bash.
- `.` means the current directory.
- `..` means the parent directory.

Examples:

```bash
cd ~
mkdir bash-practice
cd bash-practice
pwd
cd ..
pwd
```

On WSL, Linux paths look like `/home/your-name/project`. Windows drives are usually mounted under `/mnt`, such as `/mnt/c/Users/your-name`.

On Git Bash, Windows paths may appear in a Unix-like form, such as `/c/Users/your-name`.

## Basic Quoting

Spaces separate words in Bash. If a filename or directory name contains spaces, quote it.

This creates one directory:

```bash
mkdir "practice notes"
```

This tries to create two directories:

```bash
mkdir practice notes
```

Use double quotes around text and paths unless you have a reason not to:

```bash
echo "My shell is $SHELL"
cat "notes from today.txt"
```

Double quotes still allow variables like `$SHELL` to expand. Single quotes print most characters literally:

```bash
echo '$SHELL'
```

Expected output:

```text
$SHELL
```

## Creating And Reading Files

Create a directory and a file:

```bash
mkdir bash-practice
cd bash-practice
touch notes.txt
```

Write text into the file:

```bash
echo "I am learning Bash." > notes.txt
echo "Commands are small tools." >> notes.txt
```

Display the file:

```bash
cat notes.txt
```

Expected output:

```text
I am learning Bash.
Commands are small tools.
```

The `>` operator replaces a file with new output. The `>>` operator appends output to the end of a file.

## Finding Help

Bash gives you several ways to ask what a command is or how it works.

```bash
type cd
type ls
help cd
man ls
```

What they do:

- `type cd` tells you whether `cd` is built into Bash, an alias, a function, or an external program.
- `type ls` shows where Bash finds `ls`.
- `help cd` shows help for the Bash built-in command `cd`.
- `man ls` opens the manual page for `ls` on systems that include manual pages.

To exit `man`, press `q`.

## Your First Script

A Bash script is a text file containing Bash commands. Create a file named `where-am-i.sh`:

```bash
cat > where-am-i.sh <<'EOF'
#!/usr/bin/env bash
echo "Current directory:"
pwd
echo "Current user:"
whoami
EOF
```

Run it with Bash:

```bash
bash where-am-i.sh
```

Expected output shape:

```text
Current directory:
/home/your-name/bash-practice
Current user:
your-name
```

The first line, `#!/usr/bin/env bash`, is called a shebang. It tells Unix-like systems which program should run the script when the script is executed directly. In this beginner lesson, run the file with `bash where-am-i.sh`; that works even before you learn executable permissions.

Later, you may see scripts run like this:

```bash
./where-am-i.sh
```

If that gives a permission error, the file probably is not executable yet. That is normal. For now, use `bash where-am-i.sh`.

## Common Mistakes

- Typing the prompt: if an example shows `$ pwd`, type `pwd`, not `$ pwd`.
- Running Bash commands in PowerShell or CMD by accident: commands like `pwd` may work differently, and commands like `man ls` may not exist there. Open WSL or Git Bash for this lesson.
- Forgetting quotes around paths with spaces: use `cat "my notes.txt"`, not `cat my notes.txt`.
- Assuming `cd` prints your location: `cd` changes directory quietly. Run `pwd` after `cd` if you want to see where you are.
- Confusing `~`, `/`, `.`, and `..`: home, root, current directory, and parent directory are different places.
- Mixing Windows paths and WSL paths: in WSL, use paths like `/mnt/c/Users/name`, not `C:\Users\name`.
- Trying `./script.sh` before learning executable permissions: use `bash script.sh` first.
- Ignoring command errors: read the first error line carefully, then check spelling, paths, quotes, and whether the file exists.

## Exercise

Use Bash to do the following:

1. Go to your home directory.
2. Create a directory named `bash-practice`.
3. Move into that directory.
4. Create `notes.txt`.
5. Write two lines into `notes.txt`.
6. Display `notes.txt`.
7. Create a script named `where-am-i.sh`.
8. Run the script with `bash where-am-i.sh`.

Your script should print:

- A label for the current directory.
- The current directory.
- A label for the current user.
- The current user.

## Worked Answer

```bash
cd ~
mkdir bash-practice
cd bash-practice
touch notes.txt
echo "I am practicing Bash commands." > notes.txt
echo "I can create files and run scripts." >> notes.txt
cat notes.txt
```

Expected output:

```text
I am practicing Bash commands.
I can create files and run scripts.
```

Now create the script:

```bash
cat > where-am-i.sh <<'EOF'
#!/usr/bin/env bash
echo "Current directory:"
pwd
echo "Current user:"
whoami
EOF
```

Run it:

```bash
bash where-am-i.sh
```

Expected output shape:

```text
Current directory:
/home/your-name/bash-practice
Current user:
your-name
```

If your output shows a different home path or user name, that is expected.

## Next Step

Return to this level's README and continue with the next numbered lesson.

## Sources Used

- GNU Bash Reference Manual: https://www.gnu.org/software/bash/manual/bash.html
- GNU Coreutils Manual: https://www.gnu.org/software/coreutils/manual/coreutils.html
- Microsoft WSL basic commands: https://learn.microsoft.com/en-us/windows/wsl/basic-commands
- Git for Windows: https://gitforwindows.org/
