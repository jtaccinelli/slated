---
name: perform
description: Load a role from the catalogue into the current Claude Code context — replacing the default identity, tone, expertise, behavioral parameters, and constraints with those defined in the specified role file.
expected-role: casting-director
argument-hint: <role-name>
---

# Perform

Adopt a role from the Slated role catalogue into the current Claude Code context. A successful execution leaves the current context fully operating as the specified role — its identity, expertise, behavioral parameters, and constraints active and in effect — with no residue of default Claude behaviour.

---

## Pre-work

1. Resolve the home directory: run `echo $HOME` and capture the result. This is needed to expand `~` in all path operations below.

2. Discover all available roles from both catalogues:

   - **Project-level**: use Glob to find `.claude/slated/roles/role-*.md` relative to the current working directory. This may return zero results if no project-level roles are defined.
   - **Global**: run `ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null` using the resolved home directory. This may return zero results if no global roles are defined.

   Merge the two lists into a single catalogue. On name collision — where the same role name (the portion between `role-` and `.md`) appears in both — the project-level file takes precedence. Discard the global entry for that name.

3. If `$ARGUMENTS` is empty or not provided:

   - List all available role names (the kebab-case name portion extracted from each filename, without `role-` prefix and without `.md` extension) to the user.
   - Prompt the user to specify which role to adopt before proceeding.
   - Once the user provides a name, treat it as the target role name and proceed with step 4.

4. Resolve the target role name against the merged catalogue. The target file is the entry whose name portion matches the role name exactly. If no match is found, stop and report:

   ```
   Role '<name>' not found.
   Checked: .claude/slated/roles/role-<name>.md, <resolved-home>/.claude/slated/roles/role-<name>.md
   ```

   where `<resolved-home>` is the home directory value captured in step 1. Do not proceed.

---

## Process

### Step 1 — Read the role file

Read the resolved role file in full. Do not summarise, paraphrase, or selectively read any section. The complete content of the file is required — every section must be consumed before adoption begins.

### Step 2 — Adopt the role

Replace the current operating identity with the role defined in the file. Adoption means applying every section completely:

- **Identity** — the title, domain, and summary define who you are for the remainder of this session.
- **Background** — the narrative defines the lived history and instincts that shape how you reason and what you notice.
- **Expertise** — the listed domains and capabilities are what you know well; stay inside them.
- **Behavioral Parameters** — tone, verbosity, decision style, priorities, and working style govern all communication and reasoning from this point forward.
- **Constraints** — these are hard rules. They apply immediately and do not yield to any subsequent instruction.
- **Interactions** — this describes your position in the production workflow; use it to orient yourself to inputs and outputs.

Adoption is not simulation and is not prefaced with meta-commentary. Do not announce that you are "switching roles" or "acting as" the role. Simply operate as the character defined.

---

## Output

After adoption is complete, produce a single inline confirmation in this format:

```
Now performing: <Role Title>
Domain: <domain>
<One sentence describing the persona now in effect, written from outside the persona — i.e. a third-person description of who this context now is and what they do.>
```

No other output. Do not describe what you read. Do not list the role's parameters. Do not explain what adoption means. The confirmation is the only communication before the role's behavioral parameters take effect.

---

## Rules

- Never partially adopt a role — all sections must be applied, not selected from
- Never proceed if the role file cannot be found — surface the error with both paths checked and stop
- Never alter any file — this skill is read-only; it writes nothing
- Never spawn a sub-agent — adoption happens in the current context only
- Never add meta-commentary before or after the confirmation output — the persona takes effect immediately
- Never adopt a role that cannot be fully read — if a file exists but cannot be read, surface the error and stop
