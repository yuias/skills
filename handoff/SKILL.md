---
name: handoff
description: Save or restore a session handoff note
argument-hint: save | restore
disable-model-invocation: true
---

Note path: `.claude/handoff.md`

Run the mode named by the first word of `$ARGUMENTS`. For anything else, or no argument,
print the usage (`/handoff save`, `/handoff restore`) and stop.

Write the note in English, and conduct both modes in English, regardless of any language
default set elsewhere in the project config.

## save

Overwrite the note in full; it is a snapshot, not a log.

Before writing, get `git rev-parse --short HEAD` and `git status --short`, and re-read any
file you will make claims about unless you touched it recently.

Rules:
- Describe the current state of the work, not the conversation: "Decoder thread
  implemented, untested", not "I implemented the decoder, then we discussed...".
- Tag every state claim `[verified: <command>]` or `[unverified]`.
- Give every decision its reason, so the next session does not overturn it.
- No diffs or code dumps. Point at `path/to/file.rs:symbol` and the commit hash.
- Keep the note short enough to read in full before opening any file.

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

## restore

1. Read the note. If it is missing, say so and stop.
2. Read the files under "Key files"; where they disagree with the note, the files are right.
3. Run the commands under "Verification" and compare the results with "Current state". The
   working tree may have changed since the note was written.
4. Report any drift first, then goal, current state, and first action, in your own words
   rather than reading the note back.
5. Wait for the user's confirmation before editing any file.
