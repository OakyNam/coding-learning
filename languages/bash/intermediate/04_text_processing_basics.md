# 04 - Text Processing Basics

## Learning Goal

Learn how Bash text processing tools read line-oriented streams, compose with stdin and stdout, and form small pipelines for simple reports.

By the end of this lesson, you should be able to use `head`, `tail`, `wc`, `sort`, `uniq`, `cut`, `tr`, and `printf` together without reaching for `grep`, `sed`, or `awk` yet.

## Why It Matters

Many command-line programs are designed to do one small job with text. They usually read from standard input, write to standard output, and pass data to the next command through a pipe.

That gives Bash a simple text-processing model:

1. Start with lines of text.
2. Select, count, sort, split, or translate those lines.
3. Pipe the result into another small tool.
4. Print a clear final result.

Later lessons cover `grep`, `sed`, and `awk`. This lesson builds the foundation first: stream shape, fields, sorted input, and predictable output.

## Sample File

Create a small log file to use in the examples:

```bash
cat > access.log <<'LOG'
2026-06-20T09:00:01Z auth 200 /login
2026-06-20T09:01:13Z api 200 /v1/users
2026-06-20T09:02:45Z billing 500 /invoice
2026-06-20T09:03:02Z auth 401 /login
2026-06-20T09:04:18Z search 200 /query
2026-06-20T09:05:31Z api 404 /v1/missing
2026-06-20T09:06:09Z billing 200 /invoice
2026-06-20T09:07:42Z search 503 /query
LOG
```

Each line has four space-separated fields:

```text
timestamp service status path
```

That regular shape is what makes simple tools useful.

## The Line-Oriented Stream Model

A pipeline connects the stdout of one command to the stdin of the next command:

```bash
cut -d ' ' -f2 access.log | sort | uniq
```

Read it left to right:

1. `cut -d ' ' -f2 access.log` prints the second space-separated field, the service name.
2. `sort` sorts those service names.
3. `uniq` removes adjacent duplicate service names.

The output is:

```text
api
auth
billing
search
```

Most tools in this lesson can read either from files named as arguments or from stdin:

```bash
head -n 2 access.log
cut -d ' ' -f2 access.log
cut -d ' ' -f2 < access.log
```

Using stdin makes tools easy to compose in pipelines.

## First And Last Lines

Use `head` to inspect the beginning of a stream:

```bash
head -n 2 access.log
```

Output:

```text
2026-06-20T09:00:01Z auth 200 /login
2026-06-20T09:01:13Z api 200 /v1/users
```

Use `tail` to inspect the end:

```bash
tail -n 2 access.log
```

Output:

```text
2026-06-20T09:06:09Z billing 200 /invoice
2026-06-20T09:07:42Z search 503 /query
```

These are useful before and after a pipeline because they let you check the shape of the data.

## Counting Lines With wc

`wc` counts lines, words, bytes, characters, or maximum line length depending on its options. For log summaries, `wc -l` is common.

With a filename:

```bash
wc -l access.log
```

Output:

```text
8 access.log
```

With input redirection:

```bash
wc -l < access.log
```

Output:

```text
8
```

The count is the same, but the display is different. `wc -l file` includes the filename. `wc -l < file` reads from stdin, so there is no filename to print. Scripts often use the redirected form when they only need the number.

## Sorting And Unique Values

`uniq` only compares adjacent lines. It does not find duplicates spread throughout the whole input.

This command is not enough for unsorted data:

```bash
cut -d ' ' -f2 access.log | uniq
```

Because the services are mixed together, repeated services are not adjacent. Sort first:

```bash
cut -d ' ' -f2 access.log | sort | uniq
```

To count each group, add `-c`:

```bash
cut -d ' ' -f2 access.log | sort | uniq -c
```

Output:

```text
      2 api
      2 auth
      2 billing
      2 search
```

Sorting can depend on your locale, especially for mixed case, accents, and punctuation. When a script needs reproducible byte-order sorting across machines, set the locale for that command:

```bash
cut -d ' ' -f2 access.log | LC_ALL=C sort | uniq
```

For this sample log, normal `sort` and `LC_ALL=C sort` produce the same order.

## Cutting Fields

`cut` selects parts of each line. With `-d`, you choose a single-character delimiter. With `-f`, you choose fields.

Print the service field:

```bash
cut -d ' ' -f2 access.log
```

Print the status field:

```bash
cut -d ' ' -f3 access.log
```

Print service and status:

```bash
cut -d ' ' -f2,3 access.log
```

Output:

```text
auth 200
api 200
billing 500
auth 401
search 200
api 404
billing 200
search 503
```

`cut` is fast and clear when the input has a dependable delimiter. Its limitation is that it does not understand quoting, escaping, nested formats, or "one or more spaces" as a delimiter. With `-d ' '`, every literal space is a delimiter. If your data has variable spacing or quoted fields, save that for a later tool such as `awk` or a real parser.

## Translating Characters With tr

`tr` translates or deletes characters. It does not replace words.

Turn lowercase letters into uppercase letters:

```bash
printf 'auth api billing\n' | tr '[:lower:]' '[:upper:]'
```

Output:

```text
AUTH API BILLING
```

Change spaces to newlines:

```bash
printf 'auth api billing\n' | tr ' ' '\n'
```

Output:

```text
auth
api
billing
```

Delete spaces from a count:

```bash
wc -l < access.log | tr -d ' '
```

Output:

```text
8
```

Do not use `tr` for word replacement:

```bash
printf 'auth api\n' | tr 'auth' 'user'
```

This translates individual characters from the set `a`, `u`, `t`, `h` into characters from the set `u`, `s`, `e`, `r`. It does not replace the word `auth` with the word `user`.

## Printing Clear Output

`printf` is better than `echo` when a script needs predictable formatting:

```bash
printf 'Total entries: %s\n' 8
printf '%s\n' 'Unique services:'
```

`%s` means "put the next argument here as a string." `\n` means "end the line."

## Small Pipelines

A good beginner pipeline has a clear input and one small purpose:

```bash
cut -d ' ' -f3 access.log | sort | uniq -c
```

That means:

1. Extract status codes.
2. Sort equal status codes next to each other.
3. Count each adjacent group.

Output:

```text
      4 200
      1 401
      1 404
      1 500
      1 503
```

Notice what is not in this lesson: pattern matching with `grep`, stream editing with `sed`, or field programs with `awk`. Those are next-lesson tools. Here, the goal is to get comfortable with simple stream composition first.

## Common Mistakes

- Forgetting that `uniq` only removes adjacent duplicates. Use `sort | uniq` for whole-file grouping.
- Expecting `wc -l file` and `wc -l < file` to print the same shape. The filename appears only when `wc` receives a filename argument.
- Using `cut -d ' ' -f2` on data with unpredictable spacing. `cut` treats each delimiter character literally.
- Trying to use `tr` for word replacement. `tr` translates characters, not words.
- Building a long pipeline before checking the first step. Run `cut ...` alone before adding `sort` and `uniq`.
- Forgetting that sort order can vary by locale. Use `LC_ALL=C sort` when reproducible byte-order output matters.

## Exercise

Write `summarize-log.sh`. It should read `access.log` from the current directory and print:

1. Total entries.
2. First two entries.
3. Last two entries.
4. Unique services.
5. Entries per service.
6. Entries per status code.

Use only the tools from this lesson for text processing: `head`, `tail`, `wc`, `sort`, `uniq`, `cut`, `tr`, and `printf`.

## Worked Answer

Create `summarize-log.sh`:

```bash
#!/usr/bin/env bash

log_file="access.log"

total_entries=$(wc -l < "$log_file" | tr -d ' ')

printf 'Total entries: %s\n' "$total_entries"

printf '\nFirst two entries:\n'
head -n 2 "$log_file"

printf '\nLast two entries:\n'
tail -n 2 "$log_file"

printf '\nUnique services:\n'
cut -d ' ' -f2 "$log_file" | LC_ALL=C sort | uniq

printf '\nEntries per service:\n'
cut -d ' ' -f2 "$log_file" | LC_ALL=C sort | uniq -c

printf '\nEntries per status code:\n'
cut -d ' ' -f3 "$log_file" | LC_ALL=C sort | uniq -c
```

Make it executable and run it:

```bash
chmod +x summarize-log.sh
./summarize-log.sh
```

Expected output:

```text
Total entries: 8

First two entries:
2026-06-20T09:00:01Z auth 200 /login
2026-06-20T09:01:13Z api 200 /v1/users

Last two entries:
2026-06-20T09:06:09Z billing 200 /invoice
2026-06-20T09:07:42Z search 503 /query

Unique services:
api
auth
billing
search

Entries per service:
      2 api
      2 auth
      2 billing
      2 search

Entries per status code:
      4 200
      1 401
      1 404
      1 500
      1 503
```

## Explanation

The script keeps the log filename in one variable so the later commands are easy to read. It uses `wc -l < "$log_file"` to count lines without printing the filename, then uses `tr -d ' '` to remove padding spaces from the count.

`head` and `tail` print unchanged log lines. The unique-service report extracts field 2 with `cut`, sorts the names so duplicates become adjacent, and then uses `uniq`. The count reports use the same pattern with `uniq -c`.

`LC_ALL=C sort` is included to keep the script's ordering reproducible. It matters more with mixed-case or non-ASCII data, but adding it here makes the habit visible.

## Next Step

Continue to `05_grep_sed_and_awk.md` when you are ready to filter by patterns, rewrite streams, and write field-based text programs.

## Sources Used

- GNU Bash Manual: [Pipelines](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html)
- GNU Bash Manual: [Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)
- GNU Coreutils Manual: [head invocation](https://www.gnu.org/software/coreutils/manual/html_node/head-invocation.html)
- GNU Coreutils Manual: [tail invocation](https://www.gnu.org/software/coreutils/manual/html_node/tail-invocation.html)
- GNU Coreutils Manual: [wc invocation](https://www.gnu.org/software/coreutils/manual/html_node/wc-invocation.html)
- GNU Coreutils Manual: [sort invocation](https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html)
- GNU Coreutils Manual: [uniq invocation](https://www.gnu.org/software/coreutils/manual/html_node/uniq-invocation.html)
- GNU Coreutils Manual: [cut invocation](https://www.gnu.org/software/coreutils/manual/html_node/cut-invocation.html)
- GNU Coreutils Manual: [tr invocation](https://www.gnu.org/software/coreutils/manual/html_node/tr-invocation.html)
