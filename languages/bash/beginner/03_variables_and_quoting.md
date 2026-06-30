# 03 - Variables and Quoting

## Learning Goal

Create Bash variables, expand them safely, and choose single or double quotes correctly.

By the end of this lesson, you should be able to:

- Store text in a variable with `name=value`
- Use `$name` and `${name}` to expand a variable
- Quote variable expansions so spaces and wildcard characters stay safe
- Use single quotes for literal text and double quotes when you want expansion
- Capture command output with `$(...)`
- Provide a default value with `${var:-default}`

## Platform Note

This lesson teaches Bash syntax.

- On Windows 10/11, run these examples inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- The examples use project-relative files and variables, so they do not depend on a hard-coded Windows or macOS path.

## Variables Store Text

A Bash variable is a name that points to a value.

```bash
language="Bash"
greeting="hello"
count=3
```

The assignment form is:

```bash
name=value
```

There must be no spaces around `=`.

```bash
name="Ada"     # correct
name = "Ada"   # wrong
```

Variable names can contain letters, numbers, and underscores, but they cannot start with a number.

```bash
user_name="Ada Lovelace"
project2="notes"
_temporary="draft"
```

In Bash, variable values are strings by default. Even this stores the text `42`:

```bash
answer=42
```

Bash can do arithmetic, but plain assignment does not make a special number type.

## Expanding Variables

To read a variable, put `$` before its name.

```bash
name="Ada"
echo "$name"
```

Output:

```text
Ada
```

You can also write the name inside braces:

```bash
echo "${name}"
```

Both forms work when the variable name stands alone. Braces become important when text touches the variable name.

```bash
name="ada"
echo "${name}_notes.txt"
```

Output:

```text
ada_notes.txt
```

Without braces, Bash looks for a variable named `name_notes`:

```bash
name="ada"
echo "$name_notes.txt"
```

That usually prints only:

```text
.txt
```

Bash did not know you meant `name` followed by `_notes.txt`. Use `${name}` when adjoining text.

## Single Quotes And Double Quotes

Quotes control what Bash treats as special.

Single quotes keep everything literal:

```bash
name="Ada"
echo 'Hello, $name'
```

Output:

```text
Hello, $name
```

Double quotes still allow variable expansion:

```bash
name="Ada"
echo "Hello, $name"
```

Output:

```text
Hello, Ada
```

Use single quotes when you want the exact characters. Use double quotes when you want variables or command substitutions to expand while keeping the result together.

## Why Quote Variable Expansions?

Most of the time, write variable expansions inside double quotes:

```bash
file_name="Ada Lovelace notes.txt"
echo "$file_name"
```

This matters because unquoted expansions can go through two extra steps:

- Word splitting: spaces, tabs, and newlines can split one value into multiple words.
- Pathname expansion: wildcard characters such as `*` can be treated as filename patterns.

Example:

```bash
file_name="Ada Lovelace notes.txt"
printf '<%s>\n' $file_name
```

Possible output:

```text
<Ada>
<Lovelace>
<notes.txt>
```

Bash split one value into three words. Quoting keeps it as one value:

```bash
file_name="Ada Lovelace notes.txt"
printf '<%s>\n' "$file_name"
```

Output:

```text
<Ada Lovelace notes.txt>
```

Quoting also protects values that contain wildcard characters:

```bash
pattern="*.txt"
echo "$pattern"  # prints the text *.txt
echo $pattern    # may print matching .txt filenames
```

Beginner rule: quote variable expansions unless you have a specific reason not to.

## Command Substitution

Command substitution runs a command and uses its output as text.

Prefer the modern `$(...)` form:

```bash
today="$(date +%F)"
echo "$today"
```

Example output:

```text
2026-06-20
```

The older backtick form also exists:

```bash
today=`date +%F`
```

Prefer `$(...)` because it is easier to read and easier to nest.

Use double quotes around command substitution assignments when the output might contain spaces or special characters:

```bash
current_directory="$(pwd)"
```

## Empty And Unset Variables

An unset variable has not been assigned.

```bash
echo "$missing"
```

By default, Bash expands an unset variable to an empty string.

An empty variable has been assigned, but its value is empty:

```bash
title=""
echo "$title"
```

Both unset and empty variables can look the same when printed. Use a default value when you want a fallback:

```bash
name=""
echo "${name:-Guest}"
```

Output:

```text
Guest
```

`${var:-default}` means: use `var` if it is set and not empty; otherwise use `default`.

```bash
project="reports"
echo "${project:-untitled}"
```

Output:

```text
reports
```

## Common Mistakes

### Spaces Around `=`

Wrong:

```bash
name = "Ada"
```

Correct:

```bash
name="Ada"
```

Bash reads `name = "Ada"` as a command named `name` with arguments, not as an assignment.

### Forgetting `$`

Wrong:

```bash
name="Ada"
echo "Hello, name"
```

Output:

```text
Hello, name
```

Correct:

```bash
echo "Hello, $name"
```

### Using Single Quotes When Expansion Is Wanted

Wrong:

```bash
name="Ada"
echo 'Hello, $name'
```

Correct:

```bash
echo "Hello, $name"
```

### Leaving Expansions Unquoted

Risky:

```bash
file_name="Ada Lovelace notes.txt"
rm $file_name
```

Safer:

```bash
rm "$file_name"
```

The safer version treats the full filename as one argument.

### Missing Braces Beside Other Text

Wrong:

```bash
name="ada"
echo "$name_notes.txt"
```

Correct:

```bash
echo "${name}_notes.txt"
```

### Confusing Unset And Empty

These can both print nothing:

```bash
unset nickname
nickname=""

echo "$nickname"
```

When a blank value is not useful, choose a default:

```bash
echo "${nickname:-friend}"
```

## Exercise

Create a script named `report-name.sh` that:

1. Sets `user_name="Ada Lovelace"`
2. Stores today's date in a variable with `today="$(date +%F)"`
3. Creates a report filename from the user name and date
4. Prints a literal template
5. Prints the expanded report filename

Your script should print output shaped like this:

```text
Template: ${user_name}_${today}_report.txt
Report: Ada Lovelace_2026-06-20_report.txt
```

The exact date will be the date when you run the script.

## Worked Answer

Create the file:

```bash
cat > report-name.sh <<'EOF'
#!/usr/bin/env bash

user_name="Ada Lovelace"
today="$(date +%F)"
report_file="${user_name}_${today}_report.txt"

echo 'Template: ${user_name}_${today}_report.txt'
echo "Report: $report_file"
EOF
```

Inspect it before running:

```bash
cat report-name.sh
```

Run it with Bash:

```bash
bash report-name.sh
```

The script contents should be:

```bash
#!/usr/bin/env bash

user_name="Ada Lovelace"
today="$(date +%F)"
report_file="${user_name}_${today}_report.txt"

echo 'Template: ${user_name}_${today}_report.txt'
echo "Report: $report_file"
```

Expected output shape:

```text
Template: ${user_name}_${today}_report.txt
Report: Ada Lovelace_current-date_report.txt
```

## Explanation

`user_name="Ada Lovelace"` uses no spaces around `=`, so Bash treats it as assignment. The value contains a space, so the value is quoted.

`today="$(date +%F)"` runs `date +%F` and stores output such as `2026-06-20`. The `$(...)` form is command substitution.

`report_file="${user_name}_${today}_report.txt"` uses braces because each variable touches underscores and other text. Without braces, Bash might look for longer variable names such as `user_name_`.

The first `echo` uses single quotes because it should print the template literally. The second `echo` uses double quotes because it should expand `$report_file`, while keeping the full filename as one argument.

## Next Step

Continue to the next Bash beginner lesson. As you practice, make double-quoted variable expansion your default habit:

```bash
echo "$variable"
```

Then remove the quotes only when you can explain exactly why you need Bash to split or expand the value.

## Sources Used

- [GNU Bash Reference Manual: Shell Parameters](https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameters)
- [GNU Bash Reference Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion)
- [GNU Bash Reference Manual: Single Quotes](https://www.gnu.org/software/bash/manual/bash.html#Single-Quotes)
- [GNU Bash Reference Manual: Double Quotes](https://www.gnu.org/software/bash/manual/bash.html#Double-Quotes)
- [GNU Bash Reference Manual: Word Splitting](https://www.gnu.org/software/bash/manual/bash.html#Word-Splitting)
- [GNU Bash Reference Manual: Filename Expansion](https://www.gnu.org/software/bash/manual/bash.html#Filename-Expansion)
- [GNU Bash Reference Manual: Command Substitution](https://www.gnu.org/software/bash/manual/bash.html#Command-Substitution)
- [Microsoft WSL documentation](https://learn.microsoft.com/windows/wsl/)
- [Git for Windows](https://gitforwindows.org/)
- [Apple Terminal default shell documentation](https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac)
