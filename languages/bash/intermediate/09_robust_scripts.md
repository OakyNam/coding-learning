# 09 - Robust Scripts

## Learning Goal

Write Bash scripts that fail early, report useful errors, handle spaces in filenames, clean up temp files, and return meaningful exit statuses.

## Why It Matters

Small shell scripts often start as quick commands pasted into a file. That is fine for experiments, but real scripts need to behave predictably when input is missing, filenames contain spaces, a command fails, or a temporary directory needs to be removed.

A robust Bash script is not one that can never fail. It is one that fails clearly, cleans up after itself, and tells other programs whether the work succeeded.

## Platform Note

This lesson uses Bash arrays, traps, `mktemp`, and `tar`.

- On Windows 10/11, run these examples inside WSL or Git Bash, not directly in PowerShell.
- On macOS Apple Silicon, Terminal normally starts `zsh`. Type `bash` first when following Bash-specific examples.
- Archive metadata and command details can vary slightly between GNU tar and BSD tar. The exercise focuses on script structure, cleanup, quoting, and exit behavior.
- Run commands from a scratch directory so generated directories, temporary archives, and test files stay easy to inspect and remove.

## Core Idea

A robust script makes its assumptions visible:

- Declare the interpreter with a shebang.
- Enable shell options intentionally.
- Validate arguments before doing work.
- Quote expansions so spaces and glob characters are preserved.
- Send status and error messages to STDERR.
- Handle failures with clear messages.
- Clean temporary resources.
- Use arrays for command arguments and file lists.
- Avoid parsing `ls`.

The goal is to make each risky part of the script explicit: inputs, filenames, command failures, temporary files, and exit status.

## The Robust Script Starter Pattern

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

usage() {
  printf 'Usage: %s SOURCE_DIR DEST_DIR EXTENSION\n' "${0##*/}" >&2
}

die() {
  printf 'error: %s\n' "$*" >&2
  exit 1
}
```

The shebang asks the environment to run the script with Bash. That matters because this lesson uses Bash features such as arrays and `shopt`.

`set -e` asks Bash to exit when an unhandled simple command fails. It is useful, but it is not magic. Some contexts, such as commands tested by `if`, `while`, `until`, `&&`, and `||`, change how `-e` behaves.

`set -u` treats an unset variable as an error. This catches misspelled variable names, but it also means optional variables need defaults such as `${name:-}`.

`set -o pipefail` makes a pipeline fail if any command in the pipeline fails, not only the last command.

`set -E` makes an `ERR` trap inherited by shell functions, command substitutions, and subshells. Even with `-E`, prefer explicit checks for failures you expect:

```bash
if ! mkdir -p "$dest_dir"; then
  die "could not create destination directory: $dest_dir"
fi
```

Shell options help catch mistakes early. They do not replace clear validation, readable error messages, or deliberate handling of expected failures.

## Example

This script safely archives direct child files from a source directory. It accepts `SOURCE_DIR DEST_DIR EXTENSION`, supports `txt` or `.txt`, handles spaces in filenames, writes the final archive path to STDOUT, and sends status or errors to STDERR.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

usage() {
  printf 'Usage: %s SOURCE_DIR DEST_DIR EXTENSION\n' "${0##*/}" >&2
}

die() {
  printf 'error: %s\n' "$*" >&2
  exit 1
}

if (($# != 3)); then
  usage
  exit 2
fi

source_dir=$1
dest_dir=$2
extension=$3
extension=${extension#.}

[[ -d $source_dir ]] || die "source directory does not exist: $source_dir"
[[ -n $extension ]] || die "extension must not be empty"

if ! mkdir -p "$dest_dir"; then
  die "could not create destination directory: $dest_dir"
fi

tmp_dir=$(mktemp -d) || die "could not create temporary directory"
cleanup() {
  rm -rf "$tmp_dir"
}
trap cleanup EXIT

shopt -s nullglob
files=("$source_dir"/*."$extension")

regular_files=()
for file in "${files[@]}"; do
  if [[ -f $file ]]; then
    regular_files+=("$file")
  fi
done

((${#regular_files[@]} > 0)) || die "no .$extension files found in: $source_dir"

manifest="$tmp_dir/manifest.txt"
: > "$manifest"

for file in "${regular_files[@]}"; do
  cp -- "$file" "$tmp_dir/" || die "could not copy file: $file"
  printf '%s\n' "${file##*/}" >> "$manifest"
done

source_name=${source_dir%/}
source_name=${source_name##*/}
timestamp=$(date +%Y%m%d-%H%M%S)
archive="$dest_dir/$source_name-$timestamp.tar.gz"

if ! tar -czf "$archive" -C "$tmp_dir" .; then
  die "could not create archive: $archive"
fi

printf 'archived %d file(s)\n' "${#regular_files[@]}" >&2
printf '%s\n' "$archive"
```

Notice the separation of output streams: the archive path goes to STDOUT so another command can capture it, while progress and errors go to STDERR for humans.

## How To Think About Robustness

Start by asking what can go wrong:

- Are the required arguments present?
- Does the source directory exist?
- Can the destination directory be created?
- What happens when no files match?
- Do filenames with spaces still work?
- Will temporary files be removed after success and failure?
- Does the script return a nonzero exit status when it cannot complete?

Then make each answer visible in the script. Use `if ! command; then ... fi` around failures you expect and want to explain. Use arrays for lists of paths so each filename stays one item, even when it contains spaces. Use globs or `find` for files; do not parse `ls`.

## Common Mistakes

- Writing `cp $file "$dest_dir/"` instead of `cp -- "$file" "$dest_dir/"`.
- Assuming `set -e` catches every failure in every context.
- Forgetting `pipefail`, so `grep pattern missing.txt | sort` can appear successful because `sort` succeeded.
- Using `for file in $(ls "$dir")`, which breaks on spaces and parses output meant for humans.
- Creating predictable temp paths such as `/tmp/my-script`.
- Sending errors to STDOUT instead of STDERR.
- Referencing optional variables under `set -u` without a default, such as `$name` instead of `${name:-}`.
- Using `((count++))` with `set -e`; when `count` is `0`, the arithmetic command returns status `1`. Prefer `((count += 1))`.

## Exercise

Write `safe_archive.sh`.

Requirements:

- Accept exactly three arguments: `SOURCE_DIR DEST_DIR EXTENSION`.
- Accept an extension with or without a leading dot, such as `txt` or `.txt`.
- Verify that the source directory exists.
- Create the destination directory if needed.
- Archive only direct matching regular files, not recursive matches.
- Correctly handle filenames with spaces.
- Exit nonzero when no files match.
- Stage files in a temporary directory created with `mktemp -d`.
- Include a `manifest.txt` file in the archive.
- Create `DEST_DIR/<source-name>-<timestamp>.tar.gz`.
- Clean up the temporary staging directory on success or failure.
- Send errors and status messages to STDERR.
- Print only the final archive path to STDOUT.
- Do not parse `ls`.

Manual tests:

```bash
mkdir -p demo-src demo-out
printf 'alpha\n' > 'demo-src/one.txt'
printf 'beta\n' > 'demo-src/two words.txt'
printf 'ignore\n' > 'demo-src/skip.md'

bash safe_archive.sh demo-src demo-out txt
bash safe_archive.sh demo-src demo-out .txt
bash safe_archive.sh missing demo-out txt
bash safe_archive.sh demo-src demo-out csv
```

Expected results:

- The first two commands print an archive path to STDOUT and status to STDERR.
- The created archive contains `one.txt`, `two words.txt`, and `manifest.txt`.
- The missing source test exits nonzero and reports an error to STDERR.
- The no-match test exits nonzero and reports an error to STDERR.

## Worked Answer

```bash
cat > safe_archive.sh <<'EOF'
#!/usr/bin/env bash
set -Eeuo pipefail

usage() {
  printf 'Usage: %s SOURCE_DIR DEST_DIR EXTENSION\n' "${0##*/}" >&2
}

die() {
  printf 'error: %s\n' "$*" >&2
  exit 1
}

if (($# != 3)); then
  usage
  exit 2
fi

source_dir=$1
dest_dir=$2
extension=$3
extension=${extension#.}

[[ -d $source_dir ]] || die "source directory does not exist: $source_dir"
[[ -n $extension ]] || die "extension must not be empty"

if ! mkdir -p "$dest_dir"; then
  die "could not create destination directory: $dest_dir"
fi

tmp_dir=$(mktemp -d) || die "could not create temporary directory"
cleanup() {
  rm -rf "$tmp_dir"
}
trap cleanup EXIT

shopt -s nullglob
matches=("$source_dir"/*."$extension")

files=()
for file in "${matches[@]}"; do
  if [[ -f $file ]]; then
    files+=("$file")
  fi
done

((${#files[@]} > 0)) || die "no .$extension files found in: $source_dir"

manifest="$tmp_dir/manifest.txt"
: > "$manifest"

for file in "${files[@]}"; do
  cp -- "$file" "$tmp_dir/" || die "could not copy file: $file"
  printf '%s\n' "${file##*/}" >> "$manifest"
done

source_name=${source_dir%/}
source_name=${source_name##*/}
timestamp=$(date +%Y%m%d-%H%M%S)
archive="$dest_dir/$source_name-$timestamp.tar.gz"

if ! tar -czf "$archive" -C "$tmp_dir" .; then
  die "could not create archive: $archive"
fi

printf 'archived %d file(s)\n' "${#files[@]}" >&2
printf '%s\n' "$archive"
EOF
```

Expected behavior:

- With matching files, the script exits `0`, prints the archive path to STDOUT, and prints a short status message to STDERR.
- With a missing source directory, invalid argument count, empty extension, destination creation failure, or no matching files, the script exits nonzero and prints the problem to STDERR.
- The temporary staging directory is removed because the `EXIT` trap runs whether the script succeeds or fails.
- Filenames with spaces are preserved because path expansions are quoted and file lists are stored in arrays.

## Next Step

Return to this level's README and continue with the next numbered lesson. As you write future Bash scripts, start from the starter pattern and add validation for the specific risks in that script.

## Sources Used

- GNU Bash Manual for shell options, traps, arrays, quoting, and `shopt`: https://www.gnu.org/software/bash/manual/bash.html
- ShellCheck SC2086 for why unquoted expansions cause word splitting and globbing: https://www.shellcheck.net/wiki/SC2086
- Google Shell Style Guide for clear error handling, quoting, and safe temporary-file practices: https://google.github.io/styleguide/shellguide.html
- GNU coreutils `mktemp` manual for safe temporary file and directory creation: https://www.gnu.org/software/coreutils/manual/html_node/mktemp-invocation.html
- GNU tar manual for archive creation with `tar -czf`: https://www.gnu.org/software/tar/manual/tar.html
- Microsoft Learn, "Install WSL": https://learn.microsoft.com/windows/wsl/install
- Git for Windows: https://gitforwindows.org/
- Apple Terminal User Guide, "Change the default shell": https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac
