# 08 - CI Automation

## Learning Goal

Build a Bash-driven CI check that runs the same locally and in GitHub Actions, fails deterministically, validates shell syntax, runs ShellCheck and tests, handles environment safely, and publishes useful logs/artifacts.

## Why Bash Belongs Behind the CI YAML

GitHub Actions YAML should describe when and where work runs: triggers, runners, permissions, matrix entries, caches, artifacts, and secrets. The reusable project logic should live in a Bash script that developers can run before pushing.

That split gives you local parity:

```bash
./ci/check.sh
```

and CI parity:

```yaml
- name: Run CI checks
  shell: bash
  run: ./ci/check.sh
```

If the YAML contains all of the real checking logic, a developer has to mentally translate Actions behavior into local shell commands. If `ci/check.sh` contains the logic, CI becomes a remote runner for the same command.

Make the script locate the repository from its own path, not from the caller's current directory. `${BASH_SOURCE[0]}` points at the Bash file being executed or sourced, so a script under `ci/check.sh` can find the repo root like this:

```bash
script_dir="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd -P)"
repo_root="$(cd -- "${script_dir}/.." && pwd -P)"
cd "$repo_root"
```

That means these both work:

```bash
./ci/check.sh
(cd /tmp && /path/to/repo/ci/check.sh)
```

## Deterministic Failure

CI checks must have boring exit behavior:

- Exit `0` only when required checks passed.
- Exit nonzero when required checks failed, arguments are invalid, required tools are missing, or the script could not finish safely.
- Treat optional checks explicitly. For example, "ShellCheck is optional locally" is a policy decision; "the command was missing and the script accidentally continued" is a bug.

Start CI scripts with:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
```

The options mean:

- `-E`: inherit `ERR` traps in functions, command substitutions, and subshells.
- `-e`: exit on many unhandled command failures.
- `-u`: error on unset variables.
- `-o pipefail`: make a pipeline fail if any command in it fails, not only the last command.

Do not treat `errexit` as a complete error-handling model. Bash intentionally ignores `errexit` in contexts such as `if` tests, `while` and `until` conditions, most commands before the final command in `&&` or `||` lists, and non-final pipeline commands unless `pipefail` is set. Use explicit handling when failure is expected:

```bash
if grep -R --include='*.sh' -n 'TODO' scripts; then
  printf 'TODO markers found\n'
else
  printf 'No TODO markers found\n'
fi
```

Use an accumulator when you want all checks to run before the final exit:

```bash
status=0
bash -n ./script.sh || status=1
shellcheck ./script.sh || status=1
exit "$status"
```

That pattern is better for CI reports than stopping after the first file, because one run can show every syntax or lint failure.

## Validation Stages

A practical Bash CI check usually has three stages:

1. `bash -n`: parse scripts without executing them. This catches syntax errors early and safely.
2. `shellcheck`: catch risky shell patterns such as unquoted expansions, unused variables, unreachable code, suspicious redirects, and portability mistakes.
3. `bats`: run behavior tests for scripts and command-line tools.

Syntax checking is required because it needs only Bash. ShellCheck and Bats may be required or optional depending on the project, but the policy must be visible in the script and documented in CI. For a serious Bash project, install both in CI and fail when either reports a problem.

## Complete `ci/check.sh`

This script:

- Finds the repo root from `${BASH_SOURCE[0]}`.
- Writes `artifacts/ci.log`.
- Finds shell files deterministically.
- Runs `bash -n` on every shell file.
- Runs ShellCheck when installed.
- Runs Bats when test files are present and `bats` is installed.
- Exits nonzero when a required check fails.

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

script_dir="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd -P)"
repo_root="$(cd -- "${script_dir}/.." && pwd -P)"
cd "$repo_root"

artifact_dir="${ARTIFACT_DIR:-artifacts}"
log_file="${artifact_dir}/ci.log"

mkdir -p -- "$artifact_dir"
: > "$log_file"

log() {
  printf '%s\n' "$*" | tee -a "$log_file"
}

run_logged() {
  log "+ $*"
  "$@" 2>&1 | tee -a "$log_file"
}

status=0

log "repo_root=${repo_root}"
log "bash_version=${BASH_VERSION}"
log "runner_os=${RUNNER_OS:-local}"

mapfile -d '' shell_files < <(
  find . \
    \( -path './.git' -o -path './artifacts' -o -path './node_modules' -o -path './vendor' \) -prune -o \
    -type f \
    \( -name '*.sh' -o -name '*.bash' -o -name 'bashrc' -o -name 'bash_profile' \) \
    -print0 |
  sort -z
)

mapfile -d '' bats_files < <(
  find . \
    \( -path './.git' -o -path './artifacts' -o -path './node_modules' -o -path './vendor' \) -prune -o \
    -type f \
    -name '*.bats' \
    -print0 |
  sort -z
)

log "shell_files=${#shell_files[@]}"
log "bats_files=${#bats_files[@]}"

if ((${#shell_files[@]} == 0)); then
  log "No shell files found."
else
  log "Stage: bash -n"
  for file in "${shell_files[@]}"; do
    if ! run_logged bash -n "$file"; then
      log "FAIL bash -n: $file"
      status=1
    fi
  done
fi

if command -v shellcheck >/dev/null 2>&1; then
  if ((${#shell_files[@]} > 0)); then
    log "Stage: ShellCheck"
    if ! run_logged shellcheck -- "${shell_files[@]}"; then
      log "FAIL ShellCheck"
      status=1
    fi
  fi
else
  log "SKIP ShellCheck: shellcheck is not installed"
fi

if ((${#bats_files[@]} > 0)); then
  if command -v bats >/dev/null 2>&1; then
    log "Stage: Bats"
    if ! run_logged bats -- "${bats_files[@]}"; then
      log "FAIL Bats"
      status=1
    fi
  else
    log "FAIL Bats: .bats files exist, but bats is not installed"
    status=1
  fi
else
  log "SKIP Bats: no .bats files found"
fi

if ((status == 0)); then
  log "CI checks passed."
else
  log "CI checks failed."
fi

exit "$status"
```

Two details are easy to miss:

- The `find ... -print0 | sort -z` pipeline gives stable ordering and handles file names with spaces.
- `run_logged` uses `if ! run_logged ...; then` so expected command failures are captured and accumulated instead of letting `errexit` stop the whole script immediately.

## GitHub Actions Workflow

Place this in `.github/workflows/bash-ci.yml`.

```yaml
name: Bash CI

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  check:
    name: bash-ci (${{ matrix.os }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest]

    env:
      CI_MODE: github-actions
      EXAMPLE_API_TOKEN: ${{ secrets.EXAMPLE_API_TOKEN }}

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Cache Bats repository
        uses: actions/cache@v4
        with:
          path: ~/.cache/bats-core
          key: ${{ runner.os }}-bats-core-v1.11.1

      - name: Install tools
        shell: bash
        run: |
          set -Eeuo pipefail
          if [[ "${RUNNER_OS}" == "Linux" ]]; then
            sudo apt-get update
            sudo apt-get install -y shellcheck
          elif [[ "${RUNNER_OS}" == "macOS" ]]; then
            brew update
            brew install shellcheck
          fi

          mkdir -p ~/.cache
          if [[ ! -d ~/.cache/bats-core/.git ]]; then
            git clone --depth 1 --branch v1.11.1 https://github.com/bats-core/bats-core.git ~/.cache/bats-core
          fi
          sudo ~/.cache/bats-core/install.sh /usr/local

      - name: Run CI checks
        shell: bash
        run: ./ci/check.sh

      - name: Upload CI artifacts
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: ci-log-${{ runner.os }}
          path: artifacts/
          if-no-files-found: warn
```

The safe environment variable is `CI_MODE`; it is ordinary configuration and may appear in logs. `EXAMPLE_API_TOKEN` is a secret reference; the script should validate whether it exists only when a check needs it, and it should never print it.

## Environment Variables And Secrets

Environment is input. Treat it with the same care as arguments:

- Use explicit names such as `ARTIFACT_DIR`, `CI_MODE`, or `RUNNER_OS`.
- Use defaults for non-secret local settings: `artifact_dir="${ARTIFACT_DIR:-artifacts}"`.
- Require values only at the point they are needed: `: "${DEPLOY_TOKEN:?DEPLOY_TOKEN is required}"`.
- Never use `set -x` around secrets.
- Never print secret values, derived authorization headers, or generated config files containing secrets.
- Prefer GitHub secrets for sensitive values and GitHub variables or plain `env` entries for non-sensitive configuration.

Be careful with `nounset`: `${EXAMPLE_API_TOKEN}` will fail if the variable is unset, while `${EXAMPLE_API_TOKEN:-}` safely expands to empty. That distinction is useful when a secret is optional for pull requests but required for release jobs.

## Logs And Artifacts

CI logs should help a maintainer answer three questions quickly:

- What runner and Bash version ran the check?
- Which files and stages were checked?
- Which required command failed?

Use artifacts for files that are easier to inspect after the job finishes: `artifacts/ci.log`, test reports, coverage reports, command traces, or generated diagnostics. Upload artifacts with `if: always()` so failed jobs still publish evidence.

Do not put secrets in artifacts. Treat artifacts as shareable debugging output unless your organization has a stricter access model and retention policy.

## Matrix And Platform Concerns

Running on both Ubuntu and macOS catches useful differences:

- macOS often ships an older Bash as `/bin/bash`; `shell: bash` in GitHub Actions still makes the shell choice explicit, but your scripts should state the Bash version they require.
- GNU and BSD userland tools differ. `sed`, `find`, `readlink`, `stat`, and `xargs` flags are common portability traps.
- Package installation differs. Use `apt-get` on Ubuntu and Homebrew on macOS.
- File systems differ in case sensitivity and default permissions.

Use `fail-fast: false` for learning and diagnostic workflows so every matrix leg reports. That does not hide failures; it only prevents one failing platform from cancelling the other platform before it produces logs.

## Caching Caveats

Cache dependencies that are expensive and safe to reuse, but do not cache correctness:

- Include tool names and versions in cache keys.
- Do not cache secrets, generated credentials, or `.env` files.
- Do not assume a cache hit. The workflow must work from a cold cache.
- Keep install steps idempotent.
- Remember that caches can hide stale tooling if keys are too broad.

In the workflow above, the cache stores the Bats source checkout, not the test result. The check still runs every time.

## Local Parity Checklist

Before trusting a Bash CI workflow, verify:

- `./ci/check.sh` works from the repo root.
- `/absolute/path/to/repo/ci/check.sh` works from another directory.
- The script creates `artifacts/ci.log`.
- A syntax error in a `.sh` file makes the script exit nonzero.
- A ShellCheck violation makes the script exit nonzero when ShellCheck is installed.
- A failing `.bats` test makes the script exit nonzero.
- Missing Bats fails only when `.bats` tests exist.
- The GitHub Actions job runs `./ci/check.sh` instead of duplicating its logic.
- The artifact upload step uses `if: always()`.
- No secret values appear in logs or artifacts.

## Common Mistakes

- Putting all logic in YAML so local runs and CI runs drift apart.
- Using `$0` for repo-root detection even though sourced scripts and wrappers can change it; use `${BASH_SOURCE[0]}` in Bash scripts.
- Depending on the caller's current directory.
- Assuming `set -e` catches every failure.
- Running `find` without stable sorting, producing noisy order changes.
- Expanding file lists with unquoted command substitution, which breaks on spaces and newlines.
- Treating "ShellCheck not installed" as a pass in CI when the project requires linting.
- Hiding failures with `continue-on-error` instead of using `fail-fast: false` for matrix diagnostics.
- Uploading artifacts only on success.
- Printing secrets while debugging with `set -x`, `env`, or `printenv`.
- Caching generated outputs and accidentally skipping the real check.

## Exercise

Create a Bash CI setup for a small repository.

Requirements:

- Add `ci/check.sh`.
- Resolve the repo root from `${BASH_SOURCE[0]}`.
- Use `set -Eeuo pipefail`.
- Write logs to `artifacts/ci.log`.
- Find shell files deterministically with `find`, null separators, and sorting.
- Run `bash -n` on shell files.
- Run ShellCheck when installed.
- Run Bats tests when `.bats` files exist.
- Fail if `.bats` files exist but `bats` is missing.
- Accumulate failures so syntax, lint, and tests can all report in one run.
- Add `.github/workflows/bash-ci.yml`.
- Use an Ubuntu/macOS matrix with `fail-fast: false`.
- Use explicit `shell: bash`.
- Include one non-secret env var and one secret reference.
- Upload `artifacts/` with `if: always()`.

Test your setup by creating:

```bash
scripts/hello.sh
test/hello.bats
```

The script should print `hello, ci`, and the Bats test should assert that output.

## Worked Answer

`scripts/hello.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

printf 'hello, ci\n'
```

`test/hello.bats`:

```bash
#!/usr/bin/env bats

@test "hello script prints the expected message" {
  run bash scripts/hello.sh
  [ "$status" -eq 0 ]
  [ "$output" = "hello, ci" ]
}
```

`ci/check.sh`:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

script_dir="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd -P)"
repo_root="$(cd -- "${script_dir}/.." && pwd -P)"
cd "$repo_root"

artifact_dir="${ARTIFACT_DIR:-artifacts}"
log_file="${artifact_dir}/ci.log"
mkdir -p -- "$artifact_dir"
: > "$log_file"

log() {
  printf '%s\n' "$*" | tee -a "$log_file"
}

run_logged() {
  log "+ $*"
  "$@" 2>&1 | tee -a "$log_file"
}

status=0

mapfile -d '' shell_files < <(
  find . \
    \( -path './.git' -o -path './artifacts' \) -prune -o \
    -type f \( -name '*.sh' -o -name '*.bash' \) \
    -print0 |
  sort -z
)

mapfile -d '' bats_files < <(
  find . \
    \( -path './.git' -o -path './artifacts' \) -prune -o \
    -type f -name '*.bats' \
    -print0 |
  sort -z
)

log "repo_root=${repo_root}"
log "shell_files=${#shell_files[@]}"
log "bats_files=${#bats_files[@]}"

for file in "${shell_files[@]}"; do
  if ! run_logged bash -n "$file"; then
    status=1
  fi
done

if command -v shellcheck >/dev/null 2>&1; then
  if ((${#shell_files[@]} > 0)); then
    run_logged shellcheck -- "${shell_files[@]}" || status=1
  fi
else
  log "SKIP ShellCheck: shellcheck is not installed"
fi

if ((${#bats_files[@]} > 0)); then
  if command -v bats >/dev/null 2>&1; then
    run_logged bats -- "${bats_files[@]}" || status=1
  else
    log "FAIL Bats: .bats files exist, but bats is not installed"
    status=1
  fi
else
  log "SKIP Bats: no .bats files found"
fi

exit "$status"
```

`.github/workflows/bash-ci.yml`:

```yaml
name: Bash CI

on:
  push:
  pull_request:

permissions:
  contents: read

jobs:
  check:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest]
    env:
      CI_MODE: github-actions
      EXAMPLE_API_TOKEN: ${{ secrets.EXAMPLE_API_TOKEN }}
    steps:
      - uses: actions/checkout@v4

      - name: Install tools
        shell: bash
        run: |
          set -Eeuo pipefail
          if [[ "${RUNNER_OS}" == "Linux" ]]; then
            sudo apt-get update
            sudo apt-get install -y shellcheck bats
          elif [[ "${RUNNER_OS}" == "macOS" ]]; then
            brew update
            brew install shellcheck bats-core
          fi

      - name: Run checks
        shell: bash
        run: ./ci/check.sh

      - name: Upload artifacts
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: ci-artifacts-${{ runner.os }}
          path: artifacts/
          if-no-files-found: warn
```

Run locally:

```bash
chmod +x ci/check.sh scripts/hello.sh
./ci/check.sh
```

Expected result:

- `bash -n` parses `scripts/hello.sh`.
- ShellCheck runs if installed.
- Bats runs `test/hello.bats`.
- `artifacts/ci.log` contains the command output.
- The script exits `0` when all checks pass.

## Next Step

Return to the [advanced Bash README](README.md) and continue with the next numbered lesson.

## Sources Used

- GNU Bash Manual, The Set Builtin: https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html
- GNU Bash Manual, Bash Variables (`BASH_SOURCE`): https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html
- GitHub Actions workflow syntax: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- GitHub Actions variables: https://docs.github.com/en/actions/reference/workflows-and-actions/variables
- GitHub Actions secrets: https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets
- GitHub Actions artifacts: https://docs.github.com/en/actions/tutorials/store-and-share-data
- GitHub Actions dependency caching: https://docs.github.com/en/actions/reference/workflows-and-actions/dependency-caching
- ShellCheck: https://www.shellcheck.net/
- ShellCheck GitHub repository: https://github.com/koalaman/shellcheck
- Bats-core documentation: https://bats-core.readthedocs.io/
- Bats-core GitHub repository: https://github.com/bats-core/bats-core
