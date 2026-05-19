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
**Build command:** `<quick compile/typecheck, e.g. make -j, npx tsc --noEmit>`
**Test command:** `<full test suite, e.g. make test, npm test, pytest>`

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

**Issues:** [001](<plan-dir>/001_slug.md), [002](<plan-dir>/002_slug.md)

---

## Priority 2: ...

**Issues:** [003](<plan-dir>/003_slug.md)

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

---

## Issue Map

**Single source of truth for plan progress.** `/implement-masterplan` reads
this table to determine execution order and current state.

| # | Title | Type | Session | Blocked by | Priority | Status |
| - | ----- | ---- | ------- | ---------- | -------- | ------ |
| 001 | [Title](<plan-dir>/001_slug.md) | AFK | AFK | None | P1 | ⬜ |
| 002 | [Title](<plan-dir>/002_slug.md) | AFK | AFK | 001 | P1 | ⬜ |
| 003 | [Title](<plan-dir>/003_slug.md) | HITL | HITL — UX review | 002 | P2 | ⬜ |
| ... | ... | ... | ... | ... | ... | ... |

### Issue Map Status Legend

| Symbol | Meaning |
| ------ | ------- |
| ⬜ | Not started |
| 🔄 | In progress |
| ✅ | Completed |
| ⏸️ | Blocked (dependency not met) |
| 🔴 | Failed (needs attention) |
| ⏭️ | Skipped (user chose to skip) |
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
Populate it with numbered issue files using the `/write-issue` format — one per
vertical slice.

**Ordering:** Issues are numbered in **implementation order** (topological sort
of the dependency graph), not priority order. Foundational infrastructure comes
first (even if it's Priority 3), because other issues depend on it.

**AFK vs HITL classification:**

| Type | Meaning | Implementation session |
| ---- | ------- | --------------------- |
| **AFK** | Can be implemented end-to-end without human decisions | Agent works autonomously via `/implement-issue` |
| **HITL** | Requires a design decision, review, or external input mid-flight | Agent pauses for user input; schedule when user is available |

Classify each issue explicitly. When in doubt, mark HITL — it's safer to pause
than to guess wrong. Issues that only need code changes with clear specs are
AFK. Issues involving UX choices, API contracts with external teams, or
ambiguous requirements are HITL.

**Issue file format:**

```markdown
# NNN: <Title>

**Type:** AFK | HITL
**Blocked by:** NNN (description) | None — can start immediately
**Priority:** N (Section title from master plan)
**Plan:** [<plan-name>](../<YYYY-MM-DD>_<app>_<plan-name>.md)
**Session:** AFK | HITL — <one-line reason>

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

## Files to modify

- `path/to/file.ext` — what changes in this file

## Test scenario

1. Step-by-step verification
2. ...
```

**Slice rules** (same as `/write-issue`):

- Each slice is a thin vertical cut through all affected layers
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- AFK slices can be implemented without human input
- HITL slices require a design decision or review
- Every issue links back to the plan via the **Plan:** field

After creating all issue files, **update the plan document**:

1. Add `**Issues:**` links to each Priority section
2. Fill the **Issue Map** table at the bottom of the plan

### Phase 7 — Review with User

Present the complete plan:

- The governing model and key principles
- Issue breakdown as a numbered list with titles, types, sessions, and blockers
- Total action item count and AFK/HITL split

Ask:

- Does the granularity feel right?
- Are the dependency relationships correct?
- Should any issues be merged or split?
- Any missing scenarios or invariants?
- Are the AFK/HITL classifications correct?

Iterate until approved, then write all files.

Report:
- The full path to the plan file
- The full path to the issues directory
- Number of priority sections
- Number of issues created (N AFK + M HITL)
- Total acceptance criteria count

## Skill Composition

This skill orchestrates other skills:

- **During Phase 3:** Uses `/grill-me` interrogation style to resolve decisions
- **During Phase 6:** Uses `/write-issue` format for issue files
- **After completion:** Plan is ready for `/implement-masterplan` to execute
- **During implementation:** `/tdd` can be used per issue when adding tests
