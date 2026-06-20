# Curriculum Audit And Rewrite Workflow

Use this workflow when asked to improve the Coding Learning curriculum without user babysitting.

## Role

Act as the coordinator. Keep the repo moving through repeated batches:

1. Build or update the root `TODO.md` inventory.
2. Dispatch read-only audit agents for individual files.
3. Review audit returns for quality and correctness.
4. Dispatch separate worker agents to rewrite approved files.
5. Review worker changes.
6. Validate links and lesson requirements.
7. Commit and push each accepted batch.
8. Continue with the next batch.

## File Ownership

- One audit agent owns exactly one file.
- One implementation worker owns exactly one file.
- Workers must edit only their assigned file.
- Workers are not alone in the codebase; they must not revert unrelated edits.
- README files are navigation/overview files.
- Lesson files must include exercises and answers.

## Audit Agent Prompt Template

```text
Read-only audit exactly one curriculum file: `<PATH>` in this repo.
Do not edit files.
Research what this lesson or README should teach using authoritative sources.
Return:
1. Current file quality problems.
2. Detailed rewrite outline.
3. Practical exercise or project prompt if this is a lesson file.
4. Complete worked answer if this is a lesson file.
5. Authoritative sources used.
Make the output ready for a separate worker to rewrite only this file.
```

## Worker Agent Prompt Template

```text
Rewrite exactly one curriculum file: `<PATH>`.
You own only this file. Do not edit any other file.
You are not alone in the codebase; do not revert unrelated edits.

Use the reviewed audit notes below as the rewrite spec:

<AUDIT NOTES>

Requirements for lesson files:
- Specific learning goal.
- Real explanation of the topic.
- Correct, topic-specific examples.
- Common mistakes specific to the topic.
- At least one exercise.
- A complete worked answer for each exercise.
- Sources used.

Requirements for README files:
- Clear overview.
- Accurate links.
- Suggested learning order.
- No filler/template prose.

After editing, report the file changed and a short summary.
```

## Quality Gate

Reject or revise any rewrite that:

- Uses generic template language.
- Shows examples unrelated to the lesson title.
- Omits a worked answer for a lesson exercise.
- Provides code that is obviously invalid for the language.
- Adds unverified claims where official docs are available.
- Edits files outside the assigned path.

## Validation Commands

Run these after each batch:

```powershell
rg -n "input -> process -> output|api\", \"json\", \"test|Recreate the example from memory|This lesson helps you move" README.md languages topics

$missing=@()
Get-ChildItem -Recurse -Filter *.md | ForEach-Object {
  $file=$_.FullName
  $dir=$_.DirectoryName
  $text=Get-Content -Raw -LiteralPath $file
  [regex]::Matches($text, '\[[^\]]+\]\(([^)]+)\)') | ForEach-Object {
    $link=$_.Groups[1].Value
    if($link -match '^(https?|mailto):'){ return }
    if($link -match '^#'){ return }
    $target=$link.Split('#')[0]
    if([string]::IsNullOrWhiteSpace($target)){ return }
    $resolved=Join-Path $dir $target
    if(-not (Test-Path -LiteralPath $resolved)){ $missing += "$file -> $link" }
  }
}
if($missing.Count){ $missing } else { 'ALL_LINKS_OK' }

git status --short --branch
```

## TODO Status Values

Use these statuses in `TODO.md`:

- `needs-audit`
- `audit-dispatched`
- `audit-reviewed`
- `rewrite-dispatched`
- `rewrite-reviewed`
- `complete`

## Batch Strategy

Prefer coherent batches:

1. Current user-open files.
2. One topic folder at a time.
3. One language level at a time.
4. README polish after lesson rewrites.

Keep batches small enough to review carefully, usually 5-10 files.
