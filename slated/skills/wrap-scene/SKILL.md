---
name: wrap-scene
description: Close out a completed scene — appends the Completion Record to the manuscript, writes wrap.md, and dispatches a visualiser to move the scene into the storyboard.
expected-role: director
argument-hint: [scene name]
---

# Wrap Scene

Close out a scene whose final take has passed. The director compiles the Completion Record, marks the manuscript complete, writes `wrap.md`, and updates the storyboard.

---

## Pre-work

1. Resolve the scene from `$ARGUMENTS` — find `.claude/slated/scenes/scene-<name>/manuscript.md`
2. Read the manuscript in full
3. Glob `.claude/slated/scenes/scene-<name>/takes/take-*.md` and read all take files
4. Confirm the most recent take has status `pass` — if not, stop and report that the scene cannot be wrapped until a take passes
5. Confirm the manuscript status is not already `complete` — if it is, report that the scene has already been wrapped and stop
6. Check for `.claude/slated/scenes/scene-<name>/review.md` — if it exists, read it in full; the Completion Record must confirm that all findings from this review are addressed in the take history

---

## Process

### Step 1 — Compile the Completion Record

Gather the following from the take files:

- **Completed date**: today's date (YYYY-MM-DD)
- **Takes required**: total count of take files
- **Take log**: each take's number and status; for failed takes, a brief primary failure reason drawn from the Director's Notes
- **Role improvement notes**: any patterns of repeated error across takes that suggest a gap in a role definition or background document; omit entirely if no meaningful patterns emerged — do not leave placeholder text
- **Manuscript improvement notes**: any structural issues in the manuscript that caused actor confusion or deviation — ambiguous action wording, missing sequencing constraints, underspecified objectives; omit entirely if none emerged

### Step 2 — Update the manuscript

Replace the HTML comment block and stub placeholder in `manuscript.md` with the completed Completion Record, following the template structure exactly. Set the `**Status**` field to `complete`. The comment and stub must not remain in the file.

### Step 3 — Produce wrap.md

Write `.claude/slated/scenes/scene-<name>/wrap.md` using this structure (reference template at `${CLAUDE_SKILL_DIR}/../../templates/scenes/wrap.md`):

- **Description**: 2–4 sentences in plain language — what capability was added, what problem was solved, or what structure was established. No implementation detail. Written to be understood by an actor with no prior context on this scene.
- **Files**: a complete list of every file created or meaningfully modified during the scene, grouped by concern (e.g. "Schema & Handlers", "Routes", "Components"). Draw this from the **Files Changed** sections of all take files — the `git status --short` output recorded in each take is the authoritative record.
- **Editor Notes**: any shared or foundational artefacts this scene modified — entry points, shared handlers, configuration files, design system primitives — and any implicit dependencies introduced. Consider: what could a future scene inadvertently break if they touched the same files? If the scene only created new files with no effect on shared artefacts, omit this section.

### Step 4 — Invalidate prior review

If `.claude/slated/scenes/scene-<name>/review.md` exists, delete it. The scene is being wrapped (or re-wrapped after a review finding) and the prior review is no longer valid. This ensures the next run of `/slated:review-project` treats this scene as unreviewed and performs a fresh review of the corrected output.

### Step 5 — Dispatch the visualiser

Spawn a sub-agent using the actor agent:

- Pass `--role visualiser` as arguments
- Include in the agent prompt:
  - The path to the completed `wrap.md`
  - The path to `.claude/slated/scenes/storyboard.md`
  - The scene directory path
  - Instructions to:
    1. Read `wrap.md` in full
    2. Read the current storyboard in full to understand existing categories before making any changes
    3. Move the scene from the Pending section to the Completed section of `storyboard.md`, placing it within an existing category or establishing a new one if no existing category genuinely fits

Wait for the visualiser to complete before proceeding.

### Step 6 — Open a pull request

Push the scene branch to the remote and open a PR:

1. Push the scene branch: `git push -u origin scene/<scene-name>`
2. Create the PR using `gh pr create`:
   - **Title**: derived from the scene name (e.g. `scene(add-user-auth): complete`)
   - **Body**: the Description and Files sections from `wrap.md`, verbatim
   - **Base branch**: `main` (or the repo's default branch)

If `gh` is not available, or the push or PR creation fails, surface the error and instruct the user to open the PR manually:

```
PR creation failed: <error>

Push and open the PR manually:
  git push -u origin scene/<scene-name>
  gh pr create --base main
```

### Step 7 — Report completion

Re-read the Completion Record from the written `manuscript.md` to check for Role Improvement Notes.

Surface a clear summary:

```
Scene <name> — COMPLETE

Completion Record appended. Takes required: <N>.
wrap.md written. Storyboard updated.
PR: <url>
```

If Role Improvement Notes were present in the Completion Record, append to the summary:

```
Role improvements identified. Run the following before the next scene is planned:
- /slated:refine-role <role-name> — <one-line summary of the improvement needed>
- /slated:refine-background <background-name> — <one-line summary of the improvement needed>
```

Then use `AskUserQuestion` to pause before returning: "Role improvements were identified during this scene. Review the notes above and confirm you have noted them before this scene is closed." Do not return until the user acknowledges.

---

## Output

- `.claude/slated/scenes/scene-<name>/manuscript.md` — status set to `complete`, Completion Record appended
- `.claude/slated/scenes/scene-<name>/wrap.md` — written by the director
- `.claude/slated/scenes/scene-<name>/review.md` — deleted if it existed, so the next `/slated:review-project` run treats this scene as unreviewed
- `.claude/slated/scenes/storyboard.md` — scene moved from Pending to Completed
- A PR opened from `scene/<scene-name>` into `main` (or fallback instructions surfaced if `gh` is unavailable)

---

## Rules

- Never wrap a scene whose most recent take does not have status `pass`
- Never wrap a scene already marked `complete`
- Never skip dispatching the visualiser — `wrap.md` and storyboard updates are not optional
- Never write the Completion Record before confirming the passing take exists
- Never skip the PR creation step — if `gh` is unavailable, surface fallback instructions rather than silently omitting it
