# Atomicity recipes

A commit is atomic when it captures one focused, logically coherent change. The diff should be reviewable as a unit and revertable as a unit. When the staging area mixes concerns, propose a split — how to present that split to the user is covered in SKILL.md's atomicity section.

## How to recognise a non-atomic stage

Run `git diff --staged --stat` and look at the file list. Red flags:

- Files in two or more unrelated subsystems (e.g. `src/auth/` and `src/billing/`).
- A change to `package.json` / `Cargo.toml` / `go.mod` alongside unrelated source edits — dependency bumps almost always deserve their own commit.
- Formatter / linter churn mixed with substantive edits (whitespace-only hunks alongside logic changes).
- New tests for an *old* behavior bundled with a fix to a *different* behavior.
- A rename or move bundled with a behavior change inside the renamed file. Move first, change second — this preserves `git log --follow` traceability.

Read the actual hunks too. Two changes can sit in the same file and still be unrelated (e.g. an import reorder and a new function).

## Splitting recipes

### Recipe A — peel off a subset of files

```
# Inspect what's staged
git diff --staged --stat

# Unstage files that belong to a later commit
git restore --staged path/to/unrelated.ts path/to/also-unrelated.ts

# Commit the focused set
git commit -m "<subject>" -m "<body>"

# Re-stage the rest and commit separately
git add path/to/unrelated.ts path/to/also-unrelated.ts
```

### Recipe B — split hunks within a single file

When two unrelated changes live in the same file, use interactive staging.

`git add -p` is interactive and cannot run in this environment — hand these commands to the user and let them drive, or fall back to Recipe A at file granularity.

```
# Unstage the file entirely
git restore --staged path/to/file.ts

# Stage hunk by hunk
git add -p path/to/file.ts
```

In the `git add -p` prompt:

- `y` — stage this hunk
- `n` — skip this hunk
- `s` — split into smaller hunks
- `e` — manually edit the hunk (drop lines you don't want)
- `q` — quit, keep what you've staged so far

After the first commit, run `git add -p` again to stage the rest.

### Recipe C — separate the refactor from the behavior change

The single highest-leverage split. If a commit both moves code around *and* changes behavior, splitting it produces:

1. A pure refactor commit — easy to review, trivially revertable.
2. A small behavior commit — bisectable, the diff shows exactly what behavior changed.

Order: refactor first, behavior change second. The behavior diff against the refactored code is usually much smaller and clearer than the diff against the original.

### Recipe D — extract formatter churn

If a formatter has rewritten files you also edited substantively:

```
# Stash everything
git stash

# Apply only the formatter, commit it
<run formatter>
git add -u
git commit -m "style: apply <formatter> to <scope>"

# Pop your real changes back
git stash pop
```

Now your real change can be staged and committed without whitespace noise.

Caveat: the stashed hunks were written against the *unformatted* files, so `git stash pop` will often conflict. Resolve by taking your own side and re-running the formatter on the result. When the conflict risk isn't worth it, use Recipe B instead and leave the formatter churn for a later commit.

## When NOT to split

Don't split if the changes are genuinely interdependent:

- A new function and the call site that uses it. Splitting leaves a dead function in commit 1.
- A schema migration and the code that depends on the new schema. Splitting breaks bisect.
- A test and the production code it tests (when added together). Splitting hides whether the test was passing before.

Atomicity is about logical coherence, not file count. One commit touching 30 files for a single rename is atomic. Two commits each touching one file for unrelated reasons is not.
