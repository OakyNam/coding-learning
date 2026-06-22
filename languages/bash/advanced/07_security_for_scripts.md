# 07 - Security for Scripts

## Learning Goal

Write Bash scripts that treat file names, command arguments, temporary files, secrets, permissions, and user input as untrusted by default.

## Why It Matters

Bash scripts often run in places where mistakes have a large blast radius: CI jobs, deployment servers, backup tasks, production maintenance shells, and developer laptops with access to private keys. A script that works for friendly input can still be unsafe when an argument contains spaces, wildcards, command separators, `..`, or an unexpected option.

Security in Bash is mostly about reducing ambiguity. Quote expansions, keep data separate from code, validate paths before deleting, use private temporary files, pin a safe `PATH`, and run static checks before trusting a script.

## Threat Model for Bash Scripts

Start by naming what an attacker or accidental caller can influence:

- Command-line arguments such as `--name "$value"`.
- Environment variables such as `PATH`, `HOME`, `IFS`, tokens, and service URLs.
- Files and directories created by other users or earlier jobs.
- Output from commands, APIs, config files, and `git` metadata.
- The current working directory and shell options.

A secure script assumes these values may contain spaces, newlines, glob characters, leading dashes, shell metacharacters, symlinks, or traversal sequences.

Bad:

```bash
rm -rf $backup_dir/$name
```

Good:

```bash
target="${backup_dir}/${name}"
rm -rf -- "$target"
```

The good version is not complete path validation yet, but it avoids word splitting, globbing, and option confusion.

## Quote Expansions by Default

Unquoted parameter expansion is one of the most common Bash security bugs. After expansion, Bash performs word splitting and filename expansion. That means a single value can become many arguments, or a literal `*` can become matching files.

Bad:

```bash
cp $src $dest
```

If `src='my report.txt'`, `cp` receives `my` and `report.txt` as separate arguments.

Good:

```bash
cp -- "$src" "$dest"
```

Quote command substitutions too:

```bash
version="$(git describe --tags)"
printf 'version=%s\n' "$version"
```

The main exception is array expansion. Use `"${array[@]}"` when each array element should stay one argument.

```bash
args=(--color=always -- "$pattern" "$file")
grep "${args[@]}"
```

## Word Splitting And Globbing

Word splitting uses `IFS` to split unquoted expansions. Filename expansion, also called globbing, expands patterns such as `*`, `?`, and `[abc]`.

Bad:

```bash
files=$(find "$base" -type f -name '*.log')
rm -f $files
```

This breaks on spaces and newlines, and it lets glob characters be interpreted again.

Good:

```bash
find "$base" -type f -name '*.log' -print0 |
  while IFS= read -r -d '' file; do
    rm -f -- "$file"
  done
```

For known values, arrays are usually clearer:

```bash
files=("$base/app.log" "$base/error log.txt")
rm -f -- "${files[@]}"
```

## Avoid eval

`eval` takes a string and runs it as shell code. If any part of that string is influenced by input, data becomes executable code.

Bad:

```bash
cmd="grep $pattern $file"
eval "$cmd"
```

If `pattern='x; rm -rf "$HOME"'`, the shell parses the injected command.

Good:

```bash
grep -- "$pattern" "$file"
```

When you need dynamic arguments, use arrays instead of building a command string:

```bash
args=(-R --include='*.log' -- "$pattern" "$base")
grep "${args[@]}"
```

## Command Injection

Command injection happens when untrusted input changes the command being run. Quoting helps, but it is not a substitute for choosing safe APIs and passing arguments directly.

Bad:

```bash
user_input="$1"
bash -c "printf '%s\n' $user_input"
```

Good:

```bash
user_input="$1"
printf '%s\n' "$user_input"
```

If you must use `bash -c`, pass input as positional parameters, not by interpolating it into the code string:

```bash
bash -c 'printf "%s\n" "$1"' bash "$user_input"
```

## Path Traversal

Path traversal uses values such as `../`, absolute paths, or symlinks to escape the directory a script meant to use.

Bad:

```bash
name="$1"
rm -rf -- "$base/$name"
```

If `name='../../important'`, this can delete outside `base`.

Good:

```bash
case "$name" in
  ''|/*|*/*|*..*) echo "unsafe name" >&2; exit 1 ;;
esac

base_real="$(realpath -m -- "$base")"
target_real="$(realpath -m -- "$base/$name")"

case "$target_real" in
  "$base_real"/*) ;;
  *) echo "target escapes base" >&2; exit 1 ;;
esac
```

This lesson uses GNU coreutils `realpath -m`, which resolves paths even if the final component does not exist. For destructive scripts, also consider whether symlinks should be followed, rejected, or handled with a more specialized tool.

## Safe Temporary Files

Do not invent temporary names with `$$`, timestamps, or predictable paths. Attackers can pre-create those paths or replace them with symlinks.

Bad:

```bash
tmp="/tmp/my-script.$$"
echo "work" > "$tmp"
```

Good:

```bash
tmpdir="$(mktemp -d)"
trap 'rm -rf -- "$tmpdir"' EXIT HUP INT TERM
printf 'work\n' > "$tmpdir/input"
```

Use `mktemp -d` for a private directory, create files inside it, quote it, and clean it up with `trap`.

## Permissions And umask

The process `umask` controls default permissions for newly created files and directories. Scripts that create secrets, reports, or intermediate files should set restrictive defaults before creating anything sensitive.

Good:

```bash
umask 077
tmpdir="$(mktemp -d)"
printf '%s\n' "$private_value" > "$tmpdir/value"
```

With `umask 077`, new files are not readable or writable by group or others unless a command changes permissions explicitly.

## Secrets And Environment Variables

Environment variables are convenient, but they are not secret storage. They can leak through logs, child processes, crash reports, debug output, shell history, and process inspection on some systems.

Bad:

```bash
set -x
echo "deploying with token $API_TOKEN"
curl -H "Authorization: Bearer $API_TOKEN" "$url"
```

Good:

```bash
set +x
: "${API_TOKEN:?API_TOKEN is required}"
curl -H "Authorization: Bearer ${API_TOKEN}" -- "$url"
```

Do not print secrets. Do not store them in temporary files unless the file is private and short-lived. Prefer a secret manager or CI secret store when available.

## PATH Hijacking

If a script runs `tar`, `rm`, or `curl`, the shell finds the executable through `PATH`. An attacker who controls `PATH` or the working directory may cause your script to run a different program.

Bad:

```bash
export PATH=".:$PATH"
tar -czf "$archive" "$dir"
```

Good:

```bash
export PATH='/usr/sbin:/usr/bin:/sbin:/bin'
tar -czf "$archive" -- "$dir"
```

For especially sensitive scripts, use absolute command paths after checking the target platform.

## Safe rm Dry Runs And Confirmation

Deletion scripts need friction. Print exactly what will be deleted, support a dry run, and ask for confirmation unless the caller explicitly passes a yes flag.

Bad:

```bash
rm -rf -- "$target"
```

Better:

```bash
printf 'Target: %s\n' "$target"

if [[ "$dry_run" == true ]]; then
  printf 'Dry run: rm -rf -- %q\n' "$target"
elif [[ "$yes" == true ]]; then
  rm -rf -- "$target"
else
  read -r -p "Delete this target? Type yes: " answer
  [[ "$answer" == yes ]] || exit 1
  rm -rf -- "$target"
fi
```

Dry runs must not delete anything. Confirmation prompts should require a clear answer, not just pressing Enter.

## End Of Options

Many commands treat operands starting with `-` as options. Use `--` to mark the end of options before file names, patterns, and user-controlled operands.

Bad:

```bash
rm -rf "$name"
grep "$pattern" "$file"
```

Good:

```bash
rm -rf -- "$name"
grep -- "$pattern" "$file"
```

This matters even when input validation rejects most dangerous values, because `--` is cheap defense in depth.

## ShellCheck

ShellCheck catches many common Bash mistakes, including unquoted expansions. Warning SC2086 is the classic "double quote to prevent globbing and word splitting" warning.

Run it before trusting a script:

```bash
shellcheck safe-clean.sh
```

ShellCheck does not prove a script is secure. It is a fast reviewer that points out risky patterns so you can decide intentionally.

## Least Privilege Checklist

- Run as the least privileged user that can do the job.
- Restrict `PATH` before running external commands.
- Use `umask 077` before creating temporary or sensitive files.
- Validate user input before building paths.
- Resolve paths and verify containment before destructive operations.
- Quote expansions and use arrays for dynamic argument lists.
- Use `--` before user-controlled operands.
- Avoid `eval`, `source` of untrusted files, and command strings.
- Keep secrets out of logs, dry-run output, and temporary files.
- Add dry-run mode and explicit confirmation for destructive commands.
- Run ShellCheck and test with spaces, wildcards, leading dashes, and traversal attempts.

## Common Mistakes

- Writing `rm -rf "$base/$name"` without rejecting `..`, `/`, and absolute names.
- Building commands in strings instead of arrays.
- Assuming quotes prevent path traversal. Quotes preserve arguments; they do not make a path safe.
- Creating temporary files under predictable names in `/tmp`.
- Forgetting `--` before a file name that might begin with `-`.
- Printing tokens while debugging with `set -x`.
- Trusting the caller's `PATH`.
- Treating a dry-run flag as a log message while still running the destructive command.
- Checking `[[ "$target" == "$base"* ]]`, which wrongly accepts paths such as `/tmp/base-evil`.

## Exercise

Build `safe-clean.sh`.

Requirements:

- Accept `--base DIR`, `--name NAME`, `--dry-run`, and `--yes`.
- Delete only a single target inside `base`.
- Reject an empty `NAME`.
- Reject absolute names.
- Reject names containing `..`.
- Reject names containing `/`.
- Resolve the base and target path, then verify the target is under the resolved base.
- Use `umask 077`.
- Set a safe `PATH`.
- Use `mktemp -d` and `trap` for cleanup.
- Quote expansions.
- Use `--` before operands.
- Do not use `eval`.
- Print the resolved target before deleting.
- In `--dry-run` mode, delete nothing.
- Ask for confirmation unless `--yes` is passed.
- Note in a comment that secrets should not be printed or stored by this script.
- Run `shellcheck safe-clean.sh`.

Try these cases:

```bash
./safe-clean.sh --base ./scratch --name old-build --dry-run
./safe-clean.sh --base ./scratch --name '../oops' --dry-run
./safe-clean.sh --base ./scratch --name '/tmp/oops' --dry-run
./safe-clean.sh --base ./scratch --name '-starts-with-dash' --dry-run
```

## Worked Answer

This answer assumes Bash and GNU coreutils, including `realpath -m`, `mktemp`, and GNU `rm` common option handling.

```bash
#!/usr/bin/env bash
set -euo pipefail

umask 077
export PATH='/usr/sbin:/usr/bin:/sbin:/bin'

usage() {
  printf 'Usage: %s --base DIR --name NAME [--dry-run] [--yes]\n' "$0" >&2
}

die() {
  printf 'error: %s\n' "$1" >&2
  exit 1
}

base=''
name=''
dry_run=false
yes=false

while (($#)); do
  case "$1" in
    --base)
      (($# >= 2)) || die '--base requires DIR'
      base="$2"
      shift 2
      ;;
    --name)
      (($# >= 2)) || die '--name requires NAME'
      name="$2"
      shift 2
      ;;
    --dry-run)
      dry_run=true
      shift
      ;;
    --yes)
      yes=true
      shift
      ;;
    --help|-h)
      usage
      exit 0
      ;;
    *)
      usage
      die "unknown argument: $1"
      ;;
  esac
done

[[ -n "$base" ]] || die '--base is required'
[[ -n "$name" ]] || die '--name is required'

command -v realpath >/dev/null 2>&1 || die 'realpath is required'
command -v mktemp >/dev/null 2>&1 || die 'mktemp is required'

# Do not print or store secrets in this script. Keep deletion logs limited to paths.
tmpdir="$(mktemp -d)"
cleanup() {
  rm -rf -- "$tmpdir"
}
trap cleanup EXIT HUP INT TERM

[[ "$name" != /* ]] || die 'NAME must not be absolute'
[[ "$name" != */* ]] || die 'NAME must not contain /'
[[ "$name" != *..* ]] || die 'NAME must not contain ..'

base_real="$(realpath -m -- "$base")"
target_real="$(realpath -m -- "${base_real}/${name}")"

[[ "$base_real" != '/' ]] || die 'refusing to use / as base'
[[ -d "$base_real" ]] || die 'base must be an existing directory'

# The target must be a direct child path under base, not the base itself and not a prefix match.
case "$target_real" in
  "$base_real"/*) ;;
  *) die 'resolved target escapes base' ;;
esac

[[ "$target_real" != "$base_real" ]] || die 'target must not be the base directory'
[[ -e "$target_real" ]] || die 'target does not exist'
[[ -d "$target_real" ]] || die 'target must be a directory'

printf 'Resolved target: %s\n' "$target_real"

if [[ "$dry_run" == true ]]; then
  printf 'Dry run: would run rm -rf -- %q\n' "$target_real"
  exit 0
fi

if [[ "$yes" != true ]]; then
  printf 'Delete this directory? Type yes to continue: '
  IFS= read -r answer
  [[ "$answer" == yes ]] || die 'cancelled'
fi

rm -rf -- "$target_real"
printf 'Deleted: %s\n' "$target_real"
```

Review the important containment details:

- `NAME` is only one path component. Empty, absolute, slash-containing, and `..` names are rejected before joining paths.
- `base_real` and `target_real` are resolved with GNU `realpath -m`.
- The check uses `"$base_real"/*`, so `/tmp/base-evil` is not accepted as being under `/tmp/base`.
- The script refuses `/` as a base and refuses to delete the base directory itself.
- `rm` receives `-- "$target_real"`, so a target beginning with `-` is not treated as an option.

Run:

```bash
shellcheck safe-clean.sh
```

## Next Step

Return to the advanced Bash README and continue with the next numbered lesson, `08 - CI Automation`.

## Sources Used

- GNU Bash Manual, Quoting: https://www.gnu.org/software/bash/manual/html_node/Quoting.html
- GNU Bash Manual, Word Splitting: https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html
- GNU Bash Manual, Filename Expansion: https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html
- GNU Bash Manual, Bourne Shell Builtins (`eval`, `export`): https://www.gnu.org/software/bash/manual/html_node/Bourne-Shell-Builtins.html
- OWASP, Command Injection: https://owasp.org/www-community/attacks/Command_Injection
- OWASP, Path Traversal: https://owasp.org/www-community/attacks/Path_Traversal
- OWASP Cheat Sheet Series, Secrets Management: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- GNU Coreutils Manual, `mktemp`: https://www.gnu.org/software/coreutils/manual/html_node/mktemp-invocation.html
- GNU Coreutils Manual, Common Options (`--`): https://www.gnu.org/software/coreutils/manual/html_node/Common-options.html
- GNU Coreutils Manual, `rm`: https://www.gnu.org/software/coreutils/manual/html_node/rm-invocation.html
- ShellCheck SC2086: https://www.shellcheck.net/wiki/SC2086
- ShellCheck: https://www.shellcheck.net/
- MITRE CWE-427, Uncontrolled Search Path Element: https://cwe.mitre.org/data/definitions/427.html
