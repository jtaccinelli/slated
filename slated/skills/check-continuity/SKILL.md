---
name: check-continuity
description: Log an out-of-band codebase change into the storyboard as a Recorded entry, and flag whether it warrants a set outline update.
expected-role: visualiser
argument-hint: [brief description of the out-of-band change — omit to use git history]
---

# Check Continuity

Log work that happened outside the framework — changes made directly to the codebase without a scene — so the storyboard remains an honest map of the project. A successful execution produces a single Recorded entry in the storyboard, confirmed with the user before it is written. All other files are read-only.

---

## Pre-work

1. Read `.claude/slated/scenes/storyboard.md`. Identify:
   - The names of all existing scenes referenced in the Pending and Completed sections — these are the known scene slugs.
   - Whether a "Recorded" section already exists in the document.
2. If `$ARGUMENTS` is provided, use it as the starting description of the change. Skip to Step 1 with this description in hand.
3. If `$ARGUMENTS` is not provided, run:
   ```
   git log --oneline -10
   ```
   Review the output. Cross-reference each commit message against the known scene slugs from the storyboard. Any commit that does not correspond to a recognised scene is a candidate undocumented change. Surface the candidates to the user before proceeding to Step 1.

---

## Process

### Step 1 — Establish the change

Present the discovered or provided change description to the user. Ask the user to confirm or refine:

1. **What changed** — a plain-language description of the modification
2. **Why it was made** — the motivation or context
3. **Which files were affected** — the specific files touched (use git output or ask the user to list them)
4. **One-line summary** — a concise summary suitable for a log entry
5. **Slug** — propose a short kebab-case label for the change; present it for confirmation before using it

Do not proceed to Step 2 until all five points are confirmed by the user. Do not invent a slug without presenting it first.

### Step 2 — Assess architecture impact

From the confirmed file list, identify whether the change affects the project's architecture baseline. Consider the following signals:

- **New dependencies** — additions to `package.json` or lock files
- **Config changes** — modifications to `wrangler.jsonc`, `tsconfig.json`, `drizzle.config.ts`, `vite.config.ts`, `react-router.config.ts`, `sanity.config.ts`, or equivalent config files
- **New entry points** — new worker scripts, new route files, new build entry targets
- **Structural reorganisation** — directories added, moved, or removed; import path changes

If any of these signals are present, surface a recommendation to the user:

> This change appears to affect the project's architecture baseline. Consider updating the set outline at `.claude/slated/set/outline.md` to reflect: [specific aspect that changed].

Do not write to the set outline. Do not write to any file other than the storyboard.

If none of the signals are present, note that no architecture update appears necessary and proceed to Step 3.

### Step 3 — Write the storyboard entry

Open `storyboard.md`. Determine whether a "Recorded" section exists.

**If the "Recorded" section does not exist**: create it between the Pending section and the Completed section (or at the end of the document if Completed does not exist). Use this structure:

```markdown
## Recorded

Changes made outside the framework — logged for continuity. These are not scenes and have no manuscript, takes, or wrap record.

| Change | Date | Summary |
|---|---|---|
```

**Add the log entry** to the Recorded table:

```
| `<confirmed-slug>` | <YYYY-MM-DD> | <confirmed one-line summary> |
```

Use today's date in `YYYY-MM-DD` format. Write only this entry — do not modify any other section of the storyboard.

---

## Output

- `storyboard.md` — updated with the Recorded entry (section created if absent; row added to the table)
- Architecture recommendation — surfaced in the conversation if applicable; not written to any file

---

## Rules

- Never write to any file except `storyboard.md`
- Never describe or log a change without user confirmation of all five points in Step 1
- Never invent or use a slug without presenting it for confirmation first
- Never prefix a slug with `scene-` — Recorded entries are not scenes
- Never create a manuscript, takes directory, or wrap file for a Recorded entry — these are not scenes
- Never write to the set outline, regardless of architecture impact — surface the recommendation only
