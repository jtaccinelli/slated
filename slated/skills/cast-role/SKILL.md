---
name: cast-role
description: Define a new role in a single drafting pass — covering identity, background, expertise, behavioral parameters, constraints, and interactions. The user confirms the destination (global vs project) before the file is written.
expected-role: casting-director
argument-hint: [brief description of the role needed]
---

# Cast Role

Define a new role from scratch in a single drafting pass. After establishing the need through dialogue, the casting director drafts the complete role definition and presents it to the user. The user confirms only the destination (global vs project-specific) before the file is written. The output is a complete, production-ready role definition — no placeholders, no assumptions left by the casting director.

---

## Pre-work

1. Use Bash (`ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null`) to list global roles, and Glob `.claude/slated/roles/role-*.md` for project-level ones — merge both results, with project-level taking precedence; read each in full to understand what roles already exist, what they cover, and where their edges are
2. If `$ARGUMENTS` describes a need that an existing role could plausibly cover, surface this to the user before proceeding — ask whether refining the existing role via `/slated:refine-role` is preferable to defining a new one

Only proceed with a new definition once the user has confirmed the need is genuinely distinct from existing roles.

---

## Process

### Step 1 — Establish the need

Use `$ARGUMENTS` as a starting point. Ask the user:

1. What is this role responsible for? What problem does it solve in the production?
2. What kind of work will actors in this role be doing?
3. What makes this role distinct from any existing role in the catalogue?

Do not proceed until the need is clearly distinct from what already exists. If the user's answers reveal overlap with an existing role, name the overlap and ask directly whether a new role is still warranted.

### Step 2 — Draft the complete role definition

Using the understanding established in Step 1 and the context of the current project, draft the full role definition in a single pass. Produce all sections:

1. **Identity** — title, domain (one or two words), and a one-sentence summary. The summary should read as a character description, not a job posting.
2. **Background narrative** — a 2–3 paragraph professional history that shapes how this role thinks. It is not a skills list — it is a narrative explaining the role's perspective, instincts, and accumulated experience. It should read as a convincing, lived-in character.
3. **Expertise** — a focused list of specific domains, capabilities, or artefacts this role owns. Every item must be specific enough that it would not appear on another role's list. Reject generic items such as "problem-solving" or "communication".
4. **Behavioral parameters** — all five: tone, verbosity, decision style, priorities (in priority order), and working style. Each must be specific enough to meaningfully constrain actor behaviour, not describe any capable person in general.
5. **Constraints** — hard limits on what this role does and does not do. Each constraint must be precise enough that an actor knows exactly when to stop. For **cast roles**: state scope boundaries in terms of the work itself, not in terms of which other role owns it. For **crew roles**: peer role references are acceptable.
6. **Interactions** — what this role receives as input, what it produces as output, and at what point in the scene lifecycle it appears. For **cast roles**: describe handoffs in terms of what is passed, not who passes it — cast roles discover co-stars from the manuscript. For **crew roles**: other roles may be named directly.

Present the complete draft to the user and note any sections that may need refinement. If the user requests changes, revise and re-present before writing.

### Step 3 — Confirm the destination

Derive a kebab-case name from the role title (e.g. "Design Engineer" → `design-engineer`).

Ask the user whether this role should be:

1. **Global** — available across all projects on this device → writes to `$HOME/.claude/slated/roles/role-<name>.md`
2. **Project-specific** — only available in this project → writes to `.claude/slated/roles/role-<name>.md`

Present the full file path for confirmation before writing.

### Step 4 — Write the role file

Write the role file to the confirmed location. Every section must be fully populated — no placeholder text, no empty fields.

---

## Output

- `$HOME/.claude/slated/roles/role-<name>.md` or `.claude/slated/roles/role-<name>.md` — complete, production-ready role definition at the confirmed location

Report the file path. If the role's effective deployment will require one or more background documents that do not yet exist, name them specifically so they can be defined before any scene casts this role.

---

## Rules

- Never leave a section unpopulated — if the draft cannot produce a specific value for a section, surface the gap to the user before writing
- Never embed domain knowledge in the role definition — behavioral parameters only; knowledge belongs in backgrounds
- Never create a new role if an existing one can be adjusted — surface the overlap and ask first
- Never write vague constraints — every constraint must name exactly what the actor will not do
- Never write generic expertise items — every item must be specific enough that it would not appear on another role's list
- Never write without confirming the destination (global vs project) first
- For cast roles: never name a specific peer role in a constraints or interactions section — cast roles discover co-stars from the manuscript at execution time
- For crew roles: peer role references are acceptable in constraints and interactions — crew operate within the meta-narrative with full production awareness
