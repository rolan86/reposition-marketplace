# Agentic Workflows

This document explains how to use Claude Code's agent capabilities effectively. Agents are not magic — they are a tool with specific tradeoffs. Use them when the tradeoffs work in your favor.

---

## What Are Agents?

An agent is a fresh Claude instance dispatched to complete a specific, bounded task. Fresh means a clean context: no accumulated state from prior conversation turns, no drift from earlier instructions, no "forgetting" what was established at the start of the session.

Each agent:
- Receives an explicit task description and any relevant context
- Operates independently within its scope
- Returns a result that you (or a review agent) evaluate before proceeding

Agents are most valuable for two things: isolating tasks that benefit from clean context, and running independent tasks in parallel.

---

## Subagent-Driven Development

Subagent-driven development executes a written plan one task at a time, each in a fresh agent. The pattern:

1. **Write a plan** (via `/writing-plans`) — a list of discrete, independently implementable tasks
2. **Dispatch task 1** — a subagent receives the task description, relevant file context, and constraints
3. **Two-stage review:**
   - Stage 1 — Spec compliance: did the agent do what the task asked?
   - Stage 2 — Code quality: is the implementation clean, tested, and consistent with project conventions?
4. **Accept or reject** — if the output passes both stages, commit and move to task 2. If not, correct and re-dispatch.
5. **Repeat** for each task in the plan

Run this via `/subagent-driven-development`.

**Why two stages?** Spec compliance and code quality are different lenses. An implementation can satisfy the spec and still be unreadable or inconsistent with your conventions. Checking both separately catches more issues than a single combined pass.

---

## Parallel Agents

Parallel agents run two or more independent tasks simultaneously. When tasks do not share files and do not depend on each other's output, there is no reason to run them sequentially.

**When to use parallel agents:**
- 2+ tasks that write to different files
- Tasks that can be fully specified without knowing the outcome of the other
- Any situation where total wall-clock time matters and tasks are independent

Run this via `/dispatching-parallel-agents`.

**Example:** refactoring the auth module and adding a new API endpoint to a different service. These are independent — run them in parallel, review both outputs, commit both.

**What parallel agents do not help with:**
- Tasks that write to the same file — one agent will overwrite the other
- Tasks where task B depends on task A's output — you must run them sequentially
- Tasks that require shared state or shared memory

If in doubt, run sequentially. Parallel agent conflicts are harder to debug than sequential slowness.

---

## Agent Review Loops

Every agent output should go through a review loop before being accepted. The minimal loop:

```
implementer agent
    → spec compliance reviewer
    → code quality reviewer
    → decision: accept / reject / correct and re-dispatch
```

For high-stakes changes, add a security reviewer between code quality and acceptance.

**Spec compliance reviewer** asks:
- Did the agent complete the task as described?
- Are all acceptance criteria met?
- Are there missing cases not addressed?

**Code quality reviewer** asks:
- Is the code consistent with CLAUDE.md conventions?
- Are tests present and meaningful?
- Is there dead code, unnecessary complexity, or missing error handling?

The reviewer agents do not need to be elaborate. A focused prompt — "Review this implementation against the following task description and flag any gaps" — is enough. The value is in the systematic check, not the sophistication of the reviewer.

---

## Model Selection

Different tasks warrant different models. Using a capable model for mechanical tasks is waste; using a fast model for architecture decisions is risk.

| Task Type | Model Choice | Rationale |
|-----------|-------------|-----------|
| Boilerplate, scaffolding, repetitive patterns | Fast model (Haiku) | Mechanical work; speed matters; errors are obvious |
| Integration, feature implementation | Standard model (Sonnet) | Moderate complexity; balanced cost and capability |
| Architecture, security review, complex logic | Most capable model (Opus) | High stakes; reasoning depth required |
| Review agents | Standard model | Reviewing is cheaper than generating |

When dispatching parallel agents, you can use different models per agent based on task complexity.

---

## When NOT to Use Agents

Agents have overhead: context construction, dispatch, review. For small tasks, this overhead exceeds the benefit.

**Do not use agents for:**
- A single small change to one file — just implement it directly
- Tightly coupled work where each step depends on the previous — agents need independent tasks
- Shared state across tasks — agents cannot coordinate on mutable shared state without explicit handoffs
- Exploratory work where requirements are still unclear — agents need a clear spec; exploration is better done interactively

The signal that you should not be using agents: you are spending more time writing task descriptions and reviewing agent outputs than you would spend writing the code yourself.

---

## Example Workflow

Assume you have a plan with three tasks:
1. Add a `UserPreferences` model and repository
2. Add a `GET /users/{id}/preferences` endpoint
3. Write integration tests for the endpoint

Tasks 1 and 2 are sequential — the endpoint depends on the model existing. Task 3 can be dispatched in parallel with the final review of task 2, or sequentially after.

```
Dispatch: Task 1 (UserPreferences model + repository)
    → Spec review: model has correct fields, repository has correct interface
    → Quality review: follows naming conventions, has unit tests
    → Accept → commit

Dispatch: Task 2 (GET /preferences endpoint)
    → Spec review: correct route, correct response shape, correct error handling
    → Quality review: consistent with other endpoints, input validated
    → Accept → commit

Dispatch: Task 3 (integration tests)
    → Spec review: tests cover happy path, 404, auth failure
    → Quality review: tests are readable, no implementation leakage
    → Accept → commit
```

Total: three agents, three review cycles, three commits. Clean history, reviewable diffs, clear accountability per task.
