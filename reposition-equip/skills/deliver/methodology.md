# AI-Assisted Development Methodology

This document describes how to work effectively with Claude Code in your day-to-day development. Read it once, refer back when something feels off.

---

## Core Principle: You Direct, AI Executes

Your role shifts when working with AI. You are no longer the primary writer of code — you are the architect, reviewer, and decision-maker. The AI generates; you evaluate and approve.

This is not a reduction in responsibility. It is a change in where your expertise is applied. Bad direction produces bad output. Good direction — clear problem statements, tight constraints, explicit acceptance criteria — produces output you can trust.

**What this means in practice:**
- Invest more time upfront in problem definition than you used to
- Treat every AI output as a draft, not a final product
- Your judgment is the quality gate — AI has no stake in whether the code is correct

---

## Planning First

Never start coding without a plan. This applies whether you are writing the code or asking AI to write it. A vague prompt produces vague code.

**Before writing a line of implementation:**

1. Run `/brainstorming` — explore multiple approaches before committing. AI can enumerate tradeoffs you haven't considered. Get 3–5 candidates, evaluate them, pick one.
2. Run `/writing-plans` — produce a written implementation plan. Specify modules, interfaces, test cases, and edge cases before touching the codebase. Get the plan approved (by yourself or a colleague) before execution.

A plan that takes 20 minutes to write can save 4 hours of rework. With AI, rework is faster — but it is still waste.

---

## Test-Driven Development

Tests are the contract. AI must satisfy the contract. This is how you keep AI output honest.

**The workflow:**

1. Write tests first — or ask AI to write tests first, then review them before proceeding
2. Tests should specify exact inputs, expected outputs, and edge cases
3. Run `/test-driven-development` to structure this process
4. Only after tests exist and are reviewed: ask AI to implement
5. AI implementation must make all tests pass — no skipping, no mocking the assertions

Tests serve a second purpose: they are documentation that does not lie. When you return to a module in six months, the tests tell you what it is supposed to do.

**When AI writes tests:** review them before accepting. AI will sometimes write tests that trivially pass rather than tests that verify real behavior. Look for assertions that could never fail.

---

## Small Commits, Frequent Verification

Do not let AI write 500 lines and then review the whole thing. You will miss things. The review surface is too large.

**The discipline:**
- Commit after each logical change — one concept per commit
- Run tests after every change, not at the end
- If a commit makes something break, you want to know immediately, not after 10 more AI turns
- Keep diffs reviewable: if a PR diff is too large to read carefully, it was too large to generate in one session

This also gives you natural rollback points. If AI takes the implementation in the wrong direction, `git reset` to the last good commit and redirect.

---

## When to Trust AI Output

Not all code carries the same risk. Calibrate your review effort accordingly.

**High trust — review for correctness but spot-check is enough:**
- Boilerplate: file structure, imports, class scaffolding
- CRUD operations: standard create/read/update/delete against a known schema
- Repetitive patterns: serializers, mappers, similar transformations across types
- Test generation: unit tests for pure functions
- Documentation: docstrings, README sections

**Always review manually — no exceptions:**
- Business logic: rules that encode how your domain works
- Security-sensitive code: authentication, authorization, input validation, encryption
- Data migrations: schema changes, data transformations on production data
- Concurrency: locks, queues, async coordination
- Financial calculations: rounding, precision, aggregation

The rule is simple: if the code being wrong would be invisible until it caused harm, review it manually.

---

## When to Intervene

Stop and correct when:

- AI output contradicts your CLAUDE.md conventions — naming, structure, patterns
- AI introduces a dependency you did not approve
- AI rewrites code you did not ask it to touch
- AI ignores an edge case you explicitly mentioned
- Output is technically correct but does not match the spirit of the requirement

Intervention is not failure. It is the process working as designed. The earlier you intervene, the cheaper the correction.

When you correct AI, be explicit about what was wrong and what the correct approach is. Vague corrections ("this isn't quite right") produce vague improvements. Specific corrections ("function names should be camelCase per our conventions, not snake_case") produce specific fixes.

---

## CLAUDE.md Is the Source of Truth

Your CLAUDE.md defines how code is written in this project. It is the contract between you, your team, and the AI.

When AI output and CLAUDE.md disagree, CLAUDE.md wins — always. Do not accept AI output that violates your conventions just because it works. Technical correctness is not the same as project correctness.

**If CLAUDE.md is missing a rule that keeps coming up:** add it. CLAUDE.md is a living document. When you catch AI making the same mistake repeatedly, that is a signal that the rule needs to be explicit, not just implied.

**If CLAUDE.md is wrong:** update it deliberately, with intent. Do not let AI drift your conventions through accumulated exceptions.
