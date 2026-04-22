---
name: review-project
description: Unified post-production QA — reviews the set build if unreviewed, then reviews all completed scenes without a review record in storyboard order. Produces a review file per subject and a project-level report aggregating all verdicts.
expected-role: producer
argument-hint: []
---

# Review Project

Conduct post-production QA across the entire project. The producer determines what needs reviewing — the set build, outstanding scenes, or both — and works through each in order. Every subject gets a full two-track review: static analysis against documented conventions, and interactive verification in the running application. The producer does not fix — they identify, locate, and commission.

---

## Pre-work

1. Check for `.claude/slated/set/outline.md` — if it exists, the set was defined and needs reviewing unless `.claude/slated/set/review.md` already exists
2. Read `.claude/slated/scenes/storyboard.md` — identify all scenes with `complete` status
3. For each complete scene, check whether `.claude/slated/scenes/scene-<name>/review.md` exists — collect scenes that have no review
4. If neither the set nor any scene needs reviewing, report the project state and stop:
   ```
   Nothing to review.
   Set: reviewed ✓
   Scenes: <N> reviewed ✓
   ```
5. Use Bash (`ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`) to list global backgrounds, and Glob `.claude/slated/backgrounds/background-*.md` for project-level ones — merge both results, with project-level taking precedence; hold all available for convention checks across all reviews

---

## Process

### Phase 1 — Set Review (if `.claude/slated/set/review.md` does not exist)

#### Step 1 — Read the set record

1. Read `.claude/slated/set/outline.md` in full — this is what the set-designer was instructed to build
2. Read the project's `CLAUDE.md` — this is the set-designer's description of what they built
3. Read every background listed in the outline's tech stack

#### Step 2 — Verify the build against the outline

For each item in the outline (directory structure, configuration files, entry points, dependencies, bindings), verify that the corresponding artefact exists and reflects the outline's specification.

Mark each as:
- **Present and correct** — the artefact exists and matches the outline and relevant background conventions
- **Present but incorrect** — the artefact exists but deviates from the outline or background
- **Absent** — the artefact was specified in the outline but not produced

#### Step 3 — Check convention compliance

For each background in the tech stack, assess whether the built artefacts are consistent with its documented conventions.

Flag every deviation. Partial compliance is not compliance.

#### Step 4 — Interactive verification

Confirm the application is running locally. If it is not running, attempt to start it using the dev command in the project's `package.json` scripts. If it cannot be started, record `Application startup: fail` as a finding and proceed without the remaining checks in this step.

Navigate the running application to confirm it starts, routes resolve, and the basic integration points function. This is not a feature test — the set has no user objectives. This is a structural check: the scaffold must be capable of supporting scene work immediately.

Check:
1. Application starts without errors
2. Root route resolves
3. Any runtime bindings (D1, KV, etc.) are reachable
4. No console errors on cold start

#### Step 5 — Write the set review

Write `.claude/slated/set/review.md` using this structure (reference template at `${CLAUDE_SKILL_DIR}/../../templates/scenes/review.md`):

```markdown
# Set Review

**Date**: <YYYY-MM-DD>
**Verdict**: pass | flag

---

## Outline Compliance

| Item | Status | Notes |
|---|---|---|
| <outline item> | present and correct | — |
| <outline item> | present but incorrect | <what is wrong> |
| <outline item> | absent | <what is missing> |

---

## Convention Compliance

### <Background Name>

- **Finding**: <what deviates>
  **Convention**: <specific rule>
  **Location**: <file>
  **Required change**: <what must change>

*(omit if no deviations)*

---

## Interactive Verification

- Application starts: pass | fail
- Root route resolves: pass | fail
- Runtime bindings reachable: pass | fail | n/a
- Console errors on cold start: none | <errors found>

---

## Findings

1. <finding — source — required action>

*(omit if verdict is pass)*
```

The set review **passes** only if all outline items are present and correct, no convention deviations were found, and the application starts cleanly. Otherwise the verdict is **flag**.

If flagged: surface the findings to the user, instruct them to run `/slated:build-set` to action corrections before the set review can be re-run, and **stop**. Do not proceed to Phase 2 — scene reviews conducted on a defective scaffold may produce unreliable findings.

---

### Phase 2 — Scene Reviews (for each complete scene without a review)

Work through scenes in storyboard order. For each:

#### Step 1 — Read the scene record

1. Read `.claude/slated/scenes/scene-<name>/manuscript.md` in full — requirement, objectives, cast, and Completion Record
2. Read `.claude/slated/scenes/scene-<name>/wrap.md` in full — description, file list, and editor notes
3. Read the `wrap.md` of every other completed scene — build a picture of the project's current state and what each scene established
4. Read each background relevant to the technologies this scene touched

#### Step 2 — Verify objectives against the scene record

For each objective in the manuscript, assess whether the scene record provides clear evidence it was met.

Mark each as:
- **Met** — `wrap.md` and the Completion Record clearly demonstrate the objective was achieved
- **Unverified** — the output exists but the record does not provide sufficient evidence; hand to the interactive track
- **Unmet** — the objective is not reflected in the scene's output at all

Unverified is not a pass — it must be confirmed through live testing.

#### Step 3 — Check convention compliance

For each relevant background, assess whether the output described in `wrap.md` and the editor notes is consistent with its documented conventions.

Flag every deviation.

#### Step 4 — Map the regression surface

From the scene's file list, identify every file that was modified (as opposed to created). For each:

1. Check whether any other completed scene's file list includes the same file
2. Check whether any other completed scene's editor notes flag a dependency on that area
3. Check whether any background describes the file as shared or foundational

For every affected scene identified, assess whether this scene's changes are consistent with what it established.

#### Step 5 — Navigate the application as a user

Confirm the application is still running before beginning this step. If it has stopped, attempt to restart it using the dev script from `package.json`. If it cannot be restarted, surface the error to the user and wait before proceeding.

Approach this as a user encountering the application for the first time — not as a technician inspecting implementation.

If browser automation is unavailable, surface a manual verification checklist and wait for the user to report results:

```
Browser automation is unavailable. Manual verification required:

<for each objective, list the URL or flow to navigate and the expected outcome>

Confirm each result before the review report is written.
```

If browser automation is available:

For each objective in the manuscript:
1. Navigate to where the objective would be experienced
2. Execute the user flow
3. Confirm the outcome matches the objective
4. For any objective marked **Unverified**, this check is definitive

Then spot-check flows from other completed scenes whose underlying files overlap with this scene's changes.

Note any visual, functional, or navigational issues not captured in the objectives.

#### Step 6 — Write the scene review

Write `.claude/slated/scenes/scene-<name>/review.md` using the scene review template structure (template at `${CLAUDE_SKILL_DIR}/../../templates/scenes/review.md`).

The scene review **passes** only if all objectives are met or confirmed through live testing, no convention deviations were found, no cross-scene inconsistencies were found, and no broken flows were found.

If flagged: overwrite the `review.md` just written (it is the canonical finding record for this run), set the manuscript `**Status**` field back to `in-progress`, and dispatch the visualiser to move the scene from Completed back to Pending in `storyboard.md` — this restores the scene to its pre-wrap state so the re-shoot and re-wrap flow works correctly. The existing `wrap.md` remains in place — it will be overwritten when the scene is re-wrapped after corrections; a future re-run of `/slated:review-project` will read the new `wrap.md` from the corrected re-wrap. Surface findings to the user and instruct them to run `/slated:shoot-take <scene-name>` or `/slated:shoot-scene <scene-name>` to action the findings, then re-run `/slated:review-project` after the scene is re-finalised.

Do not proceed to the next scene until the current scene's review file is written.

---

### Phase 3 — Write the project report

After all subjects have been reviewed in this run, write `.claude/slated/report.md`:

```markdown
# Project Review Report

**Date**: <YYYY-MM-DD>
**Reviewed by**: producer

---

## Summary

| Subject | Type | Verdict |
|---|---|---|
| Set | set | pass | flag |
| <scene-name> | scene | pass | flag |

---

## Outstanding

Subjects not yet reviewed (to be addressed in future runs):

- <scene-name> — complete, awaiting review

*(omit if nothing is outstanding)*

---

## Action Required

<For each flagged subject, one-line summary of what must be addressed>

*(omit if all verdicts are pass)*
```

---

## Output

- `.claude/slated/set/review.md` — set review (if the set was reviewed in this run)
- `.claude/slated/scenes/scene-<name>/review.md` — per-scene review (for each scene reviewed in this run)
- `.claude/slated/report.md` — project-level report aggregating all verdicts

Report the final tally: how many subjects were reviewed, how many passed, how many were flagged.

---

## Rules

- Never review a scene that is not `complete`
- Never mark an objective as met without evidence — unverified through the record must be confirmed through live testing
- Never skip the interactive track — static analysis alone cannot confirm a scene works as a user experience
- Never skip the regression check for scenes — a scene that only creates files still has a regression surface if it touches shared configuration
- Never fix or suggest implementation approaches — identify findings and state what must change; leave the how to the actor who addresses it
- Never produce a partial review for any subject — both tracks must complete before the review file is written
- Never proceed with the interactive track if the application is not running — attempt to restart it first; if that fails, surface the error and wait
- Never skip writing `.claude/slated/report.md` — the project report is a required output of every run
