---
name: implement-issue
description: 'Implement a .plans/ issue file produced by /write-issue. Reads the issue, plans implementation order, executes changes with tests, and marks acceptance criteria complete. Use when: user wants to implement an issue, execute a plan issue, work on a .plans/ task, start an AFK issue, or mentions "implement issue".'
argument-hint: 'Path to .plans/ issue file, or issue number (e.g. 014)'
---

# Implement Issue

Execute a `.plans/` issue file end-to-end: read the spec, implement changes, run tests, mark the issue complete, update the parent plan, and commit.

## Process

### 1. Load the issue

Resolve the issue file:
- If the user passes a path → read it directly.
- If the user passes a number (e.g. `014`) → find the matching `NNN_*.md` file in the nearest `.plans/` subdirectory (check both `issues/` and plan-named dirs).
- If ambiguous → ask.

Read the full issue. Extract:
- **Acceptance criteria** (the checkbox list)
- **Files to modify** (the implementation scope)
- **Hard Invariants** (constraints that must never be violated)
- **Blocked by** (prerequisites — abort if unmet)
- **Test scenario** (verification steps)
- **Type** (`AFK` or `HITL`)

### 2. Check prerequisites

If the issue has a **Blocked by** entry that is not marked complete, warn the user and ask whether to proceed anyway. For `AFK` issues, abort unless the user overrides.

### 3. Discover repo conventions

**Before writing any code, find the repo's own contributor guidance and treat it as binding.** For each module you'll touch, look for and read:

- `CONTRIBUTING.md` in the module root (and any files it links to).
- `.github/copilot-instructions.md` and `AGENTS.md` at the repo or module level.
- Composed/nested guides — many repos build their rules from submodules (e.g. under `External/`, `lib/`, `packages/`). Follow the links these files reference and read them too.

If such files exist, follow them for everything you generate: namespaces, async/coroutine patterns, logging, error handling, validation at trust boundaries, naming, header conventions, test fixtures, test names, platform guards, and the build/test commands. When this guidance conflicts with this skill's generic defaults, **the repo's guidance wins**. If no such files exist, fall back to the generic defaults below.

### 4. Explore and plan implementation order

Use subagents or file reads to explore each file listed in **Files to modify**. Then decide on an implementation order that:
- Starts with the lowest-dependency changes (types, data structures).
- Moves to producers, then consumers.
- Ends with test updates.

**AFK issues:** Proceed directly — do not ask for approval.
**HITL issues:** Present the implementation order and wait for approval.

### 5. Implement — one acceptance criterion at a time

For each acceptance criterion:

1. **Mark the todo in-progress.**
2. **Read the relevant files** to understand current state.
3. **Make the code changes.** Follow the issue's spec closely — do not add unrequested features or refactors. Honor the conventions found in step 3 (namespaces, async/logging/error patterns, naming, tests).
4. **Run the build command** (from the parent plan's `Build command:` field, the module's `CONTRIBUTING.md` Build & Test section, or discover: check for Makefile/CMakeLists, package.json, Cargo.toml, pyproject.toml). If the issue's "Test scenario" specifies a different command, use that instead.
5. **If build fails**, diagnose and fix before moving on.
6. **Mark the todo completed.**

### 6. Run full verification

After all acceptance criteria are addressed:

1. Run the **full test command** (from the parent plan's `Test command:` field, the module's `CONTRIBUTING.md`, or discover). Only treat *new* failures as blockers if pre-existing failures were noted.
2. Verify every **Hard Invariant** is upheld by reviewing the diff.
3. If the issue includes a **Test scenario**, follow its steps.

### 7. Mark issue complete

Update the issue `.md` file:

- Check off every acceptance criterion checkbox (`- [ ]` → `- [x]`).
- Append a completion block:

```markdown

---

## Completion

**Status:** Done
**Date:** YYYY-MM-DD
**Notes:** <one-line summary of what was done, or "Implemented as specified">

Commit: <type>(<scope>): <short description>
```

### 8. Update the parent plan

If this issue belongs to a master plan (check for a **Plan:** field in the issue
header, or look for a `.md` file in the parent directory):

1. Find the corresponding checkboxes in the plan's Priority sections → mark `[x]`.
2. Find this issue's row in the **Issue Map** table → change status from `⬜` to `✅`.
3. Check if any downstream issues (those with `Blocked by: NNN` pointing to this
   issue) can now be unblocked → change their status from `⏸️` to `⬜` if all
   their blockers are now `✅`.

This keeps the master plan as a live progress dashboard for `/implement-masterplan`.

### 9. Commit

**Pre-commit checklist (MUST complete ALL before staging):**

1. ✅ All acceptance criteria checkboxes marked `[x]` in the issue file
2. ✅ `## Completion` block appended to the issue file (see step 7)
3. ✅ Parent plan updated (if applicable, see step 8)

Only after all three are verified, stage all changed files (implementation + issue file + plan file) and commit:

```
<type>(<scope>): <concise title>

- <bullet 1: key change>
- <bullet 2: key change>
- ...

Issue: NNN_slug
```

Commit type mapping:
| Issue content | type |
|---|---|
| New feature | `feat` |
| Bug fix | `fix` |
| Refactor | `refactor` |
| Test-only | `test` |
| Docs | `docs` |

**Do NOT push.** Commits stay local until user explicitly pushes.

### 10. Report completion

For **AFK** issues: brief one-line confirmation + offer to proceed to next issue.
For **HITL** issues: present a summary table of changes and wait for user review.

## Rules

- **Honor repo conventions.** When a module provides `CONTRIBUTING.md`, `.github/copilot-instructions.md`, `AGENTS.md`, or guides they link to (including submodule guides under `External/`, `lib/`, `packages/`, etc.), read and follow them for all generated code and tests. They override this skill's generic defaults when they conflict.
- **Follow the spec.** The issue file is the source of truth. Do not deviate from its behavioral description unless something is clearly wrong (in which case, ask).
- **One criterion at a time.** Do not batch multiple acceptance criteria into a single editing pass. This keeps changes reviewable and rollback-friendly.
- **Tests must pass.** Never mark a criterion complete if tests are failing.
- **Respect invariants.** Before marking completion, re-read every Hard Invariant and verify the implementation upholds it.
- **Do not modify files outside scope.** Only touch files listed in **Files to modify** unless a transitive dependency clearly requires it (e.g. fixing a call-site type mismatch). If you need to touch an unlisted file, note it in the completion block.
- **Preserve issue format.** When updating checkboxes, change only the `[ ]` → `[x]` characters. Do not reformat or rewrite the issue content.
- **AFK = autonomous.** For AFK issues, do not pause for approval between steps. Implement → test → update issue → update plan → commit → report. One fluid pass.
- **HITL = pause points.** For HITL issues, pause after presenting the plan and after completion for user review before committing.
- **Never commit without `## Completion`.** The `## Completion` block in the issue file is mandatory. If you are about to run `git commit` and the issue file doesn't have it yet, STOP and add it first.
