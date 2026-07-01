# Python Beginner

This level is a guided first pass through the minimum Python you need to read, run, modify, and organize small programs. It is written for new Python learners, including people who are new to programming.

Work through the lessons in order. Each lesson builds vocabulary and habits you will reuse in later levels, where virtual environments, files, exceptions, classes, testing, APIs, and packaging are covered more deeply.

## What You Will Learn

By the end of this level, you will have practiced how to:

- Install Python and confirm the interpreter runs from a terminal.
- Create `.py` files and run them from the command line.
- Use variables, strings, numbers, booleans, lists, and dictionaries.
- Read input, print output, and convert between simple data types.
- Choose between paths with `if`, `elif`, and `else`.
- Repeat work with `for` loops, `while` loops, and `range()`.
- Define functions with arguments and return values.
- Import modules, use standard library modules, and split a small program across files.

## Before You Start

You need a working Python installation and a terminal. Detailed install troubleshooting belongs in [Setup and Install](01_setup_and_install.md); this page only shows the commands you should be able to run before starting.

Windows PowerShell:

```powershell
python --version
py --version
python path\to\script.py
py path\to\script.py
```

macOS Apple Silicon with `zsh`:

```zsh
python3 --version
python3 path/to/script.py
```

The command names differ by platform. On Windows, either `python` or `py` may be available. On macOS, `python3` is the command to try first.

## How To Use This Level

For each lesson:

1. Read the lesson once for the main idea.
2. Run the examples in your terminal.
3. Change one thing in an example and predict the result before running it again.
4. Fix at least one intentional error or mistake you create while experimenting.
5. Complete the exercise.
6. Compare your work with the worked answer or expected behavior.

Keep short notes as you go. For every lesson, write down new syntax, error messages you learned to read, and one sentence in your own words about what the lesson taught.

## Learning Flow

```mermaid
flowchart LR
    Learner[User at terminal] --> Script[Run a .py file]
    Script --> Input[input()]
    Input --> Values[Variables and data types]
    Values --> Branches[if / elif / else]
    Branches --> Loops[for and while loops]
    Loops --> Collections[Lists and dictionaries]
    Collections --> Functions[Functions]
    Functions --> Modules[Modules and packages]
    Modules --> Output[print() and program results]
```

The diagram shows the main beginner data flow: you run a script, accept or define values, make decisions, repeat work, organize data, move repeated logic into functions and modules, then show results.

## Lessons

1. [Setup and Install](01_setup_and_install.md) - Install Python, confirm the interpreter works, and run the prompt and a script.
2. [Project and File Structure](02_project_and_file_structure.md) - Learn folders, filenames, relative paths, and where code lives.
3. [Variables and Data Types](03_variables_and_data_types.md) - Use names, assignment, strings, numbers, booleans, and simple conversions.
4. [Input and Output](04_input_and_output.md) - Practice `input()`, `print()`, f-strings, and type conversion.
5. [Control Flow](05_control_flow.md) - Make decisions with comparisons, booleans, and branching.
6. [Loops](06_loops.md) - Repeat work with `for`, `while`, `range()`, and loop control basics.
7. [Lists and Collections](07_lists_and_collections.md) - Store ordered data, use indexes and slices, append or remove items, and iterate.
8. [Dictionaries and Objects](08_dictionaries_and_objects.md) - Store labeled data with dictionaries; full classes and objects come later.
9. [Functions](09_functions.md) - Define functions, pass arguments, return values, and reduce duplication.
10. [Modules and Packages](10_modules_and_packages.md) - Import modules, use standard library modules, and split a small program into files.

## Practice Expectations

Each lesson should give you a runnable example, a small change task, an exercise, and a worked answer or expected behavior to compare against. The point is not to memorize every rule on the first pass; it is to build the habit of running code, reading errors, changing one idea at a time, and explaining what happened.

For a capstone, build a command-line study tracker or mini quiz. It should use input, lists or dictionaries, decisions, loops, functions, and at least one helper module. Keep the project small enough that you can explain how data moves through it from terminal input to final output.

## Completion Checklist

You are ready to move on when you can:

- Run Python from a terminal.
- Create and run a `.py` file.
- Explain strings, numbers, booleans, lists, and dictionaries.
- Use `if` statements, loops, and functions in a small program.
- Read an error message well enough to find the file and line number.
- Split simple code into modules.
- Complete the capstone and explain its data flow.

## Official References

- [The Python Tutorial](https://docs.python.org/3/tutorial/index.html)
- [An Informal Introduction to Python](https://docs.python.org/3/tutorial/introduction.html)
- [More Control Flow Tools](https://docs.python.org/3/tutorial/controlflow.html)
- [Data Structures](https://docs.python.org/3/tutorial/datastructures.html)
- [Input and Output](https://docs.python.org/3/tutorial/inputoutput.html)
- [Modules](https://docs.python.org/3/tutorial/modules.html)
- [Using Python on Windows](https://docs.python.org/3/using/windows.html)
- [Using Python on a Mac](https://docs.python.org/3/using/mac.html)
- [Virtual Environments and Packages](https://docs.python.org/3/library/venv.html)
- [Installing Python Modules](https://docs.python.org/3/installing/index.html)
