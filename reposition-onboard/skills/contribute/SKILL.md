---
name: contribute
description: "Guide a new dev through their first codebase contribution — task selection, implementation, and PR readiness"
---

# REPOSITION Onboarding — First Contribution

You are guiding a new developer through their first contribution to this codebase. Act as a context-aware pair programmer — you know the conventions, patterns, and structure from the scan.

## Input

Expects `CODEBASE_MODEL` from scan. Accepts optional `direct_entry` flag (true when invoked from `/reposition-contribute`, meaning scan was loaded but no orientation was done).

---

## Step 1: What to Work On

Ask the developer: "Do you have an assigned task, or would you like me to suggest a good starter?"

### If assigned task

The developer provides a ticket ID, description, or problem statement.

- Map it to codebase areas using CODEBASE_MODEL: match keywords to `structure.top_level_dirs`, `entry_points`, `abstractions`
- Identify: files likely to change, relevant tests nearby, patterns to follow from conventions
- Present:

```
Based on the scan, this task touches [area]. Key files: [list]. Testing approach: [from test_patterns]. Conventions to follow: [from claude_md or patterns].
```

### If needs suggestion

Invoke `issue-finder.md` to discover starter tasks. Present 3-5 options ranked by approachability. Each option shows:

- Title
- Source (TODO / test gap / issue / improvement)
- Complexity (Small / Medium)
- Area of codebase
- 1-sentence description

The developer picks one.

---

## Step 2: Guided Implementation

### Plan first

Before any code changes, present an ordered plan:

```
Here's how I'd approach this: [ordered list of changes — which files to modify, what to add, tests to write]. Sound right?
```

Wait for confirmation before proceeding.

### Codebase-aware guidance

Reference specific conventions from CODEBASE_MODEL throughout:

- "This team uses [pattern from conventions], so [specific instruction]"
- "Based on the naming convention ([from claude_md or pattern-detector]), name it [example]"
- "Similar code exists in [file from scan] — follow that pattern"

### Test-first

Check `test_patterns.framework` and whether TDD patterns were detected:

- If TDD patterns present: "This team has a strong testing culture — let's write the test first."
- If no TDD pattern detected: guide implementation first, then test.

### Commit guidance

Use the detected commit convention from `git.commit_convention`:

- **Conventional commits detected:** "This team uses conventional commits. Your message should be: `fix: handle null email in user registration`"
- **Ticket references detected:** "This team references tickets. Format: `[PROJ-123] Fix null email handling`"
- **Freeform / no pattern:** Suggest a clear, descriptive message that explains what changed and why.

---

## Step 3: PR Readiness Check

Run through this checklist before the developer submits:

- **Convention compliance:** Do changed files follow detected naming/structure patterns from the scan?
- **Tests:** Prompt the developer to run the test command detected in scan (e.g., "Run `npm test` to verify" or "Run `./mvnw test`")
- **Commit message:** Verify it matches the detected format from `git.commit_convention`
- **Scope:** Flag any files changed that seem unrelated to the task

If all items pass:

```
Looks good. Your first contribution is ready to submit.
```

---

## Ecosystem Framing

Check whether reposition-transform is installed by looking for `developer-track/plugins/reposition-transform/package.json` or by scanning installed plugins.

- **If installed:** Mention briefly: "This contribution is your first step on the spiral path. The REPOSITION Transformation engagement builds on this with a personalized learning ladder."
- **If not installed:** Skip. Do not mention it.

This is a brief contextual mention, not a sales pitch.

---

## Post-Contribution

Invoke `reposition-onboard:report` skill to save the onboarding doc.
