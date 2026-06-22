# 03 - Portable Shell Scripts

## Learning Goal

Write small scripts that honestly target either POSIX `sh` or Bash, avoid Bash-only syntax under `#!/bin/sh`, check command availability portably, and test with ShellCheck and `dash`.

## What Portable Means

A portable shell script is written for a specific shell language and runtime environment, then keeps that promise.

The most common portable target is POSIX `sh`. POSIX `sh` is a standardized shell language supported by many Unix-like systems. It is smaller than Bash and does not include many Bash conveniences.

The path `/bin/sh` does not always mean Bash. Depending on the system, `/bin/sh` might be:

- `dash`, common on Debian and Ubuntu systems
- `ash`, common on BusyBox and Alpine-style systems
- `ksh` or another POSIX-compatible shell
- Bash running in a POSIX compatibility mode

Portability is not only about shell syntax. It also includes the external commands and command options your script uses. A script can use POSIX shell syntax and still fail because it depends on a nonportable option such as a GNU-only flag.

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

Use `#!/bin/bash` when the environment is controlled and Bash is known to live at that exact path:

```bash
#!/bin/bash
set -euo pipefail
```

Do not write `#!/bin/sh` and then use Bash-only syntax. That script may work on your machine and fail immediately on another system.

## Bash vs POSIX sh Examples

Bash arrays are not POSIX `sh`:

```bash
#!/usr/bin/env bash
tools=(git curl awk)
for tool in "${tools[@]}"; do
  printf '%s\n' "$tool"
done
```

A POSIX `sh` script can often use positional parameters or a plain loop instead:

```sh
#!/bin/sh
for tool in git curl awk; do
  printf '%s\n' "$tool"
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

Bash case conversion such as `${name^^}` is not POSIX:

```bash
name="api"
printf '%s\n' "${name^^}"
```

For portable scripts, avoid that expansion or use a standard external utility when appropriate:

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

## Portable Tests

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

Avoid `-a` and `-o` inside `[ ... ]` because their behavior can be confusing and less portable. Use separate tests with shell operators:

```sh
if [ -n "$user" ] && [ -n "$home" ]; then
  printf '%s\n' "$user:$home"
fi
```

To test whether a variable is set, even if it is set to an empty string, use parameter expansion:

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

## Command Availability

Use `command -v` to check whether a command is available:

```sh
if command -v git >/dev/null 2>&1; then
  printf '%s\n' "git is available"
else
  printf '%s\n' "git is required" >&2
  exit 1
fi
```

The redirection keeps successful checks quiet and hides implementation-specific messages from failed lookups.

Give clear errors. A portable script should fail in a way the user can understand:

```sh
if ! command -v curl >/dev/null 2>&1; then
  printf '%s\n' "error: curl was not found in PATH" >&2
  exit 1
fi
```

Remember that finding a command is only the first check. External utilities and their options may still be nonportable. For example, one system's `sed`, `date`, `grep`, or `xargs` may not support the same options as another system's version.

## Testing Portability

For a POSIX `sh` script, test more than one thing:

```sh
shellcheck --shell=sh check-tools.sh
dash -n check-tools.sh
dash ./check-tools.sh git sh
sh ./check-tools.sh git sh
```

`shellcheck --shell=sh` asks ShellCheck to warn about non-POSIX shell features.

`dash -n` checks syntax with `dash` without running the script.

Running the script with `dash` or `sh` catches runtime behavior that syntax checks cannot.

`bash --posix` is useful, but it is not enough by itself. Bash in POSIX mode can still differ from smaller `/bin/sh` implementations, so test with a real POSIX-oriented shell such as `dash` when that is your target.

## When Portability Is Not Worth It

Portability is useful when the script may run on different Unix-like systems or under unknown `/bin/sh` implementations. It is not always the clearest choice.

Require Bash honestly when Bash features make a controlled-runtime script simpler or safer, including:

- arrays
- regular expression matching with `[[ string =~ regex ]]`
- `[[ ... ]]` conditional expressions
- associative arrays
- process substitution
- Bash-specific safety patterns such as `set -euo pipefail` with Bash-aware handling

In that case, use a Bash shebang and make the dependency clear:

```bash
#!/usr/bin/env bash
```

Honest Bash is better than accidental Bash hidden behind `#!/bin/sh`.

## Common Mistakes

- Using `#!/bin/sh` with Bash arrays such as `items=(one two)` or `"${items[@]}"`.
- Using `[[ ... ]]` in a script that claims to be POSIX `sh`.
- Using `==` inside `[ ... ]` instead of portable `=`.
- Forgetting to quote variables in tests, such as `[ $name = admin ]`.
- Using `source ./file.sh` instead of `. ./file.sh`.
- Assuming `/bin/sh` is Bash because it is Bash on one machine.
- Checking command availability with `which` instead of `command -v`.
- Testing only with Bash and never running the script with `dash` or `/bin/sh`.
- Using GNU-only command options in a script intended to run on minimal systems.
- Treating `bash --posix` as a complete substitute for testing with another shell.

## Exercise

Create a portable script named `check-tools.sh`.

Requirements:

- Start with `#!/bin/sh`.
- Accept command-line arguments as tool names.
- If no tool names are given, print usage to standard error and exit with status `2`.
- For each tool name, check availability with `command -v`.
- Print `ok: TOOL` when a tool is found.
- Print `missing: TOOL` when a tool is not found.
- Exit with status `0` if all tools are found.
- Exit with status `1` if any tool is missing.
- Use POSIX `sh` syntax only.
- Include test commands for ShellCheck, `dash -n`, and runtime checks.

Example runs:

```sh
./check-tools.sh sh printf
./check-tools.sh definitely-not-a-real-tool
./check-tools.sh
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

Test commands:

```sh
chmod +x check-tools.sh
shellcheck --shell=sh check-tools.sh
dash -n check-tools.sh
dash ./check-tools.sh sh printf
sh ./check-tools.sh sh printf
dash ./check-tools.sh definitely-not-a-real-tool
dash ./check-tools.sh
```

Expected behavior:

- `./check-tools.sh sh printf` prints `ok:` lines and exits `0` when both are available.
- `./check-tools.sh definitely-not-a-real-tool` prints `missing: definitely-not-a-real-tool` and exits `1`.
- `./check-tools.sh` prints usage to standard error and exits `2`.

Why this is portable:

- It uses `#!/bin/sh`.
- It uses `[ ... ]` instead of `[[ ... ]]`.
- It quotes variable expansions.
- It loops over `"$@"` instead of using an array.
- It uses `command -v` instead of `which`.
- It uses `printf` instead of relying on implementation-specific `echo` behavior.

## Next Step

Return to the advanced Bash README and continue with the next numbered lesson.

## Sources Used

- POSIX Shell Command Language: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html
- POSIX `test`: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/test.html
- POSIX `command`: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/command.html
- Bash Reference Manual, Bash POSIX Mode: https://www.gnu.org/software/bash/manual/bash.html#Bash-POSIX-Mode
- ShellCheck SC2039: https://www.shellcheck.net/wiki/SC2039
- ShellCheck SC3016: https://www.shellcheck.net/wiki/SC3016
- Ubuntu DashAsBinSh: https://wiki.ubuntu.com/DashAsBinSh
