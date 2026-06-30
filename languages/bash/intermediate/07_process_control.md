# 07 - Process Control

## Learning Goal

Start background commands, capture PIDs with `$!`, wait for completion, inspect exit statuses, and clean up background work with signals and traps.

## Why It Matters

Shell scripts often need to do several slow checks at once: ping a service, generate a report, copy files, or run test shards. Process control lets one script start those tasks, keep track of them, wait for the right results, and stop unfinished work if the script is interrupted.

Without process control, scripts tend to leave stray background commands behind or report success before the real work has finished.

## Platform Note

This lesson uses Bash background processes, PIDs, signals, and traps.

- On Windows 10/11, prefer WSL for these examples. Git Bash supports many examples, but process IDs and signals can behave differently from Linux.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- Run the examples from a scratch directory and use short-lived commands such as `sleep` so cleanup is easy to observe.
- Avoid testing signal examples against important real processes. The exercises start their own child processes and clean those up.

## Core Idea

Adding `&` after a command starts it in the background:

```bash
long_command &
```

Bash immediately continues to the next line. The special parameter `$!` contains the process ID of the most recent background pipeline. Save it immediately:

```bash
long_command &
pid=$!
```

Then use `wait "$pid"` to wait for that child and make `$?` hold the child's final exit status.

## Background Jobs

A background command runs while the shell keeps going. This is useful for independent work, but it changes what the shell knows at each step.

```bash
sleep 5 &
pid=$!
echo "started sleep as $pid"
wait "$pid"
echo "sleep finished with status $?"
```

The `sleep 5 &` line only means Bash successfully started the command in the background. It does not mean `sleep` has finished successfully. The final result is available after `wait`.

## Jobs vs PIDs

Bash has two related ways to talk about background work:

- A PID is an operating system process ID, such as `12345`.
- A job number is Bash's interactive job table entry, such as `%1`.

The `jobs` builtin shows background and stopped jobs known to the current shell:

```bash
jobs
jobs -l
```

Job specs like `%1` are mainly for interactive use with commands such as `fg`, `bg`, and `jobs`. Scripts should store PIDs from `$!` and use those PIDs with `wait` and `kill`. PIDs are direct, explicit, and do not depend on the interactive job table.

## Waiting For Work

Use `wait PID` when you want the status for one specific background child:

```bash
run_check &
pid=$!

wait "$pid"
status=$?
```

Save `$?` immediately after `wait`; another command, even `echo`, will replace it.

To wait for several children and preserve each status, store the PIDs and wait for them one by one:

```bash
names=(api db cache)
pids=()
statuses=()

(sleep 2; exit 0) &
pids+=("$!")

(sleep 3; exit 1) &
pids+=("$!")

(sleep 1; exit 0) &
pids+=("$!")

for i in "${!pids[@]}"; do
  if wait "${pids[$i]}"; then
    statuses[$i]=0
  else
    statuses[$i]=$?
  fi

  echo "${names[$i]} finished with status ${statuses[$i]}"
done
```

Plain `wait` with no PID waits for all active background children and returns a combined status that is not enough when you need to report each child's result. Bash also has `wait -n`, which waits for the next child to finish; it is useful for worker pools, but it is Bash-specific and needs extra bookkeeping if you want to match each result back to a name.

## Signals And Cleanup

Signals are messages sent to processes. The ones you will see most often are:

- `SIGTERM`: a polite request to terminate. `kill "$pid"` sends `SIGTERM` by default.
- `SIGINT`: the interrupt signal, usually sent by pressing `Ctrl-C`.
- `SIGKILL`: an immediate kill signal. It cannot be trapped or ignored, so it should not be your first choice.

A cleanup function can stop background work before the script exits:

```bash
cleanup() {
  if [[ -n ${pid:-} ]] && kill -0 "$pid" 2>/dev/null; then
    kill "$pid" 2>/dev/null || true
  fi
}

trap cleanup EXIT INT TERM
```

`EXIT` runs the cleanup when the shell exits normally or because of an error. `INT` and `TERM` let the same cleanup run when the user interrupts the script or another process asks it to terminate. If cleanup needs different exit codes for interrupts, use a small signal handler that calls `cleanup` and then exits with the intended status.

## Example

This script starts three background tasks, stores their PIDs immediately, waits for each one, and reports the final status.

```bash
#!/usr/bin/env bash

names=(api db cache)
pids=()
statuses=()

(sleep 2; exit 0) &
pids+=("$!")
echo "started ${names[0]} as ${pids[0]}"

(sleep 3; exit 1) &
pids+=("$!")
echo "started ${names[1]} as ${pids[1]}"

(sleep 1; exit 0) &
pids+=("$!")
echo "started ${names[2]} as ${pids[2]}"

failed=0

for i in "${!pids[@]}"; do
  if wait "${pids[$i]}"; then
    statuses[$i]=0
  else
    statuses[$i]=$?
    failed=1
  fi

  echo "${names[$i]} finished with status ${statuses[$i]}"
done

exit "$failed"
```

The tasks run in parallel even though the script waits for them in array order. `cache` finishes first, but the script reports statuses in the order stored in `names`.

## Common Mistakes

- Forgetting to save `$!` immediately after `cmd &`.
- Using `$?` too late after `wait`; another command has already overwritten it.
- Assuming the status of `cmd &` is the child's final status.
- Defaulting to `kill -9`; try `SIGTERM` first with plain `kill "$pid"`.
- Relying on `%1` in scripts instead of storing PIDs.
- Starting background work without a cleanup trap.

## Exercise

Create `parallel_checks.sh`.

The script should:

- Start three background checks.
- Make `api` sleep 2 seconds and succeed.
- Make `db` sleep 3 seconds and fail.
- Make `cache` sleep 1 second and succeed.
- Save each PID immediately.
- Print each check name and PID after starting it.
- Wait for each check and report its exit status.
- Exit `1` if any check failed; otherwise exit `0`.
- Trap cleanup on `Ctrl-C` and `TERM`, and clean up unfinished background checks on exit.
- Manage PIDs directly. Do not use `fg` or `bg`.

## Worked Answer

Run this in Bash from a scratch directory to create `parallel_checks.sh`:

```bash
cat > parallel_checks.sh <<'EOF'
#!/usr/bin/env bash

names=()
pids=()
statuses=()

cleanup() {
  for pid in "${pids[@]:-}"; do
    if kill -0 "$pid" 2>/dev/null; then
      kill "$pid" 2>/dev/null || true
    fi
  done
}

on_signal() {
  echo "received signal; stopping unfinished checks" >&2
  cleanup
  exit 130
}

start_check() {
  local name=$1
  local seconds=$2
  local status=$3

  (sleep "$seconds"; exit "$status") &
  local pid=$!

  names+=("$name")
  pids+=("$pid")

  echo "started $name as $pid"
}

trap cleanup EXIT
trap on_signal INT TERM

start_check api 2 0
start_check db 3 1
start_check cache 1 0

failed=0

for i in "${!pids[@]}"; do
  name=${names[$i]}
  pid=${pids[$i]}

  if wait "$pid"; then
    status=0
  else
    status=$?
    failed=1
  fi

  statuses[$i]=$status
  echo "$name finished with status $status"
done

exit "$failed"
EOF
```

Run it with Bash:

```bash
bash parallel_checks.sh
```

Expected behavior:

- It prints the names and PIDs for `api`, `db`, and `cache`.
- It reports `api` with status `0`, `db` with status `1`, and `cache` with status `0`.
- Because `db` fails, the script exits with status `1`.
- If interrupted, it sends `SIGTERM` to any still-running checks before exiting.

## Next Step

Return to this level's README and continue with the next numbered lesson. When you write real scripts, practice saving `$!` immediately and writing the cleanup trap before adding more background commands.

## Sources Used

- GNU Bash Reference Manual, [Job Control Basics](https://www.gnu.org/software/bash/manual/html_node/Job-Control-Basics.html)
- GNU Bash Reference Manual, [Job Control Builtins](https://www.gnu.org/software/bash/manual/html_node/Job-Control-Builtins.html)
- GNU Bash Reference Manual, [Lists of Commands](https://www.gnu.org/software/bash/manual/html_node/Lists.html)
- GNU Bash Reference Manual, [Special Parameters](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)
- GNU Bash Reference Manual, [Signals](https://www.gnu.org/software/bash/manual/html_node/Signals.html)
- GNU Bash Reference Manual, [`trap` in Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html)
- Microsoft Learn, "Install WSL": https://learn.microsoft.com/windows/wsl/install
- Git for Windows: https://gitforwindows.org/
- Apple Terminal User Guide, "Change the default shell": https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
