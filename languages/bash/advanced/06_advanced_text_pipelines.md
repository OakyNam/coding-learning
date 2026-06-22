# 06 - Advanced Text Pipelines

## Learning Goal

Build reliable Bash text pipelines that combine small tools, handle errors correctly, preserve important records such as headers, and safely process filenames and log data.

## Why Pipelines Matter

A pipeline connects the output of one command to the input of the next command:

```bash
cmd1 | cmd2
```

Most Unix text tools follow the same stream model:

- `stdin` is the command's input stream.
- `stdout` is the normal output stream.
- `stderr` is the error and diagnostic stream.

That lets you build larger programs out of smaller pieces:

```bash
grep 'ERROR' app.log | cut -d' ' -f1-3 | sort | uniq -c
```

In that pipeline, `grep` selects lines, `cut` extracts fields, `sort` groups equal lines together, and `uniq -c` counts adjacent duplicates.

Redirection changes where streams come from or go:

```bash
grep 'ERROR' < app.log > errors.txt
grep 'ERROR' app.log 2> grep-errors.txt
```

The first command reads from `app.log` and writes normal output to `errors.txt`. The second writes diagnostics to `grep-errors.txt`.

Bash also supports `|&`, which pipes both `stdout` and `stderr` into the next command:

```bash
make |& tee build.log
```

Use `|&` only when you really want normal output and diagnostics mixed together. Many scripts should keep `stdout` for machine-readable results and `stderr` for progress or errors.

## Pipeline Exit Statuses

By default, a Bash pipeline returns the exit status of the last command:

```bash
grep 'ERROR' app.log | sort
echo "$?"
```

If `grep` fails but `sort` succeeds, the pipeline can still report success because `sort` is the last command. This is often surprising in scripts.

Use `set -o pipefail` when the script should fail if any command in a pipeline fails:

```bash
set -o pipefail
grep 'ERROR' missing.log | sort
```

With `pipefail`, the pipeline returns a failure status if any command in the pipeline fails. The returned status is the status of the rightmost failed command, or `0` if all commands succeed.

Bash also stores each command's status in the special array `PIPESTATUS`:

```bash
grep 'ERROR' app.log | sort | uniq -c
printf 'grep=%s sort=%s uniq=%s\n' "${PIPESTATUS[0]}" "${PIPESTATUS[1]}" "${PIPESTATUS[2]}"
```

Timing caveat: read or copy `PIPESTATUS` immediately after the pipeline. Running another command, even a simple `echo`, changes it.

```bash
grep 'ERROR' app.log | sort
statuses=("${PIPESTATUS[@]}")
printf 'grep status was %s\n' "${statuses[0]}"
```

One more important nuance: `grep` exits with status `1` when it searched successfully but found no matches. That is not the same as a broken command. Status `2` usually means an actual error such as an unreadable file or invalid option.

## Core Text Tools

`grep` filters lines by pattern:

```bash
grep -n 'ERROR' app.log
grep -E 'status=(500|503)' app.log
```

`cut` extracts fields or character ranges from simple delimited text:

```bash
cut -d' ' -f1 app.log
cut -f1,3 table.tsv
```

`tr` translates or deletes characters:

```bash
printf 'Api Server\n' | tr '[:upper:]' '[:lower:]'
printf 'a,b,c\n' | tr ',' '\n'
```

`sort` orders lines:

```bash
sort names.txt
sort -n numbers.txt
sort -t $'\t' -k2,2nr report.tsv
```

`uniq` works on adjacent duplicate lines, so it is commonly used after `sort`:

```bash
sort status-codes.txt | uniq -c
```

`awk` is good for field-based reports and small transformations:

```bash
awk '$9 >= 500 { print $9, $1 }' access.log
awk -F'\t' '{ count[$2]++ } END { for (key in count) print count[key], key }' report.tsv
```

`sed` is good for stream editing:

```bash
sed 's/status=/code=/' app.log
sed -n '1,5p' app.log
```

These tools are strongest on regular, line-oriented text. Once the input has real quoting, escaping, nesting, or embedded delimiters, choose a parser made for that format.

## Build Pipelines Incrementally

Do not write a long pipeline all at once. Build one stage, inspect the output, then add the next stage.

Start with a small sample:

```bash
head -n 20 app.log
```

Select the records you care about:

```bash
grep -E 'status=(500|503)' app.log | head
```

Extract the fields:

```bash
grep -E 'status=(500|503)' app.log |
  awk 'match($0, /status=([0-9]{3})/, code) { print code[1] }' |
  head
```

Count them:

```bash
grep -E 'status=(500|503)' app.log |
  awk 'match($0, /status=([0-9]{3})/, code) { print code[1] }' |
  sort |
  uniq -c |
  sort -nr
```

Use `tail` when the newest records are at the end:

```bash
tail -n 1000 app.log | grep 'ERROR'
```

Use `tee` when you want to inspect or save an intermediate stage:

```bash
grep 'ERROR' app.log |
  tee errors.sample |
  awk 'match($0, /status=([0-9]{3})/, code) { print code[1] }' |
  sort |
  uniq -c
```

`tee` writes a copy to a file while still passing the stream to the next command.

## Preserving Headers

Sorting a whole file with a header usually moves the header into the data. Keep the header separate, sort the body, and print both:

```bash
{
  IFS= read -r header
  printf '%s\n' "$header"
  sort -t $'\t' -k2,2
} < report.tsv
```

Inside the group, `read` consumes the first line from `report.tsv`. Then `sort` reads the remaining lines from the same input stream.

For a comma-separated file, this pattern is only safe for simple comma-delimited text with no quoted commas or embedded newlines. For real CSV, use Python's `csv` module.

## Stable Sorting

When two records have the same sort key, normal `sort` may use the rest of the line as a final tie-breaker. A stable sort preserves the original order of records that compare equal on the selected keys.

Use `sort -s` when equal keys should stay in input order:

```bash
sort -s -t $'\t' -k2,2 users.tsv
```

This matters when an earlier pipeline stage already put records into a meaningful order:

```bash
sort -s -t $'\t' -k3,3 status-report.tsv
```

The locale can affect sorting and character classes. For bytewise, predictable sorting in scripts, set `LC_ALL=C` around the command or near the top of the script:

```bash
LC_ALL=C sort -s names.txt
```

## Null-Delimited Filenames

Filenames can contain spaces, tabs, quotes, backslashes, and newlines. Newline-delimited filename pipelines are fragile.

Use null-delimited records for filenames:

```bash
find logs -type f -name '*.log' -print0
```

Pass those names to commands with `xargs -0`:

```bash
find logs -type f -name '*.log' -print0 |
  xargs -0 grep -H 'ERROR'
```

Use a Bash loop with `read -r -d ''` when you need per-file logic:

```bash
find logs -type f -name '*.log' -print0 |
  while IFS= read -r -d '' file; do
    printf 'checking %s\n' "$file" >&2
    grep -H 'ERROR' "$file"
  done
```

Use `sort -z` for null-delimited records:

```bash
find logs -type f -name '*.log' -print0 |
  sort -z |
  xargs -0 grep -H 'ERROR'
```

Null-delimited filenames protect the filename stream. If the file contents are normal log lines, the content stream is still newline-delimited after `grep`.

## CSV Logs JSON And Tool Choice Caveats

Simple line-oriented logs and TSV files are a good fit for `grep`, `cut`, `sort`, `uniq`, `awk`, and `sed`.

Real CSV is more complicated than splitting on commas. Quoted fields can contain commas, quotes, and newlines. Use Python's `csv` module for real CSV:

```bash
python3 - <<'PY'
import csv
import sys

for row in csv.DictReader(sys.stdin):
    print(row["status"], row["source"])
PY
```

JSON is structured data, not line fields. Use `jq` for JSON:

```bash
jq -r 'select(.status >= 500) | [.status, .source] | @tsv' events.json
```

Locale settings can affect `sort`, `tr`, bracket expressions, and character ordering. Use `LC_ALL=C` when bytewise behavior is desired and appropriate:

```bash
LC_ALL=C grep -E 'status=[0-9]{3}' app.log | LC_ALL=C sort
```

## Common Mistakes

- Assuming a pipeline failed because `grep` returned `1`; `grep` uses `1` for no matches.
- Forgetting that a pipeline's default status is the status of the last command.
- Reading `PIPESTATUS` too late, after another command has already overwritten it.
- Using `set -e` and expecting it to explain which pipeline stage failed; inspect statuses when the distinction matters.
- Mixing `stdout` and `stderr` with `|&` when the next command expects clean data.
- Sorting a header line together with the data rows.
- Expecting `uniq -c` to count duplicates that are not adjacent.
- Parsing real CSV with `cut -d,` or simple `awk -F,`.
- Parsing JSON with `grep` and `sed` instead of `jq`.
- Passing filenames through whitespace-delimited loops such as `for file in $(find ...)`.
- Forgetting to quote filename variables such as `"$file"`.
- Relying on default locale sorting when the script needs repeatable bytewise order.

## Exercise

Write a script named `summarize_errors.sh`.

The script should:

- Recursively find `*.log` files under a directory argument.
- Handle filenames with spaces safely.
- Keep machine-readable output on `stdout`.
- Send progress or diagnostic messages to `stderr`.
- Use `set -o pipefail`.
- Explain, in comments or notes, how `PIPESTATUS` should be captured immediately after a pipeline.
- Treat the logs as simple line-oriented text, not JSON or CSV.
- Report rows in this format:

```text
count status_code source_file
```

The script should find log lines containing a status code in the form `status=500`, `status=404`, and so on. Count matches by status code and source file. Sort the output by descending count, then by status code.

Example output:

```text
12 500 logs/api server.log
7 503 logs/worker.log
2 404 logs/api server.log
```

## Worked Answer

One complete solution:

```bash
#!/usr/bin/env bash
set -o pipefail

if [[ $# -ne 1 ]]; then
  printf 'usage: %s LOG_DIR\n' "$0" >&2
  exit 2
fi

log_dir=$1

if [[ ! -d $log_dir ]]; then
  printf 'error: not a directory: %s\n' "$log_dir" >&2
  exit 2
fi

printf 'scanning log files under %s\n' "$log_dir" >&2

find "$log_dir" -type f -name '*.log' -print0 |
  sort -z |
  while IFS= read -r -d '' file; do
    awk -v source_file="$file" '
      match($0, /status=[0-9][0-9][0-9]/) {
        status = substr($0, RSTART + 7, 3)
        print status "\t" source_file
      }
    ' "$file" || exit $?
  done |
  awk -F '\t' '
    {
      status = $1
      file = $2
      key = status SUBSEP file
      counts[key]++
    }
    END {
      for (key in counts) {
        split(key, parts, SUBSEP)
        print counts[key], parts[1], parts[2]
      }
    }
  ' |
  sort -k1,1nr -k2,2n

statuses=("${PIPESTATUS[@]}")

# PIPESTATUS must be copied immediately after the pipeline. Any command run before
# the assignment above would replace the array with statuses for that newer command.
if (( statuses[0] != 0 || statuses[1] != 0 || statuses[2] != 0 || statuses[3] != 0 || statuses[4] != 0 )); then
  printf 'error: pipeline failed: find=%s sort_files=%s scan_loop=%s count_awk=%s final_sort=%s\n' \
    "${statuses[0]}" "${statuses[1]}" "${statuses[2]}" "${statuses[3]}" "${statuses[4]}" >&2
  exit 1
fi
```

Notes:

- `find -print0`, `sort -z`, and `read -r -d ''` keep filenames safe even when names contain spaces.
- The scan loop passes each filename to `awk` with `-v source_file="$file"`, avoiding fragile parsing of `grep -H` output.
- The first `awk` script uses `match` and `substr` to extract the status code from each log line.
- `stdout` contains only the final report rows.
- Progress, usage errors, and warnings go to `stderr`.
- This script is for simple line-oriented logs. Use `jq` for JSON logs and Python's `csv` module for real CSV.

## Next Step

Return to this level's README and continue with the next numbered lesson.

## Sources Used

- GNU Bash Manual, Pipelines: https://www.gnu.org/software/bash/manual/html_node/Pipelines.html
- GNU Bash Manual, The Set Builtin (`pipefail`): https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html
- GNU Bash Manual, Bash Variables (`PIPESTATUS`): https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html
- GNU Bash Manual, Bash Builtins (`read -r -d`): https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html
- GNU Coreutils Manual, `sort`: https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html
- GNU Coreutils Manual, `uniq`: https://www.gnu.org/software/coreutils/manual/html_node/uniq-invocation.html
- GNU Coreutils Manual, `cut`: https://www.gnu.org/software/coreutils/manual/html_node/cut-invocation.html
- GNU Coreutils Manual, `tr`: https://www.gnu.org/software/coreutils/manual/html_node/tr-invocation.html
- GNU Findutils Manual, Safe File Name Handling: https://www.gnu.org/software/findutils/manual/html_node/find_html/Safe-File-Name-Handling.html
- GNU Grep Manual, Usage and exit statuses: https://www.gnu.org/software/grep/manual/html_node/Usage.html
- GNU Awk User's Guide, Fields: https://www.gnu.org/software/gawk/manual/html_node/Fields.html
- GNU sed Manual, The `s` Command: https://www.gnu.org/software/sed/manual/html_node/The-_0022s_0022-Command.html
- Python documentation, `csv` module: https://docs.python.org/3/library/csv.html
- jq Manual: https://jqlang.github.io/jq/manual/
