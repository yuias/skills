---
name: atomicity-commit
description: Draft a Conventional Commits-style commit message for the staged changes and run `git commit` in a single step. Use whenever the user asks to commit, write a commit message, finalize changes, or types /atomicity-commit. Inspects the staged diff, drafts an English subject following the Conventional Commits 1.0.0 spec, adds a short body only when there is non-obvious rationale (why, not what), and commits directly. If the staged set mixes unrelated concerns, it stops first and proposes a split — the user decides whether to split or proceed.
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git rev-parse:*), Bash(git add:*), Bash(git restore:*), Bash(git stash:*), Bash(git commit:*), Bash(git config:*)
---

# Atomicity Commit

## Gather inputs

Run these before drafting:

```
git rev-parse --is-inside-work-tree
git status --short
git diff --staged --stat
git diff --staged
git log -n 20 --oneline
```

Follow the conventions the log shows (type vocabulary, scope style, casing). Where the repo clearly differs from the rules below, the repo's style wins; mention the deviation to the user once.

- **Nothing staged:** `git add` the files that form one focused change.
- **Something staged:** the user has chosen the contents. Ask before adding other files.

## Atomicity check

A commit should capture one focused, logically coherent change. Before drafting, check the staged diff for mixed concerns:

- Unrelated `feat`/`fix` candidates in different modules.
- A refactor, rename, or move bundled with a behavior change.
- Formatting or whitespace churn mixed with substantive edits.
- A dependency bump (`package.json`, `Cargo.toml`, `go.mod`, …) alongside unrelated work.

Read the hunks, not only the file list: one file can hold two unrelated changes. Changes that depend on each other (a new function and its call site, a migration and the code that needs it, a test and the code it tests) belong together.

If the stage is mixed, do not commit in this turn. Name the distinct changes, propose a split (which files or hunks go in which commit) with the commands to do it, and ask whether to split or proceed as one commit (Example 4). If the user proceeds, note the mixing in the body in one sentence. Splitting recipes are in `references/atomicity.md`.

## Message format

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

### Subject

- Type: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, or `revert`. Pick by the user-visible effect, not the largest file touched. Use `feat`/`fix` only for behavior changes.
- Scope: include it only when there is an obvious, stable module or area (`auth`, `parser`, `ci`); do not invent one.
- Breaking change: append `!` after the type/scope and add a `BREAKING CHANGE:` footer.
- Description: imperative mood, lowercase first letter (unless the log uses Sentence case), no trailing period, ≤ 50 characters as a target and 72 at most.
- Write the message in English unless the user asks for another language. Keep the type prefix in English either way.

### Body

Default to subject-only; most commits need no body. The diff already shows what changed, so add a body only for rationale a future reader would otherwise miss, and state the single most valuable point:

- the problem or need that motivated the change, when the subject doesn't show it;
- why this approach over a plausible alternative;
- a constraint, tradeoff, or non-obvious consequence.

Do not paraphrase the diff ("Added X. Removed Y."), and do not use "improve" or "enhance" without saying what was wrong before. Wrap at 72 characters.

### Footers

One per line, only when warranted:

- `BREAKING CHANGE: <description>` — required for any breaking change; it may run as long as the break needs.
- `Refs: #123`, `Closes: #123`, `Fixes: #123` — only for issues the user mentioned or the branch name contains. Never invent numbers.
- `Co-authored-by: Name <email>` — only for a co-author the user names. Do not add Claude or Anthropic attribution.

## Commit

On a clean stage, draft the message and commit in the same response without asking for approval. Commit with `git commit -m "<subject>" -m "<body>"` (each `-m` is one paragraph, so a third carries a footer); see `references/committing.md` when quoting gets awkward. Then run `git log -1 --format=fuller` and show the result so the user can verify it.

Do not commit, and tell the user why, when:

- a pre-commit hook fails: show its output verbatim and stop; the user decides whether to fix, use `--no-verify`, or abandon;
- the user asked for a draft only ("just draft the message", "don't commit yet"): show the message and stop;
- the repository is in a detached HEAD or mid-rebase/merge: mention the state and ask first.

If the user asks for a shorter message, tighten the body but keep the reason.

## Examples

### Example 1 — feature with rationale

```
feat(uploader): retry S3 uploads with exponential backoff

Transient 503s from S3 were failing ~2% of CI runs. The 5-attempt cap
keeps a genuine outage failing the build within a minute.
```

The body gives the failure that motivated the change and one consequence of the design, without re-describing the diff.

### Example 2 — breaking fix

```
fix(parser)!: throw on empty input instead of returning null

Three production crashes last quarter traced back to a caller that
forgot the null-check the old contract required.

BREAKING CHANGE: parse("") previously returned null; it now throws
EmptyInputError. Callers relying on the null return must catch the
error or pre-check for empty strings.
```

Both the `!` and the required `BREAKING CHANGE` footer are present; the body cites concrete evidence.

### Example 3 — no body needed (the common case)

```
chore: bump prettier to 3.3.3
```

### Example 4 — mixed stage

The user staged an auth-module refactor and a search-pagination fix. Respond with something like:

> The staged changes are two separate things: an auth-module rename and a search-pagination off-by-one. Splitting them lets you revert either one alone. Suggested split:
>
> - Commit 1 (refactor): `src/auth/*`
> - Commit 2 (fix): `src/search/pagination.ts`, `src/search/pagination.test.ts`
>
> ```
> git restore --staged src/search/pagination.ts src/search/pagination.test.ts
> # commit the auth refactor first, then re-stage and commit the fix
> ```
>
> Want me to proceed with the auth commit, or would you rather stage things differently?
