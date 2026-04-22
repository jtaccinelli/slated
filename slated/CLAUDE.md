# Framework: Agentic Coordination

This project uses a film-production-inspired agentic coordination structure. All autonomous work is performed by **actors** — a single, general-purpose agent type that adopts a **role** and optionally loads **backgrounds** before operating.

---

## Folder Reference

The framework distinguishes between **global** resources (shared across all projects on a device) and **project-level** resources (specific to this project). Run `/slated:init` once per device to create the global directories.

### Global (`~/.claude/`)

| Folder | Purpose |
|---|---|
| `~/.claude/slated/roles/` | Crew roles and reusable cast role definitions — available in every project |
| `~/.claude/slated/backgrounds/` | Reusable domain knowledge — technology backgrounds shared across projects |
| `~/.claude/agents/` | The `actor` agent — the only agent type in this framework |
| `~/.claude/skills/` | All framework skills — available in every project |

### Project-level (`.claude/`)

| Folder | Purpose |
|---|---|
| `.claude/slated/roles/` | Project-specific role overrides — takes precedence over global roles with the same name |
| `.claude/slated/backgrounds/` | Project-specific background overrides — takes precedence over global backgrounds with the same name |
| `.claude/slated/scenes/` | Individual scenes — manuscripts, takes, wrap records, and reviews |
| `.claude/slated/set/` | Set outline and set review — the producer's build specification and the review of what was built |

**Lookup order**: When searching for a role or background, always check project-level first, then fall back to global. This allows projects to override shared definitions without affecting other projects.

---

## How It Works

1. A **role** defines *how* an actor behaves — tone, priorities, constraints, interaction style.
2. **Backgrounds** define *what* an actor knows — domain knowledge that informs better execution of the role.
3. An **actor** is instantiated with a role and optional backgrounds, then directed by a scene or skill.
4. A **skill** is a discrete job that maps to an expected role — actors in that role pick it up and execute it.
5. A **scene** captures a single feature or fix — its manuscript, all takes, a wrap record, and a review.

---

## Production Lifecycle

### Pre-production

1. If any required backgrounds are missing, the **casting director** defines them (`/slated:establish-background`).
2. The **producer** gates the project for production readiness (`/slated:produce-project`) — interviewing to understand what is being built, verifying all required backgrounds and roles exist, and authoring a set outline in `.claude/slated/set/`. No scaffolding begins until the outline is confirmed.
3. The **set-designer** builds the project from the outline (`/slated:build-set`) — constructing the complete scaffold, installing dependencies, writing `CLAUDE.md`, and producing the pilot.

### Scene Planning

4. The **writer** drafts a `manuscript.md` from the requirement (`/slated:write-scene`) — casting roles, defining verifiable objectives, and sequencing actions. The writer self-validates the cast against available roles and backgrounds. On confirmation, the **visualiser** is dispatched to add the scene to the Pending section of `storyboard.md`.

### Production

5. The **director** runs takes against the manuscript (`/slated:shoot-take` for a single take, `/slated:shoot-scene` to iterate automatically). Each take dispatches actors per the cast, commissions a **writer** review of manuscript fidelity, and produces a take file with Director's Notes.
6. If a take **fails**: next-take instructions are surfaced and actors re-run.
7. If a take **passes**: the director wraps the scene (`/slated:wrap-scene`) — appending the Completion Record to the manuscript, writing `wrap.md` (including Editor Notes for shared artefacts touched), and dispatching the **visualiser** to move the scene from Pending to Completed in `storyboard.md`.

### Post-production

8. The **producer** reviews the project (`/slated:review-project`) — checking the set build first (if unreviewed), then reviewing all completed scenes without a review record in storyboard order. Each subject gets a static analysis pass and an interactive review from the user's perspective. The output is a review file per subject and a project-level `report.md`.

---

## Key Roles

Roles divide into two categories:

**Crew** operate within the meta-narrative — building the production's foundations, governing how scenes run, and reviewing what was built. Crew roles have full awareness of the production structure and may reference other roles by name.

**Cast** operate through the meta-narrative — delivering actual features for the project. Cast roles discover their co-stars from the manuscript at execution time and do not name peers in their definitions.

### Crew

| Role | Responsibility |
|---|---|
| `director` | Evaluates actor output against role definitions and background conventions. High strictness. Most naturally operates in the main context. |
| `writer` | Creates manuscripts from requirements. Assesses manuscript fidelity during takes. |
| `producer` | Gates production readiness and authors the set outline in pre-production. Conducts post-production QA across the set and all completed scenes. Never builds and never reviews their own work. |
| `set-designer` | Builds the project scaffold from the producer's set outline. Never authors the outline. |
| `visualiser` | Maintains `storyboard.md` as a navigable map of all scenes. Writes only to the storyboard — all other files are read-only inputs. |
| `casting-director` | Defines and maintains roles, backgrounds, and skills. Interviews before writing. Never produces draft definitions. |

### Cast

Cast roles are project-specific and are not bundled with the framework. They must be defined using `/slated:cast-role` before any scene that requires them can be planned.

---

## Skills Reference

| Skill | Expected Role | Purpose |
|---|---|---|
| `/slated:init` | `casting-director` | Bootstrap global framework directories (`~/.claude/slated/roles/`, `~/.claude/slated/backgrounds/`) on a new device |
| `/slated:update` | `casting-director` | Overwrite local crew role files with the latest bundled definitions — run after pulling framework changes |
| `/slated:produce-project` | `producer` | Gate production readiness — verify backgrounds and roles, author the set outline, scaffold project directories |
| `/slated:onboard-project` | `casting-director` | Bring an existing project into the framework — scan the codebase, draft backgrounds from discovered patterns, define cast roles through interview, and author a retroactive set outline |
| `/slated:build-set` | `set-designer` | Build the project scaffold from the set outline |
| `/slated:write-scene` | `writer` | Plan a new scene — manuscript, cast, objectives, storyboard entry |
| `/slated:shoot-take` | `director` | Execute one take and return a verdict |
| `/slated:shoot-scene` | `director` | Execute all takes automatically, then finalise |
| `/slated:wrap-scene` | `director` | Close out a passing scene — Completion Record, wrap.md, storyboard |
| `/slated:check-continuity` | `visualiser` | Log an out-of-band codebase change into the storyboard as a Recorded entry, and flag whether it warrants a set outline update |
| `/slated:review-project` | `producer` | Post-production QA — review the set build and all unreviewed scenes; produce a project report |
| `/slated:cast-role` | `casting-director` | Define a new role through a structured interview |
| `/slated:refine-role` | `casting-director` | Adjust an existing role through targeted interview |
| `/slated:establish-background` | `casting-director` | Define a new background through a structured interview |
| `/slated:refine-background` | `casting-director` | Adjust an existing background through targeted interview |
| `/slated:perform` | `casting-director` | Load a role from the catalogue into the current Claude Code context |

---

## Naming Conventions

- Global roles: `~/.claude/slated/roles/role-<name>.md`
- Global backgrounds: `~/.claude/slated/backgrounds/background-<name>.md`
- Project-specific roles: `.claude/slated/roles/role-<name>.md`
- Project-specific backgrounds: `.claude/slated/backgrounds/background-<name>.md`
- Skills: `.claude/skills/<skill-name>/SKILL.md`
- Set outline: `.claude/slated/set/outline.md`
- Set review: `.claude/slated/set/review.md`
- Pilot (initial scene backlog): `.claude/slated/scenes/pilot.md`
- Scenes: `.claude/slated/scenes/scene-<name>/`
- Takes: `.claude/slated/scenes/scene-<name>/takes/take-NNN.md`
- Wrap record: `.claude/slated/scenes/scene-<name>/wrap.md`
- Scene review: `.claude/slated/scenes/scene-<name>/review.md`
- Project report: `.claude/slated/report.md`
