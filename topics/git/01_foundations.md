# 01 - Git Foundations

## Learning Goal

Explain what Git is and why developers use it; configure your Git identity; create a local repository; track a file from untracked to staged to committed; and inspect repository state with `git status`, `git diff`, and `git log --oneline`.

## What Git Is

Git is a distributed version control system. Version control means it records snapshots of a project over time, so you can review what changed, return to an earlier state, and understand why code looks the way it does. Distributed means each local repository contains its own project history. Many everyday Git operations read and write local files first, even before any remote service is involved.

Git is not the same thing as GitHub. Git is the version control tool you run on your computer. GitHub is one service that can host Git repositories online for sharing, review, backup, and collaboration. This lesson stays local: you will not push anything to GitHub.

Version control matters even when you work alone. It gives you checkpoints before experiments, a history of decisions, and a way to compare your current code with a known working version. On a team, the same history makes code review, collaboration, and recovering from mistakes much easier.

## The Three Places

A Git project has three important places to keep straight:

| Place | What it means | Common state |
| --- | --- | --- |
| Working tree | The files you can see and edit in the project folder. | Modified or untracked |
| Staging area, also called the index | The next snapshot you have prepared with `git add`. | Staged |
| Git directory | The repository database where Git stores commits and metadata. | Committed |

The basic workflow is:

```mermaid
flowchart LR
    A[User edits hello.py] --> B[Working tree: modified file]
    B -->|git add hello.py| C[Staging area: next snapshot]
    C -->|git commit -m "Add hello script"| D[Git history: committed snapshot]
    B -->|git status / git diff| E[Inspect current changes]
    D -->|git log --oneline| F[Inspect saved commits]
```

The important detail is that `git add` does not save permanent history. It stages the current content for the next commit. `git commit` records the staged snapshot in the repository history.

## Check Your Setup

Run these commands from any folder:

```powershell
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global --list
```

The same commands work in `zsh`:

```zsh
git --version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global --list
```

`user.name` and `user.email` are written into future commits as authorship information. Changing them later does not rewrite commits you already made. If Git is not installed, follow the setup instructions for your operating system before continuing; installation steps differ across Windows, macOS, and Linux, so this lesson does not assume one installer.

## Create a Disposable Local Repository

Create this practice repository outside this curriculum repository. For example, if you are currently inside the curriculum repo, first move to its parent folder with `cd ..`. Keep the folder name relative and simple:

```powershell
mkdir git-foundations-practice
cd git-foundations-practice
git init
git status
```

The same commands work in `zsh`:

```zsh
mkdir git-foundations-practice
cd git-foundations-practice
git init
git status
```

After `git init`, Git creates a new local repository. The default branch name in the status output can vary, commonly `main` or `master`, depending on your Git configuration and environment. The branch name is not important for this lesson.

## Track Your First File

Create a small Python file, then inspect the state.

PowerShell:

```powershell
Set-Content -LiteralPath .\hello.py -Value "print('Hello from Git')" -Encoding UTF8
git status
```

zsh:

```zsh
printf "print('Hello from Git')\n" > hello.py
git status
```

At this point, `hello.py` should appear as untracked. Git can see the file in your working tree, but it is not part of the next commit yet.

Stage and commit it:

```powershell
git add hello.py
git status
git commit -m "Add hello script"
git status
```

The same commands work in `zsh`:

```zsh
git add hello.py
git status
git commit -m "Add hello script"
git status
```

After `git add hello.py`, `git status` should show the file under changes to be committed. After `git commit`, Git stores the staged snapshot in local history. After that, `git status` should report a clean working tree unless another file changed.

## Inspect History and Changes

View the commit history:

```powershell
git log --oneline
```

The same command works in `zsh`:

```zsh
git log --oneline
```

`git log --oneline` shows a compact history: a short commit identifier and the commit message, such as `Add hello script`.

Now modify the file and inspect the current change.

PowerShell:

```powershell
Set-Content -LiteralPath .\hello.py -Value "print('Hello from Git again')" -Encoding UTF8
git status
git diff
```

zsh:

```zsh
printf "print('Hello from Git again')\n" > hello.py
git status
git diff
```

`git status` answers "what state is my repository in?" It tells you whether files are untracked, modified, staged, or clean.

`git diff` answers "what changed but is not staged yet?" In this example, it should show the original line being replaced by `print('Hello from Git again')`.

`git log --oneline` answers "what commits are already saved in history?" It does not show unstaged edits that have not been committed.

## Quick Mental Model

| Question | Command |
| --- | --- |
| Do I have Git installed? | `git --version` |
| What name and email will future commits use? | `git config --global --list` |
| Am I in a Git repository, and what changed? | `git status` |
| What exactly changed in unstaged files? | `git diff` |
| What should be included in the next commit? | `git add hello.py` |
| How do I save the staged snapshot? | `git commit -m "Add hello script"` |
| What commits have been saved? | `git log --oneline` |

## Common Mistakes

- Running Git commands in the wrong folder. Use `git status` to confirm you are inside the intended repository.
- Trying to commit before configuring `user.name` and `user.email`. Configure them once with `git config --global`.
- Confusing Git with GitHub. A local commit is saved in Git, but it is not automatically pushed to GitHub.
- Thinking `git add` saves history. `git add` stages content; `git commit` saves the staged snapshot.
- Committing generated files, cache folders, or local build artifacts by accident. Check `git status` before staging.
- Ignoring `git status`. It is the fastest way to understand what Git thinks is happening.

## Exercise

Create a disposable local repository and explain the path a file takes through Git.

1. Create `git-foundations-practice` outside this curriculum repository.
2. Run `git --version`.
3. Configure `user.name` and `user.email` if they are missing.
4. Run `git init`.
5. Create `hello.py` with `print('Hello from Git')`.
6. Run `git status`.
7. Run `git add hello.py`.
8. Run `git status` again.
9. Run `git commit -m "Add hello script"`.
10. Run `git log --oneline`.
11. Replace the file content with `print('Hello from Git again')`.
12. Run `git status` and `git diff`.
13. In your own words, explain what happened in the working tree, staging area, and history.

## Worked Answer

Your exact output will vary. Commit hashes, branch names, Git versions, and status wording do not need to match exactly.

A correct setup check includes output like this:

```text
git version 2.x.x
user.name=Your Name
user.email=you@example.com
```

After `git init`, a correct `git status` says you are on a branch, often `main` or `master`, and that there are no commits yet.

After creating `hello.py`, a correct `git status` shows `hello.py` as an untracked file. That means the file exists in the working tree, but Git has not been told to include it in the next snapshot.

After `git add hello.py`, a correct `git status` shows `hello.py` as a new file under changes to be committed. That means the file is staged in the index.

After `git commit -m "Add hello script"`, a correct `git status` reports a clean working tree. A correct `git log --oneline` includes one commit with the message:

```text
Add hello script
```

The short hash before the message will be different on each computer.

After replacing the file content, a correct `git status` shows `hello.py` as modified but not staged. A correct `git diff` shows that:

```text
print('Hello from Git')
```

was replaced with:

```text
print('Hello from Git again')
```

The explanation should say:

- The working tree changed when `hello.py` was created and later edited.
- The staging area changed when `git add hello.py` prepared the first version for commit.
- The Git history changed when `git commit -m "Add hello script"` saved the staged snapshot.
- The final edit is visible in `git status` and `git diff`, but it is not in history until it is staged and committed.

## Sources Used

- [What is Git?](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F)
- [Recording Changes to the Repository](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)
- [Set up Git](https://docs.github.com/en/get-started/git-basics/set-up-git)
- [Setting your username in Git](https://docs.github.com/en/get-started/git-basics/setting-your-username-in-git)
- [Setting your commit email address](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/setting-your-commit-email-address)
- [git-diff Documentation](https://git-scm.com/docs/git-diff)
