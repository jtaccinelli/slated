---
name: {{SHOWREEL_NAME}}
description: {{TASK_DESCRIPTION}}
expected-role: director
argument-hint: "[brief description of what to build or fix]"
---

# {{SHOWREEL_NAME}}

{{TASK_DESCRIPTION}}

This showreel runs entirely in a single context — no sub-agents, no worktrees, no shell. The director plans the work, casts from the bundled roles, runs up to three take cycles inline, self-assesses after each take, and wraps on a passing take.

---

## Bundled Cast

The following cast roles are available in this showreel. All are loaded from `${CLAUDE_SKILL_DIR}/roles/`.

- `director` — loaded from `${CLAUDE_SKILL_DIR}/roles/role-director.md`
- `writer` — loaded from `${CLAUDE_SKILL_DIR}/roles/role-writer.md`
{{CAST_MANIFEST}}

---

## Bundled Backgrounds

The following backgrounds are available in this showreel. All are loaded from `${CLAUDE_SKILL_DIR}/backgrounds/`.

{{BACKGROUND_MANIFEST}}

---

## Pre-work

1. **Adopt the director role.** Read `${CLAUDE_SKILL_DIR}/roles/role-director.md` in full. Internalise every section — Identity, Background, Expertise, Behavioral Parameters, Constraints, Interactions. Operate as the director from this point forward. Do not announce the transition.

2. **Load all backgrounds.** For each background in the Bundled Backgrounds list above, read the corresponding file from `${CLAUDE_SKILL_DIR}/backgrounds/` in full. Absorb each as domain knowledge — available to all cast roles for the duration of this session. Skip this step if the list reads `(none)`.

3. **Establish the requirement.** If `$ARGUMENTS` is provided and sufficient, use it as the requirement. If vague or absent, use AskUserQuestion to establish:
   - What is being built or fixed — one clear sentence.
   - What does success look like — what will be observably true when the task is complete?

4. **Name the work.** Derive a short, descriptive label for the task — used for internal tracking only.

---

## Planning Phase

Operating as director.

### Step 1 — Cast the task

From the Bundled Cast list, identify which role(s) are needed to execute this task. Consider:
- What type of output does the task require?
- Which role's domain and constraints match that output?
- Are multiple roles needed, or is this a single-actor task?

For each actor needed:
- Assign a role from the Bundled Cast
- Identify which loaded backgrounds are relevant to their work
- Define which part of the task they own

If the task requires a role not in the Bundled Cast, stop and surface this as a limitation:

```
This showreel does not include a role suited to <task type>.
Bundled cast: {{CAST_NAMES}}
The task cannot proceed as described. Adjust the requirement or use a different showreel.
```

Return control to the user.

### Step 2 — Define objectives

Write 2–5 verifiable objectives. Each must describe a concrete, observable outcome. Vague outcomes are not objectives.

Good objective: `The component renders correctly with all required props and handles the empty state`
Bad objective: `The component is implemented`

### Step 3 — Sequence the shots

Break the work into ordered shots, one per actor. For each shot:
- **Actor**: which role executes it
- **Actions**: 2–6 atomic steps, each with a clear observable output
- **Dependencies**: if this shot depends on output from a prior shot, state it explicitly

Present the plan:

```
Plan: <task label>

Objectives:
- [ ] <objective>
- [ ] <objective>

Cast:
- actor-1: <role-name>

Shots:
Shot 1 — <title> (actor-1)
  1. <action — what to produce and what the output should be>
  2. <action>
```

Use AskUserQuestion: "Does this plan look correct before I begin?" Do not proceed until confirmed.

---

## Take Loop

Maximum **3 takes**. The loop runs entirely in this context — no sub-agents are spawned.

At the start of each take, note the take number internally (Take 1, Take 2, Take 3).

### Phase A — Adopt the cast role

Read `${CLAUDE_SKILL_DIR}/roles/role-<name>.md` for the first actor in the cast. Internalise every section. Operate as that role. Do not announce the transition.

If this is Take 2 or Take 3: before executing, review the Director's Notes from the previous take cycle. Apply every required change listed for this role. If a required change is unclear, state it explicitly before proceeding — do not silently skip it.

### Phase B — Execute the shot

Execute the actor's assigned shots as the adopted role. For each action:
- Produce the output described
- If file tools are available, write outputs to their target paths; if not, produce the content inline with clear file path labels (e.g. `--- src/components/Foo.tsx ---`)
- If an action cannot be completed, state why explicitly before moving to the next — do not silently skip

### Phase C — Additional actors (if multi-actor)

If additional actors are in the cast, repeat Phase A and Phase B for each sequentially. Each subsequent actor reads the output of prior shots before beginning their own work.

### Phase D — Writer review

Read `${CLAUDE_SKILL_DIR}/roles/role-writer.md` in full. Internalise every section. Operate as the writer.

Assess:
1. Were all planned actions executed? For each action: followed / deviated / skipped — with a brief note if not followed.
2. Were any actions unclear, incorrect, or missing information that caused the actor to guess?

Produce Writer's Notes:

```
Writer's Notes — Take <N>

Manuscript fidelity:
- Shot 1, Action 1: followed | deviated | skipped — <note if not followed>
- Shot 1, Action 2: followed | deviated | skipped — <note if not followed>

Manuscript issues:
- <issue> → <proposed correction>
  (omit if none)
```

### Phase E — Director self-assessment

Re-read `${CLAUDE_SKILL_DIR}/roles/role-director.md` in full. Internalise every section. Operate as the director.

Review the take output against:

1. **Objectives** — is each objective from the Planning Phase met? Pass or fail per objective.
2. **Role definitions** — did each actor operate within their role's behavioral parameters and constraints? Check the role file for each actor against what they produced.
3. **Background conventions** — did each actor apply the loaded background conventions correctly?

Review the full output before writing notes. Do not flag issues mid-review.

For every failure, record:
- The specific finding (what was wrong)
- The rule violated (with source — role file section or background file section and name)
- The location (file path, section, or output block)
- The required change (exact and actionable)

Produce Director's Notes:

```
Director's Notes — Take <N>

actor-1 (<role-name>): pass | fail
- Finding: <what was wrong>
  Rule violated: <specific rule — source: role-<name>.md § Constraints, or background-<name>.md § Semantics, etc.>
  Location: <file path or output section>
  Required change: <exactly what must be different on the next take>

(omit findings block entirely if verdict is pass)
```

### Phase F — Take verdict

The take **passes** only if all of the following are true:
- All objectives are met
- No actor violated their role's constraints
- No actor materially violated background conventions
- The writer found no significant fidelity failures

Surface the verdict:

```
Take <N> — PASS
```

or

```
Take <N> — FAIL

Required changes before next take:
1. <specific instruction attributed to actor-1>
2. <specific instruction>
```

**If the take passed**: proceed immediately to the Wrap Phase. Do not return control to the user.

**If the take failed and the limit has not been reached**: surface the verdict above, then proceed immediately to Phase A for Take N+1. Do not return control to the user between takes.

**If Take 3 failed**: proceed to the Blocked Summary.

---

## Wrap Phase

Executed immediately after a passing take. No user confirmation required.

### Wrap Step 1 — Compile the completion record

Gather from the take cycles:
- Takes required: total count
- Take log: number and verdict for each; primary failure reason in one sentence for failed takes
- Repeated patterns across takes that suggest a gap in a bundled role or background — omit if none emerged

### Wrap Step 2 — Surface the wrap summary

```
{{SHOWREEL_NAME}} — COMPLETE

<2–4 sentences describing what was produced — what capability was added, what problem was solved, or what structure was established. Written to be understood by someone with no prior context on this task. No implementation detail.>

Takes required: <N>
Take log:
  Take 1: pass | fail — <primary reason if fail>
  Take 2: pass | fail — <primary reason if fail; omit if not run>
  Take 3: pass | fail — <primary reason if fail; omit if not run>
```

### Wrap Step 3 — List outputs

If file tools were available and files were written:

```
Files written:
  <file-path> — <one-line description of what this file is or does>
  <file-path> — <one-line description>
```

If no file tools were available, list what was produced inline:

```
Content produced:
  <label> — <one-line description of what was produced>
```

### Wrap Step 4 — Surface improvement notes (if any)

If repeated failures across takes suggest a gap in a bundled role or background:

```
Improvement notes:
- role-<name>: <specific gap that caused repeated errors>
- background-<name>: <convention that was missing or unclear>
```

These are informational only — a deployed showreel cannot modify its own bundled artefacts. To act on these findings: update the source files in the framework and regenerate this showreel with `/slated:showreel`.

---

## Blocked Summary

Reached only if Take 3 fails.

```
{{SHOWREEL_NAME}} — BLOCKED

The take limit (3) was reached without a passing take.

Repeated failure reason(s):
<concise summary of issues that persisted across takes, drawn from Director's Notes>

Options:
- Clarify the requirement and run the skill again
- If a bundled role is underspecified for this task, update it in the framework and regenerate: /slated:showreel
- If a required background is missing or insufficient, update it in the framework and regenerate: /slated:showreel
```

---

## Rules

- Never exceed 3 takes — hard stop at the limit; surface the Blocked Summary
- Never skip Phase D (writer review) — it runs after every take, without exception
- Never skip Phase E (director self-assessment) — it runs after every writer review, without exception
- Never approve a take that violates role constraints or background conventions, even if the output otherwise looks reasonable
- Never return control to the user between a failed take and the next take — proceed automatically
- Never return control to the user between a passing take and the wrap — wrap runs immediately
- Never attempt to use a role not in the Bundled Cast — surface the limitation and stop
- Never modify bundled role or background files — they are read-only artefacts in a deployed showreel
- Director adoption in Pre-work is not optional — nothing proceeds until the director role is read and internalised
- Role re-adoption at each phase transition is not announced — read the file and become the role
- Wrap improvement notes are informational only — the showreel cannot modify its own bundled artefacts
