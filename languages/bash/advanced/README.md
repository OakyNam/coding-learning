# Bash Advanced

Advanced Bash is the path from useful shell snippets to maintainable automation. This level covers strict mode, traps, portability, performance, parallelism, text pipelines, security, CI automation, deployment scripts, and script architecture.

Use this README as navigation for the completed lessons in this level. Lessons `11`-`13` are not part of the recommended path yet.

## Prerequisites

Before starting, you should be comfortable with:

- Shell navigation and files: `cd`, `pwd`, `ls`, `cp`, `mv`, `rm`, `mkdir`, `chmod`, and `find`.
- Variables, quoting, command substitution, and exit statuses.
- Redirection, pipelines, functions, loops, and conditionals.
- Basic text tools: `grep`, `sed`, `awk`, `sort`, `uniq`, `find`, and `xargs`.
- Reading `man` pages and `--help` output.
- Working in a Unix-like Bash environment.

## Environment

These lessons teach Bash, not PowerShell or zsh.

- Windows 10/11 learners should use WSL, Git Bash, or another Bash environment. PowerShell can launch WSL, but Bash snippets should run inside Bash.
- macOS Apple Silicon Terminal defaults to zsh. Run course scripts with a Bash shebang or with `bash script.sh`.
- Bash 4+ is a reasonable baseline; Bash 5+ is preferable when available.
- POSIX `/bin/sh` scripts must avoid Bash-only features. Make the Bash vs POSIX choice explicit in the shebang, documentation, and tests.
- Use ShellCheck for linting.
- Use bats-core when scripts need automated tests.
- Use GNU Parallel only when the job has grown beyond simple background jobs or `xargs -P`.

## Suggested Learning Order

1. [01 - Strict Mode](01_strict_mode.md): Make failures visible and predictable before adding larger automation.
2. [02 - Traps and Signals](02_traps_and_signals.md): Add cleanup and interruption handling so scripts leave systems in known states.
3. [03 - Portable Shell Scripts](03_portable_shell_scripts.md): Decide when a script should be Bash and when it should stay POSIX-compatible.
4. [04 - Bash Performance](04_performance.md): Recognize slow shell patterns and reduce unnecessary work.
5. [05 - Parallel Work](05_parallel_work.md): Run independent work concurrently without losing errors or corrupting shared state.
6. [06 - Advanced Text Pipelines](06_advanced_text_pipelines.md): Build reliable filtering, transformation, grouping, and reporting pipelines.
7. [07 - Security for Scripts](07_security_for_scripts.md): Protect scripts from unsafe input, secret leakage, and accidental destructive behavior.
8. [08 - CI Automation](08_ci_automation.md): Run scripts under repeatable checks so regressions are caught early.
9. [09 - Deployment Scripts](09_deployment_scripts.md): Apply safety habits to higher-risk release and operations workflows.
10. [10 - Script Architecture](10_script_architecture.md): Organize larger scripts into maintainable command-line tools.

## Lesson Outcomes

| Lesson | Outcome |
| --- | --- |
| [01 - Strict Mode](01_strict_mode.md) | Use strict Bash settings deliberately, explain their tradeoffs, and keep failures diagnosable. |
| [02 - Traps and Signals](02_traps_and_signals.md) | Handle `EXIT`, `INT`, and `TERM`, clean up temporary resources, and preserve useful exit statuses. |
| [03 - Portable Shell Scripts](03_portable_shell_scripts.md) | Choose Bash or POSIX shell intentionally and avoid accidental non-portable behavior. |
| [04 - Bash Performance](04_performance.md) | Measure scripts, reduce avoidable subprocesses, and improve slow paths without guessing. |
| [05 - Parallel Work](05_parallel_work.md) | Use bounded parallelism, collect failures, and avoid unsafe shared writes. |
| [06 - Advanced Text Pipelines](06_advanced_text_pipelines.md) | Compose text pipelines that filter, transform, aggregate, and report predictably. |
| [07 - Security for Scripts](07_security_for_scripts.md) | Review quoting, input handling, secrets, destructive actions, cleanup, and command construction. |
| [08 - CI Automation](08_ci_automation.md) | Add linting, tests, clear statuses, and useful logs to automated checks. |
| [09 - Deployment Scripts](09_deployment_scripts.md) | Build deployment helpers with preflight checks, dry-run behavior, operator prompts, and rollback awareness. |
| [10 - Script Architecture](10_script_architecture.md) | Structure larger scripts with focused functions, option parsing, usage output, and separated policy. |

## Optional Capstone

Build `maintain-repo.sh`, a local and CI-friendly repository maintenance CLI.

Suggested options:

- `--path PATH`: repository or project directory to inspect.
- `--dry-run`: print intended cleanup or repair actions without changing files.
- `--jobs N`: limit parallel checks.
- `--report FILE`: write a machine-readable or plain-text summary.
- `--help`: show usage, options, examples, and exit status meanings.

The capstone should demonstrate strict mode, traps, portability choices, performance awareness, bounded parallelism, text pipelines, safe input handling, CI integration, deployment-style safety gates, and maintainable script architecture.

## Completion Criteria

You are done with this level when you can:

- Explain strict mode tradeoffs and choose the settings a script actually needs.
- Use traps for cleanup, interruption handling, and meaningful final status.
- Choose Bash or POSIX shell deliberately and defend that choice.
- Review scripts for quoting bugs, unsafe input, secret handling, cleanup, and status behavior.
- Add ShellCheck and tests before a script becomes risky to change.
- Build and defend the `maintain-repo.sh` capstone.

## References

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/bash.html)
- [POSIX Shell Command Language](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)
- [Microsoft WSL documentation](https://learn.microsoft.com/windows/wsl/)
- [Apple Terminal User Guide](https://support.apple.com/guide/terminal/welcome/mac)
- [Apple support: Use zsh as the default shell on your Mac](https://support.apple.com/en-us/102360)
- [ShellCheck](https://www.shellcheck.net/)
- [bats-core Documentation](https://bats-core.readthedocs.io/)
- [GNU Parallel](https://www.gnu.org/software/parallel/)
