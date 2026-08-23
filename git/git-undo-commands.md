# Git Undo Commands

A safety-focused guide for correcting Git mistakes. Check `git status` before and after every undo operation.

## Discard unstaged changes in one file

> [!WARNING]
> This permanently discards uncommitted changes in the selected file.

```bash
# Restore one working-tree file from the index.
git restore path/to/file.md
```

## Unstage a file without discarding edits

```bash
# Remove a file from the staging area while keeping its local changes.
git restore --staged path/to/file.md
```

## Correct the latest commit

```bash
# Add a forgotten file to the staging area.
git add path/to/file.md

# Replace the latest local commit while keeping its message.
git commit --amend --no-edit
```

> [!WARNING]
> Amending rewrites the commit ID. Avoid amending a shared commit unless collaborators understand the history change.

## Revert a shared commit safely

```bash
# Create a new commit that reverses an earlier commit.
git revert COMMIT_SHA
```

### Use case

Prefer `revert` for a commit already pushed to a shared branch because it preserves history.

## Move a local branch back but keep changes

```bash
# Remove the latest commit while keeping its changes staged.
git reset --soft HEAD~1

# Remove the latest commit while keeping its changes unstaged.
git reset HEAD~1
```

> [!WARNING]
> `git reset` moves branch history. Do not use it casually on commits already shared with others.

## Recover a commit with reflog

```bash
# Show recent updates to local branch references.
git reflog

# Create a recovery branch at the required commit.
git switch -c recovery-branch COMMIT_SHA
```

## Choosing the right command

| Situation | Command |
|---|---|
| Unstage but keep edits | `git restore --staged FILE` |
| Discard unstaged edits | `git restore FILE` |
| Undo a shared commit | `git revert COMMIT_SHA` |
| Correct latest local commit | `git commit --amend` |
| Recover lost local commit | `git reflog` then create a branch |

Avoid `git reset --hard` unless you fully understand exactly which work it will destroy and have a recoverable backup.
