# Curriculum Audit And Rewrite Workflow

Use this workflow when asked to improve the Coding Learning curriculum without user babysitting.

## Role

Act as the coordinator. Keep the repo moving through a strict two-file rolling gate:

1. Build or update the root `TODO.md` inventory.
2. Select the first actionable files from the top of `TODO.md`, up to two active files total.
3. Move each active file through audit, audit review, rewrite, rewrite review, validation, and completion.
4. Do not start a third active file while two files are still active.
5. Review audit notes before any rewrite begins for that same file.
6. Rewrite a file only after its audit review passes.
7. Require Windows and macOS Apple Silicon compatibility instructions before a file becomes `complete`.
8. Use GitHub-renderable Mermaid diagrams when a concise user-to-code data flow clarifies the lesson.
9. Validate links and lesson requirements after each file is accepted.
10. When one active file becomes `complete`, immediately pull the next `needs-audit` file from the top of `TODO.md` to keep up to two active files.

## Two-File Rolling Gate Pattern

Keep at most two active files in progress. Do not create a broad backlog of audited files waiting for rewrites or rewritten files waiting for review.

For each active file, use this sequence:

1. `needs-audit` -> `audit-dispatched`: one read-only audit agent researches and scopes exactly this file.
2. `audit-dispatched` -> `audit-reviewed`: a separate read-only review checks that the audit notes are specific, source-backed, implementation-ready, and scoped to the assigned file.
3. `audit-reviewed` -> `rewrite-dispatched`: one writer rewrites exactly this file from the reviewed audit notes.
4. `rewrite-dispatched` -> `rewrite-reviewed`: a separate read-only review checks the rewritten file against the reviewed audit notes and the workflow quality gate.
5. `rewrite-reviewed` -> `complete`: coordinator validation passes, compatibility requirements are satisfied, links are valid, and lesson requirements are present.

The two-file window is a cap, not a queue:

- Count files with `audit-dispatched`, `audit-reviewed`, `rewrite-dispatched`, `rewrite-reviewed`, `compatibility-review-needed`, `compatibility-review-dispatched`, or `compatibility-reviewed` as active unless they are deliberately reset to `needs-audit` or marked `complete`.
- Start a new audit only when fewer than two files are active.
- Prefer moving the oldest active file to its next gate before starting fresh work.
- If a review returns `NEEDS_FIX`, keep that same file active and repair that gate before advancing it.
- When the user says “continue,” resume this two-file rolling process from the current `TODO.md` state without asking them to re-specify it.

## Context Hygiene

Keep the working context small and current. On every resume or long-running continuation:

- Treat `TODO.md` as the source of truth for file state.
- Re-read only the workflow, active TODO rows, active agent results, and the specific files being audited, rewritten, or reviewed.
- Carry forward only actionable state: active file paths, owner/status/source fields, active agent IDs, reviewed audit notes, open blockers, and validation results.
- Do not keep old audit notes, prompts, command output, or completed-file details in context after the file is marked `complete`, except for concise lessons that affect the current file.
- Summarize long agent outputs into compact reviewed specs before giving them to writers or reviewers.
- Prefer targeted validation for the file that just passed review; avoid loading broad repo output unless a global check is explicitly needed.
- If context was compacted, restart from `TODO.md` and active agents instead of reconstructing every prior step from memory.

The review steps are the quality gate:

- Audit notes become `audit-reviewed` only after review confirms they are specific, source-backed, and implementation-ready.
- Rewrites become `rewrite-reviewed` only after review confirms the file meets the audit spec and quality gate.
- Previously approved files that predate the platform gate become `compatibility-review-needed` and must pass that review before becoming `complete`.
- Files become `complete` only after review findings are fixed, compatibility review passes, and validation is clean.
- If review returns `NEEDS_FIX`, the coordinator or assigned writer fixes only the reported issues, then sends the same file back to the relevant review step.
- Do not exceed two active files unless the user explicitly asks for a larger window.

## File Ownership

- One audit agent owns exactly one file.
- One implementation worker owns exactly one file.
- One review agent owns exactly one rewritten file.
- Agents should use the configs in `.agents/curriculum-audit.agent`, `.agents/curriculum-writer.agent`, and `.agents/curriculum-review.agent`.
- Workers must edit only their assigned file.
- Workers are not alone in the codebase; they must not revert unrelated edits.
- Auditors and writers should treat review as the anti-hallucination and quality gate.
- Auditors must cite authoritative sources for language/runtime/library claims.
- Writers must preserve source-backed claims from reviewed audit notes and avoid adding unsupported claims.
- README files are navigation/overview files.
- Lesson files must include exercises and answers.

## Cross-Platform And Mermaid Gate

Every curriculum file that gives setup, run, test, file-system, environment-variable, HTTP, or package-management instructions must be usable on both:

- Windows 10/11 with PowerShell.
- macOS on Apple Silicon (`arm64`) with the default `zsh` shell.

Apply these rules:

- Prefer portable tool commands such as `dotnet`, language runtimes, and relative paths when their syntax is genuinely identical.
- When shell syntax differs, show short, clearly labeled `powershell` and `bash` blocks. Do not present one platform's syntax as universal.
- Do not hard-code `C:\\...`, `/Users/...`, `cmd.exe`, or architecture-specific paths in instructions. Use project-relative paths and the language's path APIs in code.
- Explain environment-variable syntax separately when it differs: `$env:NAME` in PowerShell and `$NAME` in `zsh`.
- When an HTTP example uses curl, avoid the PowerShell alias ambiguity: use `curl.exe` for Windows PowerShell examples and `curl` for macOS examples, or use an unambiguous alternative.
- For installation or native-dependency guidance, explicitly account for the Apple Silicon (`arm64`) build or installer. Do not claim architecture support without an authoritative source.
- Reviewers must check that a learner can follow the stated steps on both platforms without translating shell syntax themselves.

Mermaid is optional and should earn its space. Use a fenced `mermaid` diagram only when it materially clarifies how a user's action or input moves through code, a service, storage, or an output. Mermaid diagrams must:

- Use syntax rendered by GitHub and the VS Code Mermaid extension.
- Stay short, readable, and faithful to the surrounding example.
- Show concrete user-to-code or data-flow relationships, not decorative architecture.
- Accompany, not replace, the written explanation and runnable example.

## Audit Agent Prompt Template

```text
Read-only audit exactly one curriculum file: `<PATH>` in this repo.
Do not edit files.
Research what this lesson or README should teach using authoritative sources.
Return:
1. Current file quality problems.
2. Detailed rewrite outline. with lesson for the topic included. the outline should be detailed enough to show, explain, and teach the concept.
3. Practical exercise or project prompt if this is a lesson file.
4. Complete worked answer if this is a lesson file.
5. Authoritative sources used.
6. Windows PowerShell and macOS Apple Silicon (`arm64`, `zsh`) instruction requirements or risks.
7. Whether a GitHub-renderable Mermaid user-to-code data-flow diagram would clarify this file; include a proposed diagram only when useful.
Make the output ready for a separate worker to rewrite only this file.
Flag any claims that require reviewer attention.
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

Cross-platform requirements:
- Keep commands portable where possible.
- When instructions differ, provide tested PowerShell and `zsh` variants.
- Account for macOS Apple Silicon when installation or native dependencies matter.
- Use only project-relative paths in instructions and examples.

Mermaid requirements:
- Add a GitHub-renderable `mermaid` diagram only when it clarifies a concrete user-to-code data flow.
- Keep diagrams aligned with the accompanying code and explanation.

After editing, report the file changed and a short summary.
Expect a separate review agent to verify factual accuracy, code validity, source quality, and scope control.
```

## Review Agent Prompt Template

```text
Read-only review exactly one rewritten curriculum file: `<PATH>` in this repo.
Do not edit files.

Use the reviewed audit notes and workflow quality gate as the review spec:

<AUDIT NOTES>

Return:
1. PASS or NEEDS_FIX.
2. Blocking issues with file/line references when possible.
3. Factual accuracy concerns or unsupported claims.
4. Invalid, misleading, or non-runnable code fences.
5. Missing lesson requirements, stale template language, bad links, or out-of-scope edits.
6. Windows PowerShell and macOS Apple Silicon compatibility issues, including shell syntax, paths, environment variables, HTTP commands, installers, and native dependencies.
7. Mermaid validity and usefulness: GitHub/VS Code-compatible syntax, faithful user-to-code data flow, and alignment with the prose.
8. Residual risks, including anything not executed.

Approve only when the file is ready for coordinator validation.
```

## Quality Gate

Reject or revise any rewrite that:

- Uses generic template language.
- Shows examples unrelated to the lesson title.
- Omits a worked answer for a lesson exercise.
- Provides code that is obviously invalid for the language.
- Adds unverified claims where official docs are available.
- Edits files outside the assigned path.
- Labels intentionally invalid or fragmentary snippets as runnable language code fences.
- Cites non-authoritative sources for claims that official docs can verify.
- Gives instructions that require a learner to translate between Windows PowerShell and macOS Apple Silicon `zsh` without saying so.
- Uses hard-coded platform paths or unsupported architecture claims.
- Includes a Mermaid diagram that is invalid, decorative, or inconsistent with the described code/data flow.

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
- `compatibility-review-needed`
- `compatibility-review-dispatched`
- `compatibility-reviewed`
- `complete`

## Batch Strategy

Prefer coherent batches within the two-file window:

1. Current user-open files.
2. One topic folder at a time.
3. One language level at a time.
4. README polish after lesson rewrites.

Keep the active window small enough to review carefully: two files maximum by default.
