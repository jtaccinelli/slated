---
name: write-scene
description: Draft a new scene from a requirement or idea. Asks clarifying questions, produces a manuscript.md, creates the scene folder structure, and registers it in the storyboard. Use whenever a new feature or fix needs to be planned before actors run it.
expected-role: writer
argument-hint: [brief description of the feature or fix]
---

# Write Scene

Translate a requirement into a complete, confirmed `manuscript.md` and register the new scene in the storyboard. Nothing is written until the requirement is understood and the cast is validated.

---

## Pre-work

Read the available roles and backgrounds so casting decisions are grounded in what actually exists:

- Use Bash (`ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null`) to list global roles, and Glob `.claude/slated/roles/role-*.md` for project-level ones — merge both results, with project-level taking precedence; read each to understand available roles and their expertise
- Use Bash (`ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`) to list global backgrounds, and Glob `.claude/slated/backgrounds/background-*.md` for project-level ones — merge both results, with project-level taking precedence; read each in full; background content determines relevance, not just name
- Read `.claude/slated/scenes/storyboard.md` — understand what scenes already exist to avoid duplication
- If `.claude/slated/set/locations.md` exists, read it — the named shot locations it contains must be used when specifying actor actions so actors have unambiguous file paths to work from

---

## Process

### Step 1 — Establish the requirement

Use `$ARGUMENTS` as a starting point. If it is sufficient to understand the scope, proceed. If it is vague, ask focused questions to establish:

1. **What is being built or fixed?** — One clear sentence.
2. **What type of change is this?** — New feature, bug fix, refactor, infrastructure, content, or other.
3. **What does success look like?** — What will be true when this scene is complete that is not true now? Push for verifiable outcomes, not intentions.
4. **Are there constraints or edge cases to know about up front?** — Anything that would affect how the work is sequenced or who does it.

Do not proceed to casting until these are clear.

### Step 2 — Name the scene

Derive a short, descriptive kebab-case name from the requirement. This becomes the folder name: `scene-<name>`.

Present the name to the user for confirmation before creating anything.

### Step 3 — Cast the scene

Based on the requirement, determine:

1. **Which roles are needed?** — Match the work to roles whose expertise and constraints fit. Do not assign work to a role that its constraints prohibit.
2. **Which backgrounds should each actor load?** — Select from the available backgrounds. Only include backgrounds directly relevant to the work the actor will do.
3. **What external resources are required?** — APIs, services, databases, third-party platforms.

Present the proposed cast to the user. If a required role or background does not exist, draft it autonomously before proceeding:

- **Missing role**: follow the cast-role process inline — draft a complete role definition (identity, background narrative, expertise, behavioral parameters, constraints, interactions) derived from the manuscript requirement and the project context; ask the user whether it should be global or project-specific; write the file to the confirmed location.
- **Missing background**: follow the establish-background process inline — draft all three sections (Semantics, Structure, Function) from the project context and available backgrounds; ask the user whether it should be global or project-specific; write the file to the confirmed location.

Surface a brief note listing every role or background that was auto-created before proceeding to Step 4. Do not proceed to Step 4 until the cast is fully satisfied.

### Step 4 — Define objectives

Write 3–6 verifiable objectives. Each must describe a concrete outcome that the director can confirm or deny after a take. Avoid vague language like "the feature works" or "the UI is complete".

Good objective: `User can submit a form at /items/create and see the new item appear in the list`
Bad objective: `Form submission is implemented`

### Step 5 — Sequence the actions

Break the work into ordered shots, each assigned to a specific actor. Within each shot, list atomic actions — what the actor must produce, not how they produce it.

Rules for actions:
- Each action has a clear, observable output
- Actions within a shot are sequential; shots themselves may be sequential or parallel
- Do not describe implementation methods — describe outcomes
- If a shot depends on output from a previous shot, say so explicitly

### Step 6 — Draft and confirm

Present the full manuscript to the user before writing anything:

- Scene name
- Cast table
- Resources
- Objectives
- Actions (shots and steps)
- Any notes worth preserving

Ask for confirmation or adjustments. Do not write files until confirmed.

### Step 7 — Write the scene files

Once confirmed, create the scene directory and files:

1. Create `.claude/slated/scenes/scene-<name>/` directory
2. Create `.claude/slated/scenes/scene-<name>/takes/` directory
3. Write `.claude/slated/scenes/scene-<name>/manuscript.md` with status set to `confirmed`

Use the manuscript template structure (template at `${CLAUDE_SKILL_DIR}/../../templates/scenes/manuscript.md`), including the HTML comment block and stub placeholder for the Completion Record — do not populate the record data. The director replaces the comment and stub with the completed Completion Record during finalisation.

### Step 8 — Pre-shoot review

Run the review loop to validate the manuscript before it reaches the shoot stage. This step is fully automatic — no user intervention at any point.

Resolve role file paths before starting: for each role in the cast, check `.claude/slated/roles/role-<name>.md` (project-level) first, then `~/.claude/slated/roles/role-<name>.md` (global). Use whichever is found first.

#### Part 1 — Director role review loop

Repeat until the director returns `PASS` or 3 iterations have completed:

1. Dispatch the actor agent with `--role director`. Pass:
   - The path to the manuscript
   - The resolved paths to all role files in the cast
   - An instruction to evaluate whether each role's definition gives the actor enough to execute the task described without guessing — not whether the role is wrong for the job, but whether it is sufficiently detailed for this specific requirement
   - An instruction to return a structured verdict: `PASS` if all roles are adequate, or `FAIL` with specific per-role findings describing exactly what is missing

2. If `FAIL`: dispatch the actor agent with `--role casting-director`. Pass:
   - The path to the manuscript
   - The resolved paths to the flagged role files only
   - The director's findings for each flagged role, verbatim
   - An instruction to revise each flagged role definition in place to address the specific gaps — not to rewrite the role wholesale, only to add the missing detail

   Wait for the casting-director to complete. Use the updated role files in the next iteration.

3. If `PASS`: exit the loop and proceed to Part 2.

If the iteration cap is reached without `PASS`: collect the director's final findings and skip Part 2. Proceed to the Outcome step.

#### Part 2 — Actor assumption review

For each actor in the cast, dispatch the actor agent with `--role <role-name> --backgrounds <bg-list>`. Pass:
- The path to the manuscript
- The actor's assigned shots only (extracted from the Actions section)
- An instruction to outline every assumption they would need to make to complete their shots — places where the manuscript does not give them enough to act without deciding for themselves

Collect the assumption list returned by each actor.

#### Part 3 — Director evaluation of assumptions

Dispatch the actor agent with `--role director`. Pass:
- The path to the manuscript
- The resolved paths to all role files in the cast
- All actor assumption reports from Part 2
- An instruction to categorise each assumption as one of:
  - `ROLE GAP` — the role definition should specify how to handle this
  - `MANUSCRIPT GAP` — the manuscript action is underspecified; the writer needs to add detail
  - `ACCEPTABLE` — a reasonable inference that does not warrant a change
- An instruction to return a structured verdict: `CLEAN` if no ROLE GAP or MANUSCRIPT GAP findings exist, or `FLAGGED` with each non-acceptable item categorised and attributed to actor and shot

#### Outcome

If the director role review loop ended with findings (cap reached), or Part 3 returned `FLAGGED`:

Dispatch the actor agent with `--role writer`. Pass:
- The path to the manuscript
- All findings: director role assessment (if any) and categorised actor assumptions (if any)
- An instruction to replace the `## Pre-Shoot Review` section in the manuscript with the consolidated feedback — populating the Director Role Assessment table and the Actor Assumptions table as applicable, and removing the HTML comment placeholder

If all parts returned clean verdicts, remove everything from the `<!--` comment block above `## Pre-Shoot Review` through to the end of the `### Actor Assumptions` table inclusive — this entire block serves no purpose if the review passed.

Either way, proceed to Step 9.

---

### Step 9 — Dispatch the visualiser

Spawn a sub-agent using the actor agent:

- Pass `--role visualiser` as arguments
- Include in the agent prompt:
  - The path to the completed `manuscript.md`
  - The path to `.claude/slated/scenes/storyboard.md`
  - The scene directory path
  - Instructions to:
    1. Read the manuscript to extract the scene name, status, and one-line requirement
    2. Read the current storyboard in full
    3. Add the new scene to the Pending section of `storyboard.md`:
       ```
       | `scene-<name>` | confirmed | <one-line requirement> |
       ```

Wait for the visualiser to complete before proceeding.

---

## Output

- `.claude/slated/scenes/scene-<name>/manuscript.md` — complete manuscript at `confirmed` status; Pre-Shoot Review section populated if issues were found, removed if clean
- `.claude/slated/scenes/scene-<name>/takes/` — empty directory ready for take files
- `.claude/slated/scenes/storyboard.md` — updated Pending section
- Role files in the cast may be revised in place by the casting-director during the pre-shoot review loop

Report the scene folder path, the number of actors cast, the number of objectives defined, and the pre-shoot review outcome (clean or flagged with a summary count of findings).

---

## Rules

- Never write files before the user confirms the manuscript
- Never leave the Shot Locations section empty if `.claude/slated/set/locations.md` exists — always include the entries relevant to this scene's work
- Never cast a role that does not exist in the merged role catalogue — draft it autonomously using the cast-role process before continuing
- Never assign an action to a role whose constraints prohibit it
- Never write objectives that cannot be verified by observation
- Never populate the Completion Record data in a newly written manuscript — include the HTML comment block and stub as placeholders only
- Never prompt the user during the pre-shoot review loop — it runs fully automatically
- Never skip the pre-shoot review — it runs after every confirmed manuscript before the visualiser is dispatched
