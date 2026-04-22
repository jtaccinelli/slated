---
name: shoot-take
description: Execute one take of a scene — dispatch actors, review output, write the take file, and return a pass or fail verdict. Stops after each take and returns control to the user.
expected-role: director
argument-hint: [scene name]
---

# Shoot Take

Execute one take of a scene. Dispatch actors per the manuscript cast, review all output against the objectives and role definitions, commission a writer review of manuscript fidelity, write the take file, and return a verdict. If the take fails, stop and surface the next-take instructions to the user.

---

## Pre-work

1. Resolve the scene from `$ARGUMENTS` — find `.claude/slated/scenes/scene-<name>/manuscript.md`
2. Read the manuscript in full — cast, resources, objectives, and all shots
3. Read each role file referenced in the cast — check `.claude/slated/roles/role-<name>.md` first (project-specific); if not found, use Bash (`ls $HOME/.claude/slated/roles/role-<name>.md 2>/dev/null`) to check globally and read at `$HOME/.claude/slated/roles/role-<name>.md` if present; use whichever is found first
4. Read each background file referenced in the cast — check `.claude/slated/backgrounds/background-<name>.md` first (project-specific); if not found, use Bash (`ls $HOME/.claude/slated/backgrounds/background-<name>.md 2>/dev/null`) to check globally and read at `$HOME/.claude/slated/backgrounds/background-<name>.md` if present; use whichever is found first
5. Determine the take number — glob `.claude/slated/scenes/scene-<name>/takes/take-*.md`, count existing files, next take is that count + 1 (zero-padded to three digits: `001`, `002`, etc.)
6. Check for `.claude/slated/scenes/scene-<name>/review.md` — read it in full if it exists; its findings must inform the director's review this take
7. Confirm the manuscript status is `confirmed` or `in-progress` — if neither, stop and report that the scene is not ready to shoot
8. Create a git worktree for this take:
   - Derive the scene branch name: `scene/<scene-name>`
   - Check if the scene branch already exists: `git branch --list scene/<scene-name>`
   - If it does not exist, create it from the current HEAD: `git branch scene/<scene-name>`
   - Derive the take branch name: `take/<scene-name>-<NNN>` (using the zero-padded take number from step 5)
   - Derive the worktree path: `.worktrees/take-<scene-name>-<NNN>` (relative to the repo root)
   - Create the worktree from the scene branch: `git worktree add -b take/<scene-name>-<NNN> .worktrees/take-<scene-name>-<NNN> scene/<scene-name>`
   - If the worktree cannot be created because the take branch already exists from a previous attempt: force-delete the stale branch (`git branch -D take/<scene-name>-<NNN>`), remove any stale worktree at the path if present (`git worktree remove --force .worktrees/take-<scene-name>-<NNN>`), then retry the `git worktree add` command. If the retry fails for any other reason, stop and report the error — do not proceed until it is resolved.
   - Record the absolute worktree path for use in Step 2. If any cleanup was performed, note it — it will be recorded in the take file's Actor Summary.

---

## Process

### Step 1 — Update manuscript status

Set the manuscript `**Status**` field to `in-progress` if it is not already. If the status was `confirmed` (i.e., this is the first take), dispatch the visualiser to update the scene's entry in the storyboard Pending section from `confirmed` to `in-progress`:

- Pass `--role visualiser` as arguments
- Include in the agent prompt:
  - The path to `.claude/slated/scenes/storyboard.md`
  - The scene name
  - Instructions to update the status column for `scene-<name>` in the Pending section from `confirmed` to `in-progress`

Wait for the visualiser to complete before proceeding.

### Step 2 — Dispatch actors

For each actor in the Cast table, spawn a sub-agent using the actor agent:

- Pass `--role <role-name> --backgrounds <bg1,bg2,...>` as arguments
- Include in the agent prompt:
  - The full manuscript objectives
  - The specific shot(s) assigned to this actor, verbatim from the manuscript
  - The absolute path to the worktree created in Pre-work step 8 as the actor's working directory — all file reads and writes must occur within this path
  - The path to the scene directory within the worktree (i.e. `<worktree-path>/.claude/slated/scenes/scene-<name>/`) so the actor can read/write scene-relative files
  - An explicit constraint: actors must never write to `.claude/slated/` inside the worktree or in the main tree — framework artefacts are never produced in a worktree context
  - If a previous take exists, the path to the most recent take file in the main tree — instruct the actor to read it in full before beginning, paying particular attention to their own Director's Notes and the Next Take Instructions

Dispatch shots sequentially by default. A shot is independent only if none of its actions reference the output of another shot in this take as a prerequisite AND no two shots would modify the same files — check both conditions explicitly against the manuscript before deciding. If a shot is independent, it may be dispatched in parallel with others. If any dependency or file conflict exists, the dependent shot must wait until the shot it depends on has completed.

Wait for all actors to complete before proceeding.

### Step 3 — Capture changed files

Run the following command in the worktree to get an authoritative list of all files created or modified during this take:

```
git -C <worktree-path> status --short
```

Parse the output into a structured list of file paths with their status (A = added, M = modified, D = deleted). Store this list — it will be written into the take file in Step 7. Do not rely on actor output to infer the file list — this git query is the canonical source.

### Step 4 — Commission writer review

Spawn a writer actor to assess manuscript fidelity:

- Pass `--role writer` as arguments
- Instruct the writer to:
  1. Read the manuscript
  2. Read all output produced by the actors in this take
  3. Assess whether each action in each shot was followed, skipped, or deviated from
  4. Identify any part of the manuscript that proved unclear, incorrect, or missing information that caused deviation
  5. Return a structured Writer's Notes report using this format:

```
**Manuscript fidelity**:
- <action ref>: followed | deviated | skipped — <brief note if not followed>

**Manuscript issues**:
- <issue> → <proposed correction>
```

Wait for the writer to complete before proceeding.

### Step 5 — Review output as director

Review all actor output against:

1. **Objectives** — is each objective in the manuscript met? Treat each as pass or fail.
2. **Role definitions** — did each actor operate within their role's behavioral parameters and constraints?
3. **Background conventions** — did each actor apply the conventions from their loaded backgrounds correctly?
4. **Review findings** — if a `review.md` exists for this scene, assess whether each finding flagged by the producer has been addressed by this take's output.

For every failure found, record:
- The specific finding (what was wrong)
- The rule violated (with source — role file or background file)
- The location (file path, section, or line)
- The required change (exact, specific, actionable)

Review the full output before writing any notes. Do not flag issues mid-review.

### Step 6 — Determine take verdict

The take **passes** only if all of the following are true:
- All manuscript objectives are met
- No actor violated their role's constraints
- No actor materially violated background document conventions
- The writer found no significant manuscript fidelity failures
- All findings from `review.md` (if present) have been addressed

If any condition is not met, the take **fails**.

### Step 7 — Auto-resolve role and background issues (if any)

If the Director's Notes from Step 5 contain role violations or background convention failures, dispatch a casting-director actor to evaluate and resolve them before writing the take file.

For each affected role or background file, spawn a sub-agent:

- Pass `--role casting-director` as arguments
- Include in the agent prompt:
  - The path to the role or background file (check project-level first, then global — use whichever was loaded in Pre-work)
  - The specific Director's Notes findings for that file — the violation, the rule violated, and the required change
  - Instructions to execute the following process:
    1. Read the current role or background file in full
    2. Read the `## Refinement Log` section at the bottom of the file — if the section does not exist, treat the log as empty
    3. Evaluate each proposed change against the Refinement Log: if a proposed change **reverses a previously logged entry** (re-adds something explicitly removed, or removes something explicitly added in a prior entry), do not apply any changes — return a conflict description to the director instead
    4. If no conflicts are found, apply only the changes identified in the Director's Notes (nothing beyond the documented finding), then append a new row to the `## Refinement Log` table recording: today's date, the scene and take number, the finding that prompted the change, and a one-sentence summary of what was changed; if the section does not yet exist, add it at the bottom of the file with a table header before appending the first row
    5. Return a structured report: changes applied (with a brief description) or skipped (with the conflict reason)

Multiple affected files may be dispatched in parallel — there are no cross-file dependencies.

Wait for all casting-director actors to complete before proceeding.

If any casting-director actor returned a conflict, use AskUserQuestion to surface the conflict and pause — do not proceed to Step 8 until the user resolves it.

If no role or background issues were identified in Step 5, skip this step.

### Step 8 — Write the take file

Write the take file to the **main tree** path — not the worktree. The path is `.claude/slated/scenes/scene-<name>/takes/take-<NNN>.md` relative to the repo root (not relative to the worktree root). Use the take template structure (template at `${CLAUDE_SKILL_DIR}/../../templates/scenes/takes/take-001.md`):

- **Files Changed** — the bullet list of file paths captured in Step 3, grouped by status (Added, Modified, Deleted); this is the canonical record of what changed in this take
- **Actor Summary** — factual account of what each actor completed and skipped
- **Director's Notes** — per-actor verdict with specific findings (leave empty if no findings for an actor)
- **Writer's Notes** — the writer's fidelity report, formatted into the template sections
- **Next Take Instructions** — if the take failed, consolidate all required changes into a prioritised, actor-attributed list; draw from Director's Notes, Writer's Notes, and any unresolved `review.md` findings — all sources must be translated into specific, actor-attributed instructions, not left as abstract observations; omit this section if the take passed

### Step 9 — Dispatch writer to resolve manuscript issues (if any)

If the Writer's Notes from Step 4 contain manuscript issues — unclear action wording, missing sequencing constraints, incorrect information that caused actor deviation — spawn a writer actor to revise the manuscript before the next take:

- Pass `--role writer` as arguments
- Include in the agent prompt:
  - The path to `manuscript.md`
  - The manuscript issues identified in the Writer's Notes, verbatim
  - Instructions to read the manuscript in full, revise only the sections identified as problematic, and apply the changes directly — no user confirmation is required

Wait for the writer to complete before proceeding.

If no manuscript issues were identified, skip this step.

### Step 10 — Resolve the worktree

**If the take passed:**

1. Merge the take branch into the scene branch: `git checkout scene/<scene-name> && git merge take/<scene-name>-<NNN> --no-ff -m "take(<scene-name>): merge take-<NNN>"`
2. Return to the original branch: `git checkout -`
3. Remove the worktree: `git worktree remove .worktrees/take-<scene-name>-<NNN>`
4. Delete the take branch: `git branch -d take/<scene-name>-<NNN>`
5. Leave the scene branch (`scene/<scene-name>`) in place — do not merge it into main; it is the output branch for this scene and is ready for a PR

**If the take failed:**

1. Force-remove the worktree without merging: `git worktree remove --force .worktrees/take-<scene-name>-<NNN>`
2. Delete the take branch without merging: `git branch -D take/<scene-name>-<NNN>`

In both cases, confirm the worktree has been removed before proceeding. If any of these commands fail, surface the error to the user and stop — do not proceed to Step 11 until the worktree is fully resolved.

### Step 11 — Return verdict to user

**If the take failed:**

Surface a clear summary:

```
Take <NNN> — FAIL

The following issues must be resolved before the next take:

<Next Take Instructions, numbered>

Run /slated:shoot-take <scene-name> to begin take <NNN+1>.
Or run /slated:shoot-scene <scene-name> to let the director iterate automatically.
```

Stop here. Do not proceed automatically.

**If the take passed:**

Execute the wrap-scene process in full for the current scene — do not stop and return control to the user first. The wrap-scene process will produce `wrap.md`, update the storyboard, and open a PR. Only return control to the user after the wrap-scene process has completed.

---

## Output

- `.claude/slated/scenes/scene-<name>/takes/take-<NNN>.md` — take file with Files Changed, Actor Summary, Director's Notes, Writer's Notes, and Next Take Instructions (if failed)
- `.claude/slated/scenes/scene-<name>/manuscript.md` — updated by the writer if manuscript issues were identified (Step 9)
- Role and background files — updated by the casting-director if violations were identified (Step 7); each update appends a row to the file's `## Refinement Log` section
- Verdict returned to user: `pass` or `fail` with a summary of issues (if failed) and the next action to take

---

## Rules

- Never run a scene whose status is not `confirmed` or `in-progress`
- Never dispatch an actor whose role file does not exist in either the project or global catalogue
- Never skip the writer review — manuscript fidelity is not optional
- Never skip dispatching the writer when manuscript issues are identified — unresolved issues compound across takes
- Never write vague Director's Notes — every finding must have a rule, a location, and a required change
- Never auto-proceed to another take on a FAIL — always stop and return control to the user (except when orchestrated by `/slated:shoot-scene`, which explicitly overrides this at Step 1.1)
- Never stop on a PASS — always execute the wrap-scene process inline without returning control to the user
- Never approve a take where an actor violated their role's constraints, even if the output appears correct
- If a previous take exists, always pass the take file path to each actor — never run a subsequent take without the actor having access to their own prior notes
- Never create a take worktree directly from main — always create or reuse the scene branch (`scene/<scene-name>`) first and branch the take off of it
- Never merge a passing take into main — passing takes merge into the scene branch only; the scene branch is the PR branch and is never auto-merged to main
- Never skip Step 7 when role or background violations are found — role and background issues must be resolved inline before the take file is written, not deferred to the next take
- Never allow the casting-director to apply a change that reverses a logged refinement without explicit user confirmation — if a conflict is detected, use AskUserQuestion to surface it before proceeding
- Never proceed past Step 7 while a conflict is unresolved — the user must explicitly confirm how to handle it before the process continues
- Always clean up the worktree (Step 10) before returning the verdict to the user — never leave an orphaned worktree or stale take branch
- Never allow actors to write to `.claude/slated/` inside the worktree — framework artefacts are never produced in a worktree context
- Never write the take file inside the worktree — the take file is always written to the main tree path
- Never infer the file list from actor output — always derive it from `git status --short` in the worktree (Step 3); actor descriptions may be incomplete or imprecise
