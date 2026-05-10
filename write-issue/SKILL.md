---
name: write-issue
description: Write a single detailed implementation issue file to a .plans/ directory. Produces structured issues with Type, Priority, Blocked-by, What-to-build, Hard Invariants, Acceptance criteria, Files-to-modify, and Test scenario. Use when user wants to write an issue, create a plan issue, file an implementation task, or mentions "to-issue".
---

# To Issue

Write a single structured implementation issue file to the nearest `.plans/` directory (or one specified by the user). Output matches the team's canonical format with full metadata, invariants, and testability.

## Process

### 1. Gather context

Work from whatever is in the conversation. If the user passes a GitHub issue number/URL, fetch it with `gh issue view`. Explore the codebase with subagents to understand affected files and existing patterns.

### 2. Determine placement

Find the nearest `.plans/` directory (or subdirectory) relative to the affected code. If multiple exist, ask the user. Determine the next sequence number by listing existing files (e.g., if `009_*.md` exists, next is `010`).

### 3. Draft the issue

Write the issue using the template below. Fill every section — no section may be omitted (except "Test scenario" for pure design-decision issues).

### 4. Present for review

Show the user the draft in chat. Ask:

- Is the scope right? (too broad / too narrow)
- Are the invariants correct?
- Should any acceptance criteria be added/removed?
- Is the priority correct?

Iterate until approved, then write the file.

---

## Issue Template

```markdown
# NNN: Title

**Type:** AFK | HITL | Bug | Design Decision
**Blocked by:** NNN (description) | None — can start immediately
**Priority:** N (Category)

---

## What to build

Concise description of the end-to-end behavior or fix. Include subsections as needed:

### Behavior / Flow / Column spec / API shape

Detailed behavioral spec with code examples if helpful.

### Architecture

Implementation approach — patterns to follow, data structures, threading model.

### Migration (if applicable)

What existing code moves or changes shape.

## Hard Invariants

- **INV-XX:** One-line invariant statement. Brief rationale.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] ...

## Files to modify

- `path/to/file.rs` — what changes in this file
- `path/to/other.rs` — what changes here

## Test scenario

1. Step-by-step manual verification
2. ...
```

---

## Field Definitions

### Type

| Value | Meaning |
|-------|---------|
| **AFK** | Can be implemented and merged without human decisions mid-flight |
| **HITL** | Requires a human decision, design review, or external input before completion |
| **Bug** | Fix for broken existing behavior |
| **Design Decision** | No code yet — the issue captures a decision to be made |

Combine if needed: `Bug / Design Decision` (bug exists, but fix approach needs a decision).

### Priority

Use the project's existing priority categories. Common pattern:

| N | Category |
|---|----------|
| 1 | Phase Separation (core UX boundaries) |
| 2 | Multi-User UX (collaboration features) |
| 3 | Data Integrity & Sync |
| 4 | Polish & Edge Cases |

If the project has no established categories, ask the user or infer from existing issues in the same `.plans/` folder.

### Hard Invariants

Numbered project-wide invariants (INV-XX) that this issue must uphold or introduces. Reference existing invariants from sibling issues. If introducing a new one, pick the next available number.

### Files to modify

List every file expected to change, with a short note on what changes. Helps the implementer scope the PR and helps reviewers know what to look for.

### Test scenario

Step-by-step manual test instructions (or automated test description) that an implementer can follow to verify the acceptance criteria are met. For AFK issues, this should be specific enough that an AI agent can self-verify.

---

## Rules

- **One issue per file.** Never combine multiple issues into one file.
- **Filename format:** `NNN_kebab-case-title.md` (e.g., `005_per-patient-annotator-badge.md`)
- **No GitHub issue creation.** This skill writes local `.plans/` files only.
- **Preserve sibling style.** Match formatting conventions (heading levels, checkbox style, invariant numbering) from existing issues in the same directory.
- **Explore before writing.** Use subagents to find affected files, existing invariants, and related patterns before drafting.
