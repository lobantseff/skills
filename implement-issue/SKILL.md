---
name: implement-issue
description: 'Implement a .plans/ issue file produced by /write-issue. Reads the issue, plans implementation order, executes changes with tests, and marks acceptance criteria complete. Use when: user wants to implement an issue, execute a plan issue, work on a .plans/ task, start an AFK issue, or mentions "implement issue".'
argument-hint: 'Path to .plans/ issue file, or issue number (e.g. 014)'
---

# Implement Issue

Execute a `.plans/` issue file end-to-end: read the spec, implement changes, run tests, and mark the issue complete.

## Process

### 1. Load the issue

Resolve the issue file:
- If the user passes a path → read it directly.
- If the user passes a number (e.g. `014`) → find the matching `NNN_*.md` file in the nearest `.plans/issues/` directory.
- If ambiguous → ask.

Read the full issue. Extract:
- **Acceptance criteria** (the checkbox list)
- **Files to modify** (the implementation scope)
- **Hard Invariants** (constraints that must never be violated)
- **Blocked by** (prerequisites — abort if unmet)
- **Test scenario** (verification steps)

### 2. Check prerequisites

If the issue has a **Blocked by** entry that is not marked complete, warn the user and ask whether to proceed anyway. For `AFK` issues, abort unless the user overrides.

### 3. Plan implementation order

Use subagents to explore each file listed in **Files to modify**. Then decide on an implementation order that:
- Starts with the lowest-dependency changes (types, data structures).
- Moves to producers, then consumers.
- Ends with test updates.

Present the implementation order to the user as a numbered todo list. Wait for approval before proceeding. If the issue is `AFK` type, proceed without approval.

### 4. Implement — one acceptance criterion at a time

For each acceptance criterion:

1. **Mark the todo in-progress.**
2. **Read the relevant files** to understand current state.
3. **Make the code changes.** Follow the issue's spec closely — do not add unrequested features or refactors.
4. **Run the relevant tests** to check nothing is broken. If the issue specifies a test file, run that. Otherwise run the project's default test command.
5. **If tests fail**, diagnose and fix before moving on.
6. **Mark the todo completed.**

### 5. Run full verification

After all acceptance criteria are addressed:

1. Run the full test suite (or the scope specified in **Test scenario**).
2. Verify every **Hard Invariant** is upheld by reviewing the diff.
3. If the issue includes a **Test scenario**, follow its steps to manually verify.

### 6. Mark issue complete

Update the issue `.md` file:

- Check off every acceptance criterion checkbox (`- [ ]` → `- [x]`).
- Append a completion block at the bottom of the file:

```markdown

---

## Completion

**Status:** Done
**Date:** YYYY-MM-DD
**Notes:** <one-line summary of what was done, or "Implemented as specified">
```

Confirm to the user that the issue is complete.

## Rules

- **Follow the spec.** The issue file is the source of truth. Do not deviate from its behavioral description unless something is clearly wrong (in which case, ask).
- **One criterion at a time.** Do not batch multiple acceptance criteria into a single editing pass. This keeps changes reviewable and rollback-friendly.
- **Tests must pass.** Never mark a criterion complete if tests are failing.
- **Respect invariants.** Before marking completion, re-read every Hard Invariant and verify the implementation upholds it.
- **Do not modify files outside scope.** Only touch files listed in **Files to modify** unless a transitive dependency clearly requires it. If you need to touch an unlisted file, note it in the completion block.
- **Preserve issue format.** When updating checkboxes, change only the `[ ]` → `[x]` characters. Do not reformat or rewrite the issue content.
