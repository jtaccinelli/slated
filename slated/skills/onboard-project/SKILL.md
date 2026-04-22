---
name: onboard-project
description: Bring an existing project into the Slated framework — scanning the codebase to discover technologies, drafting background documents from evidence, proposing cast roles from directory structure, and authoring a retroactive set outline.
expected-role: casting-director
argument-hint: [path to existing project root, if not CWD]
---

# Onboard Project

Bring an existing project into the Slated framework by reading what is already there. Unlike `/slated:produce-project` — which gates a project before it is built — this skill starts from running code and works backwards: scanning the codebase for evidence, drafting backgrounds from discovered patterns, proposing roles from observed responsibility structure, and authoring a set outline that describes what the project *is*, not what to build. Every artefact is confirmed before it is written.

---

## Pre-work

1. Use Bash (`ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`) to list global backgrounds, and Glob `.claude/slated/backgrounds/background-*.md` for project-level ones — merge both results, with project-level files taking precedence over global ones with the same name. Read each in full. This is the catalogue of existing knowledge the skill will cross-reference during the scan — know what is already covered before assessing gaps.
2. Use Bash (`ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null`) to list global roles, and Glob `.claude/slated/roles/role-*.md` for project-level ones — merge both results, with project-level taking precedence. Note which roles are defined. These inform the cast role suggestion step.
3. If `$ARGUMENTS` specifies a project root other than the CWD, use that path for all file reads in the process below.

---

## Process

### Step 1 — Scan the codebase

Read the following files in order, recording what each reveals about the project's technology stack:

1. **`package.json`** — extract `name`, `scripts`, `dependencies`, and `devDependencies`. If a lock file is present (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `bun.lockb`), note the package manager.
2. **Config files** — detect and read any of the following that are present. Note which are found and what they configure:
   - `tsconfig.json` / `tsconfig.*.json`
   - `wrangler.jsonc` / `wrangler.toml`
   - `drizzle.config.ts`
   - `sanity.config.ts`
   - `vite.config.ts`
   - `tailwind.config.ts` / `tailwind.config.js`
   - `eslint.config.*` / `.eslintrc.*`
   - `prettier.config.*` / `.prettierrc.*`
   - `react-router.config.ts`
   - `biome.json` / `biome.jsonc`
   - Any other config files at the project root (e.g. `astro.config.ts`, `next.config.ts`, `svelte.config.js`)
3. **Source files** — identify the apparent domains present in the project (e.g. UI components, route handlers, API workers, database schema, CMS schema, server utilities). For each domain, sample 2–3 representative source files and read them. Aim to cover each apparent concern — do not read only one type of file.

After completing the scan, produce and present a **technology map**: a structured list of all identified technologies, each with evidence — the specific files that revealed the technology's presence and the key patterns observed. Present this map to the user and ask for confirmation or additions before proceeding.

Example format:

```
Technology Map:
- React — src/components/Button.tsx (JSX, hooks), src/app/routes/home.tsx
- Tailwind CSS v4 — src/app/app.css (@import "tailwindcss")
- React Router 7 (SSR) — react-router.config.ts, src/app/routes.ts
- Cloudflare Workers — wrangler.jsonc (compatibility_date, d1_databases)
- Drizzle ORM — drizzle.config.ts, src/db/schema.ts
- Sanity CMS — sanity.config.ts, src/sanity/schemaTypes/
- TypeScript — tsconfig.json
```

Do not proceed to Step 2 until the technology map is confirmed.

### Step 2 — Scaffold project directories

Ensure the following project-level Slated directories exist, creating any that are absent:

- `.claude/slated/set/` — for the set outline
- `.claude/slated/scenes/` — for manuscripts, takes, and wrap records
- `.claude/slated/roles/` — for project-specific role overrides
- `.claude/slated/backgrounds/` — for project-specific background overrides

Do not modify or delete any files inside directories that already exist.

If `.claude/slated/scenes/storyboard.md` does not exist, create it with the following initial structure:

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

### Step 3 — Draft and confirm backgrounds

For each technology in the confirmed technology map, check whether a background document already exists in the catalogue loaded during Pre-work.

Present a coverage summary to the user:

```
Background coverage:
- React → background-react.md ✓
- Tailwind CSS → background-tailwind.md ✓
- React Router 7 → background-react-router.md ✓
- Drizzle ORM → background-drizzle.md ✓
- Cloudflare Workers → NO BACKGROUND DEFINED ✗
- Sanity CMS → NO BACKGROUND DEFINED ✗
```

For technologies with an **existing background**: briefly verify coverage is sufficient for what was observed in the scan. Note any patterns observed in the code that fall outside the existing background's scope — surface these as gaps to address after onboarding is complete.

For technologies **without a background**: read the representative source files for that technology identified in Step 1 (and any additional files needed for depth). Draft a background document grounded entirely in discovered patterns, with all three sections populated:

- **Semantics** — naming conventions, typing idioms, and syntax choices as observed in the scanned files. Cite the specific files that evidence each convention.
- **Structure** — file and directory layout, import paths, and separation of concerns as they exist in this codebase. Include the actual directory tree where it clarifies the pattern.
- **Function** — runtime behaviour, patterns for achieving outcomes, and recipes for common operations. Derive these from what the code actually does — not from documentation or general knowledge.

Present each section of the draft to the user separately, with evidence citations. After presenting all three sections for a given background, ask for confirmation or requested changes before proceeding to the next background.

Once all sections are confirmed for a background, ask whether it should be written as:
1. **Global** — reusable across all projects → `$HOME/.claude/slated/backgrounds/background-<name>.md`
2. **Project-specific** — only for this project → `.claude/slated/backgrounds/background-<name>.md`

Write the file only after destination is confirmed.

After all backgrounds are handled (existing reviewed for gaps, new ones written), present the full coverage map with final status before proceeding to Step 4.

### Step 4 — Suggest and confirm cast roles

From the directory structure and file-type patterns observed in Step 1, identify natural responsibility boundaries — areas of the codebase that represent distinct enough concerns to warrant a dedicated cast role.

Consider: Does the project have a UI layer, a data layer, a CMS layer, an API/worker layer, a build/tooling layer? Are there directories that clearly belong to a single concern?

Produce a proposed cast list and present it to the user:

```
Proposed cast roles:
- design-engineer — UI components, route layouts, Tailwind styling (src/components/, src/app/routes/)
- systems-engineer — Cloudflare Workers, Drizzle ORM, D1 schema, server utilities (src/worker/, src/db/)
- content-engineer — Sanity schema definitions, content types, studio configuration (src/sanity/)
```

For each proposed role, include:
- The role name (kebab-case)
- A one-sentence domain description
- The directories and file types that suggest this responsibility boundary

Ask the user to confirm, modify, or reject each proposed role. Do not write any role file without explicit confirmation.

For each confirmed role, conduct the role definition interview directly — following the same process as `/slated:cast-role`: establish identity, background narrative, expertise, behavioral parameters, constraints, and interactions through dialogue before writing anything. Do not proceed to Step 5 until all accepted roles have been written.

### Step 5 — Write the retroactive set outline

From the confirmed backgrounds, confirmed cast roles, and codebase structure discovered in Step 1, draft `.claude/slated/set/outline.md`.

This document describes what the project **is** — not what to build. It is the baseline for all future scenes. Draft it to include:

1. **Project** — a one-sentence description of what the project is and who it is for, derived from `package.json` name/description, README if present, and the overall structure observed
2. **Tech stack** — every technology in the confirmed technology map, each mapped to its background document
3. **Directory structure** — the actual directory tree of the project as it exists, annotated to indicate what each significant directory contains
4. **Configuration files** — every config file found, with the key values observed (e.g. Wrangler compatibility date, D1 binding names, Drizzle dialect and output path, Vite plugins)
5. **Entry points** — the key files: server entrypoint, route manifest, main CSS entry, schema files, worker entrypoints
6. **Dependencies** — installed packages from `package.json`, grouped by concern (UI, data, CMS, tooling, etc.)
7. **Bindings and integrations** — runtime bindings (D1, KV, R2, Hyperdrive) and external integrations derived from config files and the codebase scan
8. **Observed conventions** — any patterns or conventions found in the code that are not yet captured by a background document; flag these as candidates for future background updates

Present the full outline to the user before writing. Ask for confirmation or adjustments. Do not write the file until confirmed.

---

## Output

When the skill completes, the following files will have been written (subject to user confirmation at each step):

- `.claude/slated/set/` — scaffolded if absent
- `.claude/slated/scenes/` — scaffolded if absent
- `.claude/slated/roles/` — scaffolded if absent
- `.claude/slated/backgrounds/` — scaffolded if absent
- `$HOME/.claude/slated/backgrounds/background-<name>.md` and/or `.claude/slated/backgrounds/background-<name>.md` — one per new background confirmed and written
- `.claude/slated/roles/role-<name>.md` — one per confirmed cast role, written by casting-director sub-agents
- `.claude/slated/set/outline.md` — the retroactive set outline describing the project as it exists

Report all file paths written at completion.

---

## Rules

- Never draft a background convention that is not evidenced in the scanned code — all patterns must be traced to specific files
- Never write any file without explicit user confirmation
- Never skip the scan phase — backgrounds and the outline must be grounded in what was discovered
- Never write the set outline before backgrounds and roles are confirmed
- Never invent cast roles — only derive them from observed responsibility structure in the scanned codebase
- Never write a background without presenting each section (Semantics, Structure, Function) separately for review before writing
- Follow the `/slated:cast-role` interview process directly for each confirmed role — the casting-director running this skill conducts the interview inline without spawning a sub-agent
- Never mark coverage as sufficient for an existing background without checking what was observed against what is documented
