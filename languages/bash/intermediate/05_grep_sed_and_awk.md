# 05 - grep sed and awk

## Learning Goal

Use `grep`, `sed`, and `awk` for line-oriented text workflows, and choose the right tool for selecting lines, transforming streams, and summarizing fields.

## Why It Matters

Many Bash programs spend most of their time moving text from one command to another. Logs, command output, configuration files, and reports are usually plain text with one record per line. `grep`, `sed`, and `awk` are classic tools for this job:

- `grep` selects lines that match a pattern.
- `sed` transforms a stream of text one line at a time.
- `awk` processes records and fields, especially when you need comparisons, totals, or summaries.

The tools overlap, but they are not the same. A clear script usually uses the smallest tool that expresses the job directly.

## Sample Data

Create this file as `access.log`:

```text
2026-06-20 api 200 87ms
2026-06-20 web 404 23ms
2026-06-20 api 500 310ms
2026-06-20 auth 200 145ms
2026-06-20 api 200 92ms
2026-06-20 billing 503 780ms
2026-06-20 web 200 18ms
```

Each line has four fields:

```text
date service status time_ms
```

The spaces matter because the examples use whitespace-separated fields.

## Choosing The Tool

Use `grep` when the question is "which lines match this pattern?"

```bash
grep ' api ' access.log
```

Use `sed` when the question is "how should each matching line be rewritten?"

```bash
sed -E 's/([0-9]+)ms$/\1/' access.log
```

Use `awk` when the question is "what do the fields mean?"

```bash
awk '$3 >= 500 { print $2, $3, $4 }' access.log
```

Pipelines are useful, but do not over-pipeline. This works:

```bash
grep ' api ' access.log | sed -E 's/ms$//'
```

But this single `awk` command is clearer when the condition depends on fields:

```bash
awk '$2 == "api" { sub(/ms$/, "", $4); print }' access.log
```

## Grep: Select Matching Lines

`grep` prints lines that match a pattern.

```bash
grep 'api' access.log
```

Output:

```text
2026-06-20 api 200 87ms
2026-06-20 api 500 310ms
2026-06-20 api 200 92ms
```

Quote patterns so the shell does not expand special characters before `grep` sees them.

Use `-E` for extended regular expressions:

```bash
grep -E ' (404|500|503) ' access.log
```

Use `-v` to invert the match:

```bash
grep -v ' 200 ' access.log
```

Use `-i` for case-insensitive matching:

```bash
grep -i 'API' access.log
```

Use `-q` when you only care whether a match exists:

```bash
if grep -q ' 5[0-9][0-9] ' access.log; then
  echo 'server error found'
fi
```

`grep` exit status is important in scripts:

- `0` means at least one selected line was found.
- `1` means no selected lines were found.
- `2` means an error happened, such as a missing file or invalid regular expression.

No match is not the same as a broken command. A script that treats status `1` as a crash will give false alarms.

## Sed: Transform A Stream

`sed` reads input, applies editing commands, and prints the result. The original file is unchanged unless you explicitly use in-place editing.

The most common command is substitution:

```bash
sed 's/pattern/replacement/flags' file
```

Remove the trailing `ms` from the response time:

```bash
sed -E 's/([0-9]+)ms$/\1/' access.log
```

Output:

```text
2026-06-20 api 200 87
2026-06-20 web 404 23
2026-06-20 api 500 310
2026-06-20 auth 200 145
2026-06-20 api 200 92
2026-06-20 billing 503 780
2026-06-20 web 200 18
```

By default, `s///` replaces only the first match on each line. Add the `g` flag to replace every match on each line:

```bash
printf 'api api web\n' | sed 's/api/service/g'
```

Output:

```text
service service web
```

Use `-n` with `p` to print only selected lines:

```bash
sed -n '/ 5[0-9][0-9] /p' access.log
```

Use `-E` when you want extended regular expressions, such as `+`, `?`, and grouping with parentheses:

```bash
sed -E 's/^(2026-[0-9]{2}-[0-9]{2}) /\1 service=/' access.log
```

Avoid `sed -i` until you have checked the output without it. First redirect to a temporary file or inspect the output:

```bash
sed -E 's/([0-9]+)ms$/\1/' access.log
```

Only after the result is correct should you consider in-place editing, and even then prefer a backup when your `sed` version supports it.

## Awk: Process Records And Fields

`awk` reads input as records. By default, each line is one record, and whitespace separates fields.

Important built-in values:

- `$0` is the whole current record.
- `$1` is the first field.
- `$NF` is the last field.
- `NR` is the current record number across all input.
- `NF` is the number of fields in the current record.

Print the whole line with a line number:

```bash
awk '{ print NR, $0 }' access.log
```

Print the first field, last field, and number of fields:

```bash
awk '{ print $1, $NF, NF }' access.log
```

`awk` programs usually have the shape:

```text
pattern { action }
```

The action runs only when the pattern is true:

```bash
awk '$3 != 200 { print }' access.log
```

Numeric comparisons are one of the reasons to use `awk` instead of `grep`:

```bash
awk '$3 >= 500 { print $0 }' access.log
```

Use `BEGIN` for setup before input is read, and `END` for final output after all input is read:

```bash
awk '
BEGIN { print "service count" }
{ count[$2]++ }
END {
  for (service in count) {
    print service, count[service]
  }
}
' access.log
```

Summaries are where `awk` shines. Average the response time for the `api` service:

```bash
awk '
$2 == "api" {
  sub(/ms$/, "", $4)
  total += $4
  count++
}
END {
  if (count > 0) {
    printf "%.2f\n", total / count
  }
}
' access.log
```

Output:

```text
163.00
```

## Combining The Tools

A pipeline is helpful when each stage has a clear job:

```bash
grep -v ' 200 ' access.log | sed -E 's/([0-9]+)ms$/\1/'
```

This selects non-200 lines with `grep`, then normalizes the time field with `sed`.

But avoid long pipelines when one `awk` command communicates the logic better:

```bash
awk '$3 != 200 { sub(/ms$/, "", $4); print }' access.log
```

That version says: for records whose status field is not `200`, remove `ms` from field 4 and print the record.

## Common Mistakes

- Leaving regular expressions unquoted, which lets the shell expand characters before the tool runs.
- Using `grep` for field comparison, such as status greater than or equal to 500. Use `awk` for field-aware numeric logic.
- Forgetting the `g` flag in `sed`, so only the first match on each line is replaced.
- Running `sed -i` before checking the output, which can damage the original file.
- Parsing real CSV with simple whitespace or comma splitting. CSV can contain quoted commas and escaped quotes; use a CSV-aware tool or language library.
- Confusing shell `$1` with `awk` `$1`. Inside single quotes in an `awk` program, `$1` means the first input field. In a shell script outside those quotes, `$1` means the script's first argument.
- Treating `grep` status `1` as an error. It usually means no match was found.

## Exercise

Write `summarize_access.sh`.

Requirements:

- Read from `access.log` by default.
- Allow a different log path as the first script argument.
- Print non-200 responses.
- Print a normalized log without trailing `ms`.
- Print a count per service.
- Print the average `api` response time.
- Print a warning if any 5xx response exists.
- Use `grep`, `sed`, and `awk` at least once.
- Do not modify the log file.

## Worked Answer

```bash
#!/usr/bin/env bash
set -u

log=${1:-access.log}

echo 'non-200 responses:'
awk '$3 != 200 { print }' "$log"

echo
echo 'normalized log:'
sed -E 's/([0-9]+)ms$/\1/' "$log"

echo
echo 'count per service:'
awk '{ count[$2]++ } END { for (service in count) print service, count[service] }' "$log" | sort

echo
echo 'average api response time:'
awk '
$2 == "api" {
  sub(/ms$/, "", $4)
  total += $4
  count++
}
END {
  if (count > 0) {
    printf "%.2f\n", total / count
  } else {
    print "no api responses"
  }
}
' "$log"

echo
if grep -Eq ' 5[0-9][0-9] ' "$log"; then
  echo 'warning: 5xx response found'
else
  echo 'warning: no 5xx response found'
fi
```

Run it:

```bash
chmod +x summarize_access.sh
./summarize_access.sh
```

Expected output:

```text
non-200 responses:
2026-06-20 web 404 23ms
2026-06-20 api 500 310ms
2026-06-20 billing 503 780ms

normalized log:
2026-06-20 api 200 87
2026-06-20 web 404 23
2026-06-20 api 500 310
2026-06-20 auth 200 145
2026-06-20 api 200 92
2026-06-20 billing 503 780
2026-06-20 web 200 18

count per service:
api 3
auth 1
billing 1
web 2

average api response time:
163.00

warning: 5xx response found
```

## Next Step

Try changing `access.log` so every status is `200`, then run the script again. Confirm that the non-200 section is empty and the warning changes to `warning: no 5xx response found`.

## Sources Used

- GNU Grep manual: https://www.gnu.org/software/grep/manual/grep.html
- GNU Sed manual: https://www.gnu.org/software/sed/manual/sed.html
- GNU Awk User Guide: https://www.gnu.org/software/gawk/manual/gawk.html
