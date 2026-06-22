# 09 - Deployment Scripts

## Learning Goal

Write a Bash deployment script that safely promotes a validated build artifact to an explicit environment using preflight checks, dry-run mode, confirmation, locking, logging, cleanup traps, safe remote commands, and rollback-aware release layout.

## Why Deployment Scripts Are Risky Production Glue

Deployment scripts sit between source control, build systems, secrets, networks, remote hosts, filesystems, and live traffic. That makes them useful, but also dangerous: a small quoting bug can delete the wrong directory, a missing checksum can promote the wrong artifact, and an unsafe SSH option can train automation to ignore host identity.

Bash is often used here because it is already available on build runners and servers. The risk is that Bash makes side effects easy. A production-quality deployment script must therefore be boring on purpose: explicit inputs, loud validation, no hidden defaults for critical choices, safe cleanup, traceable logs, and rollback-aware layout.

## Safety Checklist

- Require an explicit `--env staging|production`. Never guess the target from a branch name, hostname, or current directory.
- Run preflight checks before mutation: required commands, readable artifact, valid checksum format, reachable host, writable remote directory, and enough remote tools.
- Validate the artifact and version before upload. Use SHA-256 verification, reject suspicious version strings, and check whether `releases/$version` already exists.
- Design for idempotence. A repeated deployment should either complete the same final state or fail before changing anything.
- Provide dry-run mode that mutates nothing locally or remotely. It should print the commands it would run, not create lock files, temporary directories, uploaded files, symlinks, or remote directories.
- Require production confirmation. `--yes` is acceptable for CI, but interactive users should see the exact environment, version, host, and remote directory.
- Keep secrets out of arguments, logs, and traces. Prefer CI secret stores or environment/config injection. Do not enable `set -x` around secret-handling code.
- Use safe SSH defaults: preserve normal host-key verification and use `ssh -o BatchMode=yes` so automation fails instead of prompting. Do not use `StrictHostKeyChecking=no`.
- Log with timestamps. Logs should identify the step, environment, version, artifact path, and host, but not secret values.
- Use `mktemp` plus `trap` for local temporary files and cleanup. Cleanup must run on success, failure, and interruption.
- Use `flock` for local deploy serialization. If deployments can start from multiple laptops or CI runners, local `flock` is not enough; add a remote lock on the deployment host or use your orchestrator's locking model.
- Use a rollback-aware layout such as:

```text
/opt/myapp/
  releases/
    2026.06.22-1/
    2026.06.22-2/
  current -> releases/2026.06.22-2
  previous -> releases/2026.06.22-1
```

The deployment should upload and unpack into `releases/$version`, validate that release, then atomically move `previous` to the old `current` target and `current` to the new release.

## Minimal Skeleton

This is the shape of a safe deployment script. It omits many checks so the full worked answer can show them in context.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

env=""
version=""
artifact=""
sha256=""
host=""
remote_dir=""
dry_run=false
yes=false

usage() {
  cat <<'USAGE'
Usage: deploy-release.sh --env staging|production --version VERSION \
  --artifact FILE --sha256 FILE --host HOST --remote-dir DIR [--dry-run] [--yes]
USAGE
}

log() {
  printf '[%(%Y-%m-%dT%H:%M:%S%z)T] %s\n' -1 "$*"
}

run() {
  if "$dry_run"; then
    printf 'DRY-RUN:'
    printf ' %q' "$@"
    printf '\n'
  else
    "$@"
  fi
}

cleanup() {
  [[ -n "${tmpdir:-}" && -d "${tmpdir:-}" ]] && rm -rf -- "$tmpdir"
}
trap cleanup EXIT INT TERM

# parse arguments, validate inputs, confirm production, take a lock,
# upload the artifact, and run a parameterized remote script.
```

The important habit is separating "plan" from "do". Validation should happen first. Mutation should happen only after dry-run and confirmation gates have been handled.

## Safe Remote Commands

The riskiest remote Bash pattern is building an executable string from local variables:

```bash
ssh "$host" "mkdir -p $remote_dir/releases/$version && tar -xf $artifact"
```

That line trusts every character in `remote_dir`, `version`, and `artifact` as shell syntax. Instead, pass data as positional parameters to a remote script:

```bash
ssh -o BatchMode=yes -- "$host" bash -s -- "$remote_dir" "$version" <<'REMOTE'
set -Eeuo pipefail
remote_dir=$1
version=$2
mkdir -p -- "$remote_dir/releases/$version"
REMOTE
```

The script still needs validation, but the remote values are data, not source code.

## When Bash Is Not Enough

Bash is reasonable for a small artifact promotion script with a few remote file operations. Reach for a deployment system, orchestrator, or a real programming language when you need distributed locks, health checks across many nodes, progressive rollout, service discovery changes, complex rollback policy, database migrations, secrets broker integration, or audited approvals. Kubernetes, for example, has rollout history and rollback commands; using Bash to reimplement a cluster controller is usually the wrong trade.

## Common Mistakes

- Defaulting to production when `--env` is missing.
- Treating `--dry-run` as "mostly dry" while still creating temp files, lock files, remote directories, or test uploads.
- Uploading an artifact before checking its SHA-256.
- Logging full command lines that contain tokens, passwords, private key paths, or signed URLs.
- Disabling SSH host-key verification with `StrictHostKeyChecking=no`.
- Interpolating untrusted values into a remote shell command string.
- Updating `current` before the new release directory is complete.
- Overwriting `previous` before recording what `current` pointed to.
- Using only local `flock` when multiple deploy machines can run the script at the same time.
- Assuming `set -e` catches every error. Pipelines, conditionals, command substitutions, and remote scripts still need deliberate checks.

## Exercise

Create `deploy-release.sh`.

Requirements:

- Accept `--env`, `--version`, `--artifact`, `--sha256`, `--host`, `--remote-dir`, `--dry-run`, and `--yes`.
- Allow only `--env staging` or `--env production`.
- Require every non-flag option.
- Validate the artifact is a readable file.
- Validate `--sha256` points to a readable checksum file.
- Verify the artifact checksum with `sha256sum --check` before any mutation.
- Use `mktemp` and `trap` for local cleanup.
- Use `flock` to prevent two local deployments from running at once.
- In dry-run mode, print commands and perform no local or remote mutation.
- Require confirmation for production unless `--yes` is provided.
- Use `ssh -o BatchMode=yes` and preserve normal host-key verification.
- Upload with `rsync`.
- Use remote layout `releases/$version`, `current`, and `previous`.
- Pass remote values as positional parameters to `bash -s -- ...`.
- Do not print secrets.

## Worked Answer

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

env=""
version=""
artifact=""
sha256_file=""
host=""
remote_dir=""
dry_run=false
yes=false
tmpdir=""

usage() {
  cat <<'USAGE'
Usage:
  deploy-release.sh --env staging|production --version VERSION \
    --artifact FILE --sha256 FILE --host HOST --remote-dir DIR \
    [--dry-run] [--yes]
USAGE
}

die() {
  printf 'ERROR: %s\n' "$*" >&2
  exit 1
}

log() {
  printf '[%(%Y-%m-%dT%H:%M:%S%z)T] %s\n' -1 "$*"
}

quote_cmd() {
  printf '%q ' "$@"
  printf '\n'
}

run() {
  if "$dry_run"; then
    printf 'DRY-RUN: '
    quote_cmd "$@"
  else
    "$@"
  fi
}

cleanup() {
  if [[ -n "$tmpdir" && -d "$tmpdir" ]]; then
    rm -rf -- "$tmpdir"
  fi
}
trap cleanup EXIT INT TERM

while (($#)); do
  case "$1" in
    --env)
      (($# >= 2)) || die "--env requires a value"
      env=$2
      shift 2
      ;;
    --version)
      (($# >= 2)) || die "--version requires a value"
      version=$2
      shift 2
      ;;
    --artifact)
      (($# >= 2)) || die "--artifact requires a value"
      artifact=$2
      shift 2
      ;;
    --sha256)
      (($# >= 2)) || die "--sha256 requires a value"
      sha256_file=$2
      shift 2
      ;;
    --host)
      (($# >= 2)) || die "--host requires a value"
      host=$2
      shift 2
      ;;
    --remote-dir)
      (($# >= 2)) || die "--remote-dir requires a value"
      remote_dir=$2
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
    -h|--help)
      usage
      exit 0
      ;;
    *)
      die "unknown argument: $1"
      ;;
  esac
done

[[ "$env" == "staging" || "$env" == "production" ]] || die "--env must be staging or production"
[[ -n "$version" ]] || die "--version is required"
[[ -n "$artifact" ]] || die "--artifact is required"
[[ -n "$sha256_file" ]] || die "--sha256 is required"
[[ -n "$host" ]] || die "--host is required"
[[ -n "$remote_dir" ]] || die "--remote-dir is required"

[[ "$version" =~ ^[A-Za-z0-9._-]+$ ]] || die "--version may contain only letters, numbers, dot, underscore, and dash"
[[ "$host" =~ ^[A-Za-z0-9._@-]+$ ]] || die "--host contains unsupported characters"
[[ "$remote_dir" =~ ^/[A-Za-z0-9._/-]+$ ]] || die "--remote-dir must be an absolute path using letters, numbers, dot, underscore, dash, and slash"
[[ -r "$artifact" && -f "$artifact" ]] || die "artifact is not a readable file: $artifact"
[[ -r "$sha256_file" && -f "$sha256_file" ]] || die "sha256 file is not readable: $sha256_file"

for command_name in sha256sum ssh rsync flock; do
  command -v "$command_name" >/dev/null 2>&1 || die "missing required command: $command_name"
done

artifact_name=$(basename -- "$artifact")
artifact_dir=$(cd -- "$(dirname -- "$artifact")" && pwd -P)
artifact_abs="$artifact_dir/$artifact_name"

read -r expected_sha256 _ < "$sha256_file" || die "could not read sha256 file"
[[ "$expected_sha256" =~ ^[A-Fa-f0-9]{64}$ ]] || die "sha256 file must start with a 64-character checksum"

(
  cd "$artifact_dir"
  sha256sum --check <(printf '%s  %s\n' "$expected_sha256" "$artifact_name")
) || die "artifact checksum mismatch"

[[ "$artifact_name" =~ ^[A-Za-z0-9._-]+$ ]] || die "artifact filename contains unsupported characters"
remote_upload="$remote_dir/.incoming/$version/$artifact_name"

log "validated artifact=$artifact env=$env version=$version host=$host remote_dir=$remote_dir"

if [[ "$env" == "production" && "$yes" != true ]]; then
  printf 'Deploy version %s to PRODUCTION on %s:%s? Type "deploy %s" to continue: ' \
    "$version" "$host" "$remote_dir" "$version"
  read -r answer
  [[ "$answer" == "deploy $version" ]] || die "production deployment cancelled"
fi

if "$dry_run"; then
  log "dry-run mode: no local or remote state will be changed"
  run ssh -o BatchMode=yes -- "$host" true
  run rsync -a -- "$artifact_abs" "$host:$remote_upload"
  printf 'DRY-RUN: remote script would create %q, unpack artifact, and update previous/current symlinks under %q\n' \
    "releases/$version" "$remote_dir"
  exit 0
fi

tmpdir=$(mktemp -d)
lock_file="${TMPDIR:-/tmp}/deploy-release.lock"

exec 9>"$lock_file"
flock -n 9 || die "another local deployment is already running"

log "checking remote host"
ssh -o BatchMode=yes -- "$host" bash -s -- "$remote_dir" <<'REMOTE_PREFLIGHT'
set -Eeuo pipefail
remote_dir=$1
command -v mkdir >/dev/null
command -v ln >/dev/null
command -v tar >/dev/null
mkdir -p -- "$remote_dir/.preflight"
rmdir -- "$remote_dir/.preflight"
REMOTE_PREFLIGHT

log "creating remote incoming directory"
ssh -o BatchMode=yes -- "$host" bash -s -- "$remote_dir" "$version" <<'REMOTE_INCOMING'
set -Eeuo pipefail
remote_dir=$1
version=$2
release_dir="$remote_dir/releases/$version"
if [[ -e "$release_dir" ]]; then
  printf 'release already exists: %s\n' "$release_dir" >&2
  exit 1
fi
mkdir -p -- "$remote_dir/.incoming/$version"
REMOTE_INCOMING

log "uploading artifact"
rsync -a -- "$artifact_abs" "$host:$remote_upload"

log "activating release"
ssh -o BatchMode=yes -- "$host" bash -s -- "$remote_dir" "$version" "$artifact_name" <<'REMOTE_DEPLOY'
set -Eeuo pipefail

remote_dir=$1
version=$2
artifact_name=$3

incoming="$remote_dir/.incoming/$version/$artifact_name"
release_dir="$remote_dir/releases/$version"
current_link="$remote_dir/current"
previous_link="$remote_dir/previous"

[[ "$version" =~ ^[A-Za-z0-9._-]+$ ]] || {
  printf 'invalid remote version\n' >&2
  exit 1
}
[[ -f "$incoming" ]] || {
  printf 'uploaded artifact missing\n' >&2
  exit 1
}

if [[ -e "$release_dir" ]]; then
  printf 'release already exists: %s\n' "$release_dir" >&2
  exit 1
fi

mkdir -p -- "$remote_dir/releases"
mkdir -p -- "$release_dir"
tar -xzf "$incoming" -C "$release_dir"

old_current=""
if [[ -L "$current_link" ]]; then
  old_current=$(readlink -- "$current_link")
fi

if [[ -n "$old_current" ]]; then
  ln -sfn -- "$old_current" "$previous_link"
fi

ln -sfn -- "releases/$version" "$current_link"
rm -rf -- "$remote_dir/.incoming/$version"
REMOTE_DEPLOY

log "deployment complete: env=$env version=$version host=$host"
```

Notes about this answer:

- Dry-run exits before `mktemp`, `flock`, SSH mutation, or `rsync`, so it does not change local or remote state.
- The `flock` lock is local to this script process and machine. A production script may need remote locking if deployments can start from multiple machines.
- The script never passes `StrictHostKeyChecking=no`; SSH keeps normal host-key verification behavior.
- Remote values are passed after `bash -s --` and read as positional parameters.
- The script logs deployment metadata, not secret values.

## Next Step

Return to the [advanced Bash README](README.md) and continue with the next lesson.

## Sources Used

- [GNU Bash Reference Manual: Signals and `trap`](https://www.gnu.org/software/bash/manual/html_node/Signals.html)
- [GNU Bash / Linux manual page: `set`, shell options, and execution behavior](https://man7.org/linux/man-pages/man1/bash.1.html)
- [OpenSSH `ssh(1)` manual page](https://man.openbsd.org/ssh)
- [OpenSSH `ssh_config(5)` manual page](https://man.openbsd.org/ssh_config)
- [`rsync(1)` Linux manual page](https://man7.org/linux/man-pages/man1/rsync.1.html)
- [`flock(1)` Linux manual page](https://man7.org/linux/man-pages/man1/flock.1.html)
- [`sha256sum(1)` Linux manual page](https://man7.org/linux/man-pages/man1/sha256sum.1.html)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [The Twelve-Factor App: Config](https://12factor.net/config)
- [Kubernetes Deployment rollout and rollback documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
