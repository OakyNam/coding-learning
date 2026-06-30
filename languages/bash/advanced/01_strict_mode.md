# 01 - Strict Mode

## Learning Goal

Write and debug Bash scripts using `set -Eeuo pipefail` intentionally, handle expected failures explicitly, and use `ERR` traps where helpful.

## Why It Matters

Bash scripts often glue together commands that can fail for different reasons: missing input, unreadable files, empty search results, network errors, or broken pipelines. Without a clear error policy, a script may continue after a failed command and produce misleading output.

"Strict mode" is a common name for enabling options that make many mistakes fail fast:

```bash
set -Eeuo pipefail
```

That line is useful, but it is not magic. `set -e` has exceptions, `grep` uses exit status `1` for "no matches", pipelines normally hide failures before the final command, and command substitutions have their own nuance. Good Bash scripts use strict mode as a starting point, then make expected failures explicit.

## Baseline Script Header

Use this header for Bash scripts where failing fast is the right default:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
trap 'status=$?; printf "error: line %s failed: %s\n" "$LINENO" "$BASH_COMMAND" >&2; exit "$status"' ERR
```

What each option does:

- `-e` / `errexit`: exit when a simple command, list, compound command, or pipeline returns a non-zero status, except in documented contexts where Bash ignores `errexit`.
- `-u` / `nounset`: treat expansion of unset variables as an error, except for special parameters such as `$@` and `$*`.
- `pipefail`: change the pipeline's exit status so a failure in any command can make the whole pipeline fail, not only the final command.
- `-E` / `errtrace`: make an `ERR` trap inherited by shell functions, command substitutions, and subshells.

The `ERR` trap prints the current line number and command text, then exits with the original status. `$LINENO` and `$BASH_COMMAND` are helpful context, not a perfect diagnostic report or stack trace. Keep the trap short. A complex trap can introduce new failures while handling the first one.

## Real Examples

Require an environment variable when the script cannot choose a safe default:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
trap 'status=$?; printf "error: line %s failed: %s\n" "$LINENO" "$BASH_COMMAND" >&2; exit "$status"' ERR

environment=${ENVIRONMENT:?set ENVIRONMENT to dev, staging, or prod}

printf 'deploying to %s\n' "$environment"
```

Use a default when a missing value is normal:

```bash
log_level=${LOG_LEVEL:-info}
printf 'log level: %s\n' "$log_level"
```

When you run examples that use environment variables, use the syntax for your shell.

Windows PowerShell:

```powershell
$env:ENVIRONMENT = "dev"
bash ./deploy.sh
```

macOS Terminal uses `zsh` by default. On Apple Silicon Macs, this lesson does not require special native dependencies unless you choose to install an alternate Bash. You can set a variable for one command:

```bash
ENVIRONMENT=dev bash ./deploy.sh
```

Or export it for later commands in the same terminal:

```bash
export ENVIRONMENT=dev
bash ./deploy.sh
```

Use `pipefail` when an earlier command in a pipeline matters:

```bash
error_count=$(grep -R -- 'ERROR' ./logs | wc -l)
printf 'error lines: %s\n' "$error_count"
```

Without `pipefail`, `wc -l` may succeed even if `grep` fails because `./logs` is missing or unreadable. With `pipefail`, the pipeline reports the `grep` failure. It changes the pipeline exit status; it does not change the text that `grep` passes to `wc` or the text that later commands receive.

Handle expected failures in conditionals:

```bash
if grep -q -- 'ready' status.txt; then
  printf 'service is ready\n'
else
  printf 'service is not ready yet\n'
fi
```

GNU `grep` uses exit status `0` when it selects lines, `1` when it selects none, and `2` when an error occurs. With `-q`, GNU `grep` exits with status `0` as soon as it finds a match, even if an error was already detected. In the example above, status `1` means "no matching line", which is expected. The `if` makes that case explicit, so `set -e` does not abort the script.

For a pipeline where "no matches" is expected but real `grep` errors are fatal, inspect the status:

```bash
matches=0
if count=$(grep -R -- 'ERROR' ./logs | wc -l); then
  matches=$count
else
  status=$?
  if (( status == 1 )); then
    matches=0
  else
    exit "$status"
  fi
fi
```

Be careful: with `pipefail`, a pipeline status comes from the rightmost failing command. If you need to distinguish every command in a pipeline, use simpler steps or inspect `PIPESTATUS` immediately after the pipeline.

## `set -e` Sharp Edges

`set -e` is useful because it changes the default from "keep going" to "stop on unexpected failure." The sharp edge is that Bash ignores `errexit` in several common places.

`errexit` is ignored when a failing command is:

- Part of the test after `if`, `while`, or `until`.
- In a `&&` or `||` list, except the command after the final `&&` or `||`.
- Preceded by `!`, because the status is being inverted.
- A non-final command in a pipeline, unless `pipefail` changes the pipeline status.

Functions have an extra trap: if a function is called in a conditional context, commands inside that function run while `errexit` is ignored.

```bash
check_config() {
  grep -q -- 'enabled=true' app.conf
  printf 'config checked\n'
}

if check_config; then
  printf 'enabled\n'
fi
```

If `grep` finds no match, the function continues to `printf`, and the function returns success. Prefer making the function's status explicit:

```bash
check_config() {
  grep -q -- 'enabled=true' app.conf
}
```

Command substitution also deserves care. Bash runs `$(...)` in a subshell environment. By default, Bash clears `errexit` inside command substitutions unless POSIX mode or the advanced, version-sensitive `shopt -s inherit_errexit` option is active. Do not rely on `inherit_errexit` for this lesson. `set -E` makes `ERR` traps inherit into command substitutions, but it is not the same thing as `inherit_errexit`.

Another common failure mask is assigning command output with `local`:

```bash
make_name() {
  local value=$(generate_name)
  printf '%s\n' "$value"
}
```

The `local` builtin can mask the command substitution's failure. Use a two-line pattern:

```bash
make_name() {
  local value
  value=$(generate_name)
  printf '%s\n' "$value"
}
```

Now the assignment command can fail and `errexit` can act on it.

## ERR Traps

An `ERR` trap runs before the shell exits because of `errexit`. It is a good place to add context:

```bash
trap 'status=$?; printf "error: line %s failed: %s\n" "$LINENO" "$BASH_COMMAND" >&2; exit "$status"' ERR
```

The line number and command are clues for debugging the failed path. They are not a full call stack, and the command text can be surprising when the failure happens inside compound commands, functions, or substitutions.

The `ERR` trap follows the same exceptions as `errexit`. It does not run for failures that Bash is ignoring because they are in an `if` test, a `while` test, a non-final `&&` or `||` position, an inverted `!` command, or a non-final pipeline command without `pipefail`.

Use `set -E` when you want the trap to apply inside functions, command substitutions, and subshells:

```bash
set -E
```

Do not use an `ERR` trap to hide the failure or always convert it to success. The trap should report the problem and preserve the failing status unless the script has a specific recovery plan.

## Common Mistakes

- Adding `set -euo pipefail` to a script without testing the paths where commands intentionally return non-zero.
- Treating every non-zero status as fatal, even when commands like `grep -q` use status `1` for a normal "not found" result.
- Using blanket `|| true` after commands. If a failure is expected, document that exact command and handle its status.
- Forgetting that `set -u` makes unset variables fail. Use `${NAME:?message}` for required values and `${NAME:-default}` for optional values.
- Assuming `set -e` exits everywhere. It is ignored in `if`/`while`/`until` tests, many `&&`/`||` positions, `!`, and parts of pipelines.
- Calling a function inside `if some_function; then` and expecting `errexit` to stop inside the function body.
- Writing `local value=$(cmd)` and missing a failure from `cmd`.
- Using `set -o pipefail` in a script that runs under `/bin/sh`. `pipefail` is a Bash option; use a Bash shebang when you rely on it.
- Expecting an `ERR` trap to run for every non-zero status. The trap has the same exception zones as `errexit`.

## Exercise

Repair this flawed log scanner.

Requirements:

- `INPUT_DIR` is required.
- `PATTERN` is optional and defaults to `ERROR`.
- Write a report to `${REPORT:-report.txt}`.
- Use Bash, not `/bin/sh`.
- Scanning zero matching `*.log` files should finish cleanly.
- Unreadable log files are fatal.
- `grep` finding no matching lines in a readable file is expected.
- Do not use blanket `|| true`; the only allowed exception must be documented for the `grep` no-match case.

Flawed script:

```bash
#!/usr/bin/env bash
set -euo pipefail

for log in $INPUT_DIR/*.log; do
  echo "== $log ==" >> report.txt
  grep "$PATTERN" "$log" >> report.txt || true
done

echo done
```

Run your repaired script with environment variables like this.

Windows PowerShell:

```powershell
$env:INPUT_DIR = "logs"
$env:PATTERN = "ERROR"
$env:REPORT = "report.txt"
bash ./scan_logs.sh
```

macOS Terminal (`zsh` by default):

```bash
INPUT_DIR=logs PATTERN=ERROR REPORT=report.txt bash ./scan_logs.sh
```

## Worked Answer

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
trap 'status=$?; printf "error: line %s failed: %s\n" "$LINENO" "$BASH_COMMAND" >&2; exit "$status"' ERR

input_dir=${INPUT_DIR:?set INPUT_DIR to the directory containing log files}
pattern=${PATTERN:-ERROR}
report=${REPORT:-report.txt}

if [[ ! -d "$input_dir" ]]; then
  printf 'error: INPUT_DIR is not a directory: %s\n' "$input_dir" >&2
  exit 1
fi

: > "$report"

shopt -s nullglob
logs=("$input_dir"/*.log)
shopt -u nullglob

if (( ${#logs[@]} == 0 )); then
  printf 'no log files found in %s\n' "$input_dir" >> "$report"
  printf 'done: wrote %s\n' "$report"
  exit 0
fi

for log in "${logs[@]}"; do
  if [[ ! -r "$log" ]]; then
    printf 'error: unreadable log file: %s\n' "$log" >&2
    exit 1
  fi

  printf '== %s ==\n' "$log" >> "$report"

  # grep status 1 means "no matching lines", which is expected for this scanner.
  if grep -n -- "$pattern" "$log" >> "$report"; then
    :
  else
    status=$?
    if (( status == 1 )); then
      printf '(no matches)\n' >> "$report"
    else
      exit "$status"
    fi
  fi
done

printf 'done: wrote %s\n' "$report"
```

Notes:

- `${INPUT_DIR:?message}` makes the required input fail before the script scans the wrong place.
- `${PATTERN:-ERROR}` and `${REPORT:-report.txt}` give safe defaults for optional values.
- `nullglob` makes an unmatched `*.log` pattern expand to nothing, so "no log files" is handled deliberately.
- The script checks readability before running `grep`, making unreadable files fatal with a clear message.
- The only tolerated non-zero command status is `grep` status `1`, and the comment explains why. Other `grep` errors, including unreadable input that somehow passed the earlier check, remain fatal.

## Next Step

Use this header in a small script you already have, then run through each command that can return non-zero. For each one, decide whether the failure is fatal, expected, or recoverable, and encode that decision with an `if`, `case`, or clear exit message.

## Sources Used

- GNU Bash Reference Manual: [The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)
- GNU Bash Reference Manual: [Pipelines](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html)
- GNU Bash Reference Manual: [Bash Builtin Commands](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html)
- GNU Bash Reference Manual: [Command Execution Environment](https://www.gnu.org/software/bash/manual/html_node/Command-Execution-Environment.html)
- GNU Bash Reference Manual: [Command Substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)
- GNU Bash Reference Manual: [The Shopt Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html)
- GNU grep Manual: [Exit Status](https://www.gnu.org/software/grep/manual/grep.html#Exit-Status)
- GNU grep Manual: [`-q`, `--quiet`, `--silent`](https://www.gnu.org/software/grep/manual/grep.html#index-_002dq)
- Wooledge BashFAQ: [BashFAQ/105](https://mywiki.wooledge.org/BashFAQ/105)
- ShellCheck Wiki: [SC3040](https://www.shellcheck.net/wiki/SC3040)
