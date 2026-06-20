# 02 - Commands, Files, and Directories

## Learning Goal

Use Bash to inspect where you are, move between directories, create files and directories, and remove them safely.

By the end of this lesson, you should be comfortable with:

- Checking your current location with `pwd`
- Listing files with `ls`
- Moving around with `cd`
- Creating directories with `mkdir`
- Creating empty files with `touch`
- Removing files and directories carefully with `rm` and `rmdir`

## Mental Model

Bash is usually working "inside" one directory at a time. That location is called the current working directory.

When you type a command like this:

```bash
ls
```

Bash lists the files in your current working directory.

When you type a command like this:

```bash
touch notes.txt
```

Bash creates or updates `notes.txt` in your current working directory.

That is why `pwd` matters. It tells you where Bash is currently focused.

```bash
pwd
```

Example output:

```text
/home/maya
```

### Paths

A path is a written location for a file or directory.

There are two common kinds of paths:

- Absolute path: starts from the root of the filesystem.
- Relative path: starts from your current working directory.

Example absolute path:

```bash
/home/maya/bash-practice/week1
```

Example relative path:

```bash
bash-practice/week1
```

If your current working directory is `/home/maya`, those two paths point to the same place.

### Special Path Shortcuts

Bash uses a few short names for common locations:

- `.` means the current directory
- `..` means the parent directory
- `~` means your home directory

Examples:

```bash
ls .
```

This lists the current directory.

```bash
cd ..
```

This moves up one directory.

```bash
cd ~
```

This moves to your home directory.

## Inspecting Files and Directories

Use `pwd` to print your current working directory:

```bash
pwd
```

Use `ls` to list the visible files and directories in the current directory:

```bash
ls
```

Use `ls -a` to include hidden files. Hidden files usually begin with a dot, such as `.bashrc` or `.gitignore`.

```bash
ls -a
```

Use `ls -l` for a longer listing with permissions, owner, size, and modification time:

```bash
ls -l
```

Use `ls path` to list a specific location:

```bash
ls bash-practice
```

You can combine options:

```bash
ls -la
```

That means "show a long listing, including hidden files."

## Moving Around

Use `cd` to change directories.

Move into a directory:

```bash
cd bash-practice
```

Move up one directory:

```bash
cd ..
```

Move to your home directory:

```bash
cd ~
```

Move back to the previous directory:

```bash
cd -
```

The `cd -` command is useful when you jump into a directory to check something and then want to return to where you were.

## Creating Directories

Use `mkdir` to create a directory:

```bash
mkdir bash-practice
```

Use `mkdir -p` to create a nested path. It creates parent directories if they do not already exist.

```bash
mkdir -p bash-practice/week1
```

This creates `bash-practice`, then creates `week1` inside it.

If `bash-practice` already exists, `mkdir -p` does not complain. That makes it friendly for setup commands.

## Creating Files

Use `touch` to create an empty file:

```bash
touch notes.txt
```

If `notes.txt` already exists, `touch` updates its modification timestamp instead of replacing the file.

You can create multiple empty files at once:

```bash
touch notes.txt draft.txt ideas.txt
```

## Removing Files and Directories Safely

Use `rm` to remove files:

```bash
rm draft.txt
```

Important: `rm` does not move files to the trash. In normal shell use, removal is immediate.

Use `rmdir` to remove an empty directory:

```bash
rmdir old-folder
```

If the directory contains files, `rmdir` refuses to remove it. That refusal is often helpful for beginners because it prevents accidental deletion of a whole folder.

Use `rm -r` to remove a directory and everything inside it:

```bash
rm -r old-folder
```

Be careful with `rm -r`. It can remove many files quickly.

Use `rm -i` to ask before each removal:

```bash
rm -i draft.txt
```

Example prompt:

```text
rm: remove regular empty file 'draft.txt'?
```

Use `rm -I` to ask once before removing many files or recursively removing a directory:

```bash
rm -I -r old-folder
```

This is less noisy than `rm -i`, but still gives you a chance to stop before a larger deletion.

## Beginner-Safe Workflow

This workflow creates a small practice area in your home directory, makes a few files, lists them, and removes one file.

Start from your home directory:

```bash
cd ~
pwd
```

Create a nested practice directory:

```bash
mkdir -p bash-practice/week1
```

Move into it:

```bash
cd bash-practice/week1
pwd
```

Create a few empty files:

```bash
touch notes.txt draft.txt commands.txt
```

List the files:

```bash
ls
```

Expected output will look similar to this:

```text
commands.txt  draft.txt  notes.txt
```

Use a long listing:

```bash
ls -l
```

Remove only `draft.txt`, asking for confirmation:

```bash
rm -i draft.txt
```

List again:

```bash
ls
```

Expected final files:

```text
commands.txt  notes.txt
```

## Common Mistakes

### Running `rm` in the wrong directory

Before removing anything, check where you are:

```bash
pwd
ls
```

If the directory is not the one you expected, stop and move to the right place first.

### Thinking `rm` uses the trash

Graphical file managers often have a trash or recycle bin. The `rm` command normally does not. Treat `rm` as permanent.

### Using `rm -r` casually

The `-r` option means recursive. It removes a directory and its contents. Use it only when you have checked the path.

Safer version:

```bash
rm -I -r folder-name
```

### Forgetting hidden dotfiles

Regular `ls` does not show hidden files:

```bash
ls
```

Use `ls -a` when you need to see files that begin with a dot:

```bash
ls -a
```

This matters in project directories because files like `.gitignore`, `.env`, or `.bashrc` can be important.

### Confusing `.` and `..`

These are different:

```bash
ls .
```

Lists the current directory.

```bash
ls ..
```

Lists the parent directory.

And:

```bash
cd .
```

Stays where you are.

```bash
cd ..
```

Moves up one directory.

### Forgetting to quote paths with spaces

If a path contains spaces, quote it:

```bash
cd "my practice folder"
```

Without quotes, Bash treats the words as separate pieces:

```bash
cd my practice folder
```

That command tries to use `my`, `practice`, and `folder` as separate arguments, which is not what you want.

### Using `rmdir` on a non-empty directory

This fails if `old-folder` contains files:

```bash
rmdir old-folder
```

That is normal. If you truly want to remove the directory and everything inside it, use a careful recursive remove:

```bash
rm -I -r old-folder
```

### Expecting `touch` to overwrite a file

This creates `notes.txt` if it does not exist:

```bash
touch notes.txt
```

But if `notes.txt` already exists, `touch` updates the file's timestamp. It does not erase the file's contents.

## Exercise

From your home directory, create this structure:

```text
bash-practice/
  lesson-02/
    notes.txt
    draft.txt
    examples/
```

Then inspect it and remove only `draft.txt`.

Requirements:

- Start from your home directory.
- Use `mkdir -p` for the directory structure.
- Use `touch` to create the files.
- Use `ls` or `ls -l` to inspect what you made.
- Use `rm -i` to remove only `draft.txt`.
- Confirm that `notes.txt` and `examples/` still exist.

## Worked Answer

Start from home:

```bash
cd ~
pwd
```

Create the directory structure:

```bash
mkdir -p bash-practice/lesson-02/examples
```

Create the files:

```bash
touch bash-practice/lesson-02/notes.txt bash-practice/lesson-02/draft.txt
```

Inspect the directory:

```bash
ls -l bash-practice/lesson-02
```

Expected output will look similar to this:

```text
drwxr-xr-x  2 maya  staff   64 Jun 20 10:00 examples
-rw-r--r--  1 maya  staff    0 Jun 20 10:00 draft.txt
-rw-r--r--  1 maya  staff    0 Jun 20 10:00 notes.txt
```

Your username, group, date, and spacing may be different. The important part is that you see `examples`, `draft.txt`, and `notes.txt`.

Move into the lesson directory:

```bash
cd bash-practice/lesson-02
pwd
```

Remove only `draft.txt`, with confirmation:

```bash
rm -i draft.txt
```

When Bash asks whether to remove it, type `y` and press Enter.

Check the final result:

```bash
ls
```

Expected final result:

```text
examples  notes.txt
```

`draft.txt` is gone. `notes.txt` and `examples/` are still there.

## Next Step

Practice this workflow until you can predict what each command will do before you press Enter. Then continue to the next beginner Bash lesson.

## Sources Used

- Bash manual: `cd` behavior and shell path shortcuts such as `~`
- GNU coreutils manuals: `pwd`, `ls`, `mkdir`, `touch`, `rm`, and `rmdir`
- POSIX utility descriptions for standard command behavior
