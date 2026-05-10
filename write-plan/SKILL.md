---
name: write-plan
description: 'Create a structured action plan and save it to the nearest .plans/ directory. Use when: writing a plan, creating a task plan, audit plan, migration plan, refactor plan, feature plan, improvement plan, or when user says "write a plan", "make a plan", "plan this".'
argument-hint: 'Describe what the plan is for (e.g., "migrate auth to OAuth2", "audit accessibility")'
---

# Write Plan

Create a structured, actionable plan file in the nearest `.plans/` directory.

## Output

A markdown file at `<plans_dir>/<YYYY-MM-DD>_<app_name>_<plan-name>.md` containing prioritized, checkboxed action items grouped by section.

## Procedure

### 1. Locate the Plans Directory

Search upward from the current working directory for the nearest `.plans/` folder. Check these locations in order:

1. `.plans/` in the current project root
2. `.github/.plans/`
3. `.vscode/.plans/`
4. `.claude/.plans/`
5. Parent directories (repeat search)

If no `.plans/` directory exists, create one at the project root (the directory containing `package.json`, `pyproject.toml`, `Cargo.toml`, or `.git`).

### 2. Determine Plan Metadata

Infer or ask:

| Field | Source |
|-------|--------|
| **date** | Today's date (`YYYY-MM-DD`) |
| **app_name** | From `package.json` name, directory name, or ask user |
| **plan_name** | From user's description, kebab-cased (e.g., `filament-alignment`, `oauth-migration`) |

Filename: `<date>_<app_name>_<plan-name>.md`

### 3. Gather Plan Content

If the conversation already contains enough context (e.g., audit findings, a list of tasks), use that directly. Otherwise, explore the codebase and/or ask the user:

- What is the goal of this plan?
- What areas/files are affected?
- Are there priority levels or ordering constraints?

### 4. Write the Plan

Use this structure:

```markdown
# <Plan Title>

**App:** <app name>
**Date:** <YYYY-MM-DD>
**Branch:** <current git branch>
<any other relevant metadata>

---

## Priority 1: <Section Title>

**Files:** `<affected files>`

- [ ] Action item with enough detail to execute
- [ ] Another action item

---

## Priority 2: <Section Title>

...

---

## Out of Scope (intentional exceptions)

- Item and reason it's excluded
```

#### Guidelines

- Each section groups related changes by priority (most impactful first)
- List affected **files** per section so the executor knows where to work
- Each checkbox item should be a single, completable action (not a paragraph)
- Include an "Out of Scope" section for things deliberately excluded
- Use domain-specific terminology from the codebase
- Keep items concrete: name components, functions, tokens — not vague intentions

### 5. Confirm

After writing, report:
- The full path to the created file
- Number of priority sections
- Total number of action items
