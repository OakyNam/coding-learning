# 10 - Project Automation Scripts

## Learning Goal

Write one Bash script that automates common project tasks with safe argument handling, clear output, and reliable failure behavior.

## What Project Automation Means

Project automation means collecting the commands people run again and again into one small, predictable entry point. Instead of remembering separate commands for linting, testing, building, removing generated files, or printing help, a project can provide one script such as `project.sh`.

Common project tasks include:

- `check`: validate scripts, formatting, configuration, or other quick static checks.
- `test`: run the project's test suite.
- `build`: create generated output or release artifacts.
- `clean`: remove generated output created by the project.
- `help`: show the supported commands and options.

Bash is a good fit for small wrappers around existing tools. It is especially useful when the script mostly coordinates commands such as `bash -n`, `shellcheck`, `pytest`, `npm test`, `make`, `cp`, `rm`, or project-specific helper scripts. If the automation grows into complex data processing, a general-purpose language may become easier to test and maintain.

## Script Skeleton

A project automation script should make its contract visible near the top: strict-enough shell settings, usage text, logging, failure helpers, a command runner, and one `main "$@"` call at the bottom.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

usage() {
  printf 'Usage: %s [-n] [-v] COMMAND\n' "$0"
  printf 'Commands: check, test, build, clean, help\n'
}

log() {
  printf '%s\n' "$*" >&2
}

die() {
  printf 'Error: %s\n' "$*" >&2
  exit 2
}

run() {
  log "+ $*"
  "$@"
}

main() {
  # Parse options and dispatch commands here.
  :
}

main "$@"
```

The shebang asks the system to run the script with Bash. `set -Eeuo pipefail` makes many failures visible:

- `-E` keeps `ERR` traps active in functions and subshells if you add a trap later.
- `-e` exits on many unhandled command failures.
- `-u` treats unset variables as errors.
- `pipefail` makes a pipeline fail when any command in the pipeline fails, not only the last command.

These settings help, but they are not a substitute for deliberate checks. `errexit` has exceptions in conditionals, loops, boolean lists, and other contexts. `nounset` catches typos but requires defaults such as `${name:-}` when a variable may be empty or absent.

## Safe Paths

Project scripts should work from any directory. Do not assume the caller is already standing in the project root. Resolve paths from the script's own location instead.

```bash
script_dir=$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)
project_root=$script_dir
build_dir=$project_root/build
```

`BASH_SOURCE[0]` is the path to the current script. `dirname` gets the script's directory. `cd` and `pwd` turn that into an absolute directory. Quote each expansion so spaces, tabs, and glob characters in paths stay part of the path instead of becoming separate words or filename patterns.

Use project-rooted paths for destructive commands:

```bash
rm -rf -- "$project_root/build"
```

That is much safer than `rm -rf build`, which depends on the current working directory.

## Command Dispatch

After parsing options, choose the requested task with `case`. This keeps the supported command list explicit.

```bash
command_name=${1:-help}

case "$command_name" in
  check)
    do_check
    ;;
  test)
    do_test
    ;;
  build)
    do_build
    ;;
  clean)
    do_clean
    ;;
  help)
    usage
    ;;
  *)
    die "unknown command: $command_name"
    ;;
esac
```

The `*)` branch is the error path for unsupported commands. Send that error to STDERR and exit with a non-zero status.

## Options With getopts

Use `getopts` for short options such as `-n`, `-v`, and `-h`.

```bash
dry_run=0
verbose=0

while getopts ':nvh' opt; do
  case "$opt" in
    n)
      dry_run=1
      ;;
    v)
      verbose=1
      ;;
    h)
      usage
      exit 0
      ;;
    \?)
      die "unknown option: -$OPTARG"
      ;;
  esac
done

shift "$((OPTIND - 1))"
```

`OPTIND` is the index of the next argument to process. After the loop, `shift "$((OPTIND - 1))"` removes parsed options so the remaining arguments begin with the command name.

`OPTARG` contains the value for an option that takes an argument. In this lesson the options do not take values, but if you added `-c CONFIG`, the optstring would include `c:` and the `c)` branch would read the path from `$OPTARG`.

## Functions And Local Variables

Put each task in a function. Inside functions, use `local` for variables that should not leak into the rest of the script.

```bash
do_build() {
  local src_dir=$project_root/src
  local out_dir=$project_root/build/src

  run mkdir -p -- "$out_dir"
  run cp -R -- "$src_dir/." "$out_dir/"
}
```

Keep `main "$@"` as the last line of the script. That makes the file easy to scan: definitions first, execution last.

## Quoting And Arrays

Quote variable expansions unless you intentionally want word splitting and glob expansion. In project scripts, you almost never want that.

Risky:

```bash
cp -R $src_dir/. $out_dir/
```

Safe:

```bash
cp -R -- "$src_dir/." "$out_dir/"
```

Use arrays when building commands from pieces. Then run them with `"${cmd[@]}"` so each argument stays separate.

```bash
cmd=(bash -n "$file")
run "${cmd[@]}"
```

This matters for paths like `scripts/run tests.sh`. The array preserves that as one argument.

## Failure And Output

Make normal progress output clear, and send errors to STDERR.

```bash
log() {
  printf '%s\n' "$*"
}

die() {
  printf 'Error: %s\n' "$*" >&2
  exit 2
}
```

A `run` helper makes executed commands visible. It can also support dry-run mode in one place.

```bash
run() {
  printf '+' >&2
  printf ' %q' "$@" >&2
  printf '\n' >&2

  if [ "$dry_run" -eq 0 ]; then
    "$@"
  fi
}
```

With `set -e`, a failed command run this way usually stops the script. With `pipefail`, a failed command inside a pipeline is more likely to stop the script too. Still check important preconditions yourself, such as whether `src/` or `tests/run.sh` exists, because a clear project-specific error is better than a confusing tool failure.

## Common Mistakes

- Assuming the script is run from the project root, then accidentally building or deleting files in the caller's current directory.
- Writing `rm -rf build` instead of removing the project-rooted `"$project_root/build"` path.
- Expanding paths as `$project_root/src` instead of `"$project_root/src"`, which breaks paths with spaces and can trigger glob expansion.
- Building command strings such as `cmd="bash -n $file"` and running them with `eval`. Use arrays instead.
- Forgetting `shift "$((OPTIND - 1))"` after `getopts`, so `-n` or `-v` is mistaken for the command.
- Letting `clean` remove paths outside the project because the path was not resolved from `BASH_SOURCE[0]`.
- Printing errors to STDOUT, which makes it harder to separate normal output from failures in CI logs.
- Treating `set -e` as complete error handling instead of validating required files and directories.

## Exercise

Create `project.sh` in a small Bash project.

Requirements:

- Support `./project.sh check`, `./project.sh test`, `./project.sh build`, `./project.sh clean`, and `./project.sh help`.
- Support flags `-n` for dry-run, `-v` for verbose output, and `-h` for help.
- Work from any current directory.
- Resolve the project root from the script location using `BASH_SOURCE[0]`, `dirname`, and `pwd`.
- Quote paths.
- Send errors to STDERR.
- Use functions, `case`, `getopts`, arrays, and `main "$@"`.
- `check` runs `bash -n` on `.sh` files outside `.git`.
- `test` runs `tests/run.sh`.
- `build` copies `src/` into `build/src/`.
- `clean` removes only the project's `build/` directory.

Suggested project shape:

```text
my-project/
  project.sh
  src/
    app.sh
  tests/
    run.sh
```

## Worked Answer

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

dry_run=0
verbose=0

script_dir=$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd)
project_root=$script_dir
build_dir=$project_root/build

usage() {
  printf 'Usage: %s [-n] [-v] COMMAND\n' "$0"
  printf '\n'
  printf 'Options:\n'
  printf '  -n  Show commands without running them\n'
  printf '  -v  Print extra progress messages\n'
  printf '  -h  Show this help\n'
  printf '\n'
  printf 'Commands:\n'
  printf '  check  Run bash -n on .sh files outside .git\n'
  printf '  test   Run tests/run.sh\n'
  printf '  build  Copy src/ into build/src/\n'
  printf '  clean  Remove the project build/ directory\n'
  printf '  help   Show this help\n'
}

log() {
  printf '%s\n' "$*" >&2
}

debug() {
  if [ "$verbose" -eq 1 ]; then
    log "$@"
  fi
}

die() {
  printf 'Error: %s\n' "$*" >&2
  exit 2
}

run() {
  printf '+' >&2
  printf ' %q' "$@" >&2
  printf '\n' >&2

  if [ "$dry_run" -eq 0 ]; then
    "$@"
  fi
}

do_check() {
  local file
  local found=0
  local cmd

  debug "checking shell scripts under $project_root"

  while IFS= read -r -d '' file; do
    found=1
    cmd=(bash -n "$file")
    run "${cmd[@]}"
  done < <(find "$project_root" -path "$project_root/.git" -prune -o -type f -name '*.sh' -print0)

  if [ "$found" -eq 0 ]; then
    log "No .sh files found."
  fi
}

do_test() {
  local test_script=$project_root/tests/run.sh
  local cmd

  [ -f "$test_script" ] || die "missing test script: $test_script"

  cmd=(bash "$test_script")
  run "${cmd[@]}"
}

do_build() {
  local src_dir=$project_root/src
  local out_dir=$build_dir/src
  local cmd

  [ -d "$src_dir" ] || die "missing source directory: $src_dir"

  cmd=(mkdir -p "$out_dir")
  run "${cmd[@]}"

  cmd=(cp -R "$src_dir/." "$out_dir/")
  run "${cmd[@]}"
}

do_clean() {
  local target=$build_dir
  local cmd

  case "$target" in
    "$project_root"/build)
      cmd=(rm -rf "$target")
      run "${cmd[@]}"
      ;;
    *)
      die "refusing to clean unexpected path: $target"
      ;;
  esac
}

main() {
  local opt
  local command_name

  while getopts ':nvh' opt; do
    case "$opt" in
      n)
        dry_run=1
        ;;
      v)
        verbose=1
        ;;
      h)
        usage
        return 0
        ;;
      \?)
        die "unknown option: -$OPTARG"
        ;;
    esac
  done

  shift "$((OPTIND - 1))"

  command_name=${1:-help}

  case "$command_name" in
    check)
      do_check
      ;;
    test)
      do_test
      ;;
    build)
      do_build
      ;;
    clean)
      do_clean
      ;;
    help)
      usage
      ;;
    *)
      die "unknown command: $command_name"
      ;;
  esac
}

main "$@"
```

Expected checks:

```bash
bash -n project.sh
./project.sh help
./project.sh -n check
./project.sh -n build
./project.sh -n clean
(cd /tmp && /path/to/my-project/project.sh -n check)
```

Expected behavior:

- `bash -n project.sh` reports no syntax errors.
- `./project.sh help` prints usage and exits successfully.
- `./project.sh -n check` prints the `bash -n` commands it would run.
- `./project.sh -n build` prints the `mkdir` and `cp` commands without creating `build/`.
- `./project.sh -n clean` prints the project-rooted `rm -rf` command without deleting anything.
- Running the script from another directory still checks files in the project, not in the caller's directory.

## Next Step

Add one project-specific command, such as `format` or `package`, using the same pattern: validate paths first, build the command in an array, run it through `run`, and dispatch it with `case`.

## Sources Used

- GNU Bash Reference Manual, Shell Scripts: https://www.gnu.org/software/bash/manual/html_node/Shell-Scripts.html
- GNU Bash Reference Manual, Shell Functions: https://www.gnu.org/software/bash/manual/html_node/Shell-Functions.html
- GNU Bash Reference Manual, Bourne Shell Builtins (`getopts`, `set`): https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html
- GNU Bash Reference Manual, The Set Builtin: https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html
- GNU Bash Reference Manual, Arrays: https://www.gnu.org/software/bash/manual/html_node/Arrays.html
- ShellCheck SC2086, Double quote to prevent globbing and word splitting: https://www.shellcheck.net/wiki/SC2086
- Google Shell Style Guide: https://google.github.io/styleguide/shellguide.html
