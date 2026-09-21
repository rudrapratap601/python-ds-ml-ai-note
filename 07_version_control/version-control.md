# Git, GitHub, GitLab, and Open Source for ML

> **Purpose:** A command-and-workflow reference for version control, reproducible ML projects, and useful open-source contributions.

## Reading order

1. [Git Fundamentals and Command Reference](#1-git-fundamentals-and-command-reference)
2. [Branching, Merging, Conflicts, and Recovery](#2-branching-merging-conflicts-and-recovery)
3. [GitHub and GitLab Collaboration](#3-github-and-gitlab-collaboration)
4. [Version Control for an ML Journey](#4-version-control-for-an-ml-journey)
5. [Open-Source Contributions and GSoC Preparation](#5-open-source-contributions-and-gsoc-preparation)

## How to use this guide

Commands are grouped by their effect and use case, not as one script to run from top to bottom. Replace placeholders such as `OWNER`, `PROJECT`, `COMMIT`, and paths. Inspect the actual default branch instead of assuming every project uses `main`.

This guide covers everyday commands and important advanced tools. Git, GitHub CLI, and GitLab CLI have additional commands and many version-specific options; use `git help -a`, `git help COMMAND`, `gh COMMAND --help`, and `glab COMMAND --help` for the complete installed interface.

Practice history-changing or destructive commands in a disposable repository. Hosted commands can publish branches, issues, comments, or reviews; read the effect column before using them. Official references were checked on 2026-09-21. Check current documentation for quotas, authentication policy, and GSoC rules/deadlines.

---

## 1. Git Fundamentals and Command Reference

> **Purpose:** Understand Git's state model and choose commands by their effect and use case. This covers everyday and important advanced commands; use Git's built-in help for the complete command and option catalog.

### Contents

- [Git versus hosting platforms](#git-versus-hosting-platforms)
- [The working tree, index, and history](#the-working-tree-index-and-history)
- [Setup and authentication](#setup-and-authentication)
- [Starting or cloning a repository](#starting-or-cloning-a-repository)
- [Inspecting changes](#inspecting-changes)
- [Staging and committing](#staging-and-committing)
- [History and revisions](#history-and-revisions)
- [Branches and remotes](#branches-and-remotes)
- [Tags and releases](#tags-and-releases)
- [Ignore rules and attributes](#ignore-rules-and-attributes)
- [Advanced command map](#advanced-command-map)
- [Everyday workflow and practice](#everyday-workflow-and-practice)

### Git versus hosting platforms

**Git** is a distributed version-control system. **GitHub** and **GitLab** host repositories and provide review, issue tracking, automation, and collaboration. You can commit locally without internet access; pushing and hosted collaboration need remote access.

| Term | Meaning |
|---|---|
| Repository | Object database, refs, configuration, and usually a working tree |
| Commit | A snapshot plus parents and metadata |
| Branch | A movable reference to a commit |
| HEAD | Usually the current branch reference, or a detached commit |
| Remote | A named repository URL, such as `origin` |
| Remote-tracking branch | Local record of a fetched remote branch, such as `origin/main` |
| Fork | Hosting-platform copy/relationship, normally under another account |
| Clone | Local repository copied from an existing repository |
| Pull request / merge request | Hosted proposal to review and integrate changes |

Git stores history as a directed acyclic graph. Commits normally have one parent; merge commits have multiple parents. Branches are not separate copies of every file.

### The working tree, index, and history

```text
edit files → working tree
git add   → staging area / index
git commit → local commit history
git push  → remote refs and required objects
```

The index describes the next snapshot to commit. A tracked file can have both staged and unstaged edits if you change it after staging. Untracked files are not automatically committed; ignored files are normally omitted from untracked listings and ordinary adds.

Three useful comparisons:

| Command | Comparison |
|---|---|
| `git diff` | Working tree versus index |
| `git diff --staged` | Index versus HEAD |
| `git diff HEAD` | Working tree versus HEAD for tracked content |

Untracked file contents do not appear in ordinary diffs until included appropriately. Always inspect `git status` as well.

### Setup and authentication

Replace example identity values with your own. These commands configure commit attribution, not website login:

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "your-verified-address@example.com"
git config --global init.defaultBranch main
git config --list --show-origin
```

Use repository-local `git config user.email ...` when a project needs a different identity. A verified hosting-platform email or its supported private no-reply address controls account attribution. Do not put an access token in `user.email` or commit messages.

HTTPS typically uses a credential manager or suitable token flow; SSH uses an SSH key registered with the host. Keep private keys private and follow the host's current setup instructions. Git identity and remote authentication are separate systems.

| Command | Use case |
|---|---|
| `git config --get user.name` | Inspect effective author name |
| `git config --local user.email "address"` | Set identity only for this repository |
| `git config --global core.editor "code --wait"` | Use VS Code for messages if its CLI is installed |
| `git help COMMAND` | Read full command documentation |
| `git COMMAND -h` | Get a short option summary |
| `git help -a` | Discover the installed command catalog |
| `git help -g` | List conceptual guides |

The [Git manual](https://git-scm.com/docs/git) is the complete reference. CLI versions and platform builds can differ; local `-h` output reflects what is installed.

### Starting or cloning a repository

```bash
git init -b main
git clone https://github.com/OWNER/REPOSITORY.git
git clone https://gitlab.com/GROUP/PROJECT.git
```

Run `init` inside a new project that is not already part of another repository. Run `clone` from the parent directory where the new checkout should be created. Do not run both as a routine sequence for the same project.

| Command | When to use | Detail |
|---|---|---|
| `git clone URL DIRECTORY` | Choose a local destination name | Destination must be suitable for cloning |
| `git clone --branch BRANCH URL` | Start on a particular branch/tag | A tag checkout may be detached |
| `git clone --depth 1 URL` | Small shallow history for a narrow task | Older history and some merge/bisect operations are unavailable |
| `git fetch --unshallow` | Expand a shallow clone | Requires access to available full history |
| `git clone --filter=blob:none URL` | Partial clone when the server supports it | Missing file objects may be fetched later |

### Inspecting changes

| Command | Use case |
|---|---|
| `git status` | Understand current branch, staging, and untracked files |
| `git status --short --branch` | Compact status during daily work |
| `git diff --stat` | Summarize changed tracked files |
| `git diff -- path/to/file` | Review an individual file |
| `git diff --staged --check` | Find whitespace errors in staged changes |
| `git diff main...HEAD` | Review branch changes from the merge base with main |
| `git show HEAD` | Inspect latest commit and patch |
| `git ls-files` | List tracked paths |
| `git ls-files --others --exclude-standard` | List untracked, nonignored paths |

The `--` separates revisions/options from paths. Quote paths with spaces. In PowerShell, quote revision syntax containing special characters, for example `"HEAD@{1}"`.

### Staging and committing

```bash
git add README.md
git diff --staged
git commit -m "Document model evaluation procedure"
```

| Command | Effect | When to use |
|---|---|---|
| `git add PATH` | Stage current content at a path | Select related files deliberately |
| `git add -p` | Interactively stage hunks | Separate unrelated edits into clean commits |
| `git add -u` | Stage modifications/deletions of tracked files | Exclude new untracked files |
| `git add -A` | Stage additions, modifications, and deletions | Only after reviewing the entire scope |
| `git restore --staged PATH` | Unstage to HEAD without discarding working edits | Remove a file from the next commit |
| `git commit` | Commit staged snapshot with editor message | Write a detailed explanation |
| `git commit -m "message"` | Commit staged snapshot with short message | One focused change |
| `git commit -a` | Stage/commit modifications and deletions of tracked files | Does not add new files; review first |
| `git commit --amend` | Replace the last commit | Correct a private, unshared commit |
| `git mv OLD NEW` | Rename and stage | Rename tracked files |
| `git rm PATH` | Remove tracked file and stage deletion | Delete intentionally |
| `git rm --cached PATH` | Remove from index, keep working file | Stop tracking a local artifact |

Git detects renames from content similarity; it does not require a special permanent rename object. A commit message should explain the change and why it matters. Avoid bundling formatting, unrelated fixes, and new behavior in one review unit.

Amending changes the commit identity. For published commits, coordinate before rewriting history. See [recovery and branching](#2-branching-merging-conflicts-and-recovery).

### History and revisions

| Command | Use case |
|---|---|
| `git log --oneline --graph --decorate --all` | Understand branch topology |
| `git log -n 5` | Read recent commit metadata |
| `git log --follow -- PATH` | Follow one file's history through suitable renames |
| `git log -S "function_name" -- PATH` | Find changes in occurrences of a string |
| `git log -G "pattern" -- PATH` | Find patches matching a regex |
| `git blame -L 10,30 PATH` | Find commits responsible for a line range |
| `git show COMMIT:PATH` | Inspect file content at a commit |
| `git rev-parse HEAD` | Record the exact current commit ID |
| `git rev-parse --show-toplevel` | Locate repository root |

`HEAD~2` follows the first-parent chain two generations. `HEAD^2` selects the second parent of a merge commit. `A..B` in `log` means commits reachable from B but not A; `A...B` in `log` means the symmetric difference. `git diff A...B` instead compares the merge base to B, so dots are not one universal operation across commands.

Use blame to understand context, not assign personal fault. Read the commit and surrounding discussion before changing behavior.

### Branches and remotes

| Command | Use case |
|---|---|
| `git branch` | List local branches |
| `git branch -a` | Include remote-tracking branches |
| `git branch -vv` | Inspect upstream tracking and divergence |
| `git switch -c feature/topic` | Create and switch to a branch |
| `git switch main` | Switch to an existing branch |
| `git switch --track origin/feature/topic` | Create a local tracking branch |
| `git branch -m NEW_NAME` | Rename current local branch |
| `git branch -d FINISHED_BRANCH` | Delete a branch subject to merge checks |
| `git remote -v` | Inspect fetch and push URLs |
| `git remote add upstream URL` | Add the original repository beside a fork |
| `git remote set-url origin URL` | Change a remote destination |
| `git fetch origin` | Update local knowledge without integrating into your branch |
| `git fetch --prune origin` | Also remove stale remote-tracking refs |
| `git pull --ff-only` | Fetch and integrate only if fast-forward is possible |
| `git push -u origin feature/topic` | Publish a branch and set its upstream |
| `git push` | Push according to configured tracking/push rules |

`origin` and `upstream` are naming conventions, not special server roles. A branch's upstream tracking configuration is a separate concept from a remote named `upstream`.

Fetching does not normally alter your current working files. Pull combines fetching with integration, according to options/configuration. Inspect divergence before choosing merge or rebase.

### Tags and releases

```bash
git tag -a v0.1.0 -m "First reproducible baseline"
git show v0.1.0
git push origin v0.1.0
```

An annotated tag includes metadata and a message. A lightweight tag is simply a ref. A hosting-platform release adds descriptions/assets around a tag; it is not the same object as the Git tag.

`git tag --list` lists tags. `git describe --tags --always` creates a human-readable revision description. `git tag -s` and `git commit -S` sign objects when signing is configured. Do not move an already published release tag casually; consumers may rely on its identity.

### Ignore rules and attributes

Example `.gitignore` for a Python/ML project; adapt dataset/artifact rules to your chosen storage policy:

```gitignore
__pycache__/
*.py[cod]
.venv/
.pytest_cache/
.mypy_cache/
.ruff_cache/
.ipynb_checkpoints/
.env
.env.*
!.env.example
data/raw/
artifacts/
```

Ignore rules do not remove already tracked files or erase previous commits. Use `git check-ignore -v PATH` to diagnose the matching rule. Keep small intentional test fixtures under a clearly permitted path.

`.gitattributes` controls attributes such as line-ending normalization and LFS filters. A cross-platform project may use:

```gitattributes
* text=auto
*.py text eol=lf
*.md text eol=lf
*.sh text eol=lf
*.png binary
```

Renormalizing existing tracked files can produce a large diff, so do it as a deliberate separate change. Do not blindly change global `core.autocrlf` settings for every repository.

### Advanced command map

| Command family | Use case |
|---|---|
| `git stash` | Temporarily shelve unfinished local changes |
| `git cherry-pick COMMIT` | Apply a specific commit to the current branch |
| `git revert COMMIT` | Create a new commit reversing an earlier change |
| `git rebase BASE` | Replay branch commits onto a new base |
| `git reflog` | Find recent local ref movements for recovery |
| `git bisect` | Locate a regression by binary search over history |
| `git worktree` | Multiple checkouts sharing one repository object store |
| `git submodule` | Track another repository at a specific commit |
| `git sparse-checkout` | Limit populated working-tree paths |
| `git archive` | Export a snapshot without repository history |
| `git bundle` | Transfer Git objects/refs as a file |
| `git format-patch`, `git am` | Exchange commits through patch workflows |
| `git apply --check PATCH` | Check whether a patch applies without applying it |
| `git range-diff OLD_RANGE NEW_RANGE` | Compare two versions of a commit series |
| `git fsck` | Inspect object connectivity/integrity |
| `git count-objects -vH` | Inspect object-storage usage |
| `git maintenance`, `git gc` | Repository housekeeping; rarely first-line troubleshooting |
| `git cat-file`, `ls-tree`, `rev-list`, `for-each-ref` | Inspect Git objects/refs in advanced tooling |

Most daily work uses the high-level “porcelain” commands. Low-level “plumbing” commands are useful for understanding and automation, but avoid directly modifying refs or object storage without a clear need.

### Everyday workflow and practice

```bash
git status
git switch main
git pull --ff-only
git switch -c docs/improve-readme
# Edit and validate the intended files.
git diff
git add README.md
git diff --staged
git commit -m "Clarify local setup instructions"
git push -u origin docs/improve-readme
```

This assumes a clean starting state, a branch named `main`, an upstream configuration, and permission to push to `origin`. Adapt those assumptions instead of copying commands blindly.

Practice in a disposable repository: stage half a file with `add -p`; inspect staged versus unstaged diffs; create two branches and compare their histories; tag a known-good snapshot; explain what each command changes in the working tree, index, local refs, and remote refs.

Next: [Branching, merging, and recovery](#2-branching-merging-conflicts-and-recovery).

---

## 2. Branching, Merging, Conflicts, and Recovery

> **Purpose:** Integrate work deliberately and recover from common mistakes without losing unrelated edits. Commands below are a reference, not a script to run all at once.

### Contents

- [Merge versus rebase](#merge-versus-rebase)
- [Resolving conflicts](#resolving-conflicts)
- [Undo decision table](#undo-decision-table)
- [Reset modes](#reset-modes)
- [Stash](#stash)
- [Cherry-pick and interactive rebase](#cherry-pick-and-interactive-rebase)
- [Reflog and recovery](#reflog-and-recovery)
- [Worktrees, bisect, and submodules](#worktrees-bisect-and-submodules)
- [Troubleshooting](#troubleshooting)

### Merge versus rebase

```text
Before:
      D---E  feature
     /
A---B---C    main

Merge main into feature:
      D---E---M  feature
     /      /
A---B------C    main

Rebase feature onto main:
A---B---C---D'---E'  feature
```

A merge preserves existing commits and may create a merge commit. A rebase replays changes, creating new commit identities. Both can require conflict resolution. Use the project's policy and consider whether other people already depend on the branch history.

```bash
git fetch origin
git switch feature/topic
git merge origin/main
```

Or, for a branch whose history you may rewrite:

```bash
git fetch origin
git switch feature/topic
git rebase origin/main
```

A fast-forward moves a branch pointer when no divergent integration is needed. `merge --ff-only` refuses divergent history. `merge --no-ff` requests a merge commit even when a fast-forward is possible. Squash merging produces the combined change without recording the original branch as a merge parent.

If an authorized rebase rewrites a published personal branch, a subsequent ordinary push may be rejected. `git push --force-with-lease origin feature/topic` adds a check against your expected remote state, but it can still overwrite history and depends on what your local remote-tracking ref says. Coordinate and inspect first; never substitute plain `--force` as a routine fix. See [push semantics](https://git-scm.com/docs/git-push).

### Resolving conflicts

1. Read `git status` to identify the operation and conflicted files.
2. Understand both intended changes and the surrounding behavior.
3. Edit the file into the correct combined result; remove conflict markers.
4. Run relevant checks.
5. Stage each resolved file.
6. Continue the operation with its matching command.

```text
<<<<<<< HEAD
one version
=======
another version
>>>>>>> incoming-branch
```

| In-progress operation | Continue after resolving/staging | Abort |
|---|---|---|
| Merge | `git merge --continue` | `git merge --abort` |
| Rebase | `git rebase --continue` | `git rebase --abort` |
| Cherry-pick | `git cherry-pick --continue` | `git cherry-pick --abort` |
| Revert sequence | `git revert --continue` | `git revert --abort` |

`--skip` discards the current replayed change in operations that support it; it is not a generic conflict fix. “Ours” and “theirs” refer to operation-specific sides, and rebase can make those labels surprising. Inspect content rather than choosing a side by name.

Abort recovery is more predictable when the operation began from a clean state. Save unrelated edits before beginning integration; do not assume an abort can reconstruct every preexisting local modification.

### Undo decision table

| Situation | Command or approach | What stays / what changes |
|---|---|---|
| Staged the wrong file | `git restore --staged PATH` | Working edits remain |
| Discard unstaged edits to a tracked file | `git restore PATH` | Replaces working file from index; discarded edits may be unrecoverable |
| Restore file from a known revision | `git restore --source=COMMIT -- PATH` | Overwrites working content for that path |
| Correct last unshared commit | `git commit --amend` | Replaces commit identity |
| Undo a shared commit | `git revert COMMIT` | Adds an inverse commit |
| Move branch back, keep all changes staged | `git reset --soft COMMIT` | Moves HEAD/branch; index and working tree stay |
| Move branch back, keep changes unstaged | `git reset --mixed COMMIT` | Resets index; working files stay |
| Recover a lost recent commit | `git reflog`, then `git branch rescue COMMIT` | Preserves the found commit with a new ref |
| Remove untracked files | Preview with `git clean -nd`, then only a deliberately scoped cleanup | Untracked files are not protected by commits |

Before an operation that discards data, inspect the exact paths and preserve wanted work. A Git commit cannot recover content that was never stored in Git. A backup branch preserves commits, not uncommitted working-tree changes.

### Reset modes

```text
reset --soft:   move branch/HEAD
reset --mixed:  move branch/HEAD + replace index
reset --hard:   move branch/HEAD + replace index + overwrite working tree
```

`git reset --hard COMMIT` discards tracked working changes and can remove obstructing untracked paths. It is a deliberate reset, not a general “fix Git” command. The [reset manual](https://git-scm.com/docs/git-reset) explains mode-specific behavior.

`restore` changes file content/index entries; `reset` can move the current branch; `revert` adds a commit. These are not interchangeable undo buttons.

### Stash

```bash
git stash push -u -m "WIP before review"
git stash list
git stash show -p "stash@{0}"
git stash apply "stash@{0}"
```

| Command | Meaning |
|---|---|
| `stash push` | Store tracked changes and reset them from the working state |
| `stash push -u` | Also include untracked files |
| `stash push -a` | Also include ignored files; inspect carefully |
| `stash apply` | Reapply without dropping the stash |
| `stash pop` | Apply, then remove on success; conflicts can retain it |
| `stash apply --index` | Attempt to restore staging state as well |
| `stash drop "stash@{N}"` | Delete a specific stash entry after verifying it is no longer needed |
| `stash branch BRANCH "stash@{N}"` | Recover work on a branch from its original base |

Stash is temporary local storage, not a durable cross-machine backup. Quote stash revision expressions in PowerShell. Avoid `stash clear` unless you intentionally want every entry removed.

### Cherry-pick and interactive rebase

`git cherry-pick COMMIT` applies a specific commit's change on the current branch, normally creating a new commit. It is useful for a focused backport, but repeated cherry-picking across branches can complicate later integration.

`git rebase -i BASE` opens an editable plan for commits after the base. Common actions include `pick`, `reword`, `edit`, `squash`, `fixup`, and `drop`. Keep a recovery reference before a substantial private-history cleanup.

```bash
git branch backup/before-cleanup
git rebase -i HEAD~3
```

This assumes at least three relevant commits and authorization to rewrite them. `fixup!` commits plus `--autosquash` help fold review corrections into their intended commits. The [rebase manual](https://git-scm.com/docs/git-rebase) documents advanced cases, including preserving merges.

### Reflog and recovery

```bash
git reflog --date=local
git show COMMIT
git branch rescue/recovered-work COMMIT
```

Replace `COMMIT` with a verified ID from the reflog. Creating a branch is a conservative first step because it preserves the object without resetting current work.

Reflogs are local and expire; they are not part of ordinary clone/push history. Garbage collection can eventually remove unreachable objects. Recover promptly, and do not assume another clone has the same reflog.

A detached HEAD means HEAD points directly at a commit. It is useful for inspection. If you make work worth keeping there, `git switch -c rescue/my-work` names it before moving elsewhere.

### Worktrees, bisect, and submodules

#### Worktrees

```bash
git worktree list
git worktree add -b fix/urgent ../project-urgent main
```

A linked worktree gives another checkout sharing the repository's object storage. It is useful for reviewing or fixing another branch without stashing current work. A branch normally cannot be checked out in multiple worktrees simultaneously. Remove a finished clean checkout using `git worktree remove PATH`, not an arbitrary recursive deletion.

#### Bisect

```bash
git bisect start
git bisect bad
git bisect good KNOWN_GOOD_COMMIT
# Test the checked-out revision, then mark it:
git bisect good
# Or: git bisect bad
git bisect reset
```

`git bisect run COMMAND` automates the test when exit codes follow the documented contract; code 125 skips an untestable commit. The test must be reproducible and detect the regression, not unrelated environmental failures.

#### Submodules and sparse checkout

`git submodule update --init --recursive` initializes required nested repositories at recorded commits. A submodule change is a pointer update in the parent; commit and publish the inner work before updating the parent pointer. Submodules are not ordinary folders with automatically synchronized branches.

`git sparse-checkout init --cone` and `git sparse-checkout set PATHS` limit populated working-tree paths in large repositories. This is separate from shallow history and partial object fetching.

### Troubleshooting

| Symptom | First checks |
|---|---|
| Push rejected as non-fast-forward | Fetch, inspect divergence, integrate intentionally |
| Authentication denied | Remote URL, active account/key, token scopes, repository access |
| File ignored unexpectedly | `git check-ignore -v PATH` |
| Secret committed | Revoke/rotate it immediately, then coordinate history cleanup and cache removal |
| Wrong branch | Inspect status; preserve work before switching/cherry-picking |
| Case-only rename fails on Windows | Use a temporary intermediate name with `git mv` |
| Huge binary bloats history | Choose artifact/LFS storage and plan migration; deletion in a later commit does not remove history |
| Merge shows whole-file changes | Check line endings, formatting, and attributes |

Practice in a disposable repository: create and resolve a conflict, abort a rebase, recover a detached commit with a branch, compare reset modes, and bisect a deliberately introduced failure.

Next: [GitHub and GitLab collaboration](#3-github-and-gitlab-collaboration).

---

## 3. GitHub and GitLab Collaboration

> **Purpose:** Use hosted repositories, forks, issues, pull/merge requests, reviews, and CI from the browser or command line. Commands are examples; replace placeholder names and numbers before use.

### Contents

- [Platform concepts](#platform-concepts)
- [Fork and upstream workflow](#fork-and-upstream-workflow)
- [GitHub CLI command reference](#github-cli-command-reference)
- [GitLab CLI command reference](#gitlab-cli-command-reference)
- [Writing a useful review request](#writing-a-useful-review-request)
- [Responding to review](#responding-to-review)
- [CI and permissions](#ci-and-permissions)
- [Releases and repository hygiene](#releases-and-repository-hygiene)

### Platform concepts

| Concept | GitHub | GitLab |
|---|---|---|
| Proposed code change | Pull request (PR) | Merge request (MR) |
| Work item | Issue | Issue / related planning tools |
| Automation | Actions workflows | CI/CD pipelines |
| CLI | `gh` | `glab` |
| Repository owner scope | User / organization | User / group / subgroup |
| Contribution copy | Fork | Fork |
| Release | Tag plus hosted release metadata/assets | Tag plus hosted release metadata/assets |

Installing Git does not install `gh` or `glab`. Authenticate the relevant CLI with the intended account and host. A GitLab self-managed host or GitHub Enterprise host can have different policies and feature availability.

An issue describes a problem or proposal. A PR/MR proposes a branch diff for review. A discussion is often more suitable for open-ended questions. Follow the project's contribution instructions before opening a new item.

### Fork and upstream workflow

The usual open-source arrangement is `origin` for your fork and `upstream` for the original project:

```bash
git clone https://github.com/YOUR_ACCOUNT/PROJECT.git
cd PROJECT
git remote add upstream https://github.com/ORIGINAL_OWNER/PROJECT.git
git fetch upstream
git switch -c fix/clear-error-message upstream/main
# Edit and run the project's checks.
git add path/to/changed-file.py
git commit -m "Clarify the error for invalid input"
git push -u origin fix/clear-error-message
```

Replace `main` with the project's actual contribution base. Starting the feature branch from fetched upstream avoids basing it on an outdated fork default branch. Each unrelated contribution should normally have its own branch.

Create the PR/MR with the original project as target and your feature branch as source. Inspect the diff and commit list to ensure unrelated changes are absent. GitHub's [contribution guide](https://docs.github.com/en/get-started/exploring-projects-on-github/contributing-to-open-source) describes the hosted workflow; GitLab documents [merge requests](https://docs.gitlab.com/user/project/merge_requests/).

### GitHub CLI command reference

| Command | When to use | Effect |
|---|---|---|
| `gh --version` | Check installation | Read-only |
| `gh auth login` | Authenticate interactively | Stores/configures credentials |
| `gh auth status` | Check active authentication | Read-only; do not expose tokens |
| `gh repo view OWNER/REPO` | Inspect project metadata | Read-only |
| `gh repo clone OWNER/REPO` | Create local checkout | Writes local files |
| `gh repo fork OWNER/REPO --clone` | Fork and clone for contribution | Creates hosted fork and local files |
| `gh issue list --repo OWNER/REPO` | Find reported work | Read-only |
| `gh issue view 123 --repo OWNER/REPO` | Read an issue | Read-only |
| `gh issue create --repo OWNER/REPO` | Report a verified problem | Publishes an issue |
| `gh pr list --repo OWNER/REPO` | Check open changes and avoid duplication | Read-only |
| `gh pr view 123 --repo OWNER/REPO` | Read a PR and status | Read-only |
| `gh pr diff 123 --repo OWNER/REPO` | Inspect the patch | Read-only |
| `gh pr checkout 123` | Check out a PR locally | Changes local checkout |
| `gh pr create --draft` | Open an early reviewable proposal | Publishes a draft PR |
| `gh pr checks 123` | Inspect CI results | Read-only |
| `gh pr ready 123` | Mark draft ready for review | Changes hosted PR state |
| `gh pr review 123 --comment --body-file review.md` | Submit review feedback | Publishes a review |
| `gh pr comment 123 --body-file comment.md` | Add discussion | Publishes a comment |
| `gh pr merge 123 --squash` | Integrate an approved PR when authorized | Changes target branch |
| `gh run list` | Find workflow runs | Read-only |
| `gh run view RUN_ID --log-failed` | Diagnose failed CI | Read-only logs |
| `gh run watch RUN_ID` | Follow a running workflow | Read-only monitoring |
| `gh release list` | Inspect released versions | Read-only |

Example with explicit source/target and a multiline description stored in a file:

```bash
gh pr create --repo ORIGINAL_OWNER/PROJECT --base main --head YOUR_ACCOUNT:fix/clear-error-message --draft --title "Clarify invalid-input errors" --body-file pr-description.md
```

The [PR creation reference](https://cli.github.com/manual/gh_pr_create) documents flags and fork behavior. `gh repo fork` can configure remotes; inspect `git remote -v` after using it instead of assuming names. See [fork options](https://cli.github.com/manual/gh_repo_fork).

`gh api` accesses GitHub APIs; the HTTP method and endpoint determine whether it reads or mutates data. Do not treat every CLI query as read-only. Use `gh help` and command-specific `--help` for complete options.

### GitLab CLI command reference

| Command | When to use | Effect |
|---|---|---|
| `glab --version` | Check installation | Read-only |
| `glab auth login` | Authenticate with the intended GitLab host | Stores/configures credentials |
| `glab auth status` | Verify active authentication | Read-only |
| `glab repo clone GROUP/PROJECT` | Clone a project | Writes local checkout |
| `glab repo fork GROUP/PROJECT` | Fork a contribution project | Creates hosted fork |
| `glab issue list` | Find project issues | Read-only |
| `glab issue view 123` | Read issue details | Read-only |
| `glab issue create` | Report a verified problem | Publishes an issue |
| `glab mr list` | Inspect open merge requests | Read-only |
| `glab mr view 123` | Read an MR | Read-only |
| `glab mr diff 123` | Review code changes | Read-only |
| `glab mr checkout 123` | Inspect MR locally | Changes local checkout |
| `glab mr create --draft` | Create a draft MR | Publishes an MR |
| `glab mr note 123 --message "Review comment"` | Add discussion | Publishes a comment |
| `glab mr approve 123` | Record an approval when authorized | Changes review state |
| `glab mr merge 123` | Merge an approved change when authorized | Changes target branch |
| `glab ci status` | Inspect current pipeline status | Read-only |
| `glab ci view` | Inspect pipeline jobs interactively | Read-only |
| `glab ci trace JOB_ID` | Read a job log | Read-only |

In the intended project checkout, a typical creation command is:

```bash
glab mr create --source-branch fix/clear-error-message --target-branch main --draft --title "Clarify invalid-input errors" --description "Explain the problem, change, and validation here."
```

For cross-project forks, explicitly verify the target project using the CLI prompts/options documented by [glab mr create](https://docs.gitlab.com/cli/mr/create/). Hosts, permissions, and fork setup vary. The [fork reference](https://docs.gitlab.com/cli/repo/fork/) and `glab COMMAND --help` describe installed options.

### Writing a useful review request

A good PR/MR description lets a reviewer understand the problem without reading your private notes:

```markdown
## Problem
Describe the concrete input or workflow that currently fails.

## Change
Describe the resulting behavior and important design choices.

## Validation
- State the actual checks run and their results.
- Include a minimal before/after example when helpful.

## Related issue
Link the issue using the project's preferred syntax.
```

Use the repository's template when present. Link an issue with closing keywords only when the change actually resolves it and the target branch/workflow supports that behavior. A draft is useful when the work is reviewable but intentionally unfinished.

### Responding to review

Read the feedback, reproduce concerns, update the same feature branch, and push follow-up commits. The existing PR/MR updates automatically. Explain material design changes and newly run checks.

Do not open a second PR for every revision or mark discussions resolved without addressing the concern. If you disagree, discuss the technical tradeoff with evidence. Avoid rewriting shared branch history unless the project asks for it or collaborators agree.

Review others' changes for correctness, tests, readability, compatibility, and scope. Distinguish blocking defects from suggestions. Do not approve work you have not assessed or claim tests you did not run.

### CI and permissions

CI checks execute on configured runners. A green run means those checks passed in that environment, not that all behavior is correct. Read failed logs before rerunning; a failure may be a real bug, a missing dependency, an environment problem, or a flaky test.

External fork contributions often have restricted secrets/tokens and may require maintainer approval to run workflows. Never work around those protections by giving untrusted code privileged credentials. Review workflow changes as executable code.

Protected branches can require approvals, successful checks, signed commits, or other policy. Having permission to push to a fork does not authorize bypassing the original project's review process.

### Releases and repository hygiene

Keep README setup steps, contribution instructions, a license, issue templates, and testing commands accurate. A public repository without a license does not automatically grant broad reuse rights. Follow the project's license and contributor agreement/sign-off requirements.

Use tags and release notes to identify tested snapshots. Attach appropriate build artifacts through the platform's release/artifact storage rather than committing every generated binary to Git.

Practice: fork a disposable project, open a draft change, inspect it through the CLI, push one review correction, and trace the CI run. Perform this practice in a repository where those hosted actions are intended.

Next: [Version control for ML](#4-version-control-for-an-ml-journey).

---

## 4. Version Control for an ML Journey

> **Purpose:** Make experiments reviewable and reproducible by versioning code, configuration, data references, environments, and results with the right tools.

### Contents

- [What belongs in Git](#what-belongs-in-git)
- [A practical project layout](#a-practical-project-layout)
- [Experiment provenance](#experiment-provenance)
- [Notebooks](#notebooks)
- [Git LFS and dataset storage](#git-lfs-and-dataset-storage)
- [DVC and experiment trackers](#dvc-and-experiment-trackers)
- [Branch and review workflow](#branch-and-review-workflow)
- [Continuous integration examples](#continuous-integration-examples)
- [Releases, secrets, and practice](#releases-secrets-and-practice)

### What belongs in Git

| Item | Typical storage choice | Reason |
|---|---|---|
| Python source and tests | Git | Reviewable code history |
| Small configuration files | Git | Explain experiment settings |
| Dependency manifests/lockfiles | Git | Recreate environments |
| Small synthetic test fixtures | Git | Fast reproducible tests |
| Large datasets | Versioned external storage / DVC / suitable LFS use | Size, licensing, access control |
| Model checkpoints | Artifact store or suitable LFS use | Large binaries and frequent versions |
| Metrics summaries | Small reviewed summaries in Git; full runs in tracker | Useful comparison without noisy history |
| Credentials and private records | Secrets/data systems, not Git | Access and lifecycle requirements |
| Virtual environments and caches | Ignore | Rebuildable local state |

Git tracks file content, not the meaning of an experiment. A clean commit is only one part of reproducibility.

### A practical project layout

```text
ml-project/
├── README.md
├── pyproject.toml
├── requirements.lock
├── configs/
│   └── baseline.toml
├── src/
│   └── project/
├── tests/
│   └── fixtures/
├── notebooks/
├── reports/
│   └── baseline-summary.md
├── data/                 # Managed according to a documented data policy
├── artifacts/            # Usually external or ignored
└── .gitignore
```

The lockfile name depends on your dependency manager; do not maintain conflicting lock formats without a reason. Document commands to obtain permitted data, run a small training example, evaluate, and reproduce a report.

### Experiment provenance

Record at least:

- Code commit ID and whether the working tree was dirty.
- Dataset version/checksum and the train/validation/test split definition.
- Feature-processing configuration fitted on training data only.
- Hyperparameters and random seeds.
- Python/library versions and relevant hardware/runtime settings.
- Metric definitions, direction, evaluation population, and uncertainty where appropriate.
- Artifact identifiers and a command/configuration that can reproduce the run.

Useful read-only commands:

```bash
git rev-parse HEAD
git status --porcelain
git describe --tags --always
python --version
python -m pip freeze
```

`pip freeze` captures installed packages but not every operating-system dependency, GPU driver, environment variable, or data version. A seed alone does not eliminate all nondeterminism. If the working tree is dirty, the commit ID by itself does not identify the exact code that ran.

### Notebooks

Notebook JSON can produce noisy diffs from cell IDs, outputs, execution counts, and embedded images. Keep reusable logic in source modules and use notebooks for exploration and explanation.

Before sharing, restart the kernel and run relevant cells in order, check hidden state, remove sensitive outputs, and follow the project's output-retention policy. A notebook that only works after an undocumented sequence of cells is not reproducible.

Tools such as notebook-aware diffing or paired text representations can help, but agree on one workflow with the project. Do not blindly strip outputs from a repository that intentionally reviews them as evidence.

### Git LFS and dataset storage

Git LFS stores small pointer files in Git and the large content in separate LFS storage. It must be installed and supported by the remote host. See [Git LFS](https://git-lfs.com/) and [GitHub's LFS guide](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage).

```bash
git lfs install
git lfs track "*.onnx"
git add .gitattributes
git add models/baseline.onnx
git commit -m "Track baseline model with Git LFS"
git lfs ls-files
git push
```

This assumes the model file exists, the project approves storing it in LFS, and the remote supports the required storage. Tracking a pattern updates `.gitattributes`; commit that file so collaborators get the same filters.

| Command | Use case |
|---|---|
| `git lfs version` | Check installed LFS client |
| `git lfs track` | Inspect tracked patterns |
| `git lfs ls-files` | Inspect files represented through LFS |
| `git lfs pull` | Download needed LFS content for the current checkout |
| `git lfs status` | Inspect pending LFS-related changes |

Tracking a file with LFS now does not remove older large blobs from history. Migration can rewrite history and requires coordination. Host quotas, bandwidth, and size limits change; check current host documentation before adopting a large-data workflow.

LFS is not automatically a dataset catalog, permission model, or experiment tracker. Restricted datasets may need access-controlled object storage and a documented retrieval process instead.

### DVC and experiment trackers

DVC can track data/artifact metadata and pipeline dependencies while storing large content in a separate configured remote. Git stores the small metadata files; the data remote stores the content. The [DVC getting-started guide](https://doc.dvc.org/start) explains this division.

An experiment tracker records parameters, metrics, run metadata, and artifacts. It complements Git rather than replacing source history. Link each run to code/data identifiers and keep artifact access policies explicit.

Choose the smallest stack that solves your current problem: a small learning project may need Git, a requirements file, a dataset checksum, and a metrics table before it needs a full experiment platform.

### Branch and review workflow

Use a branch for a coherent code/configuration change, not necessarily one long-lived branch per training run. Record run variations in configurations and tracking metadata so branches do not become an unmanageable experiment database.

```bash
git switch -c experiment/add-standardization
# Change source, configuration, and meaningful tests.
git diff
git add src/ configs/ tests/ reports/baseline-summary.md
git diff --staged
git commit -m "Fit feature standardization on training data only"
git push -u origin experiment/add-standardization
```

A useful ML review explains the baseline, changed assumptions, dataset/split, metrics, resource cost, and failure cases. Avoid claiming improvement from a single favorable random seed or from tuning repeatedly on the final test set.

### Continuous integration examples

CI should run fast deterministic checks: imports, unit tests, schema validation, small synthetic pipeline tests, and formatting/lint checks where the project uses them. Large GPU training is usually a separate controlled workflow.

This GitHub Actions example assumes `requirements.txt` and a `tests/` directory exist. Save it as `.github/workflows/tests.yml` in an actual code project after adapting dependencies and policies:

```yaml
name: Python tests
on: [push, pull_request]
permissions:
  contents: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: python -m pip install -r requirements.txt
      - run: python -m pip install pytest
      - run: python -m pytest -q
```

The action versions follow the checked [GitHub Python CI guide](https://docs.github.com/en/actions/tutorials/build-and-test-code/python). For production supply-chain controls, follow the project's policy for pinning approved action commit SHAs and maintaining updates.

A corresponding minimal GitLab `.gitlab-ci.yml` example, assuming a suitable runner, is:

```yaml
image: python:3.12
stages:
  - test
unit-tests:
  stage: test
  script:
    - python -m pip install -r requirements.txt
    - python -m pip install pytest
    - python -m pytest -q
```

Runner configuration, dependency caching, artifacts, and protected variables depend on the project. See the [GitLab pipeline tutorial](https://docs.gitlab.com/ci/quick_start/). These snippets are learning templates, not workflows installed in this notes repository.

### Releases, secrets, and practice

Tag a tested baseline, record artifact identifiers, and write release notes describing compatibility and reproduction steps. A model file without preprocessing/schema information may be unusable even if its weights are available.

Keep `.env` files, API keys, private data, and credentials out of commits. If a credential is exposed, revoke/rotate it first; merely deleting it in a new commit does not remove the older copy from history or external caches.

Practice: reproduce a result from a fresh clone using only documented inputs; create a small deterministic smoke test; track a permitted toy model through LFS; compare two runs with the same code but different data versions; write a report that includes both improvements and regressions.

Next: [Open source and GSoC preparation](#5-open-source-contributions-and-gsoc-preparation).

---

## 5. Open-Source Contributions and GSoC Preparation

> **Purpose:** Build practical contribution skills through useful work, clear communication, and a reproducible development workflow. Program-specific rules and dates must be checked for the relevant year.

### Contents

- [Choose a project](#choose-a-project)
- [Understand its contribution contract](#understand-its-contribution-contract)
- [A first-contribution workflow](#a-first-contribution-workflow)
- [Communicate with maintainers](#communicate-with-maintainers)
- [Build an ML contribution portfolio](#build-an-ml-contribution-portfolio)
- [GSoC preparation](#gsoc-preparation)
- [Proposal outline](#proposal-outline)
- [Sustained contribution habits](#sustained-contribution-habits)

### Choose a project

Start with a tool you use and a problem you can reproduce. For an ML journey, useful contributions include clearer error messages, dataset validation, numerical edge cases, tests, documentation examples, performance investigation, and supported interoperability.

Check project activity, contribution instructions, test setup, issue quality, and maintainer availability. A “good first issue” label is a starting signal, not a guarantee that the task is unclaimed or easy. Search existing issues and PRs before beginning duplicate work.

A documentation correction with verified behavior can be more valuable than a large unrequested feature. Choose work that matches the project's priorities and your ability to validate it.

### Understand its contribution contract

Read `README`, `CONTRIBUTING`, `CODE_OF_CONDUCT`, license terms, development setup, testing instructions, and relevant issue discussions. Some repositories also have directory-specific instructions.

Confirm formatting tools, supported Python versions, test subsets, changelog requirements, commit conventions, and any contributor license agreement or developer certificate of origin. `git commit -s` adds a sign-off line; use it only when you understand and can make the associated certification. It is different from cryptographic signing with `-S`.

Install the documented development environment and run a small baseline check before editing. Record preexisting failures separately from failures introduced by your change.

### A first-contribution workflow

1. Reproduce the issue with a minimal input and expected/actual behavior.
2. Check that the issue is still present on the intended branch.
3. Discuss scope if the project requires it or the change is substantial.
4. Fork/clone, fetch upstream, and create a focused branch.
5. Add a regression test that fails for the actual bug when appropriate.
6. Implement the smallest coherent fix and update relevant docs.
7. Run the project's required checks and inspect the full diff.
8. Open a clear PR/MR with evidence and a link to the issue.
9. Respond to review, update the same branch, and rerun affected checks.
10. After merge, sync your fork and choose the next useful contribution.

```bash
git fetch upstream
git switch -c fix/specific-problem upstream/main
# Reproduce, edit, test, and inspect.
git status
git diff
git add path/to/fix.py tests/test_regression.py
git diff --staged
git commit -m "Handle the documented edge case"
git push -u origin fix/specific-problem
```

Replace all paths and branch names with the real project values. Do not copy test commands from an unrelated project and call that complete validation.

### Communicate with maintainers

An effective question includes what you are trying to do, the relevant environment/version, the smallest reproduction, what you tried, and the specific point needing guidance. Keep logs focused and remove credentials/private records.

Ask about a proposed approach before implementing a large redesign. Give maintainers time to respond; repeated pings across channels add work. Follow public channels and the project's communication norms rather than privately contacting unrelated maintainers.

If you use AI assistance, verify every claim and line of code, follow the project's policy, and disclose assistance when required. You remain responsible for understanding the contribution and responding to review. Do not submit generated bulk changes you cannot explain.

### Build an ML contribution portfolio

Keep a concise record for each contribution:

| Field | What to record |
|---|---|
| Problem | Reproducible issue and why it mattered |
| Scope | Your actual work and collaboration |
| Evidence | Tests, benchmarks, examples, or documentation checks |
| Review | Important feedback and how you addressed it |
| Result | Merged, revised, declined, or still under discussion |
| Learning | Technical insight and next improvement |

A portfolio should show understanding and maintenance, not just a count of PRs. Useful ML work can include testing train/test leakage, handling NaNs consistently, improving sparse-data paths, documenting dtype behavior, or measuring performance with a fair benchmark.

For benchmarks, state hardware, versions, input sizes, warmup, repeated measurements, and correctness checks. Do not present one noisy timing as a general performance guarantee.

### GSoC preparation

Google Summer of Code is a mentored open-source program involving contributors and participating organizations. Eligibility, project sizes, schedules, and application rules are program-specific and can change. Check the current [GSoC overview](https://summerofcode.withgoogle.com/how-it-works/) and [official FAQ](https://developers.google.com/open-source/gsoc/faq) rather than relying on a copied deadline or assuming it is limited to a particular student category.

A useful preparation sequence is:

1. Identify a small number of organizations whose work you understand and want to use.
2. Read their current idea lists and contribution expectations.
3. Set up their development environment and explore relevant code.
4. Make useful, reviewable contributions and learn the review process.
5. Discuss a project idea with the designated mentors using the organization's process.
6. Write a scoped proposal with deliverables, milestones, risks, and testing.
7. Apply through the official process for the relevant year and continue contributing appropriately.

Previous contributions can demonstrate readiness, but they do not guarantee selection. Do not promise availability or technical outcomes you cannot realistically deliver. Account for exams, work, time zones, and other commitments in your plan.

### Proposal outline

```markdown
# Project title

## Problem and users
What problem exists, and who benefits from solving it?

## Existing system
Relevant architecture, limitations, and prior work.

## Proposed approach
Design, interfaces, alternatives, and reasons for the choice.

## Deliverables
Concrete code, tests, documentation, examples, and integration work.

## Milestones
Small reviewable increments with validation and contingency time.

## Risks and fallback scope
Dependencies, uncertain assumptions, and a useful reduced outcome.

## Communication
Agreed reporting and review cadence.

## Experience and availability
Relevant evidence, contribution links, and realistic commitments.
```

Adapt this to the organization's required template. Prefer measurable milestones such as “support this input with tests and documentation” over vague goals such as “improve the whole library.” Plan documentation and review time as part of delivery.

### Sustained contribution habits

Keep branches focused, tests reproducible, and discussions respectful. Learn to accept that a proposed change may not fit a project's goals. Maintain contributions after merging when feasible and help clarify documentation for the next contributor.

Practice: reproduce one issue without changing code; improve a failing example; write a regression test; review a small existing PR locally; draft a project proposal with a reduced-scope fallback; explain your patch without relying on generated text.

Return to the [version-control index](#reading-order).
