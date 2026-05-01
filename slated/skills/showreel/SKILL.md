---
name: showreel
description: Assemble a self-contained showreel package — a distributable skill directory with bundled roles, backgrounds, and a generated SKILL.md ready to zip and upload to Claude Web.
expected-role: director
argument-hint: "[showreel-name]"
---

# Showreel

Assemble a showreel — a self-contained, distributable `showreels/showreel-<name>/` directory containing copied crew and cast roles, copied backgrounds, and a generated `SKILL.md` built from the showreel template. The output is ready to zip and upload to Claude Web, where it runs in a single context with no sub-agents, no git, and no shell.

---

## Pre-work

1. Run `/slated:perform director` to adopt the director role.

2. Resolve the home directory: run `echo $HOME` and capture the result.

3. Discover all available **crew roles** from `${CLAUDE_SKILL_DIR}/../perform/roles/role-*.md` using Glob.

4. Discover all available **cast roles** from both catalogues:
   - Project-level: Glob `.claude/slated/roles/role-*.md`
   - Global: `ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null`
   - Merge with project-level taking precedence on name collision.

5. Discover all available **backgrounds** from both catalogues:
   - Project-level: Glob `.claude/slated/backgrounds/background-*.md`
   - Global: `ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`
   - Merge with project-level taking precedence on name collision.

6. Read the showreel template at `${CLAUDE_SKILL_DIR}/templates/showreel-skill.md` in full.

7. If `$ARGUMENTS` is provided, use it as the proposed showreel name — carry it into Step 1 of the interview for confirmation.

---

## Process

### Step 1 — Interview

Use AskUserQuestion to establish:

1. **What is this showreel for?** Describe the type of task it will handle — one clear sentence. This becomes `{{TASK_DESCRIPTION}}` in the generated SKILL.md.

2. **What should the showreel be named?** Propose a kebab-case name derived from the task description (or from `$ARGUMENTS` if provided). If the user's input contains spaces or mixed case, normalise to kebab-case and confirm: "The name has been normalised to `<kebab-case-name>`. Does that look right?" This becomes `<name>` in `showreel-<name>`.

3. **Which cast roles are needed?** Present the full merged cast catalogue by name. The user selects one or more. Director and writer are always included — do not ask about them.

4. **Which backgrounds are needed?** Present the full merged background catalogue by name. The user selects zero or more. Backgrounds are optional.

Do not proceed to Step 2 until all four answers are confirmed.

### Step 2 — Resolve missing artefacts

For each selected cast role: verify it exists in the merged cast catalogue. For each selected background: verify it exists in the merged background catalogue.

For each gap found:

1. Draft the missing artefact autonomously:
   - **Missing cast role**: follow the cast-role definition process (Identity, Background, Expertise, Behavioral Parameters, Constraints, Interactions), derived from the showreel task description and project context. Read `${CLAUDE_SKILL_DIR}/../cast/background-role.md` before drafting to absorb the minimum standard.
   - **Missing background**: follow the background definition process (Opening statement, Semantics, Structure, Function), derived from available knowledge. Read `${CLAUDE_SKILL_DIR}/../brief/background-background.md` before drafting.

2. Present the draft to the user.

3. Ask whether it should be global or project-specific:
   - Global cast role: `$HOME/.claude/slated/roles/role-<name>.md`
   - Project cast role: `.claude/slated/roles/role-<name>.md`
   - Global background: `$HOME/.claude/slated/backgrounds/background-<name>.md`
   - Project background: `.claude/slated/backgrounds/background-<name>.md`

4. Write to the confirmed location. Add to the resolved cast/background list.

Draft and confirm missing artefacts sequentially before proceeding.

If any artefacts were auto-created, surface a brief note:

```
Auto-created before assembly:
- role-<name>.md (global cast role)
- background-<name>.md (project background)
```

If nothing was auto-created, skip this note.

### Step 3 — Check for name collision

Check whether `showreels/showreel-<name>/` already exists. If it does, use AskUserQuestion:

```
A showreel named `showreel-<name>` already exists.
Overwrite, choose a different name, or cancel?
```

Do not proceed until resolved. If cancelled, stop and report cleanly.

### Step 4 — Confirm assembly plan

Present the full assembly plan:

```
Showreel: showreel-<name>
Task: <task description>

Roles to bundle:
  - role-director.md (crew)
  - role-writer.md (crew)
  - role-<cast1>.md (cast)

Backgrounds to bundle:
  - background-<bg1>.md
  (or: none)

Output directory: showreels/showreel-<name>/
```

Use AskUserQuestion: "Does this assembly plan look correct?" Do not write any files until confirmed.

### Step 5 — Assemble the showreel

Once confirmed, proceed in strict order.

#### 5a — Create directories

Create the following if they do not exist:
- `showreels/`
- `showreels/showreel-<name>/`
- `showreels/showreel-<name>/roles/`
- `showreels/showreel-<name>/backgrounds/` — only if at least one background was selected

#### 5b — Copy crew roles

Read `${CLAUDE_SKILL_DIR}/../perform/roles/role-director.md` in full. Write its content verbatim to `showreels/showreel-<name>/roles/role-director.md`.

Read `${CLAUDE_SKILL_DIR}/../perform/roles/role-writer.md` in full. Write its content verbatim to `showreels/showreel-<name>/roles/role-writer.md`.

If either source file cannot be read, stop and report:

```
Assembly failed: crew role file not found at <path>.
The framework's perform skill may be missing or incomplete.
```

Do not proceed until both crew roles are written successfully.

#### 5c — Copy cast roles

For each confirmed cast role, resolve its source path from the merged catalogue (project-level path if it exists there, otherwise global path). Read the file in full. Write its content verbatim to `showreels/showreel-<name>/roles/role-<name>.md`. Do not modify content.

#### 5d — Copy backgrounds

For each confirmed background, resolve its source path from the merged catalogue. Read in full. Write verbatim to `showreels/showreel-<name>/backgrounds/background-<name>.md`.

#### 5e — Generate the SKILL.md

Take the template content read in Pre-work step 6. Replace each placeholder:

- `{{SHOWREEL_NAME}}` → the confirmed showreel name (e.g. `feature-builder`)
- `{{TASK_DESCRIPTION}}` → the confirmed task description from Step 1
- `{{CAST_MANIFEST}}` → one line per cast role (excluding director and writer):
  ```
  - `<name>` — loaded from `${CLAUDE_SKILL_DIR}/roles/role-<name>.md`
  ```
  If no cast roles beyond director and writer: replace with `(none)`
- `{{CAST_NAMES}}` → comma-separated list of cast role names only (excluding director and writer), e.g. `systems-engineer, ui-developer`. If no cast roles beyond director and writer: replace with `none`
- `{{BACKGROUND_MANIFEST}}` → one line per background:
  ```
  - `<name>` — loaded from `${CLAUDE_SKILL_DIR}/backgrounds/background-<name>.md`
  ```
  If no backgrounds: replace with `(none)`

Write the result to `showreels/showreel-<name>/SKILL.md`.

#### 5f — Update the showreels index

Read `showreels/README.md` if it exists. If it does not exist, create it with this content:

```markdown
# Showreels

Distributable showreel packages for Claude Web. Each subdirectory is a self-contained skill — zip the directory and upload it to Claude Web.

## How to deploy

```
zip -r showreel-<name>.zip showreels/showreel-<name>/
```

Upload the ZIP to Claude Web via Settings → Skills → Add skill.

## Index

| Showreel | Task | Cast roles | Backgrounds |
|---|---|---|---|
```

Append a new row to the Index table:

```
| [showreel-<name>](./showreel-<name>/) | <task description> | <comma-separated cast role names, excluding director and writer, or —> | <comma-separated background names, or —> |
```

Write the updated file.

---

## Output

- `showreels/showreel-<name>/SKILL.md` — the generated runnable skill for Claude Web
- `showreels/showreel-<name>/roles/role-director.md` — copied crew role
- `showreels/showreel-<name>/roles/role-writer.md` — copied crew role
- `showreels/showreel-<name>/roles/role-<cast>.md` — one per selected cast role
- `showreels/showreel-<name>/backgrounds/background-<bg>.md` — one per selected background (directory absent if none selected)
- `showreels/README.md` — created or updated with index entry
- Any auto-created role or background files written to their confirmed global or project-level locations

Surface a completion summary:

```
Showreel assembled: showreel-<name>
Location: showreels/showreel-<name>/

Bundled roles: director, writer, <cast names, or "none beyond crew">
Bundled backgrounds: <bg names, or "none">

To deploy:
  zip -r showreel-<name>.zip showreels/showreel-<name>/
  Upload showreel-<name>.zip to Claude Web via Settings → Skills → Add skill.
```

---

## Rules

- Never assemble before the plan is confirmed — Step 4 is the gate; no files are written before AskUserQuestion confirms
- Never modify the content of a copied role or background file — read and write verbatim only
- Never omit director and writer from the bundled roles
- Never create `showreels/showreel-<name>/backgrounds/` if no backgrounds were selected
- Assemble in strict order: directories → crew roles → cast roles → backgrounds → SKILL.md → README; never skip or reorder
- Never write before checking for a name collision (Step 3)
- Never draft a missing artefact without presenting it to the user and confirming the destination first
- Never embed domain knowledge in a drafted cast role — behavioral parameters only; domain knowledge belongs in backgrounds
- Never write vague constraints in a drafted role or vague entries in a drafted background — the same specificity standard as `/slated:cast` and `/slated:brief` applies
- Never write to any path outside `showreels/`, `.claude/slated/`, or `$HOME/.claude/slated/` — framework internals under `slated/` are never modified
- Never proceed past a failed crew role read — surface the error and stop
