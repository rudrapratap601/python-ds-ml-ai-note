# Rudra's Git, GitHub & Open-Source Handbook

**Purpose:** Quick reference for personal Python/ML projects and fork-based open-source contributions.  
**Progress covered:** Days 1–5, plus practical safety notes.  
**Environment:** Windows, Git Bash, VS Code. Replace example repository and branch names with your own.

## 1. The mental model

- **Working tree:** Files you are editing on your computer.
- **Staging area:** Changes selected for the next commit.
- **Commit:** A recorded snapshot of staged changes.
- **Branch:** A movable pointer to a line of commits.
- **Remote:** A named reference to another Git repository.
- **`origin`:** Usually the repository you cloned. In a fork workflow, this is usually **your fork**.
- **`upstream`:** Conventional name for the original project, added manually after cloning your fork.
- **Fork:** A GitHub-hosted copy of someone else's repository under your account.
- **Clone:** A local copy downloaded to your computer.
- **Pull request (PR):** A proposal to integrate changes, with review and discussion.
- **Issue:** A tracked bug, task, feature request, or discussion. Read project guidelines before taking one on.

## 2. Command reference: setup and inspection

| Command | Use case | Important note |
|---|---|---|
| `git init` | Start version control for a new personal project. | Do not run inside an already cloned repo. |
| `git clone URL` | Download a personal repo or your open-source fork. | Creates `origin` automatically. |
| `cd folder-name` | Enter the repository directory. | Shell command, not Git. |
| `code .` | Open current folder in VS Code. | Requires VS Code command on PATH. |
| `git status` | Check current branch, modified/staged files, and sync status. | Run before and after risky operations. |
| `git remote -v` | See remote names and fetch/push URLs. | Verify `origin` and `upstream` before pushing. |
| `git log --oneline -5` | Inspect five recent commits. | `-10` shows ten. |
| `git log --oneline --graph --all` | Visualize commits and branches. | Useful when learning merges. |
| `git branch` | List local branches and see the active `*` branch. | A branch is not a separate folder. |
| `git diff` | Inspect unstaged changes. | Review before staging. |
| `git diff --staged` | Inspect changes staged for the next commit. | Also written `git diff --cached`. |
| `git diff -- FILE` | Inspect unstaged changes in one file. | Example: `git diff -- Contributors.md`. |

## 3. Command reference: branches, commits and merges

| Command | Use case | Important note |
|---|---|---|
| `git branch -M main` | Rename the current branch to `main`, commonly during initial setup. | `-M` forces the rename; don't use casually in existing shared repos. |
| `git switch -c feature/name` | Create and switch to a feature branch. | Start from an updated `main`. |
| `git switch main` | Return to your local main branch. | Commit or safely handle existing edits first. |
| `git add FILE` | Stage a specific file. | Preferred for focused open-source PRs. |
| `git add .` | Stage all changes under the current directory. | Inspect `git status` first to avoid unrelated files. |
| `git commit -m "Message"` | Save staged changes as a commit. | Write a clear description of what changed. |
| `git merge feature/name` | Integrate a feature branch into your currently checked-out branch. | Check the current branch before running. |
| `git merge --ff-only upstream/main` | Advance local `main` to upstream's latest commit, if fast-forward is possible. | Stop and investigate if branches diverged. |
| `git branch -d feature/name` | Delete a fully merged local branch. | Only after checking the PR/branch is no longer needed. |

## 4. Command reference: remotes, fetching and pushing

| Command | Use case | Important note |
|---|---|---|
| `git remote add origin URL` | Connect a newly initialized personal project to GitHub. | Cloned repositories already have `origin`. |
| `git remote add upstream URL` | Connect your local fork clone to the original project. | Usually done once per clone. |
| `git fetch upstream` | Download original project's latest commits and branch references. | Does **not** merge into your working branch. |
| `git push -u origin main` | First push of a new personal project's main branch. | `-u` sets upstream tracking for later `git push`. |
| `git push -u origin feature/name` | First push of a new feature branch to your fork or own repo. | Opens the way to create a PR on GitHub. |
| `git push origin main` | Update your GitHub fork's main after syncing locally. | Do not push to original project unless authorized. |
| `git pull origin main` | Update local main from your own remote main. | Run on `main`; pull may merge or rebase depending on configuration. |

**Subtle distinction:** In `git push -u origin feature/name`, `-u` sets a *tracking relationship*. The remote conventionally called `upstream` is a different concept.

## 5. Personal-project workflow

### A. Start a new local project and publish it

```bash
mkdir my-python-project
cd my-python-project
git init
git branch -M main
# Create your files, then inspect them
git status
git add README.md
git commit -m "Initial project setup"
# Create an EMPTY repository on GitHub first; then:
git remote add origin https://github.com/YOUR-USERNAME/my-python-project.git
git push -u origin main
```

If you already cloned a GitHub repository, **skip** `git init` and `git remote add origin`.

### B. Everyday personal development

```bash
git switch main
git status
git switch -c feature/add-model-evaluation
# Edit files and run your own checks
git diff
git add model.py
git diff --staged
git commit -m "Add model evaluation"
git push -u origin feature/add-model-evaluation
```

Open a PR on your own GitHub repository if you want to practice review. After merging it on GitHub:

```bash
git switch main
git pull origin main
git status
```

Alternatively, for a simple local-only branch you can merge it yourself with `git switch main` followed by `git merge feature/add-model-evaluation`, then push `main`.

## 6. Open-source workflow: fork → PR

1. Read the project's README, `CONTRIBUTING.md`, issue discussion, and testing instructions. Confirm the task is useful and not already being handled.
2. Fork the **original repository** on GitHub.
3. Clone **your fork**, not the original repository, if using the usual fork-based workflow.
4. Add the original as `upstream`.
5. Synchronize your local `main` if necessary; create a new feature branch from it.
6. Make a focused change; inspect your diff and run the relevant tests.
7. Commit and push the feature branch to **origin**.
8. Open a PR with **base = original repository's target branch** and **compare/head = your fork's feature branch**.
9. Respond to feedback by updating the same feature branch and pushing additional commits.

Example setup:

```bash
git clone https://github.com/YOUR-USERNAME/PROJECT.git
cd PROJECT
git remote add upstream https://github.com/ORIGINAL-OWNER/PROJECT.git
git remote -v
git switch main
git status
git fetch upstream
git merge --ff-only upstream/main
git push origin main
git switch -c fix/small-bug
# Make your change and run the project's required checks
git diff
git add path/to/changed_file.py
git diff --staged
git commit -m "Fix small bug"
git push -u origin fix/small-bug
```

**Do not blindly run this template** if your project uses a different default branch, you have uncommitted changes, or `--ff-only` fails. Stop and inspect first.

## 7. Synchronize your fork (Day 5)

In your **fork clone**, with a clean working tree:

```bash
git switch main
git fetch upstream
git log --oneline -5 upstream/main
git merge --ff-only upstream/main
git push origin main
git status
```

- `fetch`: downloads commits/references; leaves current branch unchanged.
- `merge --ff-only`: moves local `main` forward only if histories allow it.
- `push origin main`: updates the `main` branch on your GitHub fork.
- If fast-forward fails, **do not** force-push or reset hard as a guess. Check the histories and ask for help.

## 8. Fixing an accidental formatting change (Day 4 lesson)

Before committing, inspect `git diff -- FILE`. If a formatter changed hundreds of unrelated lines, disable format-on-save for that file type and restore **only the intended file** from a known clean revision, then reapply your intended edit. `git restore` overwrites selected content; inspect `git status` and preserve any work you want to keep first.

For example, **only if your feature branch's `Contributors.md` should match local `main` except for your new entry**:

```bash
git status
git restore --source=main --staged --worktree -- Contributors.md
# Re-add only your entry and save without autoformatting
git add Contributors.md
git diff --staged
```

If the bad formatting was already committed, a new corrective commit on the same branch can update the existing PR. Review the **final PR's Files changed tab**: it should show only the intended change.

## 9. Common mistakes and safer habits

- **Pushing to the wrong remote:** run `git remote -v` before the first push.
- **Merging into the wrong branch:** run `git branch` before `git merge`.
- **Committing formatter noise:** run `git diff` and `git diff --staged`.
- **Adding unrelated files:** prefer `git add FILE` over `git add .` in external contributions.
- **Working directly on main:** use one feature branch per distinct issue/PR.
- **Assuming a PR merged automatically:** check its GitHub status; maintainers decide.
- **Assuming fetch updated your files:** fetch alone does not merge.
- **Deleting branches too early:** confirm the work is merged and no longer needed.
- **Using destructive commands to fix confusion:** avoid `git reset --hard`, `git push --force`, and `git branch -D` until you understand their effects.

## 10. GitHub actions that are not Git commands

| GitHub action | When you use it |
|---|---|
| **Fork** | Create your copy of an external project on GitHub. |
| **Issues → search labels** | Find a scoped task; `good first issue` is a starting point, not a guarantee of ease. |
| **Compare & pull request** | Propose your pushed feature branch for review. |
| **Files changed** | Verify the PR contains only the intended edits. |
| **Merge pull request** | Integrate an approved PR where you have permission (e.g., your personal repo). |
| **Review comments** | Understand and address maintainer feedback on the existing PR. |

## 11. Quick decision guide

**My own new project?** `git init` → commit → add `origin` → push.  
**Existing personal GitHub project?** `git clone` → branch → commit → push → optional PR.  
**Someone else's open-source project?** Read guidelines/issues → fork → clone your fork → add upstream → branch → change/test → push to origin → PR to original.  
**Original project has new commits?** On clean local `main`: fetch upstream → fast-forward merge if possible → push origin main.  
**My PR shows hundreds of unexpected edits?** Inspect diffs and formatting before pushing more changes.

## References

- GitHub contributing guide: https://docs.github.com/en/get-started/exploring-projects-on-github/contributing-to-open-source
- GitHub pull request quickstart: https://docs.github.com/en/pull-requests/get-started/pull-request-quickstart
- GitHub issues: https://docs.github.com/en/issues/tracking-your-work-with-issues/learning-about-issues/about-issues
- Git reference: https://git-scm.com/docs
