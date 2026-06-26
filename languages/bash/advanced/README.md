# Bash Advanced

Advanced Bash moves beyond useful shell snippets into maintainable, testable, defensive automation for real projects. The goal is to write scripts that other people can run safely, debug quickly, review confidently, and adapt without guessing what hidden shell behavior will do.

This level focuses on production habits: strict error handling, cleanup, portability decisions, performance, parallel execution, text processing, security, CI, deployment automation, and script structure.

## Prerequisites

Before starting, you should be comfortable with:

- Navigation and common commands: `cd`, `pwd`, `ls`, `cp`, `mv`, `rm`, `mkdir`, `chmod`, `find`.
- Quoting rules, variables, command substitution, and exit statuses.
- Redirection, pipelines, functions, loops, and conditionals.
- Basic text tools: `grep`, `sed`, `awk`, `sort`, `uniq`, and `xargs`.
- Reading `man` pages and command help output.
- Working in a Unix-like shell environment.

## Recommended Environment

- Bash 4+ is the baseline; Bash 5+ is recommended where available.
- Use [ShellCheck](https://www.shellcheck.net/) while writing scripts.
- Use [bats-core](https://bats-core.readthedocs.io/) when a script needs automated tests.
- Use [GNU Parallel](https://www.gnu.org/software/parallel/) when parallel jobs outgrow simple background-process patterns.

Bash-specific features can make scripts clearer and safer, especially arrays, `[[ ... ]]`, `mapfile`, associative arrays, and improved parameter expansion. POSIX `sh` portability matters when scripts must run on minimal systems, non-Bash `/bin/sh`, containers, or deployment targets you do not control. Make the portability choice explicit in each script with the shebang, documentation, and tests.

## Suggested Learning Order

1. [Strict Mode](01_strict_mode.md): Start by making failure visible and predictable before adding more automation.
2. [Traps and Signals](02_traps_and_signals.md): Add cleanup and interruption handling so scripts leave the system in a known state.
3. [Portable Shell Scripts](03_portable_shell_scripts.md): Decide when to write Bash and when to restrict yourself to POSIX shell.
4. [Performance](04_performance.md): Learn where shell scripts become slow and how to avoid unnecessary process churn.
5. [Parallel Work](05_parallel_work.md): Build on performance by running independent work safely in parallel.
6. [Advanced Text Pipelines](06_advanced_text_pipelines.md): Strengthen the data-processing core that many shell automations depend on.
7. [Security for Scripts](07_security_for_scripts.md): Protect scripts from unsafe input, secret leakage, and accidental destructive behavior.
8. [CI Automation](08_ci_automation.md): Put scripts under repeatable checks so regressions are caught early.
9. [Deployment Scripts](09_deployment_scripts.md): Apply the earlier safety and reliability habits to higher-risk release workflows.
10. [Script Architecture](10_script_architecture.md): Finish by organizing larger scripts into maintainable command-line tools.
11. [Defensive Input Validation](11_defensive_input_validation.md): Validate external values before they affect shell behavior.
12. [Structured Data Interchange](12_structured_data_interchange.md): Exchange JSON and CSV without unsafe text parsing.
13. [Logging And Observability](13_logging_and_observability.md): Make automation failures diagnosable without leaking secrets.

## Lesson Outcomes

| Lesson | Outcome |
| --- | --- |
| [Strict Mode](01_strict_mode.md) | Use strict Bash settings deliberately, understand their edge cases, and write scripts that fail with useful diagnostics. |
| [Traps and Signals](02_traps_and_signals.md) | Handle `EXIT`, `INT`, and `TERM` paths, clean up temporary resources, and preserve meaningful exit statuses. |
| [Portable Shell Scripts](03_portable_shell_scripts.md) | Choose between Bash and POSIX shell, document the choice, and avoid accidental non-portable behavior. |
| [Performance](04_performance.md) | Identify slow shell patterns, reduce needless subprocesses, and measure changes before optimizing further. |
| [Parallel Work](05_parallel_work.md) | Run concurrent jobs with bounded parallelism, collect failures, and avoid unsafe shared-state writes. |
| [Advanced Text Pipelines](06_advanced_text_pipelines.md) | Build reliable pipelines for filtering, transforming, grouping, and summarizing text data. |
| [Security for Scripts](07_security_for_scripts.md) | Quote defensively, validate inputs, avoid command injection, protect secrets, and make destructive actions explicit. |
| [CI Automation](08_ci_automation.md) | Run linting and tests in CI, return clear statuses, and produce logs useful for debugging failed automation. |
| [Deployment Scripts](09_deployment_scripts.md) | Write deployment helpers with preflight checks, dry-run behavior, rollback awareness, and clear operator output. |
| [Script Architecture](10_script_architecture.md) | Organize larger scripts into small functions, parse options cleanly, separate policy from mechanics, and document commands. |

## Capstone Project

Build `maintain-repo.sh`, a project-maintenance CLI that can be run locally or in CI.

The command should support:

- `--path PATH`: repository or project directory to inspect.
- `--dry-run`: print intended cleanup or repair actions without changing files.
- `--jobs N`: limit parallel checks.
- `--report FILE`: write a machine-readable or plain-text summary report.
- `--help`: show usage, options, examples, and exit status meanings.

The script should:

- Scan project files while skipping ignored or generated paths.
- Run checks such as ShellCheck, formatting checks, test commands, or repository-specific validation.
- Summarize pass, warning, and failure counts.
- Clean temporary files created by the script.
- Return meaningful statuses for success, invalid arguments, failed checks, and interrupted runs.
- Keep destructive or cleanup behavior behind explicit, reviewable logic, with `--dry-run` proving what would happen.

## Capstone Acceptance Evidence

Submit the script and evidence that it is safe to run:

- ShellCheck passes, or every suppression is documented with a reason.
- `./maintain-repo.sh --help` shows complete usage and status information.
- `--dry-run` performs no destructive action.
- Interrupt handling cleans up temporary files and exits with a meaningful status.
- Tests cover a successful run, invalid arguments, and at least one failure path.

## Completion Criteria

You are done with this level when you can:

- Explain the tradeoffs behind Bash strict mode, traps, portability, and parallelism.
- Read and review a non-trivial Bash script for quoting, cleanup, status handling, and unsafe input.
- Add ShellCheck and tests to a script before it becomes risky to change.
- Build and defend the `maintain-repo.sh` capstone with the acceptance evidence above.
- State when Bash is the right tool and when a general-purpose language would be more maintainable.

## References

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/bash.html)
- [POSIX Shell Command Language](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [ShellCheck](https://www.shellcheck.net/)
- [bats-core Documentation](https://bats-core.readthedocs.io/)
- [GNU Parallel Tutorial](https://www.gnu.org/software/parallel/parallel_tutorial.html)
- [GNU Parallel Documentation](https://www.gnu.org/software/parallel/)
