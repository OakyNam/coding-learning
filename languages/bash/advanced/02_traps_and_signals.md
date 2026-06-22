# 02 - Traps and Signals

## Learning Goal

Write Bash scripts that clean temporary resources and background child processes when they exit normally, fail, or receive `INT`/`TERM`.

## Why It Matters

Long-running shell scripts often create temporary directories, open log files, start background workers, or hold locks. If the script exits without cleanup, the next run may inherit stale files or orphaned child processes.

Signals and traps let a script say, "before I leave, run this cleanup code." They are especially useful for scripts that supervise other commands, create temporary workspaces, or need predictable behavior when a user presses Ctrl-C.

## Mental Model

A signal is a small notification sent to a process. Common examples are:

- `INT`: interrupt, often sent by Ctrl-C.
- `TERM`: polite termination request, often sent by `kill PID` or service managers.
- `KILL`: immediate termination request that cannot be caught, ignored, or cleaned up.
- `STOP`: suspension request that also cannot be caught or ignored.

Bash `trap` installs shell code to run when the shell receives a signal or a Bash pseudo-signal.

Important pseudo-signals:

- `EXIT`: runs when the shell exits, whether the exit is success, failure, or a handled signal.
- `ERR`: runs when a command failure would cause the shell to exit under `errexit`.

Trap actions run in the shell process, not in a separate cleanup process. That means they can read and change shell variables, including arrays of child PIDs. It also means a cleanup function can accidentally overwrite `$?`, so preserve the original status first.

## Basic Syntax

```bash
trap 'echo "leaving"' EXIT
trap 'echo "interrupted"; exit 130' INT
trap 'echo "terminating"; exit 143' TERM

# Restore the default behavior for a signal.
trap - INT

# Ignore a signal.
trap '' TERM

# List signal names and numbers on this system.
trap -l
```

The first argument is code for Bash to evaluate when the signal arrives. Single quotes are common because they delay variable expansion until the trap actually runs:

```bash
tmpdir=$(mktemp -d)
trap 'rm -rf "$tmpdir"' EXIT
```

If you wrote `trap "rm -rf $tmpdir" EXIT`, `$tmpdir` would expand when the trap is installed. That is sometimes fine, but it is easy to get wrong when variables change later.

## EXIT Trap For Cleanup

An `EXIT` trap is the standard place to remove temporary files:

```bash
#!/usr/bin/env bash
set -euo pipefail

tmpdir=$(mktemp -d)

cleanup() {
  local status=$?
  rm -rf "$tmpdir" || true
  exit "$status"
}

trap cleanup EXIT

printf 'work area: %s\n' "$tmpdir"
printf 'result\n' > "$tmpdir/output.txt"
```

The first line in `cleanup` saves the original exit status. Without `local status=$?`, a successful `rm` could hide an earlier failure, or a failed cleanup command could replace the script's real result.

Use `|| true` for cleanup commands whose failure should not replace the original status. Cleanup should be best effort unless the cleanup result is the main thing the script is reporting.

## ERR Trap

`ERR` is a Bash pseudo-signal. It runs when a command failure would trigger `errexit`.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

on_error() {
  local status=$?
  printf 'failed with status %s at line %s\n' "$status" "$LINENO" >&2
  exit "$status"
}

trap on_error ERR

cp missing-file.txt /tmp/example
```

`set -E` enables `errtrace`, which lets the `ERR` trap be inherited by shell functions, command substitutions, and subshells. Without it, failures inside functions often surprise learners because the top-level `ERR` trap may not run where they expect.

`ERR` follows the same major exceptions as `errexit`. It does not run for every nonzero status. For example, failures used as conditions are usually expected and do not trigger `ERR`:

```bash
if grep -q 'needle' file.txt; then
  echo 'found'
fi

grep -q 'needle' file.txt || echo 'not found'
```

Those `grep` failures are part of control flow, so Bash does not treat them like unhandled script failures.

## INT And TERM

`INT` usually means the user pressed Ctrl-C. A script that exits because of `INT` conventionally exits with status `130`, which is `128 + 2` because `SIGINT` is signal 2.

`TERM` usually means a polite request to stop. A script that exits because of `TERM` conventionally exits with status `143`, which is `128 + 15` because `SIGTERM` is signal 15.

```bash
on_int() {
  exit 130
}

on_term() {
  exit 143
}

trap on_int INT
trap on_term TERM
```

If an `EXIT` trap is also installed, calling `exit 130` or `exit 143` from the signal trap will still run the `EXIT` cleanup.

## Background Child Cleanup

When a script starts background children, the shell does not automatically terminate them just because the parent script is cleaning up. Store each PID from `$!`, then terminate and wait for those children during cleanup.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

pids=()

start_worker() {
  while true; do
    date
    sleep 2
  done &
  pids+=("$!")
}

cleanup() {
  local status=$?
  trap - EXIT INT TERM ERR

  if ((${#pids[@]})); then
    kill -TERM "${pids[@]}" 2>/dev/null || true
    wait "${pids[@]}" 2>/dev/null || true
  fi

  exit "$status"
}

trap cleanup EXIT
trap 'exit 130' INT
trap 'exit 143' TERM

start_worker
start_worker
wait "${pids[@]}"
```

Key details:

- `$!` is the PID of the most recent background pipeline.
- Use an array so multiple children can be cleaned up together.
- Send `TERM` first so children get a polite shutdown chance.
- `wait` after `kill` so the script reaps children before exiting.
- Ignore `kill` and `wait` cleanup failures so the original status survives.
- Disable traps inside cleanup with `trap - ...` to avoid recursive cleanup if cleanup itself receives a signal or exits.

## Signal Limits And Timing

Traps are powerful, but they are not instant magic.

- `SIGKILL` and `SIGSTOP` cannot be caught, ignored, or handled. No Bash cleanup trap can run for them.
- A trap runs between Bash commands, not in the middle of arbitrary machine code. If Bash is waiting for an external foreground command, trap timing depends on the command and Bash's wait behavior.
- A signal sent to the shell is not always sent to every child. Terminal-generated Ctrl-C is commonly delivered to the foreground process group, but `kill "$script_pid"` targets only that process.
- Children can ignore `TERM`, exit before cleanup reaches them, or start their own descendants. That is why cleanup code should tolerate missing PIDs and failed waits.
- `EXIT` is reliable for normal shell exits, explicit `exit`, and failures handled by `errexit`, but it cannot help if the process is forcibly killed.

## Common Mistakes

- Forgetting to preserve `$?` at the start of cleanup, so cleanup changes the script's exit status.
- Using double quotes in a trap action and accidentally expanding variables when the trap is installed instead of when it runs.
- Assuming `ERR` catches every nonzero status. It follows `errexit` rules and skips expected failures in places like `if`, `while`, `until`, `&&`, and `||` control flow.
- Installing an `ERR` trap without `set -E` and then expecting it to behave consistently inside functions and subshells.
- Killing background children but not waiting for them.
- Waiting for children before terminating them in cleanup, which can make cleanup hang.
- Letting `rm`, `kill`, or `wait` failures inside cleanup replace the original exit status.
- Expecting traps to catch `SIGKILL` or `SIGSTOP`.
- Forgetting that traps run in the shell, so changing variables, directories, or options inside a trap affects the script.

## Exercise

Write `supervised_job.sh`.

Requirements:

- Use `mktemp -d` to create a temporary directory.
- Start two background workers.
- Each worker writes heartbeat lines to its own log file in the temporary directory.
- Store worker PIDs in a PID array using `$!`.
- Install traps for `EXIT`, `ERR`, `INT`, and `TERM`.
- On cleanup, terminate workers with `kill -TERM`, wait for them, remove the temporary directory, and preserve the original exit status.
- Use conventional signal statuses: `130` for `INT`, `143` for `TERM`.
- Ignore cleanup failures so the original status survives.
- Add a `--fail` path that starts the workers and then intentionally fails.

Try these runs:

```bash
bash supervised_job.sh
bash supervised_job.sh --fail
bash supervised_job.sh
# Press Ctrl-C during the final run.
```

## Worked Answer

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

tmpdir=$(mktemp -d)
pids=()

cleanup() {
  local status=$?
  trap - EXIT ERR INT TERM

  if ((${#pids[@]})); then
    kill -TERM "${pids[@]}" 2>/dev/null || true
    wait "${pids[@]}" 2>/dev/null || true
  fi

  rm -rf "$tmpdir" || true
  exit "$status"
}

on_error() {
  local status=$?
  printf 'supervised_job.sh failed with status %s\n' "$status" >&2
  exit "$status"
}

on_int() {
  exit 130
}

on_term() {
  exit 143
}

worker() {
  local name=$1
  local log_file=$2

  while true; do
    printf '%s heartbeat %(%Y-%m-%dT%H:%M:%S%z)T\n' "$name" -1 >> "$log_file"
    sleep 1
  done
}

start_worker() {
  local name=$1
  local log_file="$tmpdir/${name}.log"

  worker "$name" "$log_file" &
  pids+=("$!")
}

trap cleanup EXIT
trap on_error ERR
trap on_int INT
trap on_term TERM

printf 'temporary directory: %s\n' "$tmpdir"

start_worker worker_one
start_worker worker_two

if [[ ${1:-} == "--fail" ]]; then
  false
fi

sleep 5
printf 'worker logs were written in %s\n' "$tmpdir"
```

Explanation:

- `tmpdir=$(mktemp -d)` creates a unique temporary workspace.
- `pids+=("$!")` records each background worker immediately after it starts.
- `cleanup` saves `$?` before running any other command.
- `trap - EXIT ERR INT TERM` prevents trap recursion during cleanup.
- `kill -TERM` asks workers to stop, and `wait` reaps them.
- `2>/dev/null || true` makes cleanup tolerate workers that already exited.
- `on_int` and `on_term` convert handled signals to the conventional statuses `130` and `143`.
- The `--fail` path runs `false`, which triggers the `ERR` trap because the script uses `set -E` and `errexit`.

## Next Step

Return to this level's README and continue with the next numbered lesson. As you read other Bash scripts, look for `trap cleanup EXIT`; it is one of the most common signs that a script owns temporary resources.

## Sources Used

- GNU Bash Manual, Bourne Shell Builtins: `trap`, `exit` - https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html
- GNU Bash Manual, The Set Builtin: `errexit`, `errtrace`, `set -E` - https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html
- GNU Bash Manual, Signals - https://www.gnu.org/software/bash/manual/html_node/Signals.html
- GNU Bash Manual, Job Control Builtins: `wait` - https://www.gnu.org/software/bash/manual/html_node/Job-Control-Builtins.html
- Linux manual page `signal(7)` - https://man7.org/linux/man-pages/man7/signal.7.html
