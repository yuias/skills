---
name: atomicity-commit
description: Draft a Conventional Commits-style commit message for the staged changes and run `git commit` in a single step. Use whenever the user asks to commit, write a commit message, finalize changes, or types /atomicity-commit. Inspects the staged diff, drafts an English subject following the Conventional Commits 1.0.0 spec, adds a short body only when there is non-obvious rationale (why, not what), and commits directly. If the staged set mixes unrelated concerns, it stops first and proposes a split — the user decides whether to split or proceed.
disable-model-invocation: false
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git rev-parse:*), Bash(git add:*), Bash(git restore:*), Bash(git commit:*), Bash(git config:*)
---

# Commit Message

Draft a high-quality commit message for the staged changes and create the commit.

## Inputs you must gather first

Run these before drafting anything. Skipping them produces guesses, not messages.

```
git rev-parse --is-inside-work-tree
git status --short
git diff --staged --stat
git diff --staged
git log -n 20 --oneline
```

Use the recent log to learn the repository's own conventions (type vocabulary, scope style, casing) and follow them when they don't conflict with the rules below. If the log shows a clearly different style, prefer the repo's style and mention the deviation to the user once.

Staging behavior depends on what is already staged:

- **Nothing staged:** you may `git add` the relevant changes yourself so the commit has content. Pick the files that form one focused change.
- **Something already staged:** the user has signalled what they want in this commit. Before adding anything else, ask whether the other unstaged files should be included too — do not silently expand the staged set.

## Output format

Produce a Conventional Commits 1.0.0 message:

```
<type>(<scope>): <subject>

<body explaining why>

<optional footer(s)>
```

### Subject line

- Use one of the standard types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. Pick the one that best matches the user-visible effect, not the largest file touched.
- `feat` and `fix` map to MINOR and PATCH in SemVer. Reserve them for behavioral changes; use `refactor`, `chore`, `docs`, `test`, etc. when behavior is unchanged.
- Scope is optional. Include it when there is an obvious, stable module/package/area (e.g. `auth`, `parser`, `ci`). Omit it rather than invent one.
- Breaking changes: append `!` after the type/scope (e.g. `feat(api)!: ...`) AND add a `BREAKING CHANGE:` footer describing the break.
- Description: imperative mood, lowercase first letter (unless the repo log clearly uses Sentence case), no trailing period, target ≤ 50 characters and never exceed 72.
- Write in English unless the user has explicitly asked for another language.

### Body — optional, and short when present

Default to **subject-only**. If the subject alone conveys the change and its
intent, omit the body entirely — most commits land here. Add a body only when
there is genuinely non-obvious rationale a future reader would otherwise miss.

The diff already shows *what* changed line-by-line; re-narrating it adds
nothing. When a body *is* warranted, write only the point that actually
applies — do not pad to fill all three:

- What problem or need motivated this change (when not obvious from the subject)?
- Why this approach over a plausible alternative (only when the choice is non-obvious)?
- A constraint, tradeoff, or non-obvious consequence worth flagging.

**Length budget:** body is typically 0–2 lines; 4 lines is a hard ceiling. Pick
the single most valuable point and state it. If you find yourself writing more,
the surplus is almost always re-narrated diff — cut it. When in doubt, omit the
body rather than pad it. Footers are not part of the body and do not count
toward this budget — a `BREAKING CHANGE:` description may run as long as the
break requires.

Wrap body lines at 72 characters. Separate subject and body with a blank line.

**Anti-patterns to refuse:**

- "Update foo.py" / "Various changes" / "WIP" — these communicate nothing.
- A bullet list that paraphrases the diff ("Added X. Removed Y. Renamed Z."). If a reader wants that, they can read the diff. State the reason those changes were made together.
- Marketing language ("improve", "enhance", "better") without saying what made it worse before or what concretely improves.

### Footers

Use footers for machine-readable metadata, one per line:

- `BREAKING CHANGE: <description>` — mandatory for any breaking change.
- `Refs: #123`, `Closes: #123`, `Fixes: #123` — issue tracker references when the user provides them or they appear in branch names.
- `Co-authored-by: Name <email>` — when applicable.

Do not invent issue numbers. If unsure, leave footers off.

## Atomicity check — do this BEFORE drafting

A good commit captures one focused, logically coherent change. Before writing the message, scan the staged diff for signs the staging area mixes unrelated concerns:

- Multiple unrelated `feat`/`fix` candidates in different modules.
- A refactor bundled with a behavior change (these should almost always be separate commits — the refactor commit becomes reviewable, and the behavior commit becomes bisectable).
- Formatting / whitespace churn mixed with substantive edits.
- An unrelated dependency bump alongside feature work.

If you spot any of these, **stop and surface the issue to the user before drafting**. Your job here is to inform, not to block — name the distinct changes you see, propose a concrete split (which hunks/files belong to which commit) with the exact commands to perform it, then ask whether they want to split or to proceed with a single commit anyway. If they choose to proceed, draft the message normally and note the mixing in the body in one short sentence, so the history is honest about it. Example 4 below shows the shape of that response.

See `references/atomicity.md` for splitting recipes.

## Drafting workflow

The default mode is end-to-end: draft the message and commit, in a single response, without waiting for the user to approve the draft. The user invoked `/atomicity-commit` because they want a commit, not a draft for review. Treat the message as final and run `git commit`.

1. Read the inputs listed above.
2. Run the atomicity check. **If mixed:** stop and follow that section — do **not** commit anything in this turn. **If clean:** continue without asking.
3. Classify the change (type, scope, breaking?).
4. Write the subject — imperative, ≤ 50 chars target, ≤ 72 hard limit.
5. Decide on the body per **Body** above: default to subject-only.
6. Add footers only if warranted (BREAKING CHANGE, issue refs the user mentioned, etc.).
7. Commit immediately with `git commit -m "<subject>" -m "<body>"` — each `-m` becomes its own paragraph, so a third `-m` carries a footer. That covers any message within the length budget; see `references/committing.md` for the `-F` fallback when quoting gets awkward.
8. After the commit, run `git --no-pager log -1 --format=fuller` and show the user what was recorded, so they can verify and `--amend` if needed.
9. Do not add `Co-authored-by: Claude` or any Anthropic/Claude attribution unless the user asks for it.

**When NOT to auto-commit even on a clean stage:**

- A pre-commit hook fails. Surface the hook's output verbatim and stop. The user decides whether to fix it, retry with `--no-verify`, or abandon.
- The user's prompt explicitly asks for a draft only ("just draft the message", "show me what you'd write", "don't commit yet"). Honor that — show the message and stop.
- The repository is in a detached HEAD, in the middle of a rebase/merge, or otherwise in a state where committing would be surprising. Mention the state and ask before proceeding.

## Examples

### Example 1 — feature with rationale

Staged diff: adds a retry wrapper around the S3 upload call with exponential backoff.

```
feat(uploader): retry S3 uploads with exponential backoff

Transient 503s from S3 were failing ~2% of CI runs. The 5-attempt cap
keeps a genuine outage failing the build within a minute.
```

Why this is good: the subject states the change in imperative mood. The body never re-describes the diff — it gives the failure mode that motivated the change plus the one non-obvious consequence of the design, in two lines.

### Example 2 — bug fix with breaking semantics

Staged diff: changes the public `parse()` function to throw on empty input instead of returning `null`.

```
fix(parser)!: throw on empty input instead of returning null

Three production crashes last quarter traced back to a caller that
forgot the null-check the old contract required.

BREAKING CHANGE: parse("") previously returned null; it now throws
EmptyInputError. Callers relying on the null return must catch the
error or pre-check for empty strings.
```

Why this is good: the `!` marker and `BREAKING CHANGE` footer are both present (the spec requires the footer; the `!` is a visual aid). The body cites concrete evidence rather than abstract "improvements" and stays within budget; the footer sits outside that budget and runs as long as the break needs.

### Example 3 — chore where no body is needed

```
chore: bump prettier to 3.3.3
```

Why this is good: the subject is self-explanatory; padding it with a body would be noise. This is the common case, not the exception.

### Example 4 — refusing a mixed commit

The user has staged a refactor of the auth module *and* a fix to a pagination bug in the search module. Do not draft a single message. Respond with something like:

> The staged changes look like two separate things: an auth-module rename and a search-pagination off-by-one. Splitting them gives you a clean revert path for either one. Suggested split:
>
> - Commit 1 (refactor): `src/auth/*`
> - Commit 2 (fix): `src/search/pagination.ts`, `src/search/pagination.test.ts`
>
> To do this:
> ```
> git restore --staged src/search/pagination.ts src/search/pagination.test.ts
> # commit the auth refactor first, then re-stage and commit the fix
> ```
>
> Want me to proceed with the auth commit, or would you rather stage things differently?

## When the user pushes back

- "Just write it in one commit" → do it, with the one-sentence acknowledgement of the mixing described in the atomicity section.
- "Make the message shorter" → tighten the body; never sacrifice the *why*. If the subject is at 50 chars, that's fine.
- "Write it in Japanese" (or another language) → switch the message to that language; keep the Conventional Commits prefix (`feat:`, `fix:`, …) in English since it's a structural marker.

## Reference files

- `references/atomicity.md` — splitting recipes (partial staging, undoing mixed stages).
- `references/committing.md` — the `-F` fallback, verifying the commit, hook failures.
