---
name: implement-masterplan
description: 'Execute a master plan produced by /master-plan. Reads the plan and its Issue Map, implements issues in dependency order using /implement-issue, creates a git commit per completed issue, and tracks progress in the Issue Map. Use when: user wants to implement a master plan, execute a master plan, work through master plan issues, or mentions "implement masterplan", "implement master plan".'
argument-hint: 'Path to .plans/ master plan file, or plan name (e.g., "structured-sequence-telemetry")'
---

# Implement Master Plan

Execute a `/master-plan` document end-to-end: read the plan, resolve the issue
dependency graph, implement issues in order via `/implement-issue`, create a
git commit per completed issue, and track progress in the plan's Issue Map.

## When to use this vs /implement-plan

| Signal | Use |
| ------ | --- |
| Plan was created by `/write-plan` or `/write-planB` | `/implement-plan` |
| Plan was created by `/master-plan` (has Issue Map, Hard Invariants, Conflict Scenarios) | `/implement-masterplan` |
| Need git commit per issue with structured messages | `/implement-masterplan` |
| Need explicit AFK/HITL session routing | `/implement-masterplan` |

## Output

- All issue acceptance criteria checked off
- Plan's Issue Map updated with completion status
- One git commit per completed issue (clean, reviewable history)
- Completion block appended to the plan document

## Procedure

### Phase 1 — Load the Plan

Resolve the plan file:

- If the user passes a path → read it directly.
- If the user passes a name (e.g., `structured-sequence-telemetry`) → find the
  matching `*_<name>.md` file in the nearest `.plans/` directory.
- If ambiguous → ask.

Read the full plan. Extract:

- **Governing Model** (the conceptual framework — keep in context throughout)
- **Hard Invariants** (cross-cutting constraints for all issues)
- **Issue Map** (the table linking to issue files with dependency/status/session)
- **Implementation Log** (if resuming — the table of what prior issues produced)
- **Priority sections** (for context on ordering intent)
- **Resolved Design Decisions** (for reference during implementation)
- **Out of Scope** (to avoid scope creep)
- **Conflict Scenarios** (to be aware of edge cases during implementation)

If the plan has no `## Implementation Log` section yet (first run), create it
in the plan file between the Issue Map and the end of the document:

```markdown
## Implementation Log

| # | Summary | Key export |
|---|---|---|
```

Locate the issues subdirectory (same name as plan file minus `.md`).

### Phase 2 — Build the Execution Order

Read all issue files from the issues directory. For each issue, extract:

- Issue number and title
- **Blocked by** dependencies
- **Type** (AFK vs HITL)
- **Session** (AFK or HITL with reason)
- Current status from the Issue Map (⬜ / 🔄 / ✅ / ⏸️ / 🔴 / ⏭️)

Build a dependency graph. Compute a topological execution order that respects
`Blocked by` constraints. Issues already marked ✅ or ⏭️ in the Issue Map are
skipped.

Present the execution order to the user:

```
Execution order:
1. 001 — Extend StenosisInfo mm fields (AFK, no blockers)
2. 002 — Build status JSON dual transport (AFK, blocked by 001)
3. 003 — Frontend TypeScript interfaces (AFK, blocked by 002)
4. 007 — Activity feed UX layout (HITL — UX review needed)
...
Skipping: 001 (already done)

Summary: 8 AFK issues, 3 HITL issues, 1 already done
HITL issues requiring user input: 007, 009, 012
```

Ask user:

- Proceed with all? Or select specific issues to implement?
- For HITL issues: implement now (will pause for decisions) or defer?
- Implement AFK issues first, then come back for HITL?

### Phase 3 — Implement Issues

For each issue in execution order:

#### 3a. Check Prerequisites

- Verify all `Blocked by` issues are completed (check Issue Map status)
- If a blocker is not done, skip and move to the next unblocked issue
- For HITL issues, confirm the user is ready to provide input

#### 3b. Load Issue Context

Before starting each issue, load exactly:

1. **Governing Model** (from the plan — keep in working memory throughout)
2. **Issue Map + Implementation Log** (from the plan — current progress and
   what prior issues produced)
3. **Current issue file** (the full issue being implemented)

Read additional issue files or plan sections freely if the current context is
insufficient — the Implementation Log tells you which prior issues are relevant
and what they exported.

**For plans with 10+ issues:** Avoid loading all issue files preemptively. Use
the Implementation Log to identify which 1-2 prior issues matter for the
current one, then read only those.

#### 3c. Stage Clean Working Tree

Before starting each issue, ensure the git working tree is clean:

```bash
git status --porcelain
```

If there are unstaged changes from a previous issue, commit or stash them
before proceeding. Each issue gets its own isolated commit.

#### 3d. Execute the Issue

Use `/implement-issue` procedure for each issue:

1. Read the issue file
2. Check prerequisites (blocked-by)
3. Plan implementation order across files
4. Implement acceptance criteria one at a time
5. Run tests after each criterion
6. Mark criteria complete in the issue file

#### 3e. Commit the Issue

After the issue passes all tests and acceptance criteria are met:

1. Stage all changes: `git add -A`
2. Create a commit with a structured message:

```
<type>: <issue-title> [#NNN]

Implements .plans/<plan-dir>/NNN_slug.md

Changes:
- <bullet summary of key changes>

Acceptance criteria: all met
Hard invariants: verified (INV-X, INV-Y)
```

Where `<type>` is derived from the issue:

| Issue content | Commit type |
| ------------- | ----------- |
| New feature / capability | `feat` |
| Bug fix | `fix` |
| Refactor without behavior change | `refactor` |
| Test-only changes | `test` |
| Documentation | `docs` |
| Infrastructure / build | `chore` |

3. Do NOT push. Commits stay local until user explicitly pushes.

#### 3f. Update Plan Progress + Implementation Log

After each issue is committed, update the plan file with two things:

**1. Issue Map** — change status from `⬜` to `✅` (or `🔄` if in progress,
`⏸️` if blocked by an incomplete issue).

**2. Implementation Log** — append a row to the `## Implementation Log` table
in the plan file. Write both the issue's Completion block (in the issue file)
and this log entry at the same time — you have full context right now.

Log entry format:

```markdown
| NNN | <behavioral summary of what this issue produced> | `<key exported symbol or path>` |
```

Example:

```markdown
| 003 | JSON status dual transport (WS broadcast + file write) | `StatusWriter.emitJson()` |
```

The "Key export" column captures what downstream issues can *use* — a function,
a type, an endpoint, a file path.

Also check off the corresponding items in the Priority sections of the plan
if they map to the completed issue's acceptance criteria.

### Phase 4 — Handle Failures

If an issue implementation fails (tests don't pass, invariant violated,
unexpected complexity):

1. Mark the issue as `🔴` in the Issue Map
2. Note the failure reason in the issue file's completion block
3. Check if downstream issues are blocked — mark them `⏸️`
4. **Do NOT commit broken code.** Revert uncommitted changes if needed:
   `git checkout -- .`
5. Ask user: fix now, skip and continue, or abort?

If skipping:
- Mark as `⏭️` in Issue Map
- Check if downstream issues can still proceed (might need re-evaluation)

If aborting:
- Leave the Issue Map reflecting actual state
- All previously committed issues remain in git history
- Add an "Implementation Notes" section to the plan with what was completed
  and what remains

### Phase 5 — Verify Invariants

After all issues are implemented (or the selected subset):

1. Re-read every **Hard Invariant** from the plan
2. Review the cumulative diff against each invariant:
   `git diff <base-commit>..HEAD`
3. Run the full test suite
4. Cross-check against **Conflict Scenarios** — verify resolutions hold
5. Report any invariant violations

### Phase 6 — Mark Plan Complete

If all issues are done, append a completion block to the plan:

```markdown

---

## Completion

**Status:** Done | Partial (N/M issues completed)
**Date:** YYYY-MM-DD
**Commits:** <first-commit-hash>..<last-commit-hash>
**Notes:** <summary of what was done>
**Remaining:** <list of incomplete issues, if partial>
```

Update the Issue Map one final time with all statuses.

Report to the user:
- Issues completed vs total (N AFK done, M HITL done, K skipped)
- Git commits created (with short hashes)
- Any invariant concerns
- Files modified across all issues
- Test suite status
- Suggested next step: `git log --oneline <base>..<head>` to review history

## Rules

- **Follow the plan.** The plan document + issue files are the source of truth.
  Do not deviate from the governing model or violate hard invariants.
- **One issue, one commit.** Each issue gets exactly one commit. This creates
  a clean, reviewable, bisectable git history.
- **Never push.** Commits stay local. The user decides when to push.
- **Respect the dependency graph.** Never implement an issue whose blockers
  are incomplete, unless the user explicitly overrides.
- **Tests must pass.** Never commit an issue if tests are failing.
- **Track progress continuously.** Update the Issue Map after every issue
  completion, not just at the end.
- **Stay in scope.** If implementation reveals a needed change outside the
  plan's scope, note it but don't implement it. Add it to the plan's
  "Implementation Notes" section for future planning.
- **HITL issues pause, not block.** When hitting an HITL issue, present the
  decision to the user and wait. Don't skip unless told to.
- **AFK issues proceed silently.** For AFK issues, implement without asking
  for approval at each step. Only pause on test failure or ambiguity.

## Issue Map Status Legend

| Symbol | Meaning |
| ------ | ------- |
| ⬜ | Not started |
| 🔄 | In progress |
| ✅ | Completed |
| ⏸️ | Blocked (dependency not met) |
| 🔴 | Failed (needs attention) |
| ⏭️ | Skipped (user chose to skip) |

## Git History Example

After implementing a 5-issue plan, `git log --oneline` should look like:

```
a1b2c3d feat: Activity feed component [#005]
e4f5g6h feat: callLLM service + one-liner queue [#004]
i7j8k9l feat: LLM prompts to JSON format [#003]
m0n1o2p feat: Frontend JSON status parsing [#002]
q3r4s5t feat: Backend StenosisInfo mm fields + JSON status [#001]
```

Each commit is self-contained, tested, and maps 1:1 to a plan issue.

## Skill Composition

- **Core loop:** Uses `/implement-issue` procedure for each issue file
- **Before:** Plan was created by `/master-plan`
- **During:** May invoke `/grill-me` style questions for HITL issues
- **Testing:** Uses TDD approach per `/tdd` when adding new test coverage

## Resumability

This skill is designed to be resumable. If a session ends mid-plan:

1. The Issue Map reflects which issues are done (✅) or failed (🔴)
2. The Implementation Log provides compressed context of what prior issues
   produced — no need to re-read all completed issue files
3. Git history contains commits for all completed issues
4. Re-invoking `/implement-masterplan` on the same plan picks up where it left
   off — completed issues are skipped automatically
5. The dependency graph is re-evaluated from current state
6. Use `git log --oneline` to verify which issues have commits

## Session Planning

Before starting, estimate the session scope:

- **AFK batch:** Group consecutive AFK issues with no HITL blockers. These
  can run in a single unattended session.
- **HITL checkpoints:** Identify where the agent will need user input. Plan
  breaks at these points.
- **Suggested session plan:**

```
Session 1 (AFK): Issues 001–003 (foundational, no user input needed)
Session 2 (HITL): Issue 004 (API design review needed)
Session 3 (AFK): Issues 005–008 (implementation, blocked by 004)
Session 4 (HITL): Issue 009 (UX review)
```

Present this session plan to the user before starting so they can plan their
availability.
