---
name: update
description: Update local crew role files from the bundled definitions — overwriting any that already exist. Run after pulling framework changes to propagate updated roles to ~/.claude/slated/roles/.
expected-role: casting-director
argument-hint: []
---

# Update

Propagate the latest bundled crew role definitions to the global framework directory. Unlike `/slated:init`, this skill **overwrites** existing files — use it after pulling framework changes to ensure local crew roles match the current bundled versions.

---

## Pre-work

1. Resolve the user's home directory (expand `~` to the actual path via `echo $HOME` or equivalent)
2. Confirm that `~/.claude/slated/roles/` exists — if it does not, tell the user to run `/slated:init` first and stop

---

## Process

### Step 1 — Overwrite crew roles

The bundled crew role definitions live at `${CLAUDE_SKILL_DIR}/../init/roles/`. For each of the following files, read it from that directory and write it to `~/.claude/slated/roles/` — **overwriting any existing file**:

- `role-casting-director.md`
- `role-director.md`
- `role-producer.md`
- `role-set-designer.md`
- `role-visualiser.md`
- `role-writer.md`

### Step 2 — Report status

Report the outcome for each role file:

```
Crew roles updated in ~/.claude/slated/roles/:
  role-casting-director.md    ✓ updated
  role-director.md            ✓ updated
  role-producer.md            ✓ updated
  role-set-designer.md        ✓ updated
  role-visualiser.md          ✓ updated
  role-writer.md              ✓ updated
```

Confirm that local crew roles are now in sync with the bundled definitions.

---

## Output

- `~/.claude/slated/roles/` — global crew role definitions, overwritten with the latest bundled versions

---

## Rules

- Always overwrite — never skip existing files (that is the purpose of this skill vs `/slated:init`)
- Never delete files from `~/.claude/slated/roles/` that are not in the bundled list — project-defined cast roles must not be touched
- Never proceed if `~/.claude/slated/roles/` does not exist — surface the error and direct the user to run `/slated:init` first
- Never proceed silently if a bundled role file cannot be read — surface the error and stop
