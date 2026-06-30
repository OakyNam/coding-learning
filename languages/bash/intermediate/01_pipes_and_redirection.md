# 01 - Pipes and Redirection

## Learning Goal

Use Bash pipes and redirection to connect commands, separate normal output from errors, save results to files, and detect failures in multi-command pipelines.

## Why It Matters

Most useful shell work is not one huge command. It is small commands connected together:

- read data from a file or standard input
- filter or transform it
- write the useful result somewhere
- keep diagnostics separate from data
- fail loudly when an early step breaks

Pipes and redirection are the tools that make that possible.

## Platform Note

This lesson uses Bash pipeline and redirection syntax.

- On Windows 10/11, run these examples inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- Run commands from a scratch directory so files such as `notes.txt`, `server.log`, and `errors.log` stay together and are easy to delete later.

## Standard Streams and File Descriptors

Every command starts with three standard streams open:

| File descriptor | Name | Usual purpose |
| --- | --- | --- |
| `0` | standard input, `stdin` | data the command reads |
| `1` | standard output, `stdout` | normal data the command writes |
| `2` | standard error, `stderr` | diagnostics and error messages |

By default, `stdin` usually comes from your keyboard, while `stdout` and `stderr` both print to your terminal. Redirection changes where those streams point.

```bash
printf 'hello\n' > message.txt
cat < message.txt
ls missing-file 2> errors.txt
```

What happens:

- `>` sends `stdout` to `message.txt`.
- `<` takes `stdin` from `message.txt`.
- `2>` sends `stderr` to `errors.txt`.

## Redirection Basics

Use `>` to create or replace a file with standard output.

```bash
printf 'first line\n' > notes.txt
```

Use `>>` to append standard output to a file.

```bash
printf 'second line\n' >> notes.txt
```

Use `<` to read standard input from a file.

```bash
wc -l < notes.txt
```

Use `2>` to redirect standard error.

```bash
ls notes.txt missing.txt > found.txt 2> missing-errors.txt
```

Use `2>&1` to send standard error to the same place standard output is currently going.

```bash
ls notes.txt missing.txt > combined.txt 2>&1
```

In Bash, `&>` is a shorter way to redirect both standard output and standard error to the same file.

```bash
ls notes.txt missing.txt &> combined.txt
```

Prefer `> file 2>&1` in scripts when you want the order to be very clear and portable to shells that do not support Bash's `&>` form.

## Redirection Order Matters

Bash processes redirections from left to right. That means these two commands are different:

```bash
cmd > out.txt 2>&1
```

This sends `stdout` to `out.txt`, then sends `stderr` to wherever `stdout` is now going. Both streams end up in `out.txt`.

```bash
cmd 2>&1 > out.txt
```

This sends `stderr` to wherever `stdout` is currently going, usually the terminal. Then it sends `stdout` to `out.txt`. The result is that normal output goes to the file, but errors still appear on the terminal.

Try it:

```bash
ls notes.txt missing.txt > both.txt 2>&1
cat both.txt

ls notes.txt missing.txt 2>&1 > only-stdout.txt
cat only-stdout.txt
```

In the second command, the error for `missing.txt` prints to the terminal because `stderr` was copied before `stdout` moved to the file.

## Pipes

A pipe connects the `stdout` of one command to the `stdin` of the next command.

```bash
printf 'beta\nalpha\nalpha\n' | sort
```

The left command writes lines. The right command reads those lines as input.

Pipelines are useful because each command can do one job:

```bash
printf 'beta\nalpha\nalpha\n' | sort | uniq -c
```

This sorts the lines, then counts adjacent duplicates.

`uniq` only combines neighboring duplicate lines, so sort first when you want a count of all matching values:

```bash
printf 'beta\nalpha\nbeta\n' | sort | uniq -c
```

## Piping Standard Error Too

A regular pipe sends only `stdout`.

```bash
ls notes.txt missing.txt | wc -l
```

The file names from `stdout` go into `wc -l`, but the error message for `missing.txt` still prints to the terminal.

Use `|&` when you want to pipe both `stdout` and `stderr` into the next command. In Bash, this is shorthand for piping `stdout` after redirecting `stderr` into it.

```bash
ls notes.txt missing.txt |& wc -l
```

Use this deliberately. Mixing data and diagnostics can be useful for logging, but it can also break data processing if an error message enters a pipeline that expects clean data.

## tee

`tee` copies its standard input to standard output and also writes it to a file. It is useful when you want to save intermediate data without stopping the pipeline.

```bash
printf 'beta\nalpha\nalpha\n' |
  sort |
  tee sorted.txt |
  uniq -c
```

Use `tee -a file` to append instead of replacing the file.

```bash
printf 'new entry\n' | tee -a notes.txt
```

## Here-Docs and Here-Strings

A here-doc feeds multiple lines of text to a command through `stdin`.

```bash
cat > config.txt <<'EOF'
host=localhost
port=8080
mode=dev
EOF
```

The quoted delimiter `<<'EOF'` prevents variable expansion inside the block. Use an unquoted delimiter, such as `<<EOF`, when you intentionally want Bash to expand variables.

A here-string feeds one string to a command through `stdin`.

```bash
grep 'ERROR' <<< '2026-06-20 ERROR E42 failed login'
```

Here-docs are good for sample files, generated config, and test input. Here-strings are good for small one-line checks.

## Pipeline Status

Normally, the exit status of a pipeline is the exit status of the last command.

```bash
grep 'ERROR' missing.log | wc -l
echo "$?"
```

Even if `grep` fails because `missing.log` does not exist, `wc -l` can still succeed, so the whole pipeline may report success.

Turn on `pipefail` when a pipeline should fail if any command in it fails.

```bash
set -o pipefail
grep 'ERROR' missing.log | wc -l
echo "$?"
```

With `pipefail`, the pipeline status becomes the rightmost non-zero status in the pipeline, or zero if every command succeeds.

Use `PIPESTATUS` when you need each command's status from the most recent foreground pipeline.

```bash
set +o pipefail
grep 'ERROR' missing.log | sort | uniq -c
printf 'grep=%s sort=%s uniq=%s\n' "${PIPESTATUS[0]}" "${PIPESTATUS[1]}" "${PIPESTATUS[2]}"
```

Read `PIPESTATUS` immediately after the pipeline. Running another command replaces it.

## Pipeline Subshell Caveat

In Bash, each command in a multi-command pipeline usually runs in its own subshell. Changes made inside a pipeline segment may not be visible after the pipeline finishes.

This does not work the way many people expect:

```bash
count=0
printf 'a\nb\n' | while read -r line; do
  count=$((count + 1))
done
printf 'count=%s\n' "$count"
```

The loop runs in a pipeline subshell, so `count` may still be `0` afterward.

Use process substitution when the loop needs to update variables in the current shell:

```bash
count=0
while read -r line; do
  count=$((count + 1))
done < <(printf 'a\nb\n')
printf 'count=%s\n' "$count"
```

## Common Mistakes

- Using `>` when you meant `>>`, which truncates the existing file before writing.
- Writing `cmd 2>&1 > out.txt` when you meant `cmd > out.txt 2>&1`.
- Running `uniq -c` before `sort`, which only counts duplicates that are already adjacent.
- Trusting a pipeline's final status without `set -o pipefail`.
- Forgetting that a normal pipe carries only `stdout`, not `stderr`.
- Mixing `stderr` into a data pipeline and then wondering why a parser saw an error message as input.
- Reading `PIPESTATUS` too late, after another command has already overwritten it.

## Exercise

Create a small log-processing workflow.

1. Create a file named `server.log` with the sample log lines below.
2. Save only lines containing `ERROR` to `errors.log`.
3. Create a counted, sorted list of error codes and save it to `error_codes.txt`.
4. Write the total number of error lines to `summary.txt`.
5. Redirect an intentional error message to `diagnostics.log`.
6. Demonstrate the difference between a pipeline without `pipefail` and a pipeline with `pipefail`.

Sample log:

```text
2026-06-20T09:00:01Z INFO  E00 boot complete
2026-06-20T09:01:12Z ERROR E42 failed login for user ada
2026-06-20T09:02:30Z WARN  E17 slow response from cache
2026-06-20T09:03:44Z ERROR E42 failed login for user linus
2026-06-20T09:04:10Z ERROR E07 database timeout
2026-06-20T09:05:55Z INFO  E00 health check ok
2026-06-20T09:06:18Z ERROR E07 database timeout
```

## Worked Answer

Run this in Bash from a scratch directory:

```bash
cat > server.log <<'EOF'
2026-06-20T09:00:01Z INFO  E00 boot complete
2026-06-20T09:01:12Z ERROR E42 failed login for user ada
2026-06-20T09:02:30Z WARN  E17 slow response from cache
2026-06-20T09:03:44Z ERROR E42 failed login for user linus
2026-06-20T09:04:10Z ERROR E07 database timeout
2026-06-20T09:05:55Z INFO  E00 health check ok
2026-06-20T09:06:18Z ERROR E07 database timeout
EOF

grep 'ERROR' server.log > errors.log

grep 'ERROR' server.log |
  awk '{print $3}' |
  sort |
  uniq -c > error_codes.txt

printf 'total_errors=%s\n' "$(grep -c 'ERROR' server.log)" > summary.txt

ls definitely-missing-file 2> diagnostics.log

set +o pipefail
grep 'ERROR' missing-server.log | wc -l
printf 'without_pipefail_status=%s\n' "$?"

set -o pipefail
grep 'ERROR' missing-server.log | wc -l
status=$? parts=("${PIPESTATUS[@]}")
printf 'with_pipefail_status=%s\n' "$status"
printf 'pipeline_parts: grep=%s wc=%s\n' "${parts[0]}" "${parts[1]}"
```

Expected `errors.log`:

```text
2026-06-20T09:01:12Z ERROR E42 failed login for user ada
2026-06-20T09:03:44Z ERROR E42 failed login for user linus
2026-06-20T09:04:10Z ERROR E07 database timeout
2026-06-20T09:06:18Z ERROR E07 database timeout
```

Expected `error_codes.txt`:

```text
      2 E07
      2 E42
```

Some systems align `uniq -c` with spaces before the number. The important result is two `E07` errors and two `E42` errors.

Expected `summary.txt`:

```text
total_errors=4
```

Expected `diagnostics.log` contains an error similar to:

```text
ls: cannot access 'definitely-missing-file': No such file or directory
```

The exact wording may vary by operating system.

Expected terminal output for the pipefail demonstration is similar to:

```text
grep: missing-server.log: No such file or directory
0
without_pipefail_status=0
grep: missing-server.log: No such file or directory
0
with_pipefail_status=2
pipeline_parts: grep=2 wc=0
```

The assignment `status=$? parts=("${PIPESTATUS[@]}")` captures both values before a later command can replace them.

## Next Step

Continue with `02_exit_codes_and_status_checks.md`. Pipes and redirection move data between commands; exit codes decide what your script should do when one of those commands succeeds or fails.

## Sources Used

- GNU Bash Reference Manual, "Pipelines": https://www.gnu.org/software/bash/manual/html_node/Pipelines.html
- GNU Bash Reference Manual, "Redirections": https://www.gnu.org/software/bash/manual/html_node/Redirections.html
- GNU Bash Reference Manual, "Lists of Commands": https://www.gnu.org/software/bash/manual/html_node/Lists.html
- GNU Coreutils manual, "`uniq` invocation": https://www.gnu.org/software/coreutils/manual/html_node/uniq-invocation.html
- Microsoft Learn, "Install WSL": https://learn.microsoft.com/windows/wsl/install
- Git for Windows: https://gitforwindows.org/
- Apple Terminal User Guide, "Change the default shell": https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
