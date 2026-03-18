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

### 1. Add the marketplace and install the plugin

```
/plugin marketplace add jtaccinelli-slated
```

```
/plugin install slated@jtaccinelli-slated
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

## Example: A Scene Writing Cycle

Once a project is set up, delivering a feature looks like this:

**1. Plan the scene**

```
/write-scene add a contact form to the homepage
```

The writer interviews you if the requirement needs clarification, proposes a cast, defines verifiable objectives, and sequences the work into shots. Nothing is written until you confirm the manuscript.

**2. Run the scene**

```
/shoot-scene contact-form
```

The director dispatches actors per the cast, commissions a writer review of manuscript fidelity, evaluates output against objectives and role definitions, and either passes or fails the take. Failed takes surface a prioritised list of required changes and automatically begin the next take. When a take passes, the director wraps the scene — appending the Completion Record, writing `wrap.md`, and updating the storyboard.

Or step through it manually:

```
/shoot-take contact-form     ← one take at a time, verdict returned to you
/wrap-scene contact-form     ← once a take passes
```

**3. Review**

```
/review-project
```

The producer reviews all completed scenes — static analysis and an interactive pass — and produces a report. Findings are written to `review.md` per scene and feed into the next take if the scene is re-shot.

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


---

## License

Copyright (C) 2026 Jordan Accinelli

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.
