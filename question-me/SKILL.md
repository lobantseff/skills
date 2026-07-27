---
name: question-me
description: 'Interview the user with batched clarifying questions until there is shared understanding of a task, then summarize and WAIT for go-ahead before doing any work. Use when the user says "question me", "/question-me", "interview me first", "ask me questions before you start", "clarify before coding", or wants alignment on scope, requirements, and acceptance criteria before implementation begins.'
argument-hint: 'Optional: the task you want to be questioned about'
---

# Question Me

Interview me to clarify an upcoming task **before** doing any work. Do not start
implementing until we have shared understanding and I give the go-ahead.

## When to Use

- I invoke `/question-me`, optionally with a task description.
- I ask you to "interview me", "clarify first", or "make sure we're aligned" before starting.
- A request is ambiguous and you'd otherwise be guessing at scope, constraints, or acceptance criteria.

## Procedure

1. **Explore first.** For anything answerable from the codebase, open files, or context,
   investigate instead of asking. Never ask me what you can find yourself.

2. **Ask in batches.** Use the questions UI (multiple-choice where possible) to ask a focused
   round of ~3–6 questions at a time. Cover the dimensions that matter:
   - Goal & concrete outcome (what "done" looks like)
   - Scope & non-goals (what's explicitly out)
   - Constraints (performance, dependencies, style, compatibility)
   - Edge cases & error handling
   - Acceptance criteria / how to verify
   - Preferences (patterns, libraries, file locations)

3. **Always recommend.** For every question, propose your recommended answer/default and mark it
   as recommended, so I can accept quickly.

4. **Iterate to robust confidence.** Aim for *medium* depth: keep asking follow-up rounds until you
   have clear, robust confidence about the task — not a shallow single pass, but not an exhausting
   interrogation either. Only ask about genuine, material ambiguities the answers surfaced; don't
   manufacture questions. I can end the interview at any time by saying "go" or "just proceed".

5. **Summarize, then wait.** When ambiguity is resolved, write a short summary of the shared
   understanding: goal, scope, non-goals, key decisions, and acceptance criteria. Then STOP and
   wait for my explicit go-ahead before implementing.

## Guardrails

- Batch questions — don't drip them one at a time.
- Never request secrets (passwords, tokens, API keys) through questions.
- Prefer fewer, high-signal questions over exhaustive ones.
- Do not begin implementation until I confirm.
