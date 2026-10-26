# Splitting recipes

How to present a split to the user is covered in SKILL.md's atomicity check.

## Recipe A — peel off a subset of files

```
# Unstage files that belong to a later commit
git restore --staged path/to/unrelated.ts path/to/also-unrelated.ts

# Commit the focused set
git commit -m "<subject>" -m "<body>"

# Re-stage the rest for the next commit
git add path/to/unrelated.ts path/to/also-unrelated.ts
```

## Recipe B — split hunks within a single file

`git add -p` is interactive and cannot run in this environment. Hand these commands to the user, or fall back to Recipe A at file granularity.

```
git restore --staged path/to/file.ts
git add -p path/to/file.ts
```

After the first commit, the user runs `git add -p` again to stage the rest.

## Recipe C — separate the refactor from the behavior change

Commit the refactor (or rename/move) first, then the behavior change. The refactor commit stays easy to review and revert, the behavior commit stays bisectable, and its diff against the refactored code is usually much smaller. Moving before changing also keeps `git log --follow` working.

## Recipe D — extract formatter churn

```
git stash
<run formatter>
git add -u
git commit -m "style: apply <formatter> to <scope>"
git stash pop
```

The stashed hunks were written against the unformatted files, so `git stash pop` often conflicts. Resolve by taking your side and re-running the formatter. When that isn't worth it, use Recipe B and leave the formatter churn for a later commit.

## Atomicity is not file count

One commit touching 30 files for a single rename is atomic. Two commits each touching one file for unrelated reasons are not.
