---
name: master-plan
description: 'Build a deep, scenario-driven plan with a governing model, exhaustive conflict analysis, hard invariants, and a subdirectory of vertical-slice issue files. Use when: the user wants a thorough plan, says "master plan", needs a multi-user or multi-system audit, wants scenario analysis, or needs more depth than /write-plan provides.'
argument-hint: 'Describe the area to plan (e.g., "multi-user annotation workflow", "offline sync strategy")'
---

# Master Plan

Build a deep, scenario-driven plan and decompose it into a subdirectory of
numbered issue files ready for implementation. This is the heavy-duty planning
tool — use `/write-plan` for simpler checklists.

## When to use this vs /write-plan

| Signal                                         | Use            |
| ---------------------------------------------- | -------------- |
| Single-concern, known solution, < 20 items     | `/write-plan`  |
| Multi-actor, concurrency, or integration risks | `/master-plan` |
| Need to model failure modes exhaustively       | `/master-plan` |
| Requires design decisions with tradeoffs       | `/master-plan` |
| Items span multiple priorities / release phases | `/master-plan` |

## Output

Two artifacts inside the nearest `.plans/` directory:

```
.plans/
├── YYYY-MM-DD_<app>_<plan-name>.md          # The master plan document
└── YYYY-MM-DD_<app>_<plan-name>/            # Issue directory
    ├── 001_first-issue.md
    ├── 002_second-issue.md
    └── ...
```

## Procedure

### Phase 1 — Investigate

Explore the codebase to understand current behavior. Use `Explore` subagent
for breadth. Focus on:

- Data flow and storage patterns
- Identity / ownership model
- Existing error handling and edge cases
- What works today vs what the user thinks is broken

Don't assume — read the code. The plan must be grounded in actual architecture,
not guesses.

### Phase 2 — Establish the Governing Model

Name the conceptual framework that should govern all decisions. Borrow from
industry where applicable (e.g., "annotation adjudication pipeline",
"event sourcing", "optimistic concurrency", "CQRS"). The model should:

1. Define **phases** or **modes** the system operates in
2. State **key principles** (numbered, each one sentence + explanation)
3. Include a **"What This Means for Current Behavior"** table:

| Current behavior | Problem | Correct behavior |
| ---------------- | ------- | ---------------- |
| ...              | ...     | ...              |

Present the model to the user. If they disagree with the framing, iterate.
The model is the foundation — everything else flows from it.

### Phase 3 — Resolve Open Questions

Identify design questions that have multiple valid answers. For each:

- State the question
- List options with tradeoffs
- Recommend one
- Record the decision and rationale

Use `/grill-me` style interrogation. If user is not available not, make
recommendations and flag them as "assumed — confirm before implementing."

Track resolved decisions in a table:

| # | Question | Decision |
| - | -------- | -------- |
| 1 | ...      | ...      |

### Phase 4 — Exhaust Conflict Scenarios

Enumerate every failure mode, race condition, and edge case. For each scenario:

```markdown
### Scenario N: <Title>

**Actors:** <who>
**Setup:** <preconditions>

| Step | Action |
| ---- | ------ |
| 1    | ...    |

**Current risk:** <✅ Low / ⚠️ Medium / 🔴 High> — <explanation>
**Resolution:** <what the plan does about it, or "Accept as-is" with rationale>
```

Be exhaustive. Include scenarios that turn out to be safe (marked ✅) — they
document that you checked. Typical scenario categories:

- Parallel writes by different actors
- Stale reads after failed sync/upload
- Identity collisions
- Import/export attribution
- Mid-session state changes
- Clock skew / ordering issues

### Phase 5 — Write the Master Plan Document

Locate the `.plans/` directory (same rules as `/write-plan`). Write the plan
with this structure:

```markdown
# <Plan Title>

**App:** <name>  **Date:** <YYYY-MM-DD>  **Branch:** <branch>
**Scope:** <what's covered>
**Constraint:** <immovable constraints, e.g. "server API is immutable">

---

## Governing Model: <Model Name>

<Phases, key principles, current-vs-correct table>

---

## <Open Question Section> (if any remain partially open)

<Options, decision, rationale>

---

## Architecture Summary

<Current architecture, data types, sync/storage patterns,
 implementation approach for the proposed changes>

---

## Conflict Scenarios & Findings

### Scenario 1: ...
### Scenario 2: ...
...

---

## Priority 1: <Section Title>

**Files:** `<affected files>`

- [ ] Action item
- [ ] Action item

---

## Priority 2: ...

---

## Out of Scope (intentional exceptions)

- Item — reason

---

## Resolved Design Decisions

| # | Question | Decision |
| - | -------- | -------- |

---

## Hard Invariants

**Every implementation task MUST respect ALL invariants below.**

### <Category>

1. **<Invariant statement>** — <explanation>
2. ...
```

#### Hard Invariants — why they matter

Hard invariants are the most important section of the plan. They are
cross-cutting rules that apply to EVERY issue. Unlike acceptance criteria
(which are per-issue), invariants prevent systemic mistakes:

- Each invariant is numbered (INV-1, INV-2, ...) for cross-referencing
- Group by domain (e.g., "Phase Separation", "Identity", "Sync")
- Each issue file references the subset of invariants it must respect
- **Violation of any invariant is a ship-blocker**

### Phase 6 — Decompose into Issue Files

Create a subdirectory alongside the plan file with the same name (minus `.md`).
Populate it with numbered issue files — one per vertical slice.

**Ordering:** issues are numbered in **implementation order**, not priority
order. Foundational infrastructure comes first (even if it's Priority 3),
because other issues depend on it. Think of it as a topological sort of
the dependency graph.

**Issue file format:**

```markdown
# NNN: <Title>

**Type:** AFK / HITL
**Blocked by:** <NNN (description)> or "None — can start immediately"
**Priority:** <N> (<priority section name from the master plan>)

---

## What to build

<Concise description of the vertical slice. Describe end-to-end behavior.
 Include code sketches, API shapes, migration notes where helpful.>

## Hard Invariants

<List ONLY the invariants from the master plan that are relevant to this
 specific issue. Use the INV-N numbering.>

- **INV-N:** <statement>
- **INV-M:** <statement>

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
```

**Slice rules** (same as `/write-issue`):

- Each slice is a thin vertical cut through all affected layers
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- AFK slices can be implemented without human input
- HITL slices require a design decision or review

### Phase 7 — Review with User

Present the issue breakdown as a numbered list with titles, types, and
blockers. Ask:

- Does the granularity feel right?
- Are the dependency relationships correct?
- Should any issues be merged or split?
- Any missing scenarios or invariants?

Iterate until approved, then write the files.

## Skill Composition

This skill naturally chains with other skills:

- **Before:** `/grill-me` can be used during Phase 3 to resolve open questions
- **After:** `/write-issue` can prepare the issue files for GitHub Issues
- **During:** `/tdd` can be used when implementing individual issue files
