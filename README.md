# Slated

A film-production-inspired agentic coordination framework for Claude Code. Slated structures autonomous work through roles, backgrounds, scenes, and takes — giving every actor a clear identity, every task a plan, and every outcome a record.

---

## Concepts

| Concept | What it is |
|---|---|
| **Role** | Defines *how* an actor behaves — tone, priorities, constraints, interaction style |
| **Background** | Defines *what* an actor knows — domain knowledge that informs better execution |
| **Actor** | A general-purpose agent vessel that adopts a role and optional backgrounds |
| **Skill** | A discrete job mapped to an expected role |
| **Scene** | A single feature or fix — manuscript, takes, wrap record, and review |

---

## Testing Locally

To load the plugin from your local clone rather than the registry, pass `--plugin-dir` when starting Claude Code:

```
claude --plugin-dir /path/to/slated/plugin
```

The local copy takes precedence over any installed registry version with the same name for that session. After making changes, reload without restarting:

```
/reload-plugins
```

---

## Getting Started

### 1. Install the plugin

```
/plugin install slated
```

### 2. Initialise global directories (once per device)

```
/init
```

This creates `~/.claude/slated/roles/` and `~/.claude/slated/backgrounds/`, and seeds the bundled crew role definitions. You do not need to define crew roles manually.

### 3. Define backgrounds (as needed)

Backgrounds are domain knowledge documents. Define them globally for reuse across projects:

```
/establish-background react
/establish-background cloudflare
/establish-background drizzle
```

### 4. Start a project

**New project:**

```
/produce-project
```

The producer interviews you, verifies all required roles and backgrounds exist, scaffolds project directories, and authors a set outline.

```
/build-set
```

The set-designer builds the project from the outline.

**Existing project:**

```
/onboard-project
```

The casting-director scans the codebase, drafts backgrounds from discovered patterns, proposes cast roles from the directory structure, and authors a retroactive set outline.

---

## Production Lifecycle

```
/init                          → bootstrap global dirs (once per device)
/cast-role <name>              → define a crew or cast role
/establish-background <name>   → define a background document
/produce-project               → gate production readiness, author set outline
/onboard-project               → bring an existing project into the framework
/build-set                     → scaffold project from outline
/write-scene <requirement>     → plan a scene — manuscript + storyboard entry
/shoot-take <scene>            → execute one take, return verdict
/shoot-scene <scene>           → execute all takes automatically
/wrap-scene <scene>            → close out a passing scene
/review-project                → post-production QA across set and all scenes
/refine-role <name>            → adjust an existing role
/refine-background <name>      → adjust an existing background
```

---

## Directory Layout

### Global (`~/.claude/slated/`)

Created by `/init`. Shared across all projects on this device.

```
~/.claude/slated/
  roles/          ← crew roles + reusable cast roles
  backgrounds/    ← reusable domain knowledge
```

### Project-level (`.claude/`)

Created by `/produce-project`. Project-specific overrides and scene artefacts.

```
.claude/slated/
  roles/          ← project-specific role overrides
  backgrounds/    ← project-specific background overrides
  scenes/         ← manuscripts, takes, wraps, reviews
  set/            ← outline and set review
```

**Lookup order**: project-level takes precedence over global. A role or background in `.claude/` overrides the same name in `~/.claude/slated/`.

---

## Templates

Reference templates for roles, backgrounds, skills, and scenes live in `templates/`. They define the expected structure for each artefact and are used by the casting-director skills as guides.

---

## Roles

Roles divide into two categories:

**Crew** — operate within the meta-narrative. Seeded automatically by `/init`.

| Role | Responsibility |
|---|---|
| `casting-director` | Defines roles, backgrounds, and skills |
| `writer` | Plans scenes; assesses manuscript fidelity during takes |
| `director` | Runs takes; evaluates actor output against role and background conventions |
| `producer` | Gates production readiness; conducts post-production QA |
| `set-designer` | Builds the project scaffold from the producer's outline |
| `visualiser` | Maintains `storyboard.md` |

**Cast** — project-specific. Define per-project using `/cast-role`.
