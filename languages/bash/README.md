# Bash

Bash is both an interactive command language and a scripting language. You use it directly in a terminal to run commands, inspect files, start tools, and combine programs. You also use it in scripts to automate repeatable command-line workflows.

Bash is a good fit for composing Unix tools, file operations, CI and helper scripts, small system tasks, and glue code around programs such as `git`, `curl`, `jq`, `grep`, `find`, `sed`, and `awk`. It is not a good fit for large applications, complex parsing, data-heavy logic, or programs that need rich data structures and strong error handling. When a shell script starts growing into an application, reach for Python, Go, JavaScript, or another general-purpose language.

## Prerequisites

- A terminal and basic comfort running commands.
- A working understanding of files, directories, paths, and environment variables.
- A text editor for writing scripts.
- Bash through Linux, macOS, WSL, Git Bash, or a dev container.
- Optional but recommended: Git for version control and ShellCheck for linting shell scripts.

## What You Will Learn

- Commands, flags, manual pages, `help`, `type`, and command discovery.
- Files, directories, permissions, executables, shebangs, and `chmod`.
- Variables, quoting, command substitution, parameter expansion, and arithmetic.
- Exit statuses, `if`, `case`, `test`, `[[ ... ]]`, loops, and short-circuit operators.
- Pipes, redirection, standard input, standard output, standard error, and process composition.
- Functions, script arguments, positional parameters, arrays, and return values.
- Safe scripting with strict modes, quoted expansions, arrays for argument lists, temporary files, and cleanup.
- Debugging with `set -x`, `bash -n`, ShellCheck, and small reproducible test cases.
- Practical tools including `grep`, `find`, `sed`, `awk`, `jq`, `curl`, and `git`.

## Suggested Learning Order

1. **Beginner:** Learn the terminal workflow first: commands, paths, help, files, permissions, variables, quoting, pipes, redirects, and simple scripts.
2. **Intermediate:** Write useful automation with functions, arguments, loops, conditionals, arrays, exit statuses, and safer handling of files and command output.
3. **Advanced:** Focus on robust scripting: error handling, debugging, ShellCheck, portability, traps, process control, CI scripts, and knowing when Bash is the wrong tool.

## Levels

- [Beginner](beginner/README.md): start here for terminal basics, command syntax, file operations, quoting, pipes, redirects, and first scripts.
- [Intermediate](intermediate/README.md): build practical scripts with arguments, functions, loops, conditionals, arrays, tool composition, and safer workflows.
- [Advanced](advanced/README.md): improve reliability with debugging, linting, traps, portability tradeoffs, CI usage, and script design limits.

## Project Milestones

- Build a file organizer that sorts files by extension, date, or naming pattern.
- Build a log summarizer that uses `grep`, `awk`, `sort`, and `uniq` to report common errors.
- Build a Git helper that wraps repeated branch, status, commit, or cleanup workflows.
- Build an API fetcher that uses `curl` plus `jq` to request JSON and extract useful fields.
- Build a backup script that copies selected directories, validates inputs, writes progress to stderr, and exits clearly on failure.

## Connections To Shared Topics

- **Git:** use Bash for repeatable repository checks, commit helpers, branch cleanup, and CI automation.
- **JSON:** use `jq` instead of trying to parse JSON with shell string operations.
- **Testing:** run scripts with sample inputs, check exit statuses, and add automated checks for important workflows.
- **APIs:** combine `curl`, environment variables, headers, exit statuses, and `jq` to call HTTP APIs safely.
- **Security:** quote expansions, avoid `eval`, handle secrets carefully, validate inputs, and prefer least-privilege file permissions.
- **Linux and filesystems:** practice paths, permissions, globbing, symlinks, executables, standard streams, and process behavior.

## Style And Safety

- Start scripts with a clear shebang such as `#!/usr/bin/env bash`.
- Quote expansions by default: use `"$name"` instead of `$name`.
- Use arrays for command arguments instead of building command strings.
- Send errors and progress messages to stderr with `>&2`.
- Check exit statuses and fail clearly when required commands or files are missing.
- Run `bash -n script.sh` for syntax checks and ShellCheck for common bugs.
- Keep shell scripts small. If parsing, state management, or data logic becomes complex, switch languages.

## References

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/bash.html)
- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- [ShellCheck site](https://www.shellcheck.net/)
- [ShellCheck README](https://github.com/koalaman/shellcheck/blob/master/README.md)
- [BashGuide](https://mywiki.wooledge.org/BashGuide)
- [BashPitfalls](https://mywiki.wooledge.org/BashPitfalls)
