# Bash Beginner

This level teaches Bash as both an interactive shell and a small scripting language. Work through the lessons in order, type the examples yourself, and try each exercise before reading the worked answer.

Beginner Bash is about practical fluency: moving around safely, handling text, writing small scripts, and turning repeated terminal work into commands you can trust.

## Prerequisites

Before starting, you should have:

- A Bash shell available through Linux, macOS, WSL, Git Bash, or a similar environment.
- A terminal you can open and type commands into.
- A plain-text editor for creating small `.sh` files.
- A safe practice directory where it is okay to create and remove test files.

You do not need prior Bash scripting experience.

## How To Use This Level

- Work through the lessons in order.
- Run the examples instead of only reading them.
- Change small values and rerun commands to see what changes.
- Keep notes on commands, errors, or quoting behavior that surprise you.
- Do each lesson's exercise before checking the worked answer.
- Practice inside a disposable folder such as `bash-practice`.
- Revisit earlier lessons when paths, quotes, redirection, or permissions become confusing.

## Lessons

1. [Setup and Shell Basics](01_setup_and_shell_basics.md)
   Open and verify Bash, read prompts, run simple commands, understand paths, and run a first script.

2. [Commands, Files, and Directories](02_commands_files_and_directories.md)
   Inspect where you are, move through directories, create files and folders, and remove test files safely.

3. [Variables and Quoting](03_variables_and_quoting.md)
   Store values, expand variables, use single and double quotes, and avoid word-splitting surprises.

4. [Input and Output](04_input_and_output.md)
   Print predictable output, read user input, redirect stdout/stderr, append to files, and use simple pipelines.

5. [Conditionals](05_conditionals.md)
   Use `if`, command exit status, string tests, integer tests, and file tests to choose what a script does.

6. [Loops](06_loops.md)
   Repeat commands with `for`, `while`, `until`, arithmetic loops, `break`, and `continue`.

7. [Arrays](07_arrays.md)
   Store multiple values, access elements by index, loop safely over array values, and preview associative arrays.

8. [Functions](08_functions.md)
   Group repeated logic, pass function arguments, use `local`, and return success or failure with exit status.

9. [Scripts and Permissions](09_scripts_and_permissions.md)
   Create runnable scripts, pass arguments, check exit status, inspect permissions, and use `chmod u+x`.

10. [PATH and Reusable Commands](10_path_and_reusable_commands.md)
    Understand command lookup, add a personal command directory to `PATH`, and create a reusable command.

## Completion Goal

By the end of this level, you should be able to use Bash to navigate the filesystem, manage files safely, write small scripts, pass arguments, quote variables correctly, redirect input and output, use conditionals and loops, organize repeated logic with arrays and functions, make scripts executable, and turn one small script into a reusable command on your `PATH`.

## Final Practice

After lesson 10, build one small reusable command of your own. Good starter ideas:

- `today-note`: append a dated note to a file.
- `backup-list`: list files that would be copied before a backup.
- `project-check`: verify that important project files exist.

Your command should use variables, quoting, input/output, at least one conditional or loop, a function, script arguments, permissions, and optionally `PATH`.

## What To Study Next

Continue to intermediate Bash when you are ready to work with text-processing tools, process control, scheduling, robust scripts, and automation workflows.
