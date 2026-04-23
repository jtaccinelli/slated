# Manuscript: <Scene Name>

**Status**: confirmed | in-progress | complete
**Requirement**: <one-line description of the feature or bug being addressed>

---

## Cast

The actors required to run this scene, and what each one must be loaded with.

| Actor | Role | Backgrounds |
|---|---|---|
| actor-1 | `<role-name>` | `<bg-1>`, `<bg-2>` |
| actor-2 | `<role-name>` | `<bg-1>` |

---

## Shot Locations

Key file and directory paths actors should reference when executing this scene. Derived from `.claude/slated/locations.md` — include only entries relevant to this scene's work.

| Name | Path |
|---|---|
| `<location-name>` | `<path/relative/to/project>` |

---

## Resources

External platforms, APIs, services, or tools this scene depends on.

- <resource or integration required>
- <resource or integration required>

---

## Objectives

What a successful run of this scene must produce. Written as verifiable outcomes, not vague intentions. The director uses these to assess each take.

- [ ] <specific, verifiable outcome>
- [ ] <specific, verifiable outcome>

---

## Actions

The step-by-step sequence of work this scene requires. Each action is assigned to an actor and describes the expected output, not the implementation method. Actions are ordered — earlier outputs may be inputs to later steps.

### Shot 1: <shot title>

**Actor**: `actor-1` (`<role-name>`)

1. <atomic action — what to do and what the output should be>
2. <atomic action — what to do and what the output should be>

### Shot 2: <shot title>

**Actor**: `actor-2` (`<role-name>`)

1. <atomic action — depends on output of Shot 1>
2. <atomic action>

---

## Notes

<Any context the writer wants to preserve that does not fit the structure above — constraints discovered during planning, edge cases to watch for, or decisions made and why.>

---

<!--
  The writer replaces this comment block and the stub below with pre-shoot review feedback
  if the review loop flags unresolved ROLE GAP issues. MANUSCRIPT GAP findings are fixed
  directly in the Actions/Objectives sections above and are never documented here.
  If the review returned clean (or all gaps were resolved), remove this section entirely.
-->

## Pre-Shoot Review

> No review feedback recorded. Remove this section if the pre-shoot review returned clean.

### Director — Role Assessment

| Role | Finding |
|---|---|
| `<role-name>` | <specific gap in role definition> |

### Actor Assumptions

| Actor | Shot | Assumption | Category |
|---|---|---|---|
| `actor-1` | Shot 1 | <assumption text> | ROLE GAP |

---

<!--
  The director replaces this entire comment block and the stub below with the completed
  Completion Record when the scene is marked complete. Do not populate it manually during drafting.
-->

## Completion Record

> Replace this stub and the comment above with the completed record when finalising.

**Completed**: <YYYY-MM-DD>
**Takes required**: <N>

### Take Log

| Take | Status | Primary Failure Reason |
|---|---|---|
| 001 | fail | <brief reason> |
| 002 | pass | — |

### Role Improvement Notes

Patterns of repeated error across takes that suggest a gap in a cast role definition or background document. Omit entirely if no meaningful patterns emerged.

#### <role-name>

- <specific gap in the role definition that caused repeated errors>
- <background document that was missing or incomplete>

### Manuscript Improvement Notes

Structural issues in this manuscript that caused actor confusion or deviation — ambiguous action wording, missing sequencing constraints, underspecified objectives. Omit entirely if none emerged.

- <structural issue that led to ambiguity>
