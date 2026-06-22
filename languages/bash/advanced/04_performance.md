# 04 - Bash Performance

## Learning Goal

Learn how to measure Bash scripts, identify the common causes of slow shell code, and rewrite small data-processing scripts using builtins, efficient file reading, batching, and the right external tools.

## Why It Matters

Bash is excellent at connecting programs, handling files, and automating operating-system tasks. It is not always excellent at doing millions of tiny operations one process at a time.

Performance problems in Bash usually come from a few patterns:

- starting an external command inside a loop
- reading files in a way that changes whitespace or loses backslashes
- building long pipelines without understanding process and scope costs
- processing records one at a time when the work could be batched
- keeping a problem in Bash after it has become a better fit for `awk`, `find`, `xargs`, `jq`, or Python

The goal is not to make every Bash script clever. The goal is to measure first, fix the biggest cost, and keep the script readable.

## Mental Model

Bash has two broad kinds of work:

- Shell work: variable expansion, parameter expansion, tests, loops, arrays, and builtins such as `read`, `printf`, and `mapfile`.
- Process work: running external commands such as `tr`, `grep`, `sed`, `awk`, `find`, `xargs`, `jq`, and Python.

Shell work usually happens inside the current Bash process. Process work usually requires Bash to create another process, ask the operating system to run a program, connect input and output, and wait for the result.

One external command is usually fine. Thousands of external commands inside a loop can dominate the runtime.

That does not mean "never use external tools." Tools such as `awk`, `grep`, `find`, `xargs`, and `jq` are often much faster than Bash when they do a large amount of work in one process. The expensive pattern is often not the tool itself, but starting it repeatedly for tiny pieces of work.

## Measure First

Use timing tools before rewriting. A slow-looking line is not always the real bottleneck.

The simplest measurement is Bash's `time` reserved word:

```bash
time ./script.sh
```

It reports three useful values:

- `real`: wall-clock time from start to finish. This is what you felt while waiting.
- `user`: CPU time spent running program code.
- `sys`: CPU time spent in the kernel, often from process creation, file I/O, and other operating-system work.

You can change Bash's `time` output with `TIMEFORMAT`:

```bash
TIMEFORMAT='real=%3R user=%3U sys=%3S'
time ./script.sh
```

For more detail, use the external `time` program. On many Linux systems it is available as `/usr/bin/time`:

```bash
/usr/bin/time -p ./script.sh
/usr/bin/time -v ./script.sh
```

The `-p` option gives portable `real`, `user`, and `sys` output. The `-v` option, where available, adds details such as memory use and file-system activity.

Measure the original script, make one change, and measure again. This keeps performance work honest.

## Avoid Subprocesses In Loops

A common slow pattern is starting an external command for every line.

Bad:

```bash
while IFS= read -r name; do
  upper=$(echo "$name" | tr '[:lower:]' '[:upper:]')
  printf '%s\n' "$upper"
done < names.txt
```

This starts at least two external commands per line: `echo` and `tr`. The pipeline also adds process and pipe overhead.

Better, when Bash can do the transformation:

```bash
while IFS= read -r name; do
  printf '%s\n' "${name^^}"
done < names.txt
```

`${name^^}` is Bash parameter expansion. It uppercases the variable without starting `tr`.

Also good, when the whole task is text processing:

```bash
awk '{ print toupper($0) }' names.txt
```

This starts one `awk` process and lets it handle the file efficiently.

## Batch Work

If you need an external command, try to feed it many inputs at once instead of starting it once per item.

Bad:

```bash
for file in *.log; do
  grep -H 'ERROR' "$file"
done
```

This starts one `grep` process per file.

Better for a normal list of files:

```bash
printf '%s\0' ./*.log | xargs -0 grep -H 'ERROR'
```

Better when recursively finding files:

```bash
find . -type f -name '*.log' -print0 | xargs -0 grep -H 'ERROR'
```

`find -print0` and `xargs -0` use a null byte as the separator, so filenames with spaces, tabs, and newlines are handled correctly.

Some tools can already accept many file arguments:

```bash
grep -H 'ERROR' ./*.log
```

Use `xargs` when the file list is large, generated dynamically, or might exceed the command-line length limit.

## Builtins vs External Tools

Bash builtins and parameter expansion avoid process startup. They are good for simple operations on shell variables.

Examples:

```bash
name='ada lovelace'
printf '%s\n' "${name^^}"
printf '%s\n' "${name// /_}"
printf '%s\n' "${#name}"
```

These examples uppercase text, replace spaces with underscores, and get string length without external commands.

External tools are still the right choice when the tool can process the data in bulk:

```bash
awk -F, '$3 > 90 { print $1 }' scores.csv
jq -r '.items[].name' data.json
find . -type f -name '*.tmp' -delete
```

A good rule: use Bash builtins for small shell-level transformations, and use specialized tools for large structured or record-oriented data.

## Reading Files Efficiently

The standard safe pattern for reading a file line by line is:

```bash
while IFS= read -r line; do
  printf '%s\n' "$line"
done < input.txt
```

Important details:

- `IFS=` prevents leading and trailing whitespace from being trimmed.
- `read -r` prevents backslashes from being treated as escapes.
- `< input.txt` feeds the file to the loop without using `cat`.

Avoid this:

```bash
for word in $(cat input.txt); do
  printf '%s\n' "$word"
done
```

That command substitution reads the whole file, then word-splits it on whitespace, then applies pathname expansion. It does not preserve lines.

When the file fits comfortably in memory and you want an array of lines, use `mapfile`:

```bash
mapfile -t lines < input.txt

for line in "${lines[@]}"; do
  printf '%s\n' "$line"
done
```

When you really do need the entire file as one string, Bash can capture it without `cat`:

```bash
contents=$(< input.txt)
```

Use that only when whole-file capture is actually appropriate. It is not a line-reading pattern.

## Pipeline Costs And Scope

Pipelines are useful, but each command in a pipeline may run in a separate process. In Bash, each pipeline segment usually runs in a subshell. That means variable changes inside a pipeline loop may disappear when the loop ends.

Gotcha:

```bash
count=0

printf '%s\n' alpha beta gamma | while IFS= read -r line; do
  count=$((count + 1))
done

printf 'count=%s\n' "$count"
```

The final value may still be `0` because the `while` loop ran in a subshell.

Use redirection when you need loop variables afterward:

```bash
count=0

while IFS= read -r line; do
  count=$((count + 1))
done < names.txt

printf 'count=%s\n' "$count"
```

Pipelines are not bad. They are one of the main reasons to use shell. Just remember that they have process costs and scope behavior.

## When To Switch Languages Or Tools

Stay in Bash when the script mostly coordinates commands, handles files, checks exit statuses, or does simple variable transformations.

Switch tools when the data shape asks for it:

- Use `awk` for line-oriented text, fields, totals, grouping, and simple reports.
- Use `find` for recursive filesystem searches and file predicates.
- Use `xargs` when many paths or records should be passed to a command in batches.
- Use `jq` for JSON instead of parsing JSON with `grep`, `cut`, or `sed`.
- Use Python when the script needs complex data structures, nontrivial parsing, tests, libraries, or maintainable application logic.

The performance win often comes from moving the inner loop into a tool designed for that data.

## Common Mistakes

- Timing only the rewritten script and never measuring the original.
- Optimizing tiny scripts where readability matters more than speed.
- Running `echo "$value" | tr ...` inside a loop when Bash parameter expansion can do the job.
- Using `for word in $(cat file)` and accidentally destroying the original line structure.
- Forgetting that pipeline loops may run in a subshell.
- Starting `grep`, `sed`, `awk`, or `jq` once per record instead of once for the whole input.
- Using `xargs` without `-0` for filenames that might contain whitespace or newlines.
- Keeping complex parsing in Bash after the problem clearly belongs in `awk`, `jq`, or Python.

## Exercise

Create a file named `names.txt`:

```bash
cat > names.txt <<'EOF'
ada lovelace
Grace Hopper
alan turing
Katherine Johnson
EOF
```

Here is a slow script:

```bash
#!/usr/bin/env bash

count=0

for name in $(cat names.txt); do
  upper=$(echo "$name" | tr '[:lower:]' '[:upper:]')
  if echo "$upper" | grep -q 'A'; then
    count=$((count + 1))
    echo "$upper"
  fi
done

echo "count=$count"
```

Do the following:

1. Measure the script with `time` or `/usr/bin/time`.
2. List the performance and correctness problems.
3. Rewrite it using Bash builtins and efficient line reading.
4. Provide an `awk` solution.
5. Explain which solution you would choose for a very large file.

## Worked Answer

One way to measure the original:

```bash
TIMEFORMAT='real=%3R user=%3U sys=%3S'
time ./slow-names.sh
```

Problems in the original script:

- `$(cat names.txt)` reads the whole file and then splits it on whitespace, so `ada lovelace` becomes two separate items.
- The `for` loop iterates over words, not lines.
- `echo "$name" | tr ...` starts external commands for every item.
- `echo "$upper" | grep -q ...` starts more external commands for every item.
- `echo` is less predictable than `printf` for arbitrary text.
- The script does simple string work that Bash can do without subprocesses.

Bash rewrite:

```bash
#!/usr/bin/env bash

count=0

while IFS= read -r name; do
  upper=${name^^}
  if [[ $upper == *A* ]]; then
    count=$((count + 1))
    printf '%s\n' "$upper"
  fi
done < names.txt

printf 'count=%s\n' "$count"
```

This version preserves each input line, uses Bash uppercase parameter expansion, uses `[[ ... ]]` for the match, and avoids external commands inside the loop.

`awk` rewrite:

```bash
awk '
{
  upper = toupper($0)
  if (upper ~ /A/) {
    count++
    print upper
  }
}
END {
  print "count=" count
}
' names.txt
```

For a very large file, the `awk` version is usually the better choice. It starts one process, streams the file, and performs the record processing in a tool designed for line-oriented text. The Bash version is useful when the surrounding script is already Bash and the file is modest, but Bash loops are usually slower for large record-by-record processing.

## Next Step

Return to the advanced Bash README and continue with the next numbered lesson. As you write future scripts, measure first, look for repeated subprocesses, and move large inner loops into the right tool.

## Sources Used

- GNU Bash Manual: Pipelines - https://www.gnu.org/software/bash/manual/bash.html#Pipelines
- GNU Bash Manual: Command Substitution - https://www.gnu.org/software/bash/manual/bash.html#Command-Substitution
- GNU Bash Manual: Bash Builtin Commands - https://www.gnu.org/software/bash/manual/bash.html#Bash-Builtins
- Linux man-pages: `time(1)` - https://man7.org/linux/man-pages/man1/time.1.html
- GNU Findutils Manual: `xargs` - https://www.gnu.org/software/findutils/manual/html_node/find_html/xargs-options.html
