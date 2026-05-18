---
name: encyclopedia
description: 'Build a comprehensive repository encyclopedia for new-member onboarding. Produces a single markdown file with TLDR, chapter-per-module structure, gotchas, reading order, and glossary. Use when: user wants to understand a repo, onboard to a codebase, create a repo guide, build a knowledge base, says "encyclopedia", "repo guide", "document this repo", "explain this codebase", "onboarding guide".'
argument-hint: 'Optional: path to output file or specific area to focus on'
---

# Repository Encyclopedia

Build a single comprehensive onboarding encyclopedia for a repository — a structured reference that a mid-to-senior developer joining the project can read to understand what the repo does, how it's organized, why decisions were made, and what will bite them.

**Target audience:** Experienced developers (mid to senior, or AI) who know general technology (the programming language, CI/CD, build systems) but have zero domain knowledge and zero project history. Don't explain what a pointer is. Do explain what "streaming mode" means in this repo's context and why the retry logic uses exponential backoff with jitter.

## Output Structure

The encyclopedia is a **single markdown file** with this skeleton:

```markdown
# {Repo Name} — Encyclopedia

> Generated: {YYYY-MM-DD} | Commit: {short hash}

## TLDR

{3-5 sentence overview: what the repo builds, who it's for, key technologies, deployment target}

## Suggested Reading Order

{Agent-crafted path through chapters, optimized for building mental model fastest}

1. Start with Chapter N — {why: gives foundational mental model}
2. Then Chapter M — {why: builds on N, covers core domain}
3. ...

## Table of Contents

{Auto-generated from chapters}

---

## How Things Connect

{Prose description of the module dependency graph, main execution flows,
and how data moves end-to-end through the system. ASCII diagrams welcome.}

---

## Chapter 1: {Module/Area Name}

### Purpose
{Why this exists — one paragraph. Explain the problem it solves, not just what it does.}

### Definitions
{Domain terms introduced or owned by this module}

| Term | Definition |
|------|-----------|
| ... | ... |

### Invariants
{Rules that must ALWAYS hold in this module}

- ...

### Key Components
{3-8 most important files/classes. Name, role, key interfaces.
Reference as: `ClassName` in `path/to/file.cpp:42` — no inlined code.}

### Data Flow
{How data enters, transforms, and exits. Numbered prose list or ASCII.}

### Gotchas & Tribal Knowledge
{Things that aren't obvious from reading the code:}
- "This config looks optional but the service crashes without it"
- "Don't refactor X because it's load-bearing for Y"
- "This test is flaky when Z"

### Why It's Built This Way
{Architectural decisions WITH evidence from code/comments/ADRs.
Only state rationale when provable. If no evidence exists, omit — don't speculate.}

### Subchapter: {Topic}
{Deeper explanation of non-obvious aspects. Only if the module warrants it.}

---

## Chapter N: ...

---

## External Dependencies

{Consolidated chapter for third-party/external modules.
Each gets a short subsection — not full depth.}

### {External Name}
- **Role:** What it provides to the system
- **Used by:** Which internal modules depend on it
- **Key interfaces:** Entry points consumed by this repo

---

## Cross-Cutting Concerns

### Build System
### Configuration
### Testing Strategy
### Deployment

---

## Glossary

{Consolidated definitions from all chapters, alphabetized}
```

## Procedure

### 0. Initial Questions

Before any exploration, ask the user:

1. **Fresh or incremental?** "Should I generate a fresh encyclopedia, or update an existing one?"
   - If incremental and a previous encyclopedia exists, read it to understand current state.
2. **Output path?** "Where should I save? Default: `.plans/ENCYCLOPEDIA.md`"

### 1. Broad Reconnaissance (single subagent)

Launch one Explore subagent to map the entire repo at surface level:

- Read top-level directory listing and all READMEs
- Read CONTRIBUTING, architectural docs, ADRs if they exist
- Identify module boundaries (CMakeLists.txt, package.json, directory structure)
- Map the dependency graph: which modules include/import/link which others
- Classify each area as **own code** (full depth) or **external dependency** (consolidated)

Goal: produce a **chapter outline** with dependency relationships.

### 2. Present Chapter Outline

Show the user:
- Proposed chapters (with one-line descriptions)
- Which areas are "own code" (full chapters) vs "external" (consolidated)
- Proposed output path

Ask: "Does this capture the right boundaries? Any areas to merge, split, or skip?"

This is the **only user checkpoint**. After confirmation, the skill runs autonomously to completion.

### 3. Deep Exploration (per-chapter subagents)

For each "own code" chapter, launch an Explore subagent with this prompt:

> "Explore {module path} thoroughly. Answer ALL of the following — do not stop until every section is covered:
> 1. PURPOSE: Why does this module exist? What problem does it solve?
> 2. DEFINITIONS: Domain terms introduced here (with precise definitions)
> 3. INVARIANTS: Rules that must always hold (look in assertions, validation, test setup, comments with must/always/never)
> 4. KEY COMPONENTS: 3-8 most important files/classes with roles and file paths
> 5. DATA FLOW: How data enters, transforms, exits (numbered steps)
> 6. GOTCHAS: Non-obvious things that would bite a newcomer
> 7. WHY: Architectural decisions with evidence (from comments, ADRs, commit messages, code structure)
> Report file paths as `path/to/file.cpp:line` for all references."

**Structural completeness rule:** If a subagent returns with any section blank or marked "unclear," explore further. The template IS the checklist — every section must be confidently filled before moving on.

### 4. Synthesize

With all subagent results collected, the main agent:

1. **Writes the TLDR** — distilling the whole picture into 3-5 sentences
2. **Writes "How Things Connect"** — using the dependency graph from step 1 and cross-references from step 3
3. **Assembles chapters** — from subagent findings, adding cross-references between chapters
4. **Writes "External Dependencies"** — consolidated from the broad reconnaissance
5. **Crafts "Suggested Reading Order"** — using agent judgment: what's most conceptually foundational, what newcomers will touch first, what has highest "aha" density
6. **Writes Cross-Cutting Concerns** — build, config, testing, deployment
7. **Compiles Glossary** — all definitions alphabetized, duplicates reconciled

### 5. Handle Inaccessible Areas

When a module can't be explored (submodule not checked out, binary dependency, no source):

- Still include it in the encyclopedia
- Write what's knowable from usage patterns (call sites, include paths, link targets)
- Mark clearly: "⚠️ Source not available — documented from usage patterns only"
- Place in "External Dependencies" section

### 6. Save and Report

Save to the confirmed output path. Include metadata header with generation date and commit hash. Report:
- Total chapters written
- Total definitions captured
- Areas marked as inaccessible/opaque
- Any sections that feel thin (honest about gaps)

## Quality Criteria

- A reader should understand the repo's purpose within 30 seconds (TLDR)
- Each chapter is self-contained — can be read independently
- Definitions are precise enough to resolve ambiguity in code reviews
- Invariants are specific enough to catch violations in PR review
- Gotchas prevent real onboarding pain — not obvious things
- "Why" sections cite evidence, never speculate
- No chapter exceeds ~300 lines — split if needed
- The glossary has no undefined terms referenced in chapters
- File references use `path/to/file.cpp:line` format (verifiable, not inlined)
- Reading order rationale is explicit ("start here because...")

## Exploration Tips

- `CMakeLists.txt` and `package.json` reveal module boundaries and dependencies
- Test files reveal invariants (what's being asserted?)
- Interface headers (`.h`, `.hpp`) reveal the public contract
- Config files and manifests reveal deployment assumptions
- CI pipelines reveal the build/test/deploy flow
- README files in subdirectories often explain "why this exists"
- Comments with "NOTE:", "HACK:", "TODO:", "IMPORTANT:" are gotcha signals
- `git log --oneline -20 -- path/` reveals recent activity and decision context

## Tone

Write for a peer — not a student, not a manager. Assume the reader is smart but uninformed about this specific domain. Be direct. Skip throat-clearing. If something is weird, say it's weird and explain why it exists anyway.
