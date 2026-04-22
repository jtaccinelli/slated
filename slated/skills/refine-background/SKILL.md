---
name: refine-background
description: Refine an existing background through targeted interview — adjusting specific sections without rebuilding from scratch. Nothing is written until every proposed change is confirmed.
expected-role: casting-director
argument-hint: [background name]
---

# Refine Background

Make targeted, minimal changes to an existing background document. The casting director reads the current background in full, interviews to understand exactly what needs to change and why, proposes specific edits to the affected sections only, and writes the updated file after explicit confirmation. No section is touched that is not in scope.

---

## Pre-work

1. Resolve the background from `$ARGUMENTS` — check `.claude/slated/backgrounds/background-<name>.md` first (project-specific); if not found, use Bash (`ls $HOME/.claude/slated/backgrounds/background-<name>.md 2>/dev/null`) to check if it exists globally, and if so read it at `$HOME/.claude/slated/backgrounds/background-<name>.md`; note which location the file was found at
2. Read the background file in full — understand every section before the interview begins
3. If the background does not exist in either location, stop — suggest `/slated:establish-background` to define a new one

---

## Process

### Step 1 — Establish what needs to change

Use `$ARGUMENTS` as a starting point if a reason was provided. Ask the user:

1. What is not working about the current background? What actor behaviour or output has revealed the gap?
2. Which sections are affected — Semantics, Structure, or Function?
3. Is this a correction (something wrong), an addition (something missing), or a removal (something that does not belong)?

Do not proceed until the scope is clear. If the answers suggest the background needs fundamental rebuilding rather than targeted adjustment, surface that and ask whether a full redefinition via `/slated:establish-background` would be more appropriate.

### Step 2 — Propose targeted changes

For each section identified as needing change, present the specific proposed edit:

- Show the current text and the proposed replacement side by side
- State the reason for each change in one sentence
- Do not rewrite sections that are not in scope

If a proposed change in one section conflicts with another (e.g. a new Function pattern that contradicts an existing Semantics convention), surface the conflict and resolve it before confirming.

### Step 3 — Confirm the changes

Present all proposed changes together and ask for explicit confirmation. If adjustments are requested, revise and present again.

Do not write the file until every proposed change is confirmed.

### Step 4 — Write the updated background file

Apply only the confirmed changes to the background file at the location it was found. Every section not in scope must remain exactly as it was.

---

## Output

- The background file at its original location — updated with targeted changes only

Report the file path and summarise which sections were changed and why.

---

## Rules

- Never write the file before the user explicitly confirms the proposed changes
- Never rewrite sections that were not identified as needing change
- Never accept vague change requests — if the reason for a change is unclear, ask until it is specific
- Never add behavioural parameters to a background — backgrounds document what to know, not how to behave; that belongs in roles
- Never create a new background file — this skill only modifies existing ones; use `/slated:establish-background` for new backgrounds
- Never move a background between global and project locations — this skill only updates content, not location
