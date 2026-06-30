# 02 - Exit Codes and Status Checks

## Learning Goal

Use Bash exit statuses to decide what a script should do next, including direct command checks, explicit script exits, function returns, and pipeline status handling.

## Why Exit Codes Matter

Every command returns a status code when it finishes. Bash uses that status to decide whether a command "succeeded" or "failed" for control-flow purposes:

- `0` means success.
- Any nonzero value means failure or another non-success condition.

That does not always mean "broken." For example, `grep` returns `1` when it searched successfully but found no matching lines. Your script needs to know when a nonzero status means an expected result and when it means a real error.

## Platform Note

This lesson uses Bash status-checking syntax.

- On Windows 10/11, run these examples inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- The worked answer runs scripts with `bash check_logs.sh ...` so the lesson behaves consistently even before you practice executable permissions.

## Reading The Most Recent Status

Bash stores the status of the most recent foreground pipeline in the special parameter `$?`.

```bash
grep "ERROR" app.log
status=$?

echo "grep exited with status $status"
```

Read `$?` immediately. Even a harmless-looking command such as `echo`, `test`, or an assignment with command substitution can replace the value you meant to inspect.

```bash
grep "ERROR" app.log
echo "checking status"
echo "$?"
```

The final `echo "$?"` prints the status of `echo "checking status"`, not the status of `grep`.

If you need to keep the value, save it right away:

```bash
grep "ERROR" app.log
grep_status=$?

if [ "$grep_status" -eq 0 ]; then
  echo "found errors"
fi
```

## Prefer Direct Checks

Most of the time, you do not need `$?`. Bash conditionals already check command status.

```bash
if grep "ERROR" app.log; then
  echo "found errors"
else
  echo "no errors found, or grep failed"
fi
```

The command after `if` runs first. If it returns `0`, Bash runs the `then` block. If it returns nonzero, Bash runs the `else` block if one exists.

Use this style when you only need success versus non-success. Save `$?` when you need to distinguish several statuses.

## Lists With `&&` And `||`

Bash also uses statuses in command lists.

```bash
mkdir -p reports && echo "reports directory is ready"
```

The second command runs only if `mkdir -p reports` returns `0`.

```bash
cp app.log reports/app.log || echo "copy failed"
```

The second command runs only if `cp app.log reports/app.log` returns nonzero.

You can combine these, but keep readability in mind:

```bash
grep "ERROR" app.log && echo "errors found" || echo "no match or grep failed"
```

That one-liner is compact, but it hides an important detail: `grep` status `1` means no match, while status `2` means an error. A full `if` or `case` statement is clearer when the difference matters.

## Inverting A Status With `!`

The `!` operator reverses the success test for a command.

```bash
if ! test -f app.log; then
  echo "missing app.log"
fi
```

This reads well for guard checks: "if not a regular file, report the problem."

## Exiting Scripts And Returning From Functions

Use `exit n` to end a script or shell with status `n`.

```bash
if [ "$#" -ne 1 ]; then
  echo "usage: backup.sh FILE" >&2
  exit 2
fi
```

Use `return n` inside a function to return a status to the caller without exiting the whole script.

```bash
check_file() {
  if [ ! -f "$1" ]; then
    return 1
  fi

  return 0
}

if check_file app.log; then
  echo "file exists"
else
  echo "file is missing"
fi
```

A common convention is:

- `0`: success
- `1`: general failure
- `2`: incorrect usage, such as missing arguments

Bash also uses some special statuses:

- `126`: the command was found but could not be executed
- `127`: the command was not found

## Pipeline Status

By default, a pipeline returns the status of its last command.

```bash
grep "ERROR" app.log | wc -l
echo "$?"
```

The status printed here is the status of `wc -l`, not necessarily the status of `grep`. If `grep` fails because `app.log` is missing but `wc -l` still runs successfully, the pipeline can still return `0`.

Enable `pipefail` when the script should treat any failed command in a pipeline as a pipeline failure:

```bash
set -o pipefail

grep "ERROR" app.log | wc -l
echo "$?"
```

With `pipefail`, the pipeline returns the rightmost nonzero status, or `0` if all commands succeed.

Bash also provides the `PIPESTATUS` array, which holds each command's status from the most recent foreground pipeline.

```bash
grep "ERROR" app.log | wc -l
statuses=("${PIPESTATUS[@]}")

echo "grep status: ${statuses[0]}"
echo "wc status: ${statuses[1]}"
```

Like `$?`, read `PIPESTATUS` immediately. The next command can replace it.

## A Cautious Note About `set -e`

`set -e`, also called `errexit`, tells Bash to exit when some commands return nonzero. It can be useful, but it has exceptions around conditionals, lists, pipelines, and other contexts. Those exceptions surprise many script authors.

Explicit checks still matter because they document which failures are expected, which failures should stop the script, and which nonzero statuses have special meaning.

```bash
if grep "ERROR" app.log >/dev/null; then
  echo "errors found"
elif [ "$?" -eq 1 ]; then
  echo "no errors found"
else
  echo "grep failed" >&2
  exit 1
fi
```

This example is intentionally explicit: `grep` status `1` is not handled the same way as a grep error.

## Common Mistakes

- Reading `$?` too late, after another command has already replaced it.
- Writing `command; if [ "$?" -eq 0 ]; then ...` when `if command; then ...` is clearer.
- Treating all nonzero statuses as the same result.
- Forgetting that `grep` returns `1` for "no matches" and a different nonzero status for errors.
- Assuming a pipeline failed because an early command failed; by default, Bash reports the last command's status.
- Using `exit` inside a function when `return` was intended, which ends the whole script instead of just the function.

## Exercise

Write `check_logs.sh`.

Requirements:

1. The script accepts exactly one argument: the path to a log file.
2. If the argument count is wrong, print `usage: check_logs.sh LOGFILE` to standard error and exit with status `2`.
3. If the file does not exist or is not a regular file, print `missing file: LOGFILE` to standard error and exit with status `1`.
4. Search the file for the text `ERROR`.
5. If matches are found, print `ERROR lines found:` and then print the matching lines.
6. If no matches are found, print `no errors found`.
7. If `grep` itself fails for another reason, print `grep failed` to standard error and exit with status `1`.
8. Exit with status `0` both when errors are found and when no errors are found.

## Worked Answer

Run this in Bash from a scratch directory to create the script:

```bash
cat > check_logs.sh <<'EOF'
#!/usr/bin/env bash

if [ "$#" -ne 1 ]; then
  echo "usage: check_logs.sh LOGFILE" >&2
  exit 2
fi

log_file=$1

if [ ! -f "$log_file" ]; then
  echo "missing file: $log_file" >&2
  exit 1
fi

matches=$(grep "ERROR" "$log_file")
grep_status=$?

case "$grep_status" in
  0)
    echo "ERROR lines found:"
    echo "$matches"
    exit 0
    ;;
  1)
    echo "no errors found"
    exit 0
    ;;
  *)
    echo "grep failed" >&2
    exit 1
    ;;
esac
EOF
```

This answer saves `grep` status immediately because the script needs to distinguish three cases:

- `0`: `grep` found at least one match.
- `1`: `grep` searched successfully and found no matches.
- Any other status: `grep` failed.

## Example Runs

Wrong number of arguments:

```bash
$ bash check_logs.sh
usage: check_logs.sh LOGFILE
$ echo $?
2
```

Missing file:

```bash
$ bash check_logs.sh missing.log
missing file: missing.log
$ echo $?
1
```

File with an error:

```bash
$ printf 'INFO started\nERROR database unavailable\n' > app.log
$ bash check_logs.sh app.log
ERROR lines found:
ERROR database unavailable
$ echo $?
0
```

File with no errors:

```bash
$ printf 'INFO started\nINFO done\n' > app.log
$ bash check_logs.sh app.log
no errors found
$ echo $?
0
```

## Next Step

Practice rewriting one `$?` check as a direct `if command; then` check, then continue to the next Bash intermediate lesson on arguments and flags.

## Sources Used

- GNU Bash Manual: [Exit Status](https://www.gnu.org/software/bash/manual/bash.html#Exit-Status)
- GNU Bash Manual: [Lists of Commands](https://www.gnu.org/software/bash/manual/bash.html#Lists)
- GNU Bash Manual: [Conditional Constructs](https://www.gnu.org/software/bash/manual/bash.html#Conditional-Constructs)
- GNU Bash Manual: [Pipelines](https://www.gnu.org/software/bash/manual/bash.html#Pipelines)
- GNU Bash Manual: [Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/bash.html#Bourne-Shell-Builtins)
- POSIX Shell Command Language: [Exit Status for Commands](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_08)
- Microsoft Learn, "Install WSL": https://learn.microsoft.com/windows/wsl/install
- Git for Windows: https://gitforwindows.org/
- Apple Terminal User Guide, "Change the default shell": https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
