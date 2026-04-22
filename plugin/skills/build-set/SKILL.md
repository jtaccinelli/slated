---
name: build-set
description: Scaffold a new project from the set outline — building the complete directory structure, configuration files, entry points, and dependencies defined by the producer. Writes CLAUDE.md, shot locations, and the pilot on completion. Subsequent runs build incrementally on what already exists.
expected-role: set-designer
argument-hint: []
---

# Build Set

Scaffold the project from the set outline. The set-designer reads `.claude/slated/set/outline.md` in full, builds everything it specifies in dependency order, and writes no file that the outline does not define. The output is a complete, functional project scaffold ready for actors to begin writing scenes against.

On subsequent runs, the set-designer operates incrementally — creating missing files, updating configuration that the outline has changed, and regenerating derived artefacts (`CLAUDE.md`, `locations.md`) to reflect current project state.

---

## Pre-work

1. Read `.claude/slated/set/outline.md` in full — this is the sole source of truth for all scaffolding decisions; if it does not exist, stop and instruct the user to run `/slated:produce-project` first.
2. **Detect run mode**: Check whether `.claude/slated/set/locations.md` exists in the project.
   - If it does not exist: this is an **initial build** — create all files from scratch.
   - If it does exist: this is an **incremental build** — scan for missing or outdated artefacts and update accordingly. Do not delete or overwrite files that are not governed by the outline (source files that actors have written remain untouched).
3. If `.claude/slated/set/review.md` exists, delete it — the scaffold is being (re)built and the prior review is no longer valid.
4. Ensure the following project-level directories exist, creating any that are absent: `.claude/slated/set/`, `.claude/slated/scenes/`, `.claude/slated/roles/`, `.claude/slated/backgrounds/`. Do not modify existing files.
5. Read each background document listed in the outline's tech stack — check `.claude/slated/backgrounds/background-<name>.md` (project) first; if not found, use Bash (`ls $HOME/.claude/slated/backgrounds/background-<name>.md 2>/dev/null`) to check globally and read at `$HOME/.claude/slated/backgrounds/background-<name>.md` if present; use whichever is found first. Understand the conventions before touching any files.
6. Confirm the outline is complete — every section must be populated; if any section is missing or unclear, surface the gap and ask the user before proceeding. Once the user provides the missing information, continue autonomously without further confirmation.

---

## Process

### Step 1 — Build in dependency order

#### Initial build

Create all files in the order the outline specifies, respecting dependency constraints:

1. Root configuration files (`package.json`, `pnpm-workspace.yaml` if monorepo, `tsconfig.json`)
2. Runtime configuration (`wrangler.jsonc`)
3. CSS entry point and Tailwind configuration
4. Server entrypoint and load context declaration
5. Application entrypoint and route manifest
6. Any additional integration files (schema, drizzle config, session handler stubs)

#### Incremental build

For each file the outline specifies:

- If the file does not exist: create it as in an initial build.
- If the file is a configuration file (e.g. `package.json`, `tsconfig.json`, `wrangler.jsonc`): read the current file and compare it against what the outline specifies. If they differ in a meaningful way — a value added, changed, or removed in the outline — update the file to match. Do not change values the outline does not mention.
- If the file is a source file (entrypoints, route files, schema): do not overwrite. Actors may have modified these — treat them as owned by the production, not the scaffold.

Surface a summary of what was created, what was updated, and what was skipped (already exists and unchanged).

#### Both modes

Every file written must reflect the conventions in the relevant background document exactly. The outline defines what to build; the backgrounds define how to build it correctly.

If a conflict exists between the outline and a background — a value in the outline that violates a background convention — defer to the background convention. Write the file using the background-compliant value and record the override in the build report under a "Convention overrides" section, noting the outline value that was replaced and the background rule that took precedence. Do not silently deviate without logging.

### Step 2 — Install dependencies

Run the appropriate install command for the project's package manager (e.g. `pnpm install`). Confirm the install completes without errors before proceeding. If it fails, diagnose from the output — do not proceed with a broken install.

On an incremental build, run install only if `package.json` was created or updated in Step 1.

### Step 3 — Capture shot locations

Walk the project's built file structure and identify every file or directory that an actor might need to locate during a scene. This includes all files created or confirmed in Step 1, plus any already-existing files that are key reference points for the project.

A shot location is any file or directory that:
- Is an entry point the outline names (server, routes, CSS, schema, config)
- Is a top-level directory grouping domain logic
- Is a configuration file an actor would need to read or modify
- Is a stub or integration file that scenes will build on

Write `.claude/slated/set/locations.md`. This file is regenerated on every run — it always reflects the current state of the project layout.

Format:

```markdown
# Shot Locations

Named file and directory locations for manuscript authors. Reference these when specifying actor actions to give actors an unambiguous starting point.

| Name | Path | Description |
|---|---|---|
| `<kebab-case-name>` | `<path/relative/to/project/root>` | <what this file or directory contains and when to reference it> |
```

Rules for entries:
- Names must be unique and kebab-case (e.g. `route-manifest`, `db-schema`, `server-entry`)
- Paths are relative to the project root — never absolute
- Descriptions state what the file/directory contains and when an actor should reference it, in one sentence
- Cover all entry points, all config files, all top-level directories, and key integration stubs
- Do not include files inside `.claude/` — those are framework artefacts, not project locations

### Step 4 — Write CLAUDE.md

Write the project's `CLAUDE.md` file. This file is loaded into every future actor's context — it must describe the project accurately and concisely so actors can orient themselves without re-reading the full scaffold.

This file is regenerated on every run (initial and incremental) to reflect the current state of the project.

Derive all content from the set outline and what was built:

1. **Project** — one sentence describing what this project is and what it does (from the outline)
2. **Tech stack** — the technologies in use (from the outline's tech stack section)
3. **Structure** — a brief description of the directory layout and what each top-level area contains (from what was built)
4. **Entry points** — the key files an actor needs to know about: server entrypoint, route manifest, CSS entry, schema, etc.
5. **Key decisions** — the foundational choices from the outline's Key Decisions section, stated plainly

Do not include framework meta-documentation. Do not duplicate what background documents already cover in detail — reference the pattern, not the recipe.

### Step 5 — Initialise the storyboard

On an **initial build** only: write `.claude/slated/scenes/storyboard.md` with the following structure. If the file already exists, skip this step.

```markdown
# Storyboard

Navigation map of all scenes in this project. Maintained by the visualiser.

## Pending

Scenes confirmed or in progress.

| Scene | Status | Requirement |
|---|---|---|

## Completed

Scenes wrapped and ready for review.
```

### Step 6 — Write the pilot

Write `.claude/slated/scenes/pilot.md`. This is a lightweight scene backlog — a starting point for the writer when planning the first scenes. It is not a manuscript and contains no cast, objectives, or actions.

On an **initial build**: write the file from scratch.

On an **incremental build**: read the existing pilot and check whether new routes, schema tables, or integrations added since the original build suggest scene ideas not yet listed. If so, append the new suggestions to the existing Suggested Scenes list. Do not remove or rewrite existing entries.

Include:

- A one-sentence description of the project, restating what was established in the outline
- A suggested list of scenes: each with a short kebab-case name and a one-line description of what it would accomplish

Derive the scene suggestions from the scaffold itself — what routes exist, what the data model supports, what user flows are implied by the project's purpose. Aim for 3–6 scenes that together would make the project functional end-to-end.

Format:

```markdown
# Pilot

<one sentence describing the project>

## Suggested Scenes

- `scene-<name>` — <one-line description of what this scene would accomplish>
- `scene-<name>` — <one-line description>
```

This file is a starting point, not a plan. The writer will make their own casting and sequencing decisions when `/slated:write-scene` is run.

---

## Output

- A complete project scaffold (initial build) or incremental update (subsequent build), derived from `.claude/slated/set/outline.md`
- No placeholder files — every file created must be complete enough to be functional
- `.claude/slated/set/locations.md` — named shot locations for manuscript authors; regenerated on every run
- `CLAUDE.md` — project context file written for future actors; regenerated on every run
- `.claude/slated/scenes/pilot.md` — suggested scene backlog for the writer; created on initial build, appended on incremental build if new suggestions exist
- `.claude/slated/scenes/storyboard.md` — scene navigation map; created on initial build only

Report the full list of files created and updated, grouped by concern, and note which background document each group was derived from. For incremental builds, separately list files that were skipped because they already existed. If any conflict arose between the outline and a background, document it so it can inform future updates to both.

---

## Rules

- Never build without a complete set outline — if `.claude/slated/set/outline.md` is missing or incomplete, stop
- Never introduce a file or pattern that the outline does not specify
- Never deviate from background conventions when building what the outline defines
- Never ignore a conflict between the outline and a background — defer to the background convention and log the override in the build report
- Never produce a partial scaffold — the output must be complete enough for scene work to begin
- Never skip writing `CLAUDE.md` — project context is a required output of every build
- Never skip writing `.claude/slated/set/locations.md` — shot locations are a required output of every build
- Never skip writing `.claude/slated/scenes/pilot.md` on an initial build
- Never skip writing `.claude/slated/scenes/storyboard.md` on an initial build — skip only if it already exists
- Never make decisions the outline deferred — if the outline is ambiguous, ask rather than guess
- Never overwrite an existing source file on an incremental build — only config files may be updated, and only where the outline specifies a different value
- Never use absolute paths in `locations.md` — all paths must be relative to the project root
