---
name: explore
description: "Open exploration mode — Q&A, module drill-downs, flow tracing, and pattern comparison"
---

# REPOSITION Onboarding — Open Explore

**Role:** You are running the open explore mode of the REPOSITION Onboarding Accelerator. After a brief orientation, the developer drives — they ask questions, you answer from scan data and by reading files on demand.

**Input:** Expects CODEBASE_MODEL from the scan skill (loaded from `.reposition/scan-*.json` or passed in session). Accepts an optional `mode` parameter: `full` (default, includes kickoff) or `qa_only` (used by `/reposition-ask`, skips kickoff and guard rails).

---

## Kickoff (skip if `qa_only`)

Invoke `reposition-onboard:orient` in condensed mode.

Orient presents:
- System type and tech stack — one sentence
- Key directories — bullet list, one line per directory
- How to run it locally — numbered steps

After orient returns, display:

> "You're oriented. Ask me anything about this codebase — architecture, patterns, specific files, how things connect, why something is done a certain way. When you're ready to make your first contribution, say 'ready to contribute'."

---

## Topic Tracking (skip if `qa_only`)

Maintain a session-memory list of explored topics. Do not write this to disk.

Topics to track:

| Topic | Marked explored when |
|---|---|
| `architecture` | Kickoff completes (auto), or dev asks about structure/how things work |
| `environment` | Kickoff completes (auto), or dev asks how to run/install |
| `patterns` | Dev asks how something is implemented across the codebase |
| `testing` | Dev asks about tests, test coverage, or test patterns |
| `git_workflow` | Dev asks about commits, branches, history, or who works on what |
| `deep_dive` | Dev asks for a walkthrough of a specific module or area |

At kickoff completion, pre-mark `architecture` and `environment` as explored (orient covered them).

---

## Q&A Loop

For each dev question, determine the type and respond as follows.

### Architecture questions
Trigger: "how does X work?", "what's the structure of Y?", "where does X live?"

- Read relevant files referenced in CODEBASE_MODEL (`entry_points`, `abstractions`, `structure`)
- Trace connections between components
- Explain with code references in `file:line` format
- Mark `architecture` explored

### Module drill-down
Trigger: "walk me through the X module", "explain the Y service", "tell me about Z"

- Identify the area from the keyword in the question
- Glob for matching directories and files
- Read key files in the area
- Trace execution paths through the module
- Run `git log --oneline -10 -- [path]` to show recent history for the area
- Explain the domain model and point out any gotchas or non-obvious patterns
- Mark `deep_dive` explored

### Flow tracing
Trigger: "how does X get processed?", "what happens when Y?", "trace the flow for Z"

- Find the entry point from `entry_points` in CODEBASE_MODEL, or grep for the trigger keyword if not listed
- Follow imports and function calls step by step
- Present the complete chain from entry to exit in a single annotated trace:
  ```
  POST /webhook → WebhookController:receive → EventRouter:dispatch → PaymentHandler:process → DB
  ```
- Use actual class/method names from the code
- Mark `architecture` explored

### Pattern comparison
Trigger: "how is X handled?", "what's the pattern for Y?", "how does this codebase do Z?"

- Grep for the pattern keyword across the codebase
- Find 2–3 representative examples showing the approach
- Summarize the pattern in plain English, citing the files as evidence
- Mark `patterns` explored

### History questions
Trigger: "what changed recently?", "who works on X?", "what's the commit history for Y?"

Use git log with appropriate filters:
- Recent changes: `git log --oneline -20`
- By person: `git log --oneline --author="name"`
- For a specific area: `git log --oneline -20 -- [path]`

Present the output in context — don't just dump raw log lines. Mark `git_workflow` explored.

### Testing questions
Trigger: "how do tests work?", "what's the test coverage?", "how do I write a test for X?"

- Reference `test_patterns` from CODEBASE_MODEL
- Read 1–2 sample test files to show the pattern in practice
- Explain: framework used, file naming convention, where tests live, what gets mocked vs integrated
- Mark `testing` explored

### Environment questions
Trigger: "how do I run this?", "what do I need installed?", "how do I set up my local environment?"

- Reference `environment` from CODEBASE_MODEL
- Read Dockerfile, docker-compose.yml, and/or README setup sections as available
- Present setup steps in order
- Mark `environment` explored

---

## Guard Rails (skip if `qa_only`)

When the dev signals readiness to contribute — "ready to contribute", "let's go", "I'm ready", "time to code", or similar — do the following before transitioning:

1. Check the topic tracking list for `testing` and `git_workflow`
2. If one or both are unexplored, present a soft prompt:

   - Both unexplored:
     > "Before you jump in — you haven't looked at how testing works here or the commit conventions. Want a quick 2-minute summary, or are you good?"

   - Only `testing` unexplored:
     > "Before you jump in — you haven't looked at how testing works here. Want a quick summary, or are you good?"

   - Only `git_workflow` unexplored:
     > "Before you jump in — you haven't looked at the commit conventions. Want a quick summary, or are you good?"

3. This is not a blocker:
   - If they say "I'm good", "skip", "no thanks", or similar — proceed immediately
   - If they want the summary: present testing conventions and/or git workflow from CODEBASE_MODEL data, then proceed

4. If both `testing` and `git_workflow` are already explored — skip the prompt entirely and proceed directly to transition.

---

## Transition to Contribute

Invoke `reposition-onboard:contribute`.

Do not add anything after invoking contribute — it handles the rest of the flow.
