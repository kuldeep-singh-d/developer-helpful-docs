# GitHub Pull Request Workflow

A beginner-friendly workflow for proposing focused changes through a pull request.

## Create a branch

```bash
# Update remote references.
git fetch origin

# Create a focused branch from the current base branch.
git switch -c docs/add-adb-guide
```

## Review, commit, and push

```bash
# Review current changes.
git status
git diff

# Stage only the intended file.
git add android/adb-useful-commands.md

# Review the exact commit content.
git diff --staged

# Commit and publish the branch.
git commit -m "Add ADB useful commands guide"
git push -u origin docs/add-adb-guide
```

## Open the pull request

On GitHub, select the pushed branch and choose **Compare & pull request**. Confirm:

- The base branch is correct.
- The title explains the outcome.
- The description explains what changed and how it was checked.
- The diff contains no credentials, private URLs, generated files, or unrelated changes.

Contributors without repository write access should fork the repository, push a branch to their fork, and open a pull request from that branch.

## After review

Apply feedback in the same branch and push additional focused commits. Do not force-push unless the repository workflow explicitly permits rewritten history.
