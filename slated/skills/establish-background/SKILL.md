---
name: establish-background
description: Define a new background document in a single drafting pass — covering semantics, structure, and function for a specific technology or domain. The user confirms the destination (global vs project) before the file is written.
expected-role: casting-director
argument-hint: [technology or domain to document]
---

# Establish Background

Define a new background document from scratch in a single drafting pass. A background is domain knowledge — the conventions, patterns, and rules an actor needs to operate effectively in a given technology or domain. After establishing the domain through dialogue, the casting director drafts all three sections and presents the complete document. The user confirms only the destination (global vs project-specific) before the file is written. The output is a complete, immediately usable background document.

---

## Pre-work

1. Use Bash (`ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`) to list global backgrounds, and Glob `.claude/slated/backgrounds/background-*.md` for project-level ones — merge both results, with project-level taking precedence; read each in full to understand what is already documented and where its edges are
2. If `$ARGUMENTS` describes a domain that an existing background already covers, surface this to the user — ask whether extending the existing background is preferable to creating a new one

Only proceed with a new background once the domain is confirmed as genuinely uncovered or sufficiently distinct.

---

## Process

### Step 1 — Establish the domain

Use `$ARGUMENTS` as a starting point. Ask the user:

1. What technology, tool, or domain does this background cover?
2. What kind of actor would load this background — what role would benefit from knowing it?
3. What makes this domain distinct enough to warrant its own background rather than an addition to an existing one?

Do not proceed until the domain is clear and confirmed as distinct.

### Step 2 — Draft the complete background

Using the understanding established in Step 1, draft the full background document in a single pass. Produce all three sections:

1. **Opening statement** — a one-sentence description of what domain the background covers and why it matters to an actor filling a relevant role. It must be specific enough that an actor immediately understands what they are about to absorb and why it is relevant to their work.

2. **Semantics** — naming conventions, terminology, typing idioms, and syntax choices specific to this domain. It answers: what are things called, how are they typed, and what does correct syntax look like? Every entry must tell an actor exactly what to write, not just what to think about. Vague guidance ("use descriptive names") is not acceptable — produce concrete patterns.

3. **Structure** — file and directory layout, where things live, import paths, and how the domain organises its artefacts. It answers: where does this code go, and how does it relate to other files? Include concrete directory trees and import path examples where they add clarity. The structure must be reproducible from this section alone.

4. **Function** — runtime behaviour, patterns for achieving outcomes, and recipes for common operations. It answers: how does this domain behave when running, and how do you make it do things correctly? Include concrete code examples where they make a convention unambiguous. Every pattern must be complete enough to implement from.

Present the complete draft to the user and note any sections that may need refinement. If the user requests changes, revise and re-present before writing.

### Step 3 — Confirm the destination

Derive a kebab-case name from the domain (e.g. "Tailwind CSS" → `tailwind`, "React Router 7" → `react-router`).

Ask the user whether this background should be:

1. **Global** — reusable across all projects on this device → writes to `$HOME/.claude/slated/backgrounds/background-<name>.md`
2. **Project-specific** — only available in this project → writes to `.claude/slated/backgrounds/background-<name>.md`

Present the full file path for confirmation before writing.

### Step 4 — Write the background file

Write the background file to the confirmed location. Every section must be fully populated — no placeholder text, no empty fields.

---

## Output

- `$HOME/.claude/slated/backgrounds/background-<name>.md` or `.claude/slated/backgrounds/background-<name>.md` — complete, immediately usable background document at the confirmed location

Report the file path.

---

## Rules

- Never leave a section unpopulated — if the draft cannot produce a specific value for a section, surface the gap to the user before writing
- Never extend an existing background when a new one is warranted, and never create a new one when an existing one could be extended — surface the overlap and ask first
- Never write vague conventions — every entry must be specific enough that an actor would know exactly what to write
- Never include domain knowledge that belongs in a role definition — backgrounds document what to know, not how to behave
- Never write without confirming the destination (global vs project) first
