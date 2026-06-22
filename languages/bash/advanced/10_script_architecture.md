# 10 - Script Architecture

## Learning Goal

Build a maintainable Bash CLI that accepts global options, dispatches subcommands, validates dependencies, logs clearly, keeps business logic in small testable functions, and explains when Bash should be split into libraries or replaced with another language.

## Why Architecture Matters

Bash starts as glue: run a command, pipe some text, move a file, call `git`, wrap a deployment step. That is exactly what it is good at. The trouble begins when yesterday's five-line helper becomes today's shared command-line tool with options, cleanup behavior, CI output, dry-run mode, and three people editing it.

Architecture for shell scripts is not about imitating a large application framework. It is about making the script's promises visible:

- What arguments does it accept?
- Which output is meant for another program and which output is meant for a human?
- Which commands must be installed?
- Which failures are user mistakes, environment problems, or failed checks?
- Which functions can be tested without touching the filesystem?
- Where does execution actually begin?

A well-shaped Bash script is easier to review because the risky parts are not scattered everywhere. Parsing happens in one place, validation happens before work starts, helper functions do one thing, and command functions are the public surface of the tool.

## A Useful Script Shape

For a non-trivial Bash CLI, use a consistent order:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

readonly SCRIPT_NAME=${0##*/}
readonly DEFAULT_PATH=.

VERBOSE=0
DRY_RUN=0
TARGET_PATH=$DEFAULT_PATH

usage() { ...; }
log() { ...; }
debug() { ...; }
die() { ...; }
run() { ...; }
require_cmd() { ...; }

pure_or_small_helper() { ...; }

cmd_check() { ...; }
cmd_clean() { ...; }
cmd_help() { ...; }

parse_global_options() { ...; }
dispatch() { ...; }
main() { ...; }

main "$@"
```

The names are not magic. The shape is the point:

- Shebang: `#!/usr/bin/env bash` says this is Bash, not POSIX `sh`.
- Strict settings: `set -Eeuo pipefail` makes unexpected failures visible. Use explicit handling for expected non-zero statuses.
- Constants and defaults: keep defaults near the top so users and reviewers can see the policy.
- `usage`: document command syntax, options, commands, examples, and exit statuses.
- Logging helpers: send human-readable diagnostics to stderr.
- `die`: print one clear error and exit with a non-zero status.
- `run`: centralize dry-run behavior and command logging.
- Dependency checks: fail early when required tools are missing.
- Pure-ish helper functions: make decisions with arguments and return output or statuses without modifying global state.
- Command functions: put each subcommand behind `cmd_name`.
- Parse, dispatch, main: separate CLI mechanics from business logic.
- `main "$@"`: preserve the user's original argument boundaries.

## CLI Layout

A maintainable multi-command script usually follows this layout:

```text
script [global-options] command [command-options] [args]
```

For example:

```bash
repo-tool.sh --path ~/work/project --verbose check
repo-tool.sh --path . --dry-run clean
repo-tool.sh help
```

Global options affect the whole program. Command options belong to one command. Keep that boundary clear. If `--path` affects every command, parse it before dispatch. If `--force` only affects `clean`, parse it inside `cmd_clean`.

Good CLI behavior also follows ordinary Unix conventions:

- `-h` or `--help` prints usage and exits `0`.
- Invalid options print a short error to stderr and exit non-zero.
- Normal machine-readable results go to stdout.
- Logs, warnings, prompts, and errors go to stderr.
- Exit status `0` means success. Non-zero statuses should have documented meanings.
- `--` ends option parsing when a following operand might begin with `-`.
- Short options use one hyphen, such as `-v`; long options use two, such as `--verbose`.

POSIX utility conventions are stricter than many modern CLIs: options are normally single letters, option arguments are separate arguments, and operands follow options. Bash scripts often add GNU-style long options manually because `getopts` only handles short options portably.

## Preserving Arguments With `main "$@"`

Always pass the original arguments as `"$@"`, not `$@`, `$*`, or `"$*"`.

```bash
main "$@"
```

`"$@"` expands to one quoted word per original argument. That preserves spaces, empty strings, and glob characters:

```bash
./repo-tool.sh --path "project with spaces" check
```

Inside `main`, `"$@"` still contains `--path`, `project with spaces`, and `check` as three separate arguments. Unquoted `$@` invites word splitting and pathname expansion. `"$*"` joins all arguments into one string, which loses the original boundaries.

## Option Parsing

Use `getopts` when you only need short options:

```bash
while getopts ":hvn" opt; do
  case "$opt" in
    h) usage; exit 0 ;;
    v) VERBOSE=1 ;;
    n) DRY_RUN=1 ;;
    :) die "option -$OPTARG requires an argument" 64 ;;
    \?) die "unknown option: -$OPTARG" 64 ;;
  esac
done
shift "$((OPTIND - 1))"
```

For long options, use a manual `case` loop:

```bash
while (($#)); do
  case "$1" in
    -h|--help) usage; exit 0 ;;
    -v|--verbose) VERBOSE=1; shift ;;
    -n|--dry-run) DRY_RUN=1; shift ;;
    --path)
      (($# >= 2)) || die "--path requires an argument" 64
      TARGET_PATH=$2
      shift 2
      ;;
    --path=*)
      TARGET_PATH=${1#--path=}
      shift
      ;;
    --)
      shift
      break
      ;;
    -*)
      die "unknown option: $1" 64
      ;;
    *)
      break
      ;;
  esac
done
```

Manual parsing is more code, but it is clear and portable across Bash installations. Avoid external `getopt` unless you have checked the target systems, because behavior differs between platforms.

## Dispatch

After global options are parsed, the next argument is the command:

```bash
dispatch() {
  local command=${1:-}
  shift || true

  case "$command" in
    check) cmd_check "$@" ;;
    clean) cmd_clean "$@" ;;
    help|"") cmd_help "$@" ;;
    *) die "unknown command: $command" 64 ;;
  esac
}
```

Command functions are where the actual behavior starts. They should accept their own arguments, use local variables, call helpers, and return meaningful statuses.

## Functions, Globals, and Testability

Bash has global variables by default. Use `local` inside functions unless a variable is intentionally global configuration such as `VERBOSE`, `DRY_RUN`, or `TARGET_PATH`.

Prefer helpers that are easy to test:

```bash
is_git_repo() {
  local path=$1
  git -C "$path" rev-parse --is-inside-work-tree >/dev/null 2>&1
}
```

That function has a simple contract: given a path, return success if Git recognizes it as a work tree. It does not parse CLI options, print banners, remove files, or mutate globals. A Bats test can create a temporary repository and assert the status.

Keep side effects at the edges:

- Parsing sets configuration.
- Validation checks the environment.
- Pure-ish helpers compute or query.
- Command functions coordinate.
- `run` performs commands and honors dry-run mode.

This separation makes it possible to test the dangerous parts indirectly. For example, `clean` can call a helper that finds `*.tmp` files, and the destructive `rm` call can be routed through `run`.

## Dependency Checks

Check required commands before using them:

```bash
require_cmd() {
  command -v "$1" >/dev/null 2>&1 || die "missing required command: $1" 69
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

## ShellCheck and Bats

Run ShellCheck before treating a script as done:

```bash
shellcheck repo-tool.sh
```

ShellCheck catches many mistakes that are easy to miss in review: unquoted expansions, unreachable branches, masked statuses, invalid `[` tests, and accidental POSIX/Bash mismatches.

For behavior, use manual command examples for small scripts and Bats when the script will keep changing. Bats tests run the script as a user would, inspect stdout, stderr, exit status, and filesystem effects, and protect you from breaking CLI behavior during refactors.

## When to Split or Rewrite

Split a Bash script into sourced library files when:

- Several scripts share the same logging, parsing, or validation helpers.
- Tests need to source helpers without executing the CLI.
- One file has grown so large that command functions are hard to find.

When you split, keep the executable script small:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

script_dir=$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)
source "$script_dir/lib/logging.bash"
source "$script_dir/lib/repo.bash"

main "$@"
```

Rewrite in Python, Ruby, Go, or another general-purpose language when:

- The script does complex data structures, JSON manipulation, or API workflows.
- Error handling needs typed exceptions or richer recovery.
- Tests require heavy mocking or many fixtures.
- Portability across shells and operating systems is becoming expensive.
- The script spends more code managing Bash edge cases than doing useful work.

Bash is excellent for orchestrating processes. It is a poor place to build a large application.

## Common Mistakes

- Putting executable code throughout the file instead of using `main "$@"`.
- Calling `main $@` and breaking paths with spaces.
- Mixing global option parsing into every command function.
- Sending logs to stdout, which breaks callers that want to pipe real output.
- Returning `0` after failed checks because an `echo` or `printf` was the last command.
- Using `exit` inside helper functions that should return a status to the caller.
- Forgetting `local`, then accidentally overwriting global configuration.
- Using `eval` to build commands instead of arrays or direct quoted arguments.
- Treating dry-run as a printed message while still running the destructive command.
- Checking dependencies after half the script has already run.
- Ignoring ShellCheck warnings because the script "works on my machine."
- Keeping thousands of lines in Bash after the problem has become data-heavy application logic.

## Exercise

Write `repo-tool.sh`, a small repository maintenance CLI.

Required global options:

- `-h`, `--help`: print usage and exit `0`.
- `-v`, `--verbose`: print debug logs to stderr.
- `-n`, `--dry-run`: show cleanup actions without deleting files.
- `--path PATH`: repository path to inspect or clean. Default: current directory.

Required commands:

- `check`: verify that the path exists, Git is installed, the path is a Git work tree, print `git status --short`, and run ShellCheck on `*.sh` files if ShellCheck is installed.
- `clean`: remove `*.tmp` files under the target path. In dry-run mode, print what would be removed without deleting anything.
- `help`: print usage.

Behavior requirements:

- Use the script shape from this lesson.
- Preserve arguments with `main "$@"`.
- Send logs and errors to stderr.
- Keep stdout for command results such as `git status --short` and dry-run file lists.
- Return meaningful statuses: success, invalid usage, unavailable dependency, and failed checks.
- Keep business logic in small functions that can be tested manually or with Bats.

## Worked Answer

Save this as `repo-tool.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

readonly SCRIPT_NAME=${0##*/}
readonly DEFAULT_PATH=.
readonly EX_USAGE=64
readonly EX_UNAVAILABLE=69

VERBOSE=0
DRY_RUN=0
TARGET_PATH=$DEFAULT_PATH

usage() {
  cat <<USAGE
Usage:
  $SCRIPT_NAME [global-options] command [args]

Global options:
  -h, --help        Show this help and exit
  -v, --verbose     Print debug logs to stderr
  -n, --dry-run     Print cleanup actions without deleting files
      --path PATH   Repository path to inspect or clean (default: .)

Commands:
  check             Verify path, Git repository status, and shell scripts
  clean             Remove *.tmp files under the target path
  help              Show this help

Examples:
  $SCRIPT_NAME --path . check
  $SCRIPT_NAME --path ~/work/project --verbose check
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

require_cmd() {
  local name=$1
  command -v "$name" >/dev/null 2>&1 || die "missing required command: $name" "$EX_UNAVAILABLE"
}

validate_path() {
  local path=$1
  [[ -d "$path" ]] || die "path is not a directory: $path" "$EX_UNAVAILABLE"
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
    return 0
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
  git -C "$TARGET_PATH" status --short
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
        exit 0
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
        [[ -n "$TARGET_PATH" ]] || die "--path requires a non-empty argument" "$EX_USAGE"
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
  REMAINING_ARGS=()
  parse_global_options "$@"
  dispatch "${REMAINING_ARGS[@]}"
}

main "$@"
```

Notes on the answer:

- `main "$@"` preserves every argument exactly as the user passed it.
- `parse_global_options` stops at the first non-option so commands can have their own options later.
- `check` prints `git status --short` to stdout, while progress and warnings go to stderr.
- `clean --dry-run` prints planned removals to stdout and does not call `rm`.
- `check_shell_scripts` treats missing ShellCheck as a warning, not a hard failure, because the exercise says to run it if installed.
- The helper functions `is_git_repo`, `list_tmp_files`, and `list_shell_scripts` are small enough to test directly or through commands.

Manual verification examples:

```bash
chmod +x repo-tool.sh
./repo-tool.sh --help
./repo-tool.sh --path . check
touch one.tmp "two words.tmp"
./repo-tool.sh --path . --dry-run clean
test -e one.tmp
./repo-tool.sh --path . clean
test ! -e one.tmp
```

Minimal Bats examples:

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
  run "$tmpdir/repo-tool.sh" --help
  [ "$status" -eq 0 ]
  [[ "$output" == *"Usage:"* ]]
}

@test "dry-run clean reports tmp files without deleting them" {
  touch "$tmpdir/example.tmp"

  run "$tmpdir/repo-tool.sh" --path "$tmpdir" --dry-run clean

  [ "$status" -eq 0 ]
  [[ "$output" == *"would remove:"* ]]
  [ -e "$tmpdir/example.tmp" ]
}

@test "check rejects a non-repository path" {
  plain_dir=$(mktemp -d)

  run "$tmpdir/repo-tool.sh" --path "$plain_dir" check

  [ "$status" -eq 69 ]
  [[ "$output" == *"not a Git work tree"* ]]
  rm -rf "$plain_dir"
}
```

One Bats nuance: `run` combines stdout and stderr into `$output` by default in common Bats usage, so the final test can inspect the error text even though the script writes it to stderr.

## Next Step

Return to the [Bash Advanced README](README.md) and use this lesson's structure when building the level capstone.

## Sources Used

- GNU Bash Manual: [Shell Parameters](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameters.html)
- GNU Bash Manual: [The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)
- GNU Bash Manual: [Bourne Shell Builtins: getopts](https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html)
- POSIX: [Utility Syntax Guidelines](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html)
- Google Shell Style Guide: [Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- ShellCheck: [Shell script analysis tool](https://www.shellcheck.net/)
- bats-core: [Bash Automated Testing System documentation](https://bats-core.readthedocs.io/)
