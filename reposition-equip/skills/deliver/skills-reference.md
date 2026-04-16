# Claude Code Skills Reference

Quick reference for the skills available in this project. Use this when you know you need to do something but are not sure which skill applies.

---

## Skills at a Glance

| Task | Skill Sequence | When to Use | What It Does |
|------|---------------|-------------|--------------|
| New feature from scratch | `/brainstorming` → `/writing-plans` → `/test-driven-development` | Always for non-trivial work | Explores approaches, produces a written plan, implements test-first |
| Bug diagnosis | `/systematic-debugging` | Before guessing at the fix | Diagnoses root cause systematically before touching code |
| Code review | `/requesting-code-review` | Before merging any PR | Reviews for bugs, style, security, completeness |
| Multiple independent tasks | `/dispatching-parallel-agents` | When tasks don't share files | Runs agents in parallel, reducing total wall-clock time |
| Feature branch work | `/using-git-worktrees` | Every feature | Creates an isolated workspace; main stays clean |
| Plan execution | `/subagent-driven-development` | When you have a written plan | Dispatches one subagent per task with two-stage review |
| Receiving review feedback | `/receiving-code-review` | When someone reviews your PR | Systematically addresses feedback without missing items |
| Finishing a branch | `/finishing-a-development-branch` | When all tests pass | Cleanup, final review, merge decision |

---

## Skill Details

### `/brainstorming`

Generates multiple solution approaches for a problem before committing to one. Forces exploration before execution.

**Use it when:** starting a feature, architectural decision, or any task where there is more than one reasonable approach.

**Do not use it when:** the solution is already defined and agreed — skip to `/writing-plans`. Using brainstorming on a decision that is already made wastes time and creates noise.

---

### `/writing-plans`

Takes a chosen approach and produces a structured implementation plan: modules, interfaces, test cases, sequence of steps.

**Use it when:** you have completed brainstorming and chosen an approach, and the work has more than 2–3 steps.

**Do not use it when:** the task is a one-liner fix or a clearly scoped change to an existing function. Over-planning small tasks is its own overhead.

---

### `/test-driven-development`

Implements a feature or fix by writing tests first, then implementation. Tests define the contract; implementation satisfies it.

**Use it when:** implementing any feature or fixing any bug — this should be the default for all substantive coding work.

**Do not use it when:** writing documentation, configuration files, or pure scaffolding with no logic. TDD is for code with behavior that can be verified.

---

### `/systematic-debugging`

Diagnoses a bug or unexpected behavior by working through a structured process: reproduce, isolate, hypothesize, verify.

**Use it when:** you encounter a bug, a test failure, or unexpected behavior — before making any changes. Debugging without diagnosis produces patches, not fixes.

**Do not use it when:** the bug is already diagnosed and the fix is obvious. In that case, fix it, test it, commit it.

---

### `/requesting-code-review`

Performs a structured review of your code before it is merged. Checks for correctness, edge cases, security, style, and completeness.

**Use it when:** any PR is ready to merge — no exceptions. Even if you are confident, a second pass catches things.

**Do not use it when:** the code is still in progress (WIP). Review is for code that is ready to ship, not for early drafts. Reviewing WIP burns context on code that will change.

---

### `/receiving-code-review`

Helps you systematically work through review feedback without missing items or accidentally introducing new issues.

**Use it when:** a reviewer has left comments on your PR. Works through feedback item by item, verifies each change, and confirms nothing else was affected.

**Do not use it when:** there is no feedback. If the review came back clean, proceed to `/finishing-a-development-branch`.

---

### `/dispatching-parallel-agents`

Dispatches two or more independent tasks to separate Claude instances running simultaneously. Reduces total time when tasks are independent.

**Use it when:** you have 2+ tasks that do not share files or state and can be worked on concurrently.

**Do not use it when:** tasks are tightly coupled (one depends on the output of another), or when tasks write to the same files. Parallel agents on shared state produce merge conflicts and overwrite each other's work.

---

### `/subagent-driven-development`

Executes a written implementation plan by dispatching one subagent per task. Each subagent gets fresh context. After each task, a two-stage review runs: spec compliance, then code quality.

**Use it when:** you have a written plan (from `/writing-plans`) and are ready to execute it. Ideal for multi-task features where each task is meaningful and independently verifiable.

**Do not use it when:** the plan has only one task — just implement it directly. The subagent overhead is not worth it for single-task work.

---

### `/using-git-worktrees`

Creates a dedicated git worktree for a feature, keeping main clean and enabling parallel work across features.

**Use it when:** starting any feature branch. This is the default way to start new work in this project.

**Do not use it when:** making a hotfix that must go directly to main with no branch isolation. Even then, consider whether a worktree is faster.

---

### `/finishing-a-development-branch`

Runs end-of-branch cleanup: final test run, lint, dead code check, commit message review, and merge decision.

**Use it when:** all tests pass, review feedback is addressed, and you are ready to merge.

**Do not use it when:** there are still failing tests or open review items. This skill assumes the branch is ready — use it only when it actually is.

---

## Skill Sequencing

Most non-trivial features follow this sequence:

```
/brainstorming
    → /writing-plans
    → /using-git-worktrees   (create the worktree)
    → /subagent-driven-development   (execute the plan)
    → /requesting-code-review
    → /receiving-code-review   (if feedback exists)
    → /finishing-a-development-branch
```

For bugs:

```
/systematic-debugging
    → /test-driven-development   (write a failing test for the bug, then fix it)
    → /requesting-code-review
    → /finishing-a-development-branch
```
