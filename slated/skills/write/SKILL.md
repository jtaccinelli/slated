---
name: write
description: Plan a new scene or refine an existing manuscript — producing a confirmed manuscript.md ready to shoot.
expected-role: writer
argument-hint: "[scene-name | brief description of the feature or fix]"
---

# Write

Translate a requirement into a complete, confirmed `manuscript.md`, or identify and correct gaps in an existing one. A successful run leaves the scene directory populated, the manuscript at `confirmed` status, and a clean pre-shoot review — ready for the first take.

---

## Pre-work

1. Run `/slated:perform writer` to adopt the writer role.

2. Resolve the home directory: run `echo $HOME`.

3. Discover available roles and backgrounds:
   - Crew roles: Glob `${CLAUDE_SKILL_DIR}/../perform/roles/role-*.md`
   - Cast roles: Glob `.claude/slated/roles/role-*.md` (project) and `ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null` (global); merge with project taking precedence
   - Backgrounds: Glob `.claude/slated/backgrounds/background-*.md` (project) and `ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null` (global); merge with project taking precedence
   - Read each role and background in full — casting decisions must be grounded in what actually exists

4. Read `.claude/slated/locations.md` if it exists — the named shot locations it contains must be used when specifying actor actions so actors have unambiguous file paths.

5. Use AskUserQuestion: "Are you creating a new scene or refining an existing manuscript?"
   - If **refining**: ask which scene. Resolve `.claude/slated/scenes/scene-<name>/manuscript.md` and read it in full. Proceed to the Refine path.
   - If **creating**: use `$ARGUMENTS` as a starting point if provided. Proceed to the Create path.

---

## Process — Create Path

### Step 1 — Establish the requirement

Use `$ARGUMENTS` as a starting point. If it is sufficient to understand the scope, proceed. If it is vague, ask focused questions:

1. **What is being built or fixed?** — One clear sentence.
2. **What type of change is this?** — New feature, bug fix, refactor, infrastructure, content, or other.
3. **What does success look like?** — What will be true when this scene is complete that is not true now? Push for verifiable outcomes, not intentions.
4. **Are there constraints or edge cases to know about up front?** — Anything that would affect how the work is sequenced or who does it.

Do not proceed to casting until these are clear.

### Step 2 — Name the scene

Derive a short, descriptive kebab-case name from the requirement. Use AskUserQuestion to confirm: "The scene will be named `scene-<name>`. Does that look right?"

### Step 3 — Cast the scene

Dispatch a director actor to determine the cast:

- Pass `--role director` as arguments to the actor agent
- Include in the agent prompt:
  - The requirement established in Step 1
  - The full list of available cast roles and their summaries
  - The full list of available backgrounds and their descriptions
  - An instruction to propose: which roles are needed, which backgrounds each actor should load, and what external resources are required
  - An instruction to flag any required role or background that does not exist in the catalogue

Wait for the director actor to complete. Review the proposed cast.

**If any required role is missing**: draft it autonomously — follow the cast-role definition process (identity, background narrative, expertise, behavioral parameters, constraints, interactions) derived from the requirement and project context. Ask whether it should be global or project-specific. Write to the confirmed location before proceeding.

**If any required background is missing**: draft it autonomously — populate all three sections (Semantics, Structure, Function) from the project context and available knowledge. Ask whether it should be global or project-specific. Write to the confirmed location before proceeding.

Surface a brief note listing every role or background that was auto-created.

### Step 4 — Define objectives

Write 3–6 verifiable objectives. Each must describe a concrete outcome the director can confirm or deny after a take. Avoid vague language like "the feature works" or "the UI is complete".

Good objective: `User can submit a form at /items/create and see the new item appear in the list`
Bad objective: `Form submission is implemented`

### Step 5 — Sequence the actions

Break the work into ordered shots, each assigned to a specific actor. Within each shot, list atomic actions — what the actor must produce, not how they produce it.

Rules for actions:
- Each action has a clear, observable output
- Actions within a shot are sequential; shots themselves may be sequential or parallel
- Reference location names from `locations.md` when specifying file targets — do not use raw paths that are already named
- Do not describe implementation methods — describe outcomes
- If a shot depends on output from a previous shot, say so explicitly

### Step 6 — Enter plan mode and draft the manuscript

Call `EnterPlanMode`. Write the full manuscript draft to the plan file using the template at `${CLAUDE_SKILL_DIR}/templates/manuscript.md`. The plan file is the manuscript — populate every section:

- Scene name and status (`confirmed`)
- Cast table
- Shot locations (derived from `locations.md` — include only entries relevant to this scene's work)
- Resources
- Objectives
- Actions (shots and steps)
- Notes
- HTML comment block and stub placeholder for the Completion Record

Once the plan file is written, call `ExitPlanMode` to surface the manuscript draft to the user for approval. Do not write any scene files until the user approves the plan.

### Step 7 — Write the scene files

Once the plan is approved, create the scene directory and files:

1. Create `.claude/slated/scenes/scene-<name>/` directory
2. Create `.claude/slated/scenes/scene-<name>/takes/` directory
3. Write `.claude/slated/scenes/scene-<name>/manuscript.md` using the template at `${CLAUDE_SKILL_DIR}/templates/manuscript.md`, with status set to `confirmed`

Include the HTML comment block and stub placeholder for the Completion Record — do not populate the record data. The director replaces these with the completed Completion Record during wrap.

### Step 8 — Pre-shoot review

Run the review loop to validate the manuscript before it reaches the shoot stage. This step runs fully automatically — no user intervention at any point.

Resolve role file paths for the cast before starting: check project-level first, then global, then crew (in `${CLAUDE_SKILL_DIR}/../perform/roles/`). Use whichever is found first.

#### Part 1 — Director role review loop

Repeat until the director returns `PASS` or 3 iterations have completed:

1. Dispatch a director actor. Pass:
   - The path to the manuscript
   - The resolved paths to all role files in the cast
   - An instruction to evaluate whether each role's definition gives the actor enough to execute the task described without guessing — not whether the role is wrong for the job, but whether it is sufficiently detailed for this specific requirement
   - An instruction to return a structured verdict: `PASS` if all roles are adequate, or `FAIL` with specific per-role findings describing exactly what is missing

2. If `FAIL`: dispatch a director actor again. Pass:
   - The path to the manuscript
   - The resolved paths to the flagged role files only
   - The director's findings for each flagged role, verbatim
   - An instruction to revise each flagged cast role definition in place — adding only the missing detail identified in the findings; crew roles are not touched

   Wait for the director actor to complete. Use the updated role files in the next iteration.

3. If `PASS`: exit the loop and proceed to Part 2.

If the iteration cap is reached without `PASS`: collect the director's final findings and skip Part 2. Proceed to the Outcome step.

#### Part 2 — Actor assumption review

For each actor in the cast, dispatch the actor agent with `--role <role-name> --backgrounds <bg-list>`. Pass:
- The path to the manuscript
- The actor's assigned shots only (extracted from the Actions section)
- An instruction to outline every assumption they would need to make to complete their shots — places where the manuscript does not give them enough to act without deciding for themselves

Collect the assumption list returned by each actor.

#### Part 3 — Director evaluation of assumptions

Dispatch a director actor. Pass:
- The path to the manuscript
- The resolved paths to all role files in the cast
- All actor assumption reports from Part 2
- An instruction to categorise each assumption as one of:
  - `ROLE GAP` — the role definition should specify how to handle this
  - `MANUSCRIPT GAP` — the manuscript action is underspecified; the writer needs to add detail
  - `ACCEPTABLE` — a reasonable inference that does not warrant a change
- An instruction to return a structured verdict: `CLEAN` if no ROLE GAP or MANUSCRIPT GAP findings exist, or `FLAGGED` with each non-acceptable item categorised and attributed to actor and shot

#### Outcome

**Step A — Resolve manuscript gaps:**

If Part 3 returned `FLAGGED` with any `MANUSCRIPT GAP` findings, dispatch a writer actor. Pass:
- The path to the manuscript
- The `MANUSCRIPT GAP` findings verbatim, each attributed to actor and shot
- An instruction to fix those gaps by revising only the affected Actions (and Objectives if applicable) in the manuscript — eliminating the ambiguity that caused each assumption
- An instruction not to touch the Pre-Shoot Review section or any other sections

Wait for the writer to complete before proceeding.

**Step B — Document unresolved role gaps:**

If the director role review loop ended with findings (cap reached), or Part 3 returned `FLAGGED` with any `ROLE GAP` findings:

Dispatch a writer actor. Pass:
- The path to the manuscript
- All remaining findings: director role assessment (if the Part 1 cap was reached) and any `ROLE GAP` actor assumptions from Part 3
- An instruction to replace the `## Pre-Shoot Review` section in the manuscript with the consolidated feedback — populating the Director Role Assessment table and the Actor Assumptions table (ROLE GAP items only) as applicable, and removing the HTML comment placeholder

If all parts returned clean verdicts and all manuscript gaps were resolved in Step A, remove the entire Pre-Shoot Review section from the manuscript — including the HTML comment block.

---

## Process — Refine Path

### Step 1 — Identify what needs to change

Read the existing manuscript in full. Use AskUserQuestion: "What was missing or inaccurate in this manuscript?"

Establish:
1. Which sections need updating — cast, objectives, actions, notes, or pre-shoot review
2. Whether any roles or backgrounds referenced in the cast need to be corrected

### Step 2 — Enter plan mode and draft the revised manuscript

Call `EnterPlanMode`. Write the full updated manuscript to the plan file, incorporating all identified corrections. Call `ExitPlanMode` to surface the revised draft to the user for approval. Do not write any files until the plan is approved.

### Step 3 — Write and re-review

Write the updated manuscript. Run the full pre-shoot review (Steps 8, Parts 1–3 from the Create path) against the revised manuscript.

---

## Output

- `.claude/slated/scenes/scene-<name>/manuscript.md` — complete manuscript at `confirmed` status
- `.claude/slated/scenes/scene-<name>/takes/` — empty directory ready for take files
- Pre-Shoot Review section populated if issues were found; removed entirely if the review returned clean
- Any auto-created role or background files written to their confirmed locations

Report the scene folder path, the number of actors cast, the number of objectives defined, and the pre-shoot review outcome (clean or flagged with a summary count of findings).

---

## Rules

- Never write files before the user approves the plan — EnterPlanMode and ExitPlanMode are the confirmation gate, not AskUserQuestion
- Never leave the Shot Locations section empty if `.claude/slated/locations.md` exists — always include the entries relevant to this scene's work
- Never cast a role that does not exist in the merged role catalogue — draft it autonomously before continuing
- Never assign an action to a role whose constraints prohibit it
- Never write objectives that cannot be verified by observation
- Never populate the Completion Record data in a newly written manuscript — include the HTML comment block and stub as placeholders only
- Never prompt the user during the pre-shoot review loop — it runs fully automatically
- Never skip the pre-shoot review — it runs after every confirmed manuscript
- Never document a MANUSCRIPT GAP finding in the Pre-Shoot Review section — fix it in the manuscript directly during the Outcome Step A; only ROLE GAP findings are documented
- Never modify crew roles during the pre-shoot review loop — only cast role definitions may be updated
