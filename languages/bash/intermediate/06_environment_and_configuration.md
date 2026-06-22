# 06 - Environment and Configuration

## Learning Goal

Learn how Bash stores configuration in shell variables and environment variables, how exported values reach child processes, and how to use environment-based configuration safely in scripts.

## Why It Matters

Many command-line tools and deployment scripts are configured without editing code. They read values such as `PATH`, `LANG`, `EDITOR`, or `API_TOKEN` from the environment. If you understand how Bash variables become environment variables, you can run the same script in development, testing, and production with different settings.

Environment configuration is powerful, but it is also easy to misuse. A missing export, an overwritten `PATH`, or a secret committed to a startup file can make a script fail in confusing ways.

## Shell Variables vs Environment Variables

A shell variable belongs to the current Bash process:

```bash
mode=dev
echo "$mode"
```

That value is available to Bash itself, but it is not automatically passed to programs Bash starts.

An environment variable is a variable Bash marks for export:

```bash
export APP_MODE=dev
```

Child processes inherit exported values:

```bash
export APP_MODE=dev
bash -c 'echo "$APP_MODE"'
```

The child shell can read `APP_MODE` because it inherited a copy of the environment. It cannot modify the parent shell's environment:

```bash
export APP_MODE=dev
bash -c 'APP_MODE=test'
echo "$APP_MODE"   # still dev
```

This one-way inheritance is central to environment configuration: parents pass values down; children do not push changes back up.

## Inspecting the Environment

Use `printenv` to print exported environment variables:

```bash
printenv
printenv HOME
```

Use `env` to print the environment or run a command with a modified environment:

```bash
env
```

Use `export -p` to show variables marked for export by the current shell:

```bash
export -p
```

Use `declare -p NAME` when you want Bash's view of a specific variable, including whether it is exported:

```bash
declare -p HOME
declare -p APP_MODE
```

Do not write scripts that depend on the order of `printenv` or `env` output. Environment entries are name-value pairs, but their display order is not a stable interface.

## Setting/Exporting/Unsetting

Set a shell variable with no spaces around `=`:

```bash
mode=dev
```

Export a variable when child commands need it:

```bash
export APP_MODE=dev
./run.sh
```

Remove a variable with `unset`:

```bash
unset APP_MODE
```

You can also set first and export later:

```bash
APP_MODE=dev
export APP_MODE
```

## One-Command Environment Overrides

Put assignments before a command to set environment values only for that command:

```bash
APP_MODE=test ./run.sh
LC_ALL=C sort names.txt
```

After the command finishes, the parent shell's variables are unchanged unless they were already set separately.

Use `env -i` to start a command with an empty environment, then add back only what it needs:

```bash
env -i PATH="$PATH" HOME="$HOME" bash -c 'printenv'
```

This is useful for testing whether a script accidentally depends on your personal shell setup.

## Important Environment Variables

`PATH` is the colon-separated list of directories searched for external commands.

`HOME` is the current user's home directory.

`PWD` is the current working directory as tracked by the shell.

`OLDPWD` is the previous working directory, used by `cd -`.

`SHELL` usually names the user's login shell.

`USER` and `LOGNAME` identify the current user on many systems.

`LANG` sets the default locale category values.

`LC_ALL` overrides locale categories and is often used for predictable command behavior.

`LC_*` variables, such as `LC_COLLATE` or `LC_TIME`, control individual locale categories.

`EDITOR` and `VISUAL` tell programs which editor to open.

`PAGER` tells programs which pager to use for long output, often `less` or `more`.

`TMPDIR` tells programs where to create temporary files when they honor it.

## PATH Configuration

To add your own command directory before the existing command search path, prepend it:

```bash
export PATH="$HOME/bin:$PATH"
```

Quoting keeps the assignment safe if a directory contains spaces. Keeping the old `$PATH` preserves access to system commands.

Check which command Bash will run with `command -v`:

```bash
command -v bash
command -v my-tool
```

If `command -v my-tool` prints nothing and exits nonzero, Bash cannot find `my-tool` through `PATH`.

## Bash Startup Files

Bash reads different startup files depending on how it was started.

For an interactive login shell, Bash reads `/etc/profile` first when it exists. Then it reads the first existing readable file from this list:

```text
~/.bash_profile
~/.bash_login
~/.profile
```

For an interactive non-login shell, Bash reads:

```text
~/.bashrc
```

For a non-interactive shell, such as a script, Bash does not read `~/.bashrc`. If `BASH_ENV` is set in the environment, Bash expands its value and reads that file before running the script:

```bash
BASH_ENV="$HOME/.bash_env" bash script.sh
```

Startup files are a good place for stable interactive preferences, such as `EDITOR`, `PAGER`, or a personal `PATH` extension. They are usually a bad place for project secrets, because dotfiles are often copied, synced, or checked into personal repositories.

## Script Configuration Patterns

Use parameter expansion for defaults:

```bash
APP_MODE=${APP_MODE:-dev}
```

This means "use the current non-empty `APP_MODE`; otherwise use `dev`."

Use the `:?` form for required variables:

```bash
: "${API_TOKEN:?Set API_TOKEN before running this script}"
```

The `:` command does nothing by itself, but the expansion fails with a clear message if `API_TOKEN` is unset or empty.

For optional trusted configuration files, source them after checking that they exist:

```bash
config_file=${CONFIG_FILE:-./app.env}

if [ -f "$config_file" ]; then
  . "$config_file"
fi
```

The `.` builtin, also called `source`, executes the file in the current shell. Only source files you trust, because sourced files execute code; they are not just key-value data.

## Example

This script reads configuration from the environment, supplies defaults, validates a required secret, and writes a small status line:

```bash
#!/usr/bin/env bash
set -euo pipefail

APP_MODE=${APP_MODE:-dev}
LOG_DIR=${LOG_DIR:-./logs}
: "${API_TOKEN:?Set API_TOKEN before running this script}"

mkdir -p "$LOG_DIR"
printf 'mode=%s\n' "$APP_MODE" > "$LOG_DIR/app-status.txt"

if [ "${VERBOSE:-0}" = "1" ]; then
  printf 'Wrote %s\n' "$LOG_DIR/app-status.txt"
fi
```

Example runs:

```bash
API_TOKEN=secret ./status.sh
APP_MODE=test LOG_DIR=./tmp API_TOKEN=secret VERBOSE=1 ./status.sh
```

## Common Mistakes

- Writing `export NAME = value`. Bash treats this as separate words. Use `export NAME=value`.
- Expecting a child process to change the parent shell's environment. Children inherit copies; they cannot update the parent.
- Expanding unquoted variables such as `$REPORT_DIR`. Use `"$REPORT_DIR"` so spaces and glob characters stay data.
- Putting secrets in tracked startup files such as `.bashrc`, `.bash_profile`, or `.profile`.
- Sourcing an untrusted `.env` file. In Bash, sourced files execute commands.
- Overwriting `PATH` with `export PATH="$HOME/bin"` and losing system command directories. Extend with `export PATH="$HOME/bin:$PATH"`.
- Assuming `.bashrc` runs for scripts. Normal Bash scripts are non-interactive shells and do not read `.bashrc`.

## Exercise

Create a script named `run_report.sh` configured by environment variables.

Requirements:

- `USER_NAME` is required.
- `APP_ENV` defaults to `dev`.
- `REPORT_DIR` defaults to `./reports`.
- `VERBOSE` defaults to `0`.
- The script creates the report directory if needed.
- The script writes a file named `report-${APP_ENV}.txt` inside `REPORT_DIR`.
- The report file includes the user name and app environment.
- If `VERBOSE=1`, the script prints the report path.

Tests to try:

```bash
chmod +x run_report.sh

./run_report.sh

USER_NAME=Ada ./run_report.sh
cat ./reports/report-dev.txt

USER_NAME=Ada APP_ENV=test REPORT_DIR=./tmp/reports VERBOSE=1 ./run_report.sh
cat ./tmp/reports/report-test.txt
```

The first run should fail with a clear message because `USER_NAME` is missing. The other runs should create report files.

## Worked Answer

One complete valid solution:

```bash
#!/usr/bin/env bash
set -euo pipefail

: "${USER_NAME:?Set USER_NAME before running this script}"

APP_ENV=${APP_ENV:-dev}
REPORT_DIR=${REPORT_DIR:-./reports}
VERBOSE=${VERBOSE:-0}

report_path="$REPORT_DIR/report-${APP_ENV}.txt"

mkdir -p "$REPORT_DIR"

{
  printf 'User: %s\n' "$USER_NAME"
  printf 'Environment: %s\n' "$APP_ENV"
} > "$report_path"

if [ "$VERBOSE" = "1" ]; then
  printf 'Wrote report: %s\n' "$report_path"
fi
```

Expected behavior:

```bash
$ ./run_report.sh
run_report.sh: line 4: USER_NAME: Set USER_NAME before running this script

$ USER_NAME=Ada ./run_report.sh
$ cat ./reports/report-dev.txt
User: Ada
Environment: dev

$ USER_NAME=Ada APP_ENV=test REPORT_DIR=./tmp/reports VERBOSE=1 ./run_report.sh
Wrote report: ./tmp/reports/report-test.txt
$ cat ./tmp/reports/report-test.txt
User: Ada
Environment: test
```

The exact line number in the missing-variable error may differ if your script has extra blank lines or comments.

## Next Step

Return to this level's README and continue with the next numbered lesson. As you read future scripts, watch for which values are local shell variables and which values are exported configuration for child commands.

## Sources Used

- GNU Bash Reference Manual, Environment: https://www.gnu.org/software/bash/manual/html_node/Environment.html
- GNU Bash Reference Manual, Bash Startup Files: https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html
- GNU Bash Reference Manual, Shell Parameter Expansion: https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html
- GNU Bash Reference Manual, Bourne Shell Builtins: https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html
- Linux manual page, `environ(7)`: https://man7.org/linux/man-pages/man7/environ.7.html
- GNU coreutils manual, `env` invocation: https://www.gnu.org/software/coreutils/manual/html_node/env-invocation.html
