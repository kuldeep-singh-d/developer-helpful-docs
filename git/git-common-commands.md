# Git Common Commands

A beginner-friendly reference for everyday Git work.

## Check repository state

```bash
# Show the current branch and changed files.
git status

# Show unstaged changes.
git diff

# Show staged changes.
git diff --staged
```

## Create a branch

```bash
# Create a new branch and switch to it.
git switch -c docs/add-adb-guide
```

## Stage and commit changes

```bash
# Stage one specific file.
git add android/adb-useful-commands.md

# Review exactly what will be committed.
git diff --staged

# Create a commit with a clear message.
git commit -m "Add ADB useful commands guide"
```

## Synchronize with the remote

```bash
# Download remote references without changing local files.
git fetch origin

# Integrate the remote main branch into the current branch.
git pull origin main

# Push the current branch and set its upstream.
git push -u origin docs/add-adb-guide
```

## View history

```bash
# Show a compact branch graph.
git log --oneline --graph --decorate --all

# Show changes introduced by the latest commit.
git show --stat
```

## Temporarily store unfinished work

```bash
# Save tracked changes with a descriptive stash message.
git stash push -m "WIP: update Android guide"

# List saved stashes.
git stash list

# Reapply the latest stash and remove it from the stash list.
git stash pop
```

> [!WARNING]
> Conflicts can occur during `pull` or `stash pop`. Resolve and review them before committing.

## Daily workflow

```text
status → switch/create branch → edit → diff → add → diff --staged → commit → push
```
