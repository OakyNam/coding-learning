# Bash Intermediate

This level moves from single commands to reusable Bash scripts. Work through the lessons in order, run the examples, and keep a notes file with commands you had to debug.

## Platform Expectations

- On Windows 10/11, use WSL or Git Bash for the Bash lessons. Use WSL for process-control and cron examples when Linux-like behavior matters.
- On macOS Apple Silicon, Terminal normally starts `zsh`; type `bash` before running Bash-specific examples.
- Most worked answers use `bash script.sh` so they run before you practice executable permissions.
- Run lesson exercises from scratch directories so generated files and cleanup commands stay contained.

## Lessons

1. [Pipes and Redirection](01_pipes_and_redirection.md): connect commands, redirect streams, and understand pipeline status.
2. [Exit Codes and Status Checks](02_exit_codes_and_status_checks.md): use command statuses for control flow and error handling.
3. [Arguments and Flags](03_arguments_and_flags.md): handle positional arguments, quoting, `shift`, and `getopts`.
4. [Text Processing Basics](04_text_processing_basics.md): build small reports with `head`, `tail`, `wc`, `sort`, `uniq`, `cut`, `tr`, and `printf`.
5. [grep sed and awk](05_grep_sed_and_awk.md): select lines, transform streams, and summarize fields.
6. [Environment and Configuration](06_environment_and_configuration.md): use shell variables, exported environment variables, `PATH`, startup files, and environment-driven script configuration.
7. [Process Control](07_process_control.md): start background work, capture PIDs, wait for statuses, and clean up with traps.
8. [Cron and Scheduling](08_cron_and_scheduling.md): schedule scripts, log cron output, and account for cron's smaller environment.
9. [Robust Scripts](09_robust_scripts.md): validate inputs, quote paths, clean temporary files, and return meaningful statuses.
10. [Project Automation Scripts](10_project_automation_scripts.md): combine the level into one project task runner with safe dispatch.

## Completion Goal

By the end of this level, you should be able to explain each topic, modify the examples, and build a small project automation script that works from Windows Bash environments, macOS Apple Silicon, and Linux-like shells.

