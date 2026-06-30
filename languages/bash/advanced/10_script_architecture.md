# 10 - Script Architecture

## Learning Goal

Build a maintainable Bash CLI architecture with a clear contract: inputs from arguments, environment, and optional config; validation before behavior; subcommand dispatch; stdout for data; stderr for logs and errors; cleanup on exit; safe arrays; controlled globals; ShellCheck/shfmt verification; and manual or Bats testability.

By the end, you should be able to explain and use `main "$@"`, a source-safe guard, small functions, global option parsing, dependency checks, config/env/default separation, traps, and command arrays without reaching for `eval`.

## Why Architecture Matters

Bash starts as glue: run a command, pipe some text, move a file, call `git`, wrap a deployment step. That is exactly what it is good at. The trouble begins when yesterday's five-line helper becomes today's shared command-line tool with options, cleanup behavior, CI output, dry-run mode, and several people editing it.

Architecture for shell scripts is not about imitating a large application framework. It is about making the script's promises visible:

- Inputs: command-line arguments, environment variables, and optional trusted config.
- Validation: required commands, paths, and unsafe combinations checked before work starts.
- Behavior: public subcommands such as `check`, `clean`, and `help`.
- Output: stdout is for results that other programs may read.
- Diagnostics: stderr is for logs, warnings, prompts, and errors.
- Cleanup: temporary resources are removed by traps.
- Testability: helpers can be tested without running the whole CLI.

A well-shaped Bash script is easier to review because risky parts are not scattered everywhere. Parsing happens in one place, validation happens before work starts, helper functions do one thing, and command functions are the public surface of the tool.

```mermaid
flowchart LR
  user[User] --> main[main "$@"]
  main --> parse[parse global options]
  parse --> config[defaults + env + CLI]
  config --> dispatch[dispatch command]
  dispatch --> check[cmd_check]
  dispatch --> clean[cmd_clean]
  check --> validate[validate]
  clean --> validate
  validate --> helpers[helpers / run]
  helpers --> out[stdout data]
  helpers --> err[stderr logs/errors]
  traps[ERR/EXIT traps] --> cleanup[cleanup]
```

## Recommended File Shape

For a non-trivial Bash CLI, use a consistent order:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

readonly SCRIPT_NAME=${0##*/}
readonly EX_OK=0
readonly EX_USAGE=64
readonly EX_UNAVAILABLE=69
readonly DEFAULT_PATH=.

VERBOSE=0
DRY_RUN=0
TARGET_PATH=${REPO_TOOL_PATH:-$DEFAULT_PATH}
TEMP_DIR=
REMAINING_ARGS=()

usage() { ...; }
log() { ...; }
debug() { ...; }
die() { ...; }

cleanup() { ...; }
on_error() { ...; }

require_cmd() { ...; }
run() { ...; }
validate_path() { ...; }

cmd_check() { ...; }
cmd_clean() { ...; }

parse_global_options() { ...; }
dispatch() { ...; }
main() { ...; }

if [[ ${BASH_SOURCE[0]} == "$0" ]]; then
  main "$@"
fi
```

The names are not magic. The shape is the point:

- `#!/usr/bin/env bash` says this is Bash, not POSIX `sh`.
- `set -Eeuo pipefail` makes unexpected failures visible.
- `readonly` constants document names, defaults, and exit statuses.
- Controlled globals such as `VERBOSE`, `DRY_RUN`, and `TARGET_PATH` hold program configuration.
- `usage`, `log`, `debug`, and `die` centralize user-facing text.
- `cleanup` and `on_error` keep trap behavior visible.
- `require_cmd`, `run`, and validation helpers keep dangerous work behind small contracts.
- `cmd_check` and `cmd_clean` are the public behavior surface.
- `parse_global_options` handles global CLI flags before command dispatch.
- `main "$@"` preserves every original argument boundary.
- The source-safe guard runs the CLI only when the file is executed, not when it is sourced for tests or reuse.

## Strict Mode and Entry Points

`set -Eeuo pipefail` combines several Bash behaviors:

- `-E` lets `ERR` traps be inherited by functions, command substitutions, and subshells in more cases.
- `-e` exits on many unhandled command failures.
- `-u` treats unset variables as errors.
- `-o pipefail` makes a pipeline fail if any command in it fails, not just the last command.

These settings are useful, but they are not a replacement for clear error handling. `set -e` and `ERR` have exceptions around conditionals, `if`, `while`, `until`, `&&`, `||`, and commands whose status is being tested. Use explicit `if ! command; then ... fi` blocks for expected failures.

Always enter the script through `main "$@"`:

```bash
main "$@"
```

`"$@"` expands to one quoted word per original argument. That preserves spaces, empty strings, and glob characters:

```bash
./repo-tool.sh --path "project with spaces" check
```

Do not use `main $@`, `$*`, or `"$*"`. Those forms lose argument boundaries or allow word splitting and pathname expansion.

Use this source-safe guard at the bottom:

```bash
if [[ ${BASH_SOURCE[0]} == "$0" ]]; then
  main "$@"
fi
```

`$0` is the shell's idea of the currently executing script or shell. `${BASH_SOURCE[0]}` is the file where the current Bash code came from. When you run `bash repo-tool.sh`, they match. When a test or another script runs `source repo-tool.sh`, they do not match, so functions can be loaded without executing the CLI.

## Functions, Modules, and Globals

Bash variables are global by default. Inside functions, use `local` unless you are intentionally writing to a controlled program global:

```bash
is_git_repo() {
  local path=$1
  git -C "$path" rev-parse --is-inside-work-tree >/dev/null 2>&1
}
```

That helper has a clear contract: given a path, return success if Git recognizes it as a work tree. It does not parse options, print banners, remove files, or mutate globals.

Keep parsing separate from behavior:

- Parsing reads arguments and sets configuration.
- Validation checks paths, dependencies, and preconditions.
- Helpers compute, query, or wrap commands.
- Command functions coordinate one user-facing subcommand.
- `run` performs side-effecting commands and honors dry-run mode.

Avoid deep helper `exit` calls. Most helpers should return success or failure and let their caller decide. A small `die MESSAGE STATUS` helper is the exception: it is useful for user-facing fatal errors at clear boundaries.

When a script grows, split shared helpers into library files only when there is a real boundary. Remember that `source` executes Bash in the current shell. It shares variables, options, traps, and functions. Never source untrusted files. Keep sourced modules boring: function and constant definitions, not surprise execution.

## CLI Layout

A maintainable multi-command script usually follows this layout:

```text
script [global-options] command [command-options] [args]
```

For example:

```bash
bash repo-tool.sh --path . --verbose check
bash repo-tool.sh --path . --dry-run clean
bash repo-tool.sh help
```

Global options affect the whole program. Command options belong to one command. Keep that boundary clear. If `--path` affects every command, parse it before dispatch. If `--force` only affects `clean`, parse it inside `cmd_clean`.

Good CLI behavior follows ordinary Unix conventions:

- `-h` or `--help` prints usage and exits `0`.
- Invalid options print a short error to stderr and exit non-zero.
- Normal machine-readable results go to stdout.
- Logs, warnings, prompts, and errors go to stderr.
- Exit status `0` means success. Non-zero statuses should have documented meanings.
- `--` ends option parsing when a following operand might begin with `-`.
- Short options use one hyphen, such as `-v`; long options use two, such as `--verbose`.

POSIX utility conventions are stricter than many modern CLIs: options are normally single letters, option arguments are separate arguments, and operands follow options. Bash scripts often add GNU-style long options manually because `getopts` only handles short options portably.

## Argument Parsing

Use `getopts` when you only need short options:

```bash
while getopts ":hvn" opt; do
  case "$opt" in
    h) usage; exit "$EX_OK" ;;
    v) VERBOSE=1 ;;
    n) DRY_RUN=1 ;;
    :) die "option -$OPTARG requires an argument" "$EX_USAGE" ;;
    \?) die "unknown option: -$OPTARG" "$EX_USAGE" ;;
  esac
done
shift "$((OPTIND - 1))"
```

For long options, use a manual `case` loop:

```bash
while (($#)); do
  case "$1" in
    -h|--help) usage; exit "$EX_OK" ;;
    -v|--verbose) VERBOSE=1; shift ;;
    -n|--dry-run) DRY_RUN=1; shift ;;
    --path)
      (($# >= 2)) || die "--path requires an argument" "$EX_USAGE"
      TARGET_PATH=$2
      shift 2
      ;;
    --path=*)
      TARGET_PATH=${1#--path=}
      [[ -n $TARGET_PATH ]] || die "--path requires a non-empty argument" "$EX_USAGE"
      shift
      ;;
    --)
      shift
      break
      ;;
    -*)
      die "unknown option: $1" "$EX_USAGE"
      ;;
    *)
      break
      ;;
  esac
done
```

Manual parsing is more code, but it is clear and portable across Bash installations. Avoid external `getopt` unless you have checked the target systems, because behavior differs between platforms.

The `--` marker means "stop parsing options." This lets users pass operands that begin with a dash:

```bash
bash repo-tool.sh --path . -- clean
```

## Config, Env, and Defaults

Keep configuration precedence obvious. A common order is:

1. Defaults built into the script.
2. Optional trusted config file.
3. Environment variables.
4. CLI flags.

In this lesson, prefer defaults, environment variables, and CLI flags. That keeps the example portable and avoids executing extra files:

```bash
readonly DEFAULT_PATH=.
TARGET_PATH=${REPO_TOOL_PATH:-$DEFAULT_PATH}
```

Then `--path PATH` can override `REPO_TOOL_PATH`, and `REPO_TOOL_PATH` can override the default.

If you do support config files, remember that this executes Bash code:

```bash
source ./repo-tool.conf
```

That is appropriate only for trusted config written for your script. It is not a safe parser for untrusted input.

## Logging and Errors

Use stdout for results:

```bash
git -C "$TARGET_PATH" status --short
printf 'would remove: %s\n' "$file"
```

Use stderr for logs, warnings, prompts, and errors:

```bash
log() {
  printf '%s: %s\n' "$SCRIPT_NAME" "$*" >&2
}
```

Give fatal errors one helper:

```bash
die() {
  local message=$1
  local status=${2:-1}
  log "error: $message"
  exit "$status"
}
```

Named exit constants make scripts easier to read:

```bash
readonly EX_OK=0
readonly EX_USAGE=64
readonly EX_UNAVAILABLE=69
```

## Dependencies and Tools

Check required commands before using them:

```bash
require_cmd() {
  local name=$1
  command -v "$name" >/dev/null 2>&1 || die "missing required command: $name" "$EX_UNAVAILABLE"
}
```

Optional tools should be detected and reported without failing the whole script unless they are required for the requested command:

```bash
if command -v shellcheck >/dev/null 2>&1; then
  shellcheck repo-tool.sh
else
  log "ShellCheck not installed; skipping shell lint"
fi
```

Use ShellCheck for linting, shfmt for formatting, and Bats for behavior tests when the script will keep changing. These are Bash ecosystem tools; on Windows, run them inside WSL, Git Bash where the tools have been installed, or Docker. Bats is not a native PowerShell test runner, and Git Bash by itself does not guarantee ShellCheck, shfmt, or Bats are available.

## Cleanup and Traps

Use traps for cleanup:

```bash
cleanup() {
  if [[ -n ${TEMP_DIR:-} && -d $TEMP_DIR ]]; then
    rm -rf -- "$TEMP_DIR"
  fi
}

on_error() {
  local line=$1
  log "error near line $line"
}

trap cleanup EXIT
trap 'on_error $LINENO' ERR
```

Only delete temporary directories that your script created and validated. Do not run `rm -rf` against a variable unless you know exactly how that variable was set.

`ERR` traps are useful for diagnostics, but they do not fire in every situation where a command returns non-zero. In conditionals and intentional status checks, handle failures explicitly.

## Safe Arrays and `run`

Build commands as arrays when arguments are dynamic:

```bash
status_cmd=(git -C "$TARGET_PATH" status --short)
"${status_cmd[@]}"
```

Arrays preserve argument boundaries. Avoid `eval`; it turns strings back into code and creates quoting and injection problems.

Put destructive or important side effects behind `run`:

```bash
run() {
  if ((DRY_RUN)); then
    printf 'would run:'
    printf ' %q' "$@"
    printf '\n'
    return 0
  fi

  debug "running: $*"
  "$@"
}
```

Now dry-run mode is enforced in one place instead of scattered across every command.

## Platform Setup and Running

Windows PowerShell does not run Bash scripts by itself. Use WSL or Git Bash.

Windows PowerShell with WSL:

```powershell
wsl --install
wsl sudo apt update
wsl sudo apt install -y shellcheck shfmt bats
wsl bash repo-tool.sh --path . check
$env:REPO_TOOL_PATH = "."
wsl bash repo-tool.sh check
wsl bash -n repo-tool.sh
wsl shellcheck repo-tool.sh
wsl shfmt -w repo-tool.sh
```

Windows PowerShell with Git Bash installed and `bash` on `PATH`. Install Git for Windows first, then install ShellCheck, shfmt, and Bats separately or use WSL for those checks if they are not available in Git Bash:

```powershell
winget install --id Git.Git -e --source winget
bash repo-tool.sh --path . check
$env:REPO_TOOL_PATH = "."
bash repo-tool.sh check
bash -n repo-tool.sh
shellcheck repo-tool.sh  # if installed and on PATH
shfmt -w repo-tool.sh    # if installed and on PATH
```

macOS Apple Silicon uses zsh as the default interactive shell, but you can still invoke Bash explicitly:

```zsh
brew install bash shellcheck shfmt bats-core
bash repo-tool.sh --path . check
export REPO_TOOL_PATH=.
bash repo-tool.sh check
bash -n repo-tool.sh
shellcheck repo-tool.sh
shfmt -w repo-tool.sh
bats test/repo-tool.bats
```

The examples are project-relative: run them from the directory that contains `repo-tool.sh`. They avoid hard-coded platform paths so the same commands work in a cloned project.

## Common Mistakes

- Putting executable code throughout the file instead of using `main "$@"`.
- Forgetting the `${BASH_SOURCE[0]} == "$0"` guard, then accidentally running the script when tests source it.
- Calling `main $@` and breaking paths with spaces.
- Mixing global option parsing into every command function.
- Sending logs to stdout, which breaks callers that want to pipe real output.
- Returning `0` after failed checks because a `printf` was the last command.
- Using `exit` inside helper functions that should return a status to the caller.
- Forgetting `local`, then accidentally overwriting global configuration.
- Using `eval` to build commands instead of arrays or direct quoted arguments.
- Treating dry-run as a printed message while still running the destructive command.
- Checking dependencies after half the script has already run.
- Sourcing untrusted config files.
- Deleting temp directories that were not created and validated by the script.
- Assuming Bats is a native PowerShell tool.
- Ignoring ShellCheck warnings because the script "works on my machine."

## Exercise

Write `repo-tool.sh`, a small repository maintenance CLI.

Required global options:

- `-h`, `--help`: print usage and exit `0`.
- `-v`, `--verbose`: print debug logs to stderr.
- `-n`, `--dry-run`: show cleanup actions without deleting files.
- `--path PATH`: repository path to inspect or clean. Default: current directory.

Configuration:

- Default path is `.`.
- Environment variable `REPO_TOOL_PATH` overrides the default.
- CLI flag `--path PATH` overrides both.

Required commands:

- `check`: verify that the path exists, Git is installed, the path is a Git work tree, print `git status --short`, and run ShellCheck on `*.sh` files if ShellCheck is installed.
- `clean`: remove `*.tmp` files under the target path. In dry-run mode, print what would be removed without deleting anything.
- `help`: print usage.

Behavior requirements:

- Use `set -Eeuo pipefail`.
- Use named exit constants.
- Preserve arguments with `main "$@"`.
- Use a source-safe guard.
- Send logs and errors to stderr.
- Keep stdout for command results such as `git status --short` and dry-run file lists.
- Keep destructive operations behind `run`.
- Use command arrays where command arguments are assembled.
- Avoid `eval`.
- Return meaningful statuses: success, invalid usage, unavailable dependency, and failed checks.
- Keep helpers small enough to test manually or with Bats.

## Worked Answer

Save this as `repo-tool.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

readonly SCRIPT_NAME=${0##*/}
readonly EX_OK=0
readonly EX_USAGE=64
readonly EX_UNAVAILABLE=69
readonly DEFAULT_PATH=.

VERBOSE=0
DRY_RUN=0
TARGET_PATH=${REPO_TOOL_PATH:-$DEFAULT_PATH}
TEMP_DIR=
REMAINING_ARGS=()

usage() {
  cat <<USAGE
Usage:
  $SCRIPT_NAME [global-options] command [args]

Global options:
  -h, --help        Show this help and exit
  -v, --verbose     Print debug logs to stderr
  -n, --dry-run     Print cleanup actions without deleting files
      --path PATH   Repository path to inspect or clean

Environment:
  REPO_TOOL_PATH    Default repository path when --path is not provided

Commands:
  check             Verify path, Git repository status, and shell scripts
  clean             Remove *.tmp files under the target path
  help              Show this help

Examples:
  $SCRIPT_NAME --path . check
  $SCRIPT_NAME --path . --verbose check
  REPO_TOOL_PATH=. $SCRIPT_NAME check
  $SCRIPT_NAME --path . --dry-run clean

Exit statuses:
  0   Success
  64  Invalid command-line usage
  69  Required dependency or path is unavailable
  1   A check or command failed
USAGE
}

log() {
  printf '%s: %s\n' "$SCRIPT_NAME" "$*" >&2
}

debug() {
  if ((VERBOSE)); then
    log "debug: $*"
  fi
}

die() {
  local message=$1
  local status=${2:-1}
  log "error: $message"
  exit "$status"
}

cleanup() {
  if [[ -n ${TEMP_DIR:-} && -d $TEMP_DIR ]]; then
    debug "removing temp dir: $TEMP_DIR"
    rm -rf -- "$TEMP_DIR"
  fi
}

on_error() {
  local line=$1
  log "error near line $line"
}

require_cmd() {
  local name=$1
  command -v "$name" >/dev/null 2>&1 || die "missing required command: $name" "$EX_UNAVAILABLE"
}

run() {
  if ((DRY_RUN)); then
    printf 'would run:'
    printf ' %q' "$@"
    printf '\n'
    return "$EX_OK"
  fi

  debug "running: $*"
  "$@"
}

validate_path() {
  local path=$1
  [[ -d $path ]] || die "path is not a directory: $path" "$EX_UNAVAILABLE"
}

is_git_repo() {
  local path=$1
  git -C "$path" rev-parse --is-inside-work-tree >/dev/null 2>&1
}

list_tmp_files() {
  local path=$1
  find "$path" -type f -name '*.tmp' -print
}

list_shell_scripts() {
  local path=$1
  find "$path" -type f -name '*.sh' -print
}

check_shell_scripts() {
  local path=$1
  local found=0
  local failed=0
  local script

  if ! command -v shellcheck >/dev/null 2>&1; then
    log "ShellCheck not installed; skipping shell lint"
    return "$EX_OK"
  fi

  while IFS= read -r script; do
    found=1
    if ! shellcheck "$script"; then
      failed=1
    fi
  done < <(list_shell_scripts "$path")

  if ((found == 0)); then
    log "no shell scripts found for ShellCheck"
  fi

  return "$failed"
}

cmd_check() {
  (($# == 0)) || die "check does not accept arguments" "$EX_USAGE"

  validate_path "$TARGET_PATH"
  require_cmd git

  if ! is_git_repo "$TARGET_PATH"; then
    die "not a Git work tree: $TARGET_PATH" "$EX_UNAVAILABLE"
  fi

  log "checking repository: $TARGET_PATH"
  local status_cmd=(git -C "$TARGET_PATH" status --short)
  "${status_cmd[@]}"
  check_shell_scripts "$TARGET_PATH"
}

cmd_clean() {
  (($# == 0)) || die "clean does not accept arguments" "$EX_USAGE"

  validate_path "$TARGET_PATH"
  require_cmd find
  require_cmd rm

  local file
  local count=0

  while IFS= read -r file; do
    count=$((count + 1))
    if ((DRY_RUN)); then
      printf 'would remove: %s\n' "$file"
    else
      run rm -f -- "$file"
    fi
  done < <(list_tmp_files "$TARGET_PATH")

  log "tmp files matched: $count"
}

cmd_help() {
  (($# == 0)) || die "help does not accept arguments" "$EX_USAGE"
  usage
}

parse_global_options() {
  while (($#)); do
    case "$1" in
      -h|--help)
        usage
        exit "$EX_OK"
        ;;
      -v|--verbose)
        VERBOSE=1
        shift
        ;;
      -n|--dry-run)
        DRY_RUN=1
        shift
        ;;
      --path)
        (($# >= 2)) || die "--path requires an argument" "$EX_USAGE"
        TARGET_PATH=$2
        shift 2
        ;;
      --path=*)
        TARGET_PATH=${1#--path=}
        [[ -n $TARGET_PATH ]] || die "--path requires a non-empty argument" "$EX_USAGE"
        shift
        ;;
      --)
        shift
        break
        ;;
      -*)
        die "unknown option: $1" "$EX_USAGE"
        ;;
      *)
        break
        ;;
    esac
  done

  REMAINING_ARGS=("$@")
}

dispatch() {
  local command=${1:-help}
  if (($#)); then
    shift
  fi

  case "$command" in
    check) cmd_check "$@" ;;
    clean) cmd_clean "$@" ;;
    help) cmd_help "$@" ;;
    *) die "unknown command: $command" "$EX_USAGE" ;;
  esac
}

main() {
  trap cleanup EXIT
  trap 'on_error $LINENO' ERR

  parse_global_options "$@"
  dispatch "${REMAINING_ARGS[@]}"
}

if [[ ${BASH_SOURCE[0]} == "$0" ]]; then
  main "$@"
fi
```

Notes on the answer:

- `main "$@"` preserves every argument exactly as the user passed it.
- The source-safe guard lets tests source the file without running the CLI.
- `parse_global_options` stops at the first non-option so commands can have their own options later.
- `REMAINING_ARGS` is an intentional global. It is contained to the parse/dispatch boundary instead of being used across business logic.
- `check` prints `git status --short` to stdout, while progress and warnings go to stderr.
- `clean --dry-run` prints planned removals to stdout and does not call `rm`.
- Destructive `rm` calls go through `run`.
- `status_cmd=(git -C "$TARGET_PATH" status --short)` shows a command array that preserves argument boundaries.
- `check_shell_scripts` treats missing ShellCheck as a warning, not a hard failure, because the exercise says to run it if installed.

Manual verification:

```bash
bash -n repo-tool.sh
shellcheck repo-tool.sh
shfmt -w repo-tool.sh
bash repo-tool.sh --help
bash repo-tool.sh --path . check
tmprepo=$(mktemp -d)
git -C "$tmprepo" init
touch "$tmprepo/one.tmp" "$tmprepo/two words.tmp"
bash repo-tool.sh --path "$tmprepo" --dry-run clean
test -e "$tmprepo/one.tmp"
bash repo-tool.sh --path "$tmprepo" clean
test ! -e "$tmprepo/one.tmp"
rm -rf -- "$tmprepo"
```

Windows PowerShell verification with WSL:

```powershell
$env:REPO_TOOL_PATH = "."
wsl sudo apt update
wsl sudo apt install -y shellcheck shfmt bats
wsl bash -n repo-tool.sh
wsl shellcheck repo-tool.sh
wsl shfmt -w repo-tool.sh
wsl bash repo-tool.sh check
wsl bash repo-tool.sh --path . --dry-run clean
```

Windows PowerShell verification with Git Bash installed:

```powershell
$env:REPO_TOOL_PATH = "."
winget install --id Git.Git -e --source winget
bash -n repo-tool.sh
shellcheck repo-tool.sh  # if installed and on PATH
shfmt -w repo-tool.sh    # if installed and on PATH
bash repo-tool.sh check
bash repo-tool.sh --path . --dry-run clean
```

macOS Apple Silicon zsh verification:

```zsh
export REPO_TOOL_PATH=.
bash -n repo-tool.sh
shellcheck repo-tool.sh
shfmt -w repo-tool.sh
bash repo-tool.sh check
bash repo-tool.sh --path . --dry-run clean
```

Optional Bats tests can run in WSL, Git Bash, Docker, or macOS Terminal. They are not native PowerShell tests.

```bash
#!/usr/bin/env bats

setup() {
  tmpdir=$(mktemp -d)
  git -C "$tmpdir" init >/dev/null
  cp "$BATS_TEST_DIRNAME/../repo-tool.sh" "$tmpdir/repo-tool.sh"
}

teardown() {
  rm -rf "$tmpdir"
}

@test "help succeeds" {
  run bash "$tmpdir/repo-tool.sh" --help
  [ "$status" -eq 0 ]
  [[ "$output" == *"Usage:"* ]]
}

@test "dry-run clean reports tmp files without deleting them" {
  touch "$tmpdir/example.tmp"

  run bash "$tmpdir/repo-tool.sh" --path "$tmpdir" --dry-run clean

  [ "$status" -eq 0 ]
  [[ "$output" == *"would remove:"* ]]
  [ -e "$tmpdir/example.tmp" ]
}

@test "check rejects a non-repository path" {
  plain_dir=$(mktemp -d)

  run bash "$tmpdir/repo-tool.sh" --path "$plain_dir" check

  [ "$status" -eq 69 ]
  [[ "$output" == *"not a Git work tree"* ]]
  rm -rf "$plain_dir"
}
```

One Bats nuance: common `run` usage captures stdout and stderr together in `$output`, so the final test can inspect the error text even though the script writes it to stderr.

## Next Step

Return to the [Bash Advanced README](README.md) and use this lesson's structure when building the level capstone.

## Sources Used

- GNU Bash Manual: [Shell Parameters](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameters.html)
- GNU Bash Manual: [The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)
- GNU Bash Manual: [Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html)
- POSIX: [Utility Syntax Guidelines](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html)
- Google Shell Style Guide: [Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- ShellCheck: [Shell script analysis tool](https://www.shellcheck.net/)
- shfmt: [Shell formatter](https://github.com/mvdan/sh)
- bats-core: [Bash Automated Testing System documentation](https://bats-core.readthedocs.io/)
- Microsoft Learn: [Install WSL](https://learn.microsoft.com/windows/wsl/install)
- Microsoft Learn: [WSL documentation](https://learn.microsoft.com/windows/wsl/)
- Git: [Install Git for Windows](https://git-scm.com/install/windows)
- Git for Windows: [Project site](https://gitforwindows.org/)
- Apple Support: [Change the default shell in Terminal on Mac](https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac)
- Homebrew Formulae: [bats-core](https://formulae.brew.sh/formula/bats-core)
- Homebrew Formulae: [shellcheck](https://formulae.brew.sh/formula/shellcheck)
- Homebrew Formulae: [shfmt](https://formulae.brew.sh/formula/shfmt)
