---
name: produce
description: Initialise a project for production — interviewing to understand what is being built or scanning an existing codebase, then establishing locations, backgrounds, and cast roles before any scene is planned.
expected-role: producer
argument-hint: (none)
---

# Produce

Gate a project for production readiness. The producer interviews the user to understand what is being built — or, for an existing codebase, reads what is already there and works backwards. A successful run leaves the project with a confirmed `locations.md`, all required backgrounds, and all required cast roles, ready for the writer to begin planning scenes.

---

## Pre-work

1. Run `/slated:perform producer` to adopt the producer role.

2. Resolve the home directory: run `echo $HOME` and capture the result.

3. Discover existing backgrounds from both catalogues:
   - Project-level: Glob `.claude/slated/backgrounds/background-*.md`
   - Global: run `ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`
   - Merge both, with project-level taking precedence on name collision. Read each in full.

4. Discover existing cast roles from both catalogues:
   - Project-level: Glob `.claude/slated/roles/role-*.md`
   - Global: run `ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null`
   - Merge both, with project-level taking precedence. Note which are defined.

5. Read `${CLAUDE_SKILL_DIR}/../cast/background-role.md` in full — absorb as the minimum standard for every role file drafted in this skill.

6. Read `${CLAUDE_SKILL_DIR}/../brief/background-background.md` in full — absorb as the minimum standard for every background file drafted in this skill.

7. Use AskUserQuestion: "Are you setting up a new project or bringing an existing one into the framework?"

---

## Process — New Project

### Step 1 — Establish what is being built

Interview the user to understand the nature of the project. Ask:

1. What is this project? What is its primary purpose and who uses it?
2. Is this a micro-app (single-purpose, few routes, minimal state) or a full application (multiple domains, complex data, multi-user)?
3. Is this a standalone app or part of a monorepo?
4. What integrations are required — database, authentication, CMS, third-party APIs?
5. Are there any known constraints — runtime, deployment target, existing tooling — that must be respected?

Do not proceed until the project's nature and scope are clear.

### Step 2 — Scaffold project directories

Ensure the following project-level directories exist, creating any that are absent:

- `.claude/slated/scenes/` — for manuscripts, takes, and wrap records
- `.claude/slated/roles/` — for project-specific cast role overrides
- `.claude/slated/backgrounds/` — for project-specific background overrides

Do not modify or delete any files inside directories that already exist.

### Step 3 — Verify and draft backgrounds

From the interview, compile a complete list of every technology, runtime, and integration the project requires. For each, identify whether a background document exists in the catalogue loaded during Pre-work.

Present the mapping to the user:

```
Required technologies:
- <technology> → background-<name>.md ✓
- <technology> → background-<name>.md ⚠ (insufficient coverage for <specific gap>)
- <technology> → NO BACKGROUND DEFINED ✗
```

**If any technology has no corresponding background**: draft a full background document following the schema loaded in Pre-work — Opening statement (one sentence on what the domain covers and why it matters to an actor filling a relevant role), Semantics (naming conventions and syntax choices with concrete examples per entry), Structure (directory trees wherever more than two directories exist; import path examples wherever aliases are used), Function (runnable code examples for every non-obvious pattern — pseudo-code is not acceptable). Present the draft to the user. Ask whether it should be global or project-specific:
- **Global** → `$HOME/.claude/slated/backgrounds/background-<name>.md`
- **Project-specific** → `.claude/slated/backgrounds/background-<name>.md`
Write only after destination is confirmed.

**If a background exists but coverage is insufficient**: treat it as `⚠ insufficient`. Extend the existing background — add only the missing content to the relevant sections, at the same specificity standard. Present the proposed additions to the user before writing.

In all cases, do not introduce a convention that cannot be grounded in the project context established in Step 1. If a decision requires more information, ask rather than guess.

### Step 4 — Verify and draft cast roles

From the project nature established in Step 1, identify the cast roles required to deliver scenes for this project. Present the proposed cast list to the user:

```
Proposed cast roles:
- <role-name> — <one-sentence domain description> (<files/areas this role would own>)
```

For each role that does not yet exist: draft it autonomously without interview, following the schema loaded in Pre-work — YAML frontmatter (`name`, `version`), Identity (title, domain, one-sentence summary), Background (2–3 paragraph narrative, not a skills list), Expertise (items specific enough that none would appear on another role's list), Behavioral Parameters (all five: tone, verbosity, decision style, priorities, working style — each specific enough to direct actor behaviour), Constraints (each naming exactly what the actor will not do), Interactions (handoffs described in terms of what is passed, not who passes it). Surface the draft for confirmation before writing. Once confirmed, ask whether the role should be global or project-specific:
- **Global** → `$HOME/.claude/slated/roles/role-<name>.md`
- **Project-specific** → `.claude/slated/roles/role-<name>.md`
Write only after destination is confirmed.

### Step 5 — Author the locations document

From the intended project structure established in Step 1, author `.claude/slated/locations.md`. Include every directory and named file the project will require, derived from the tech stack and integration requirements.

Format:

```markdown
# Shot Locations

Named file and directory locations for manuscript authors. Reference these when specifying actor actions to give actors an unambiguous starting point.

| Name | Path | Description |
|---|---|---|
| `<kebab-case-name>` | `<path/relative/to/project/root>` | <what this file or directory contains and when to reference it> |
```

Rules for entries:
- Names must be unique and kebab-case
- Paths are relative to the project root — never absolute
- Descriptions state what the file/directory contains and when an actor should reference it, in one sentence
- Do not include files inside `.claude/` — those are framework artefacts, not project locations

### Step 6 — Confirm and scaffold

Surface the full locations document to the user for confirmation using AskUserQuestion: "Does this locations list look correct before I scaffold the project?"

Once confirmed:

1. Create the project directory structure from the locations list
2. Create required configuration files from the tech stack (e.g. `package.json`, `tsconfig.json`, `wrangler.jsonc`) — populate each from the background documents for the relevant technologies
3. Install dependencies using the appropriate package manager. Confirm the install completes without errors before finishing.

Every file written must reflect the conventions in the relevant background document. Do not introduce a pattern, value, or configuration that cannot be traced to an available background.

---

## Process — Existing Project

### Step 1 — Scan the codebase

Read the following files in order, recording what each reveals about the technology stack:

1. **`package.json`** — extract `name`, `scripts`, `dependencies`, and `devDependencies`. Note the package manager from any lock file present (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `bun.lockb`).
2. **Config files** — detect and read any present: `tsconfig.json`, `wrangler.jsonc`/`wrangler.toml`, `drizzle.config.ts`, `sanity.config.ts`, `vite.config.ts`, `tailwind.config.ts`, `react-router.config.ts`, `biome.json`, and any other config files at the project root.
3. **Source files** — identify the apparent domains present (UI components, route handlers, API workers, database schema, CMS schema, server utilities). For each domain, sample 2–3 representative source files and read them.

Produce a technology map and present it to the user:

```
Technology Map:
- <Technology> — <evidence file(s)> (<key pattern observed>)
- <Technology> — <evidence file(s)>
```

Use AskUserQuestion to confirm the technology map or request additions before proceeding to Step 2.

### Step 2 — Scaffold project directories

Ensure the following project-level directories exist, creating any that are absent:

- `.claude/slated/scenes/`
- `.claude/slated/roles/`
- `.claude/slated/backgrounds/`

Do not modify or delete any files inside directories that already exist.

### Step 3 — Draft and confirm backgrounds

For each technology in the confirmed map, check whether a background document exists in the catalogue from Pre-work.

Present a coverage summary:

```
Background coverage:
- <Technology> → background-<name>.md ✓
- <Technology> → NO BACKGROUND DEFINED ✗
```

For technologies **without a background**: read the representative source files identified in Step 1. Draft a background grounded entirely in discovered patterns, following the schema loaded in Pre-work:

- **Opening statement** — one sentence describing what domain the background covers and why it matters to an actor filling a relevant role; specific enough that an actor immediately understands what they are about to absorb.
- **Semantics** — naming conventions, typing idioms, and syntax choices as observed; each entry must include a concrete pattern or example cited from the specific file that evidences it.
- **Structure** — file and directory layout, import paths, and separation of concerns as they exist; include a directory tree wherever the domain has more than two directories; include import path examples wherever path aliases exist.
- **Function** — runtime behaviour, patterns, and recipes derived from what the code actually does; include runnable code examples for every non-obvious pattern — pseudo-code is not acceptable.

For technologies **with an existing background**: briefly verify coverage is sufficient for what was observed. Note any patterns that fall outside the existing background's scope — surface these as gaps.

Present each new background draft to the user before writing. Once confirmed, ask whether the background should be:
1. **Global** — `$HOME/.claude/slated/backgrounds/background-<name>.md`
2. **Project-specific** — `.claude/slated/backgrounds/background-<name>.md`

Write only after destination is confirmed.

### Step 4 — Suggest and confirm cast roles

From the directory structure and file-type patterns observed in Step 1, identify natural responsibility boundaries. Produce a proposed cast list:

```
Proposed cast roles:
- <role-name> — <one-sentence domain description> (<directories/file types>)
```

Use AskUserQuestion to confirm, modify, or reject each proposed role. For each confirmed role, draft it autonomously following the schema loaded in Pre-work — YAML frontmatter (`name`, `version`), all six required sections, all five Behavioral Parameters each specific enough to direct actor behaviour, constraints each naming exactly what the actor will not do, expertise items each distinct enough that none would appear on another role's list, and interactions describing handoffs in terms of what is passed rather than who passes it. Surface the draft for confirmation before writing. Once confirmed, ask whether the role should be global or project-specific:
- **Global** → `$HOME/.claude/slated/roles/role-<name>.md`
- **Project-specific** → `.claude/slated/roles/role-<name>.md`

Do not proceed to Step 5 until all accepted roles have been written.

### Step 5 — Author the locations document

Perform as DOP (using `/slated:perform dop`) to scan the actual project structure and produce a candidate `locations.md`. Follow the Scout process:

1. Walk the project structure; identify every meaningful directory and named file entry point
2. Draft the document — group entries by concern; label each with a short description
3. Use AskUserQuestion to surface the draft for user confirmation before writing
4. Write `.claude/slated/locations.md` once confirmed

---

## Output

**New project:**
- `.claude/slated/scenes/`, `.claude/slated/roles/`, `.claude/slated/backgrounds/` — scaffolded if absent
- `.claude/slated/locations.md` — named shot locations, confirmed and written
- Any drafted backgrounds and cast roles written to their confirmed locations
- Project directory structure, configuration files, and dependencies from the scaffold

**Existing project:**
- `.claude/slated/scenes/`, `.claude/slated/roles/`, `.claude/slated/backgrounds/` — scaffolded if absent
- `.claude/slated/locations.md` — produced from the codebase scan, confirmed and written
- Any drafted backgrounds and cast roles written to their confirmed locations

---

## Rules

- Never scaffold before the locations list is confirmed by the user
- Never write a background without presenting the draft and confirming the destination (global or project-specific) first
- Never write a cast role without presenting the draft for confirmation and confirming the destination (global or project-specific) first
- Never introduce a pattern that cannot be traced to an available background document
- Never make foundational decisions (storage approach, auth pattern, package structure) without surfacing them to the user for confirmation
- Never draft a background convention that is not evidenced in the scanned code (existing project path) — all patterns must be traced to specific files
- Never invent cast roles — derive them from interview (new project) or from observed responsibility structure (existing project)
- Does not produce or review its own output — confirmation is a user gate, not a self-check
- Never draft a background without an Opening statement as the first line of the file, before any section heading
- Never write a background convention without a concrete pattern or example — vague entries are not acceptable
- Never draft a role without all five Behavioral Parameters, each specific enough to meaningfully direct actor behaviour
- Never write a vague constraint in a role — each must name exactly what the actor will not do
- Never write a generic expertise item in a role — each must be specific enough that it would not appear on another role's list
- Never embed domain knowledge in a role definition — behavioral parameters only; domain knowledge belongs in backgrounds
- Never write a role file without YAML frontmatter containing `name` and `version`
