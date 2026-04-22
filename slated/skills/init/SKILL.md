---
name: init
description: Initialise the global Slated framework directories — creating ~/.claude/slated/roles/ and ~/.claude/slated/backgrounds/ and seeding the bundled crew role definitions on a new device or user account.
expected-role: casting-director
argument-hint: []
---

# Init

Bootstrap the global framework directories on this device. This skill is run once per device — it creates the persistent global locations where crew roles and backgrounds live, seeds the bundled crew role definitions, and makes them available across all projects.

---

## Pre-work

1. Resolve the user's home directory (expand `~` to the actual path via `echo $HOME` or equivalent)
2. Check whether `~/.claude/slated/roles/` already exists
3. Check whether `~/.claude/slated/backgrounds/` already exists

---

## Process

### Step 1 — Create global roles directory

If `~/.claude/slated/roles/` does not exist, create it. If it already exists, note that it will be left untouched.

### Step 2 — Create global backgrounds directory

If `~/.claude/slated/backgrounds/` does not exist, create it. If it already exists, note that it will be left untouched.

### Step 3 — Seed crew roles

The bundled crew role definitions live alongside this skill at `${CLAUDE_SKILL_DIR}/roles/`. For each of the following files, copy it to `~/.claude/slated/roles/` — **skipping any file that already exists there**:

- `role-casting-director.md`
- `role-director.md`
- `role-producer.md`
- `role-set-designer.md`
- `role-visualiser.md`
- `role-writer.md`

Read each file from `${CLAUDE_SKILL_DIR}/roles/<filename>` and write its contents to `$HOME/.claude/slated/roles/<filename>` (using the resolved home directory from Pre-work). Do not overwrite.

### Step 4 — Report status

Report the outcome for each directory and each role file:

```
~/.claude/slated/roles/       ✓ created   |  ✓ already exists
~/.claude/slated/backgrounds/ ✓ created   |  ✓ already exists

Crew roles:
  role-casting-director.md    ✓ written   |  — skipped (already exists)
  role-director.md            ✓ written   |  — skipped (already exists)
  role-producer.md            ✓ written   |  — skipped (already exists)
  role-set-designer.md        ✓ written   |  — skipped (already exists)
  role-visualiser.md          ✓ written   |  — skipped (already exists)
  role-writer.md              ✓ written   |  — skipped (already exists)
```

Confirm that the framework is ready to use on this device.

---

## Output

- `~/.claude/slated/roles/` — global directory for crew and cast role definitions, seeded with bundled crew roles
- `~/.claude/slated/backgrounds/` — global directory for reusable background documents

---

## Rules

- Never overwrite existing role files — skip any file that already exists in the destination
- Never delete existing files inside either directory
- Never proceed silently if the home directory cannot be resolved — surface the error and stop
- Never proceed silently if a bundled role file cannot be read from `${CLAUDE_SKILL_DIR}/roles/` — surface the error and stop
