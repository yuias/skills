---
name: handoff
description: Save or restore a session handoff note
argument-hint: save | restore
disable-model-invocation: true
---

Note path: `.claude/handoff.md`

Read the first word of `$ARGUMENTS` and run the matching mode below.
If it is neither `save` nor `restore`, or is empty, do **nothing** and just print the usage
(`/handoff save`, `/handoff restore`).

Write the note in English, and conduct both modes in English, regardless of any language
default set elsewhere in the project config.

---

## save

Rewrite `.claude/handoff.md` in full. Never append to it as a log.

Before writing:
- Get `git rev-parse --short HEAD` and `git status --short`.
- Re-read any file you are about to make claims about if you have not touched it recently;
  your in-context copy may be stale.

Rules:
- Describe the **current state of the world**, not the conversation. "Decoder thread
  implemented, untested" — not "I implemented the decoder, then we discussed...".
- Tag every state claim `[verified: <command>]` or `[unverified]`.
- Every decision carries its reason. A decision without a reason gets overturned next session.
- No diffs, no code dumps. Point at `path/to/file.rs:symbol` and the commit hash.
- Keep the whole note under 100 lines.

Format:

````markdown
# Handoff: <task>

Updated: <date> / HEAD: <short-sha> / working tree: clean|dirty

## Goal
<1-2 sentences. Done condition, in observable terms.>

## Decisions
- <what was decided> — because <why>

## Current state
- <component>: <state> [verified: `cargo test`]
- <component>: <state> [unverified]
- Not started: <item>
- Half-edited / broken right now: <where, or "none">

## Next action
<one concrete action, executable without further decisions>

## Dead ends
- <approach>: rejected because <reason>

## Key files
- `path/to/file` — <what to look at>

## Verification
```
<build / test commands>
```
````

Then report only the note path and that the next session should run `/handoff restore`.

---

## restore

1. Read `.claude/handoff.md`. If missing, say so and stop.
2. Read the files listed under "Key files". The note records intent; the files are the truth.
3. Run the verification commands and check them against "Current state". **Do not skip this**
   — the working tree may have moved since the note was written.
4. Report any drift first.
5. State goal / current state / first action in 5 lines or fewer, then wait for the user's
   confirmation. **Do not edit files before that confirmation.**
