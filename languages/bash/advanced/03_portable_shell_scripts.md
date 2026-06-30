# 03 - Portable Shell Scripts

## Learning Goal

Choose POSIX `sh` or Bash deliberately, write syntax that matches that target, check required commands with `command -v`, and test the script with the shell you claim it supports.

## Target Contract

A portable shell script keeps a contract:

1. The shebang names the intended interpreter.
2. The syntax matches that interpreter.
3. The external commands and options are available where the script will run.
4. The tests run under the same shell family the script claims to support.

For POSIX `sh`, avoid Bash-only features such as:

- arrays: `items=(one two)` and `"${items[@]}"`
- `[[ ... ]]`
- `source file.sh`
- process substitution: `<(command)` or `>(command)`
- here-strings: `command <<< "$value"`
- Bash parameter case conversion: `${name^^}` or `${name,,}`
- Bash-only `printf` formats such as `%q`

For Bash, those features can be fine. The bug is pretending Bash is POSIX `sh`.

## What Portable Means

The most common portable target is POSIX `sh`. POSIX defines a shell command language and standard utilities, but it is smaller than Bash and leaves out many Bash conveniences.

The path `/bin/sh` does not always mean Bash. Depending on the system, `/bin/sh` might be `dash`, `ash`, `ksh`, another POSIX-oriented shell, or Bash running in POSIX mode.

Portability is not only shell syntax. A script can use valid POSIX `sh` syntax and still fail because it depends on a nonportable external utility option. GNU and BSD systems often differ in details for tools such as `sed`, `date`, `grep`, `xargs`, and `readlink`.

```mermaid
flowchart LR
    A[Choose target shell] --> B[Choose shebang]
    B --> C[Use matching syntax]
    C --> D[Check required commands]
    D --> E[Lint and run under that shell]
```

## Choose The Right Shebang

The shebang is the first line of an executable script. Treat it as a contract with the person, scheduler, or operating system running your script.

Use `#!/bin/sh` when the script is written in POSIX `sh` syntax:

```sh
#!/bin/sh
printf '%s\n' "This script targets POSIX sh."
```

Use `#!/usr/bin/env bash` when the script requires Bash and you want to find Bash through the user's `PATH`:

```bash
#!/usr/bin/env bash
names=(api worker db)
printf '%s\n' "${names[@]}"
```

Use `#!/bin/bash` only in controlled environments where Bash is known to live at that exact path:

```bash
#!/bin/bash
set -euo pipefail
```

Avoid recommending `#!/usr/bin/env sh` as a default. POSIX systems standardize `sh`; using `/bin/sh` is the normal portable contract for POSIX shell scripts.

The shebang is used when a Unix-like system executes the file directly, such as `./script.sh`. If you run `sh script.sh`, the `sh` command interprets the file even if the shebang says Bash. If you run `bash script.sh`, Bash interprets the file even if the shebang says `sh`.

## Bash vs POSIX sh Examples

Bash arrays are not POSIX `sh`:

```bash
#!/usr/bin/env bash
tools=(git curl awk)
for tool in "${tools[@]}"; do
  printf '%s\n' "$tool"
done
```

A POSIX `sh` script can often use positional parameters:

```sh
#!/bin/sh
set -- git curl awk
for tool do
  printf '%s\n' "$tool"
done
```

For data with spaces, use newline-delimited input instead of trying to fake arrays:

```sh
#!/bin/sh
printf '%s\n' "build api" "run tests" "ship app" |
while IFS= read -r task; do
  printf 'task: %s\n' "$task"
done
```

Bash `[[ ... ]]` is not POSIX:

```bash
if [[ $answer == y || $answer == yes ]]; then
  printf '%s\n' "confirmed"
fi
```

Use `case` in POSIX `sh`:

```sh
case $answer in
  y|yes)
    printf '%s\n' "confirmed"
    ;;
esac
```

Use `[ ... ]` for simple POSIX tests:

```sh
if [ "$answer" = "yes" ]; then
  printf '%s\n' "confirmed"
fi
```

Bash case conversion such as `${name^^}` is not POSIX:

```bash
name="api"
printf '%s\n' "${name^^}"
```

Use a standard external utility when appropriate:

```sh
name="api"
printf '%s\n' "$name" | tr '[:lower:]' '[:upper:]'
```

Bash accepts `source`, but POSIX `sh` uses `.`:

```bash
source ./config.sh
```

Portable form:

```sh
. ./config.sh
```

Bash here-strings are not POSIX:

```bash
grep 'ready' <<< "$status_text"
```

Portable form:

```sh
printf '%s\n' "$status_text" | grep 'ready'
```

Bash process substitution is not POSIX:

```bash
diff <(sort expected.txt) <(sort actual.txt)
```

Portable form:

```sh
sort expected.txt > expected.sorted
sort actual.txt > actual.sorted
diff expected.sorted actual.sorted
rm -f expected.sorted actual.sorted
```

## Portable Tests And Quoting

Use `[ ... ]` for portable tests:

```sh
if [ "$name" = "admin" ]; then
  printf '%s\n' "allowed"
fi
```

Quote variable expansions inside tests. This prevents empty values or values with spaces from changing the test expression:

```sh
if [ -n "$filename" ]; then
  printf '%s\n' "$filename"
fi
```

Use `=` for string equality, not `==`:

```sh
if [ "$mode" = "prod" ]; then
  printf '%s\n' "production"
fi
```

Avoid `-a` and `-o` inside `[ ... ]`. Use separate tests with shell operators:

```sh
if [ -n "$user" ] && [ -n "$home" ]; then
  printf '%s\n' "$user:$home"
fi
```

To test whether a variable is set, even if it is set to an empty string, use `${VAR+x}`:

```sh
if [ -n "${VAR+x}" ]; then
  printf '%s\n' "VAR is set"
fi
```

That is different from testing whether the variable has a non-empty value:

```sh
if [ -n "$VAR" ]; then
  printf '%s\n' "VAR is not empty"
fi
```

## Command Availability And External Utilities

Use `command -v` to check whether a command exists:

```sh
if command -v git >/dev/null 2>&1; then
  printf '%s\n' "git is available"
else
  printf '%s\n' "error: git was not found in PATH" >&2
  exit 1
fi
```

The redirection keeps successful checks quiet and hides implementation-specific messages from failed lookups.

Finding a command does not prove that every option you plan to use is portable. Examples to watch:

- `sed -i` differs between GNU `sed` and BSD/macOS `sed`.
- `date -d` is common on GNU systems but not portable to macOS `date`.
- `grep -P` is not a POSIX `grep` option.
- `xargs -r` is common on GNU systems but not portable everywhere.
- `readlink -f` is common on GNU systems but not portable to default macOS `readlink`.

Prefer POSIX utility options when the script targets POSIX `sh`. If you intentionally require GNU utilities, say so in the script's documentation and test on that platform.

## Setup And Run On Windows

Windows PowerShell is not Bash and is not POSIX `sh`. Running `.\script.sh` from PowerShell does not give you Unix shebang behavior.

For POSIX `sh`, `dash`, and Linux-style tooling on Windows, use WSL:

```powershell
wsl --install
```

Open the installed Linux distribution shell, then install test tools:

```bash
sudo apt update
sudo apt install shellcheck dash
```

From PowerShell, you can ask WSL to run a POSIX shell command:

```powershell
wsl sh -lc "sh check-tools.sh sh printf"
```

Git Bash can be useful for Bash practice on Windows, but WSL is clearer when the lesson is POSIX `sh`, `dash`, and Linux command behavior.

## Setup And Run On macOS Apple Silicon

Terminal on modern macOS defaults to `zsh`, not POSIX `sh`. Launch scripts with the shell you mean:

```bash
sh check-tools.sh sh printf
bash script-that-requires-bash.sh
```

Apple Silicon does not change pure shell syntax. It can matter for native tools because package managers and binaries may install in architecture-specific locations. Do not hard-code Homebrew paths in scripts; let `PATH` and `command -v` find tools.

Homebrew can install ShellCheck and `dash`:

```bash
brew install shellcheck dash
```

## Line Endings

Save shell scripts with LF line endings, not Windows CRLF line endings. A carriage return can become part of the interpreter path or command text, causing confusing failures. ShellCheck reports this as SC1017.

## Testing Matrix

For a POSIX `sh` script:

```sh
shellcheck --shell=sh check-tools.sh
sh -n check-tools.sh
dash -n check-tools.sh
sh ./check-tools.sh sh printf
dash ./check-tools.sh sh printf
```

`shellcheck --shell=sh` asks ShellCheck to warn about non-POSIX features. `sh -n` and `dash -n` parse the script without running it. Running the script with `sh` and `dash` catches runtime behavior that syntax checks cannot.

For a Bash script:

```sh
shellcheck --shell=bash script.sh
bash -n script.sh
bash ./script.sh
```

`bash --posix` is useful for learning how Bash changes in POSIX mode, but it is not a substitute for testing with another POSIX-oriented shell such as `dash`.

## When Portability Is Not Worth It

Portability is useful when the script may run on different Unix-like systems or under unknown `/bin/sh` implementations. It is not always the clearest choice.

Require Bash honestly when Bash features make a controlled-runtime script simpler or safer, including:

- arrays
- associative arrays
- `[[ ... ]]` conditionals
- regular expression matching with `[[ string =~ regex ]]`
- here-strings
- process substitution
- Bash-specific safety patterns such as `set -euo pipefail`, used with Bash-aware error handling

In that case, use a Bash shebang and make the dependency clear:

```bash
#!/usr/bin/env bash
```

Honest Bash is fine. Accidental Bash hidden behind `#!/bin/sh` is the bug.

## Common Mistakes

- Using `#!/bin/sh` with Bash arrays such as `items=(one two)` or `"${items[@]}"`.
- Using `[[ ... ]]`, here-strings, or process substitution in a script that claims to be POSIX `sh`.
- Using `==` inside `[ ... ]` instead of portable `=`.
- Forgetting to quote variables in tests, such as `[ $name = admin ]`.
- Using `source ./file.sh` instead of `. ./file.sh`.
- Assuming `/bin/sh` is Bash because it is Bash on one machine.
- Checking command availability with `which` instead of `command -v`.
- Running `./script.sh` from PowerShell and expecting Unix shebang semantics.
- Saving shell scripts with CRLF line endings.
- Assuming macOS `zsh` is the same language as POSIX `sh`.
- Assuming ShellCheck proves runtime portability.
- Testing only with Bash and never running the script with `dash` or `/bin/sh`.
- Using GNU-only command options in a script intended to run on macOS or minimal systems.

## Exercise

Create a portable script named `check-tools.sh`.

Requirements:

- Start with `#!/bin/sh`.
- Accept command-line arguments as tool names.
- If no tool names are given, print usage to standard error and exit with status `2`.
- For each tool name, check availability with `command -v`.
- Print `ok: TOOL` to standard output when a tool is found.
- Print `missing: TOOL` to standard error when a tool is not found.
- Exit with status `0` if all tools are found.
- Exit with status `1` if any tool is missing.
- Use POSIX `sh` syntax only.
- Include lint and run commands for WSL on Windows and `zsh` on macOS.

Example runs:

```sh
sh check-tools.sh sh printf
sh check-tools.sh definitely-not-a-real-tool
sh check-tools.sh
```

## Worked Answer

`check-tools.sh`:

```sh
#!/bin/sh

if [ "$#" -eq 0 ]; then
  printf '%s\n' "usage: check-tools.sh TOOL [TOOL ...]" >&2
  exit 2
fi

status=0

for tool in "$@"; do
  if command -v "$tool" >/dev/null 2>&1; then
    printf '%s\n' "ok: $tool"
  else
    printf '%s\n' "missing: $tool" >&2
    status=1
  fi
done

exit "$status"
```

Why this is portable:

- It uses `#!/bin/sh`.
- It uses `[ ... ]` instead of `[[ ... ]]`.
- It quotes variable expansions.
- It loops over `"$@"` instead of using an array.
- It uses `command -v` instead of `which`.
- It uses `printf` instead of relying on implementation-specific `echo` behavior.

Windows PowerShell with WSL:

```powershell
wsl sh -lc "shellcheck --shell=sh check-tools.sh"
wsl sh -lc "sh -n check-tools.sh"
wsl sh -lc "dash -n check-tools.sh"
wsl sh -lc "sh check-tools.sh sh printf"
wsl sh -lc "dash check-tools.sh sh printf"
wsl sh -lc "sh check-tools.sh definitely-not-a-real-tool"
wsl sh -lc "sh check-tools.sh"
```

macOS Terminal with default `zsh`:

```bash
shellcheck --shell=sh check-tools.sh
sh -n check-tools.sh
dash -n check-tools.sh
sh check-tools.sh sh printf
dash check-tools.sh sh printf
sh check-tools.sh definitely-not-a-real-tool
sh check-tools.sh
```

Expected behavior:

- `sh check-tools.sh sh printf` prints `ok:` lines and exits `0` when both are available.
- `sh check-tools.sh definitely-not-a-real-tool` prints `missing: definitely-not-a-real-tool` to standard error and exits `1`.
- `sh check-tools.sh` prints usage to standard error and exits `2`.

## Next Step

Return to the advanced Bash README and continue with the next numbered lesson.

## Sources Used

- POSIX.1-2024 Shell Command Language: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/V3_chap02.html
- POSIX.1-2024 `sh`: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/sh.html
- POSIX.1-2024 `test`: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/test.html
- POSIX.1-2024 `command`: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/command.html
- POSIX.1-2024 `printf`: https://pubs.opengroup.org/onlinepubs/9799919799/utilities/printf.html
- Bash Reference Manual: https://www.gnu.org/software/bash/manual/bash.html
- Microsoft Learn, Install WSL: https://learn.microsoft.com/en-us/windows/wsl/install
- Apple Support, Change the default shell in Terminal on Mac: https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
- Apple Support, Intro to shell scripts in Terminal on Mac: https://support.apple.com/guide/terminal/intro-to-shell-scripts-apd53500956-7c5b-496b-a362-2845f2aab4bc/mac
- Homebrew Documentation, Installation: https://docs.brew.sh/Installation
- ShellCheck, Installing: https://github.com/koalaman/shellcheck#installing
- ShellCheck SC1017: https://www.shellcheck.net/wiki/SC1017
- ShellCheck SC3016: https://www.shellcheck.net/wiki/SC3016
