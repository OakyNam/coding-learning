# 02 - Commands, Files, and Directories

## Learning Goal

Use Bash to inspect your location, move through folders, create files and directories, copy and move items, remove files carefully, and ask commands for help.

By the end of this lesson, you should be able to:

- Read a command as `command options arguments`.
- Use `pwd`, `ls`, `cd`, `mkdir`, `touch`, `cp`, `mv`, `rm`, and `rmdir`.
- Work mostly with relative paths such as `bash-practice/lesson-02`.
- Quote file and directory names that contain spaces.
- Use `--help`, `help`, and `man` when you need more information.
- Check a command's exit status with `echo $?`.

## Platform Note

This is a Bash lesson.

- On Windows 10/11, run these commands inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. For Bash lessons, type `bash` first when Bash behavior matters.
- The examples use project-relative paths. Do not copy raw Windows paths into Bash commands for this lesson.

## Command Structure

Most shell commands follow this shape:

```text
command options arguments
```

The command name says what program or shell builtin to run. Options change behavior. Arguments tell the command what to act on.

Examples:

```bash
ls
ls -l
ls -la bash-practice
mkdir -p bash-practice/lesson-02
```

In `ls -la bash-practice`, `ls` is the command, `-la` is an option group, and `bash-practice` is the path argument.

Spaces separate words in Bash. If a file or directory name contains spaces, quote it:

```bash
mkdir "practice notes"
cd "practice notes"
```

## Working Directory and Paths

Bash always has a current working directory. Many commands use that directory when you pass a relative path.

Use `pwd` to print the current working directory:

```bash
pwd
```

Use `ls` with no path to list the current directory:

```bash
ls
```

A relative path starts from where you are now:

```bash
bash-practice/lesson-02
```

An absolute path starts from the filesystem root. You will see absolute paths in command output, but the practice work in this lesson uses relative paths so it works across WSL, Git Bash, macOS, and Linux.

Special path names:

- `.` means the current directory.
- `..` means the parent directory.
- `~` is expanded by the shell to your home directory.

Examples:

```bash
ls .
cd ..
cd ~
```

## Inspecting Files and Directories

Use `ls` to inspect before you change anything.

```bash
ls
ls -a
ls -l
ls -la
```

- `ls` lists visible files and directories.
- `ls -a` includes hidden names that start with `.`.
- `ls -l` uses a long format with details such as permissions, size, and modification time.
- `ls -la` combines both options.

Output can vary by operating system, terminal width, locale, and file metadata. Focus on the names first.

## Moving Around

Use `cd` to change directories.

```bash
cd bash-practice
cd ..
cd ~
cd -
```

`cd -` returns to the previous directory. `cd` is a Bash builtin, so `help cd` is often more useful than `cd --help`.

## Creating Directories and Files

Use `mkdir` to create a directory:

```bash
mkdir bash-practice
```

Use `mkdir -p` to create a nested path and avoid errors when the directory already exists:

```bash
mkdir -p bash-practice/lesson-02/examples
```

Use `touch` to create an empty file:

```bash
touch notes.txt
```

If the file already exists, `touch` updates its timestamp instead of erasing its contents.

## Copying and Moving

Use `cp` to copy files:

```bash
cp notes.txt notes-copy.txt
cp notes.txt examples/
```

Use `cp -r` to copy a directory and its contents:

```bash
cp -r examples examples-backup
```

Use `mv` to rename or move files:

```bash
mv draft.txt notes-draft.txt
mv notes-draft.txt examples/
```

`cp` and `mv` can overwrite destinations. Inspect with `pwd` and `ls` before running them.

## Removing Safely

Use `rm` to remove files:

```bash
rm draft.txt
```

`rm` normally does not move files to Trash or Recycle Bin. Treat removal as permanent.

Use `rm -i` to ask before removing a file:

```bash
rm -i draft.txt
```

Use `rmdir` for empty directories:

```bash
rmdir old-folder
```

`rmdir` refuses to remove a directory that still contains files. That refusal is helpful while you are learning.

Use `rm -r` only when you intentionally want to remove a directory and everything inside it:

```bash
rm -r old-folder
```

Before recursive removal, run:

```bash
pwd
ls -la old-folder
```

Avoid `rm -rf` in beginner practice. It removes recursively and skips many safety prompts.

## Getting Help

Many external commands support `--help`:

```bash
ls --help
mkdir --help
```

Bash builtins use `help`:

```bash
help cd
```

Manual pages provide more detail when available:

```bash
man ls
man rm
```

Press `q` to leave `man`. Some minimal Git Bash installations may not include full manual pages, so start with `--help` and `help cd`.

## Exit Status Basics

Commands report success or failure with an exit status. In Bash, `$?` stores the previous command's status.

```bash
ls
echo $?

ls missing-folder
echo $?
```

`0` means success. A nonzero number means failure. This matters later when scripts make decisions.

## Common Mistakes

- Running commands in the wrong directory. Use `pwd` and `ls` first.
- Forgetting quotes around names with spaces.
- Thinking `rm` uses Trash or Recycle Bin.
- Using `rm -r` on the wrong directory.
- Confusing rename and move; `mv` does both.
- Expecting `touch` to erase file contents.
- Forgetting hidden files when using plain `ls`.
- Running Bash commands in PowerShell by accident.

## Exercise

Create and organize a small project folder using relative paths.

Build this structure:

```text
bash-practice/
  lesson-02/
    inbox/
      draft.txt
      meeting notes.txt
    archive/
    README.txt
```

Then:

1. Print your current directory.
2. List the lesson folder in long format.
3. Copy `README.txt` into `archive/` as `README-copy.txt`.
4. Rename `inbox/draft.txt` to `inbox/first-draft.txt`.
5. Move `meeting notes.txt` from `inbox/` to `archive/`.
6. Remove only `inbox/first-draft.txt` using an interactive prompt.
7. Remove `inbox/` only after it is empty.
8. Confirm `archive/` and `README.txt` still exist.
9. Run one help command for a command used in the exercise.

## Worked Answer

Create the practice structure:

```bash
mkdir -p bash-practice/lesson-02/inbox bash-practice/lesson-02/archive
touch bash-practice/lesson-02/README.txt
touch bash-practice/lesson-02/inbox/draft.txt
touch "bash-practice/lesson-02/inbox/meeting notes.txt"
```

Inspect your location and the lesson folder:

```bash
pwd
ls -l bash-practice/lesson-02
```

Copy, rename, and move files:

```bash
cp bash-practice/lesson-02/README.txt bash-practice/lesson-02/archive/README-copy.txt
mv bash-practice/lesson-02/inbox/draft.txt bash-practice/lesson-02/inbox/first-draft.txt
mv "bash-practice/lesson-02/inbox/meeting notes.txt" bash-practice/lesson-02/archive/
```

Inspect before removing:

```bash
ls -l bash-practice/lesson-02/inbox
```

Remove one file interactively:

```bash
rm -i bash-practice/lesson-02/inbox/first-draft.txt
```

Type `y` only after you have checked that the path is the file you intended to remove.

Remove the now-empty `inbox` directory:

```bash
rmdir bash-practice/lesson-02/inbox
```

Confirm the remaining structure:

```bash
ls -l bash-practice/lesson-02
ls -l bash-practice/lesson-02/archive
```

Expected final shape:

```text
bash-practice/
  lesson-02/
    archive/
      README-copy.txt
      meeting notes.txt
    README.txt
```

Ask for help:

```bash
ls --help
help cd
```

Optional exit status check:

```bash
ls bash-practice/lesson-02/archive
echo $?

ls bash-practice/lesson-02/inbox
echo $?
```

After `rmdir`, the second `ls` should fail because `inbox` no longer exists. The next `echo $?` should print a nonzero status.

## Sources Used

- GNU Bash Manual, shell operation: https://www.gnu.org/software/bash/manual/html_node/Shell-Operation.html
- GNU Bash Manual, quoting: https://www.gnu.org/software/bash/manual/html_node/Quoting.html
- GNU Bash Manual, Bash builtins: https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html
- GNU Coreutils Manual: https://www.gnu.org/software/coreutils/manual/coreutils.html
- Microsoft WSL install documentation: https://learn.microsoft.com/en-us/windows/wsl/install
- Git for Windows download page: https://git-scm.com/downloads/win
- Apple Terminal default shell documentation: https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
