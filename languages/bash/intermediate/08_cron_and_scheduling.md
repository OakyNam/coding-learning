# 08 - Cron and Scheduling

## Learning Goal

Learn how cron runs scheduled commands, how to write and inspect crontab entries, and how to make Bash scripts reliable when they run outside your normal terminal session.

## What Cron Is

Cron is a Unix-style scheduler for running commands at specific times. A background service called the cron daemon wakes up regularly, usually about once per minute, checks schedule files, and starts any commands whose schedule matches the current time.

A scheduled command is often called a cron job. The schedule table that contains cron jobs is called a crontab.

Cron is useful for repeated maintenance tasks, reports, backups, cleanup scripts, reminders, and other jobs that should run without someone typing the command manually.

## Platform Note

Cron is a Unix-style scheduler, so platform behavior matters.

- On Windows 10/11, practice these examples inside WSL. Native Windows scheduling uses Task Scheduler, not cron.
- Git Bash is useful for Bash syntax practice, but it does not provide the same always-on cron service model as a Linux system.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first for Bash-specific script examples. macOS includes cron, but Apple commonly steers scheduled jobs toward `launchd`.
- The worked answer uses paths derived from `$HOME`. In an actual crontab, test with `crontab -l` and logs because cron environments are intentionally small.

## Cron Schedule Syntax

Most user crontab entries use five time fields followed by the command:

```text
minute hour day-of-month month day-of-week command
```

The fields are:

- `minute`: `0` through `59`
- `hour`: `0` through `23`
- `day-of-month`: `1` through `31`
- `month`: `1` through `12`
- `day-of-week`: commonly `0` through `7`, where both `0` and `7` may mean Sunday

Common schedule examples:

```cron
* * * * * $HOME/bin/check.sh
*/15 * * * * $HOME/bin/check.sh
0 2 * * * $HOME/bin/nightly.sh
30 9 * * 1-5 $HOME/bin/weekday-report.sh
0 0 1 * * $HOME/bin/monthly.sh
```

These mean:

- `* * * * *`: every minute
- `*/15 * * * *`: every 15 minutes
- `0 2 * * *`: every day at 02:00
- `30 9 * * 1-5`: Monday through Friday at 09:30
- `0 0 1 * *`: midnight on the first day of each month

Many cron implementations also support nicknames:

```cron
@hourly $HOME/bin/hourly.sh
@daily $HOME/bin/daily.sh
@weekly $HOME/bin/weekly.sh
@monthly $HOME/bin/monthly.sh
@reboot $HOME/bin/startup.sh
```

## Cron Runs Differently Than Your Terminal

The most important cron habit is this: do not assume cron has the same environment as your terminal.

Cron runs noninteractively. It does not open your shell prompt, and Bash does not read your usual interactive startup files such as `.bashrc` for a noninteractive script. User crontabs usually provide only a small environment. Common default variables include `SHELL=/bin/sh`, `HOME`, and `LOGNAME`.

That means a command that works in your terminal can fail under cron because:

- the current working directory is different from what you expected
- your usual `PATH` is missing
- aliases and functions from `.bashrc` are unavailable
- Bash-specific syntax is being interpreted by `/bin/sh`
- environment variables from your login session are not present

Use absolute paths for commands and files when possible:

```cron
30 2 * * * /usr/bin/env bash $HOME/bin/cleanup_reports.sh
```

Or set an explicit `PATH` near the top of the crontab:

```cron
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin

30 2 * * * $HOME/bin/cleanup_reports.sh
```

For Bash scripts, include a Bash shebang:

```bash
#!/usr/bin/env bash
```

or:

```bash
#!/bin/bash
```

## Use Wrapper Scripts

Short commands can go directly in a crontab, but real work is usually easier to maintain in a wrapper script.

Instead of writing a long crontab line, put the logic in a script:

```bash
#!/usr/bin/env bash
set -euo pipefail

PATH=/usr/local/bin:/usr/bin:/bin

report_dir="$HOME/reports"
archive_dir="$HOME/reports/archive"

mkdir -p "$archive_dir"
find "$report_dir" -maxdepth 1 -type f -name '*.log' -mtime +7 -exec mv -t "$archive_dir" {} +
```

Then schedule the wrapper:

```cron
30 2 * * * /usr/bin/env bash $HOME/bin/cleanup_reports.sh
```

Wrapper scripts are easier to test manually, easier to log, and easier to put under version control.

## Logging And Output

Cron jobs do not display output in your terminal. Depending on the system, cron may email output to the crontab owner. If local mail is not configured, that output may be hard to notice.

Redirect both standard output and standard error to a log file:

```cron
30 2 * * * /usr/bin/env bash $HOME/bin/cleanup_reports.sh >> $HOME/cron-lab/logs/cleanup.log 2>&1
```

The `>>` appends standard output. The `2>&1` sends standard error to the same destination.

If you do not want cron to send mail, many cron implementations allow:

```cron
MAILTO=""
```

Keep useful messages in your script logs, especially start time, end time, and a summary of what changed.

## Editing And Inspecting Crontabs

Each user can have a user crontab. Common commands are:

```bash
crontab -e
crontab -l
crontab -r
```

Use `crontab -e` to edit your user crontab. Use `crontab -l` to list it. Be careful with `crontab -r`: it removes your crontab.

There are also system crontabs, such as `/etc/crontab` and files under directories like `/etc/cron.d/`. System cron files may include an extra user field between the schedule and the command:

```cron
30 2 * * * alex /usr/bin/env bash /home/alex/bin/cleanup_reports.sh
```

That user field is not used in a normal per-user crontab installed with `crontab -e`.

## Debugging Cron Jobs

Debug cron jobs by making cron's environment less mysterious.

First, run the script manually:

```bash
/usr/bin/env bash "$HOME/bin/cleanup_reports.sh"
```

Make sure it is executable if you plan to run it directly:

```bash
chmod +x "$HOME/bin/cleanup_reports.sh"
"$HOME/bin/cleanup_reports.sh"
```

Then test with a minimal environment similar to cron:

```bash
env -i HOME="$HOME" PATH=/usr/bin:/bin /usr/bin/env bash "$HOME/bin/cleanup_reports.sh"
```

Check your job's logs and your system cron logs. The system log location varies by distribution, but common places include `/var/log/syslog`, `/var/log/cron`, and journal logs.

For timing problems, temporarily schedule the job every minute:

```cron
* * * * * /usr/bin/env bash $HOME/bin/cleanup_reports.sh >> $HOME/cron-lab/logs/cleanup.log 2>&1
```

After confirming it works, change the schedule back.

## Common Mistakes

- Using relative paths such as `reports/input.txt` and assuming cron starts in the same directory as your terminal.
- Assuming `.bashrc` is loaded for cron jobs.
- Writing Bash-only syntax while cron runs the command with the default `SHELL=/bin/sh`.
- Forgetting to redirect standard error with `2>&1`, then missing the real error message.
- Putting inline comments after commands. A full-line comment is safe, but text after the command may become part of the command.
- Forgetting the final newline at the end of a crontab on systems that require one.
- Misunderstanding day-of-month and day-of-week matching. Depending on the cron implementation, a job can run when either field matches if both are restricted.
- Thinking `*/35` means every 35 minutes forever. In the minute field, it means minute `0` and minute `35` within each hour.
- Ignoring daylight saving time. Jobs scheduled during skipped times may not run, and jobs scheduled during repeated times may run more than once.
- Assuming cron catches up missed jobs after the computer was off. Traditional cron usually does not run missed jobs later.

## When Cron Is Not Enough

Cron is simple and widely available, but it is not the best tool for every schedule.

Use `anacron` when the machine is not always running and you want daily, weekly, or monthly jobs to run later if they were missed.

Use `systemd` timers on systemd-based Linux systems when you want stronger service integration, dependency handling, persistent timers, randomized delays, logging through the journal, or more detailed scheduling controls.

## Exercise

Create a script named `cleanup_reports.sh`.

The script should:

- use Bash strict mode
- use paths derived from `$HOME`
- create these directories if needed:
  - `$HOME/cron-lab/archive`
  - `$HOME/cron-lab/logs`
  - `$HOME/cron-lab/incoming`
- move `*.log` files older than 7 days from `incoming` to `archive`
- append a timestamped summary to a log file
- work when run manually
- work under cron
- optionally use a lock directory so two copies do not run at the same time

Then write a crontab entry that runs it every day at 02:30 and appends standard output and standard error to a log file.

## Worked Answer

Create `$HOME/bin/cleanup_reports.sh`:

```bash
mkdir -p "$HOME/bin"
cat > "$HOME/bin/cleanup_reports.sh" <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

PATH=/usr/local/bin:/usr/bin:/bin

base_dir="$HOME/cron-lab"
incoming_dir="$base_dir/incoming"
archive_dir="$base_dir/archive"
log_dir="$base_dir/logs"
summary_log="$log_dir/cleanup-summary.log"
lock_dir="$base_dir/cleanup.lock"

mkdir -p "$incoming_dir" "$archive_dir" "$log_dir"

if ! mkdir "$lock_dir" 2>/dev/null; then
  printf '[%s] cleanup already running\n' "$(date '+%Y-%m-%d %H:%M:%S')" >> "$summary_log"
  exit 0
fi

cleanup_lock() {
  rmdir "$lock_dir"
}
trap cleanup_lock EXIT

start_time=$(date '+%Y-%m-%d %H:%M:%S')
moved_count=0

while IFS= read -r -d '' file; do
  mv -- "$file" "$archive_dir/"
  moved_count=$((moved_count + 1))
done < <(find "$incoming_dir" -maxdepth 1 -type f -name '*.log' -mtime +7 -print0)

printf '[%s] moved %d old log file(s) from %s to %s\n' \
  "$start_time" "$moved_count" "$incoming_dir" "$archive_dir" >> "$summary_log"
EOF
```

Make it executable:

```bash
chmod +x "$HOME/bin/cleanup_reports.sh"
```

Create sample files and test manually:

```bash
mkdir -p "$HOME/cron-lab/incoming"
touch "$HOME/cron-lab/incoming/new.log"
touch -d '8 days ago' "$HOME/cron-lab/incoming/old.log"

/usr/bin/env bash "$HOME/bin/cleanup_reports.sh"

ls -l "$HOME/cron-lab/incoming"
ls -l "$HOME/cron-lab/archive"
cat "$HOME/cron-lab/logs/cleanup-summary.log"
```

Test with a minimal environment similar to cron:

```bash
env -i HOME="$HOME" PATH=/usr/bin:/bin /usr/bin/env bash "$HOME/bin/cleanup_reports.sh"
```

Edit your crontab:

```bash
crontab -e
```

Add this entry:

```cron
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin

30 2 * * * /usr/bin/env bash $HOME/bin/cleanup_reports.sh >> $HOME/cron-lab/logs/cleanup-cron.log 2>&1
```

Inspect the installed crontab:

```bash
crontab -l
```

## Next Step

Return to the intermediate Bash README and continue with the next lesson.

## Sources Used

- Linux `crontab(5)` manual page for crontab file format, time fields, environment variables, nicknames, and system crontab differences: https://man7.org/linux/man-pages/man5/crontab.5.html
- Linux `crontab(1)` manual page for `crontab -e`, `crontab -l`, and `crontab -r`: https://man7.org/linux/man-pages/man1/crontab.1.html
- GNU Bash manual for noninteractive shell startup behavior and `BASH_ENV`: https://www.gnu.org/software/bash/manual/bash.html
- Linux `anacron(8)` manual page for delayed execution of periodic jobs on machines that are not always running: https://man7.org/linux/man-pages/man8/anacron.8.html
- Linux `systemd.timer(5)` manual page for systemd timer units and scheduling alternatives: https://man7.org/linux/man-pages/man5/systemd.timer.5.html
- Microsoft Learn, "Install WSL": https://learn.microsoft.com/windows/wsl/install
- Git for Windows: https://gitforwindows.org/
- Apple Terminal User Guide, "Change the default shell": https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
