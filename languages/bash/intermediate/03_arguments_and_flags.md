# 03 - Arguments and Flags

## Learning Goal

Write Bash scripts that accept positional arguments and short options safely, including required arguments, remaining operands, quoted filenames, manual `case` parsing, and `getopts`.

## Why It Matters

Most useful shell scripts are not closed boxes. They accept a target directory, a mode, a filename, a count, or a flag that changes behavior. Argument handling is the difference between a script that only works for your first test and a script that another person can run without editing it.

Bash gives you two common tools:

- Positional parameters such as `$1`, `$2`, `$#`, and `"$@"`.
- Option parsing, either manually with `case` and `shift`, or with the Bash/POSIX `getopts` builtin for short options.

## Platform Note

This lesson uses Bash argument expansion and option parsing.

- On Windows 10/11, run these examples inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- The worked examples use `bash script.sh ...` so they behave consistently before you practice executable permissions.

## Positional Parameters

When a script is called, Bash stores its command-line words in positional parameters.

```bash
#!/usr/bin/env bash

echo "script name: $0"
echo "first arg: $1"
echo "second arg: $2"
echo "arg count: $#"
```

If the file is named `show_args.sh` and you run:

```bash
bash show_args.sh alpha beta
```

the output is:

```text
script name: show_args.sh
first arg: alpha
second arg: beta
arg count: 2
```

Important positional parameters:

- `$0` is the name used to invoke the shell or script.
- `$1` through `$9` are the first nine arguments.
- `${10}` is the tenth argument. Braces are required for positional parameters with more than one digit.
- `$#` is the number of positional arguments.
- `"$@"` expands to each argument as a separate quoted word.
- `"$*"` expands to one quoted word containing all arguments joined by the first character of `IFS`, usually a space.

For scripts, `"$@"` is usually what you want when forwarding or looping over arguments.

```bash
for path in "$@"; do
  printf 'file: %s\n' "$path"
done
```

That loop preserves spaces, tabs, wildcard characters, and empty arguments. If someone passes `Quarter 1.txt`, it remains one argument.

## Quoting Arguments

In shell scripts, quote parameter expansions unless you specifically want word splitting and pathname expansion.

Good:

```bash
cp -- "$1" "$2"
for file in "$@"; do
  printf '%s\n' "$file"
done
```

Risky:

```bash
cp -- $1 $2
for file in $@; do
  printf '%s\n' "$file"
done
```

The risky version breaks filenames with spaces and can accidentally expand `*`, `?`, or bracket patterns as globs.

Use `"$@"` when you want all original arguments preserved as separate words. Use `"$*"` only when you intentionally want one combined string.

```bash
printf 'separate args:\n'
for arg in "$@"; do
  printf '<%s>\n' "$arg"
done

printf 'one string: <%s>\n' "$*"
```

With:

```bash
bash demo.sh "two words" "*.txt"
```

`"$@"` produces two values, `two words` and `*.txt`. `"$*"` produces one value, usually `two words *.txt`.

## Validating Required Arguments

Check required arguments before using them. A short `usage` function keeps error paths readable.

```bash
#!/usr/bin/env bash

usage() {
  printf 'Usage: %s SOURCE DEST\n' "$0" >&2
}

if [ "$#" -ne 2 ]; then
  usage
  exit 2
fi

source_path=$1
dest_path=$2

cp -- "$source_path" "$dest_path"
```

Status `2` is commonly used for command-line usage errors. The exact status is your script's contract, so be consistent and document it.

## Manual Parsing With case and shift

Manual parsing is useful when you need long options, custom syntax, or a small parser that is easy to see at a glance.

This parser accepts `-v`, `-n COUNT`, `--name NAME`, `--`, and file operands.

```bash
#!/usr/bin/env bash

verbose=0
count=1
name=

usage() {
  printf 'Usage: %s [-v] [-n COUNT] [--name NAME] [--] FILE...\n' "$0" >&2
}

while [ "$#" -gt 0 ]; do
  case $1 in
    -v)
      verbose=1
      shift
      ;;
    -n)
      if [ "$#" -lt 2 ]; then
        printf 'Error: -n requires a value\n' >&2
        usage
        exit 2
      fi
      count=$2
      shift 2
      ;;
    --name)
      if [ "$#" -lt 2 ]; then
        printf 'Error: --name requires a value\n' >&2
        usage
        exit 2
      fi
      name=$2
      shift 2
      ;;
    --)
      shift
      break
      ;;
    -*)
      printf 'Error: unknown option: %s\n' "$1" >&2
      usage
      exit 2
      ;;
    *)
      break
      ;;
  esac
done

if [ "$#" -eq 0 ]; then
  printf 'Error: at least one FILE is required\n' >&2
  usage
  exit 2
fi

for file in "$@"; do
  if [ "$verbose" -eq 1 ]; then
    printf 'name=%s count=%s file=%s\n' "$name" "$count" "$file"
  fi
done
```

The key idea is that each handled option removes itself from the front of the argument list with `shift`. An option with a value, such as `-n 3`, uses `shift 2` because it consumes two words. The `--` marker means "stop parsing options"; everything after it is an operand, even if it starts with `-`.

## Parsing Short Options With getopts

`getopts` is the standard shell tool for parsing short options such as `-v` and `-p PREFIX`. It does not parse long options such as `--verbose`.

The basic shape is:

```bash
while getopts ':p:uv' opt; do
  case $opt in
    p)
      prefix=$OPTARG
      ;;
    u)
      uppercase=1
      ;;
    v)
      verbose=1
      ;;
    :)
      printf 'Error: -%s requires a value\n' "$OPTARG" >&2
      exit 2
      ;;
    \?)
      printf 'Error: unknown option: -%s\n' "$OPTARG" >&2
      exit 2
      ;;
  esac
done

shift "$((OPTIND - 1))"
```

Pieces to know:

- The optstring lists accepted option letters.
- A trailing colon after an option letter means that option requires an argument. In `:p:uv`, `-p` requires a value, while `-u` and `-v` do not.
- A leading colon in the optstring enables silent error reporting. That lets your script handle missing values with `:)` and unknown options with `\?)`.
- `OPTARG` contains the option value for options that take one. With a leading colon, it also contains the option letter for missing-value and unknown-option errors.
- `OPTIND` is the index of the next argument to process.
- After the loop, `shift "$((OPTIND - 1))"` removes the parsed options so `"$@"` contains only the remaining operands.
- `getopts` handles short options only. Use manual parsing or another tool if your script needs long options.

Example:

```bash
#!/usr/bin/env bash

usage() {
  printf 'Usage: %s [-v] -p PREFIX FILE...\n' "$0" >&2
}

verbose=0
prefix=

while getopts ':p:v' opt; do
  case $opt in
    p)
      prefix=$OPTARG
      ;;
    v)
      verbose=1
      ;;
    :)
      printf 'Error: -%s requires a value\n' "$OPTARG" >&2
      usage
      exit 2
      ;;
    \?)
      printf 'Error: unknown option: -%s\n' "$OPTARG" >&2
      usage
      exit 2
      ;;
  esac
done

shift "$((OPTIND - 1))"

if [ -z "$prefix" ]; then
  printf 'Error: -p PREFIX is required\n' >&2
  usage
  exit 2
fi

if [ "$#" -eq 0 ]; then
  printf 'Error: at least one FILE is required\n' >&2
  usage
  exit 2
fi

for file in "$@"; do
  [ "$verbose" -eq 1 ] && printf 'Would tag %s with %s\n' "$file" "$prefix"
done
```

## Common Mistakes

- Writing `for arg in $@` instead of `for arg in "$@"`. The unquoted form can split and expand arguments again.
- Forgetting `shift "$((OPTIND - 1))"` after `getopts`, which leaves options mixed in with file operands.
- Expecting `getopts` to parse long options such as `--prefix`. It parses short options only.
- Writing `while getopts ':puv' opt` when `-p` needs a value. The optstring must be `:p:uv`.
- Checking options but never validating required operands, so the script silently does nothing.
- Reusing `getopts` in the same shell or function without resetting `OPTIND=1` first.

## Exercise

Create `tag_files.sh`.

Requirements:

- Accept `-p PREFIX`, which is required.
- Accept `-u`, which prints output filenames in uppercase.
- Accept `-v`, which prints verbose messages.
- Require one or more file operands.
- Print usage and exit with status `2` for an unknown option, a missing option value, a missing prefix, or no files.
- Use `getopts`.
- After parsing options, loop over file operands with `for file in "$@"; do`.
- You do not need to rename files for this exercise. Print the output filename that would be used.

Example behavior:

```bash
bash tag_files.sh -p draft report.txt notes.md
```

Expected output:

```text
draft_report.txt
draft_notes.md
```

With uppercase and verbose mode:

```bash
bash tag_files.sh -v -u -p draft "report final.txt"
```

Expected output:

```text
tagging report final.txt -> DRAFT_REPORT FINAL.TXT
DRAFT_REPORT FINAL.TXT
```

Invalid use:

```bash
bash tag_files.sh -p
```

Expected error output:

```text
Error: -p requires a value
Usage: tag_files.sh -p PREFIX [-u] [-v] FILE...
```

Expected status: `2`.

## Worked Answer

Run this in Bash from a scratch directory to create the script:

```bash
cat > tag_files.sh <<'EOF'
#!/usr/bin/env bash

usage() {
  printf 'Usage: %s -p PREFIX [-u] [-v] FILE...\n' "$0" >&2
}

prefix=
uppercase=0
verbose=0

while getopts ':p:uv' opt; do
  case $opt in
    p)
      prefix=$OPTARG
      ;;
    u)
      uppercase=1
      ;;
    v)
      verbose=1
      ;;
    :)
      printf 'Error: -%s requires a value\n' "$OPTARG" >&2
      usage
      exit 2
      ;;
    \?)
      printf 'Error: unknown option: -%s\n' "$OPTARG" >&2
      usage
      exit 2
      ;;
  esac
done

shift "$((OPTIND - 1))"

if [ -z "$prefix" ]; then
  printf 'Error: -p PREFIX is required\n' >&2
  usage
  exit 2
fi

if [ "$#" -eq 0 ]; then
  printf 'Error: at least one FILE is required\n' >&2
  usage
  exit 2
fi

for file in "$@"; do
  tagged="${prefix}_${file}"

  if [ "$uppercase" -eq 1 ]; then
    tagged=${tagged^^}
  fi

  if [ "$verbose" -eq 1 ]; then
    printf 'tagging %s -> %s\n' "$file" "$tagged"
  fi

  printf '%s\n' "$tagged"
done
EOF
```

## Teaching Notes

- Start by running the script with filenames that contain spaces. That makes the value of `"$@"` obvious.
- Ask learners to remove the colon after `p` in `:p:uv` and observe how `-p draft` is misread.
- Ask learners to comment out `shift "$((OPTIND - 1))"` and observe why option words show up as files.
- Point out that `${tagged^^}` is Bash-specific uppercase expansion, which is fine here because this is a Bash lesson.
- If this parser were placed inside a function and called more than once in the same shell, set `OPTIND=1` before the `getopts` loop.

## Next Step

Write a second version that supports a manual long option, such as `--prefix PREFIX`, using `case` and `shift`. Compare the parser to the `getopts` version and decide which one is easier to maintain for this script.

## Sources Used

- GNU Bash Reference Manual, Shell Parameters: https://www.gnu.org/software/bash/manual/html_node/Shell-Parameters.html
- GNU Bash Reference Manual, Positional Parameters: https://www.gnu.org/software/bash/manual/html_node/Positional-Parameters.html
- GNU Bash Reference Manual, Special Parameters: https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html
- GNU Bash Reference Manual, Bourne Shell Builtins (`getopts`, `shift`): https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html
- GNU Bash Reference Manual, Quoting: https://www.gnu.org/software/bash/manual/html_node/Quoting.html
- POSIX `getopts`: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/getopts.html
- Microsoft Learn, "Install WSL": https://learn.microsoft.com/windows/wsl/install
- Git for Windows: https://gitforwindows.org/
- Apple Terminal User Guide, "Change the default shell": https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
