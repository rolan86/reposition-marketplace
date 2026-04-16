---
name: equip
description: "Deliver a complete AI-assisted development toolkit configured for the developer's stack"
---

You are the `reposition-equip` orchestrator. When invoked via `/reposition-equip`, execute the following 5 stages in order. Do not skip stages. Do not show internal state to the user unless instructed.

---

## Stage 1: Stack Detection

Follow all instructions in `stack-detector.md` (located in the same directory as this file) to detect the developer's stack.

- If the stack can be reliably inferred from the project files, detect it automatically and confirm with the user before proceeding.
- If the stack cannot be determined, ask the user directly.

Store the resulting stack config with these fields:
- `language` (e.g., "TypeScript", "Python", "Java")
- `framework` (e.g., "Next.js", "FastAPI", "Spring Boot")
- `app_type` (e.g., "web", "api", "cli", "library")
- `template` (the CLAUDE.md template to use, e.g., "nextjs", "fastapi", "spring")
- `test_runner` (e.g., "jest", "vitest", "pytest", "junit")
- `services` (list of detected services, e.g., ["postgres", "redis", "s3"])

This stack config will be passed to all subsequent stages.

---

## Stage 2: Customize Q&A

Ask the developer 5 questions to populate the conventions sections of CLAUDE.md. Present all 5 questions as a single numbered list so the developer can answer them in one response. Tell the user: "Answer any you know — skip or type 'not sure' for anything you want me to default."

**Question 1 — Naming conventions:**
Ask: "How does your team name things? Type 'detect' and I'll scan your code, or describe your conventions (e.g., camelCase functions, PascalCase classes, snake_case files)."

If the user types "detect":
- Sample 10–15 source files from the project (skip node_modules, __pycache__, .git, dist, build directories)
- Read the first 50 lines of each sampled file
- Categorize observed patterns for: function names, class names, variable names, constant names, file names
- Report the detected patterns to the user and ask them to confirm or correct

**Question 2 — Error handling strategy:**
Offer multiple choice:
- A) Custom exception classes
- B) Result/Either types
- C) Let errors propagate
- D) Not sure

**Question 3 — PR review process:**
Offer multiple choice:
- A) 1 approval required
- B) 2 approvals required
- C) No formal process
- D) CI must pass before merge

**Question 4 — What gets logged:**
Offer multiple choice:
- A) Errors only
- B) Errors + warnings
- C) Everything (verbose)
- D) No logging standards yet

**Question 5 — Anything else your team enforces:**
Free text, optional. Examples: commit message format, branch naming, code formatting tools, security policies, accessibility standards.

After collecting answers:
- Store all answers in a `conventions` object within the stack config
- For any answer of "not sure", skipped, or empty: record the field with value `"default"` and note it was not specified
- Do not ask follow-up questions; proceed to Stage 3

---

## Stage 3: Pre-delivery Inventory

Before writing any file, scan the project for existing configs. Do this silently — do not show the inventory to the user at this stage.

Check for each of the following and determine the action (create / merge / skip):

| File / Path | Action Logic |
|---|---|
| `CLAUDE.md` | If missing → create. If exists → merge (preserve existing content, add missing sections). |
| `.mcp.json` | If missing → create. If exists → merge (add new servers, preserve existing entries). |
| `.githooks/pre-commit` | If missing → create. If exists → skip (note reason: "already configured"). |
| `.git/hooks/pre-commit` | If missing → create symlink or file. If exists → skip. |
| `jest.config.*` | If missing and test_runner is jest → create. Otherwise → skip. |
| `vitest.config.*` | If missing and test_runner is vitest → create. Otherwise → skip. |
| `pytest.ini` | If missing and test_runner is pytest → create. Otherwise → skip. |
| `pyproject.toml` (with `[tool.pytest]`) | If pyproject.toml exists but lacks `[tool.pytest]` and test_runner is pytest → merge. If already has pytest config → skip. |
| `pom.xml` (with surefire plugin) | If pom.xml exists but lacks surefire → merge. If already configured → skip. |
| `.gitignore` | If missing → create. If exists → merge (add missing entries only). |
| `.reposition/equip/methodology.md` | If missing → create. If exists → skip. |
| `.reposition/equip/prompt-library.md` | If missing → create. If exists → skip. |
| `.reposition/equip/code-review-checklist.md` | If missing → create. If exists → skip. |
| `.reposition/equip/skills-reference.md` | If missing → create. If exists → skip. |
| `.reposition/equip/agentic-workflows.md` | If missing → create. If exists → skip. |
| `.reposition/equip/worktree-strategy.md` | If missing → create. If exists → skip. |

Store the inventory result (file path, action, reason) internally. Proceed to Stage 4.

---

## Stage 4: Deliver

Execute delivery in this order:

**Step 4a — CLAUDE.md**
Invoke the `reposition-equip:configure` skill, passing:
- The full stack config (language, framework, app_type, template, test_runner, services)
- The conventions object from Stage 2
- The inventory entry for CLAUDE.md (action: create or merge)

The `reposition-equip:configure` skill is responsible for generating or merging CLAUDE.md.

**Step 4b — All other deliverables**
Invoke the `reposition-equip:deliver` skill, passing:
- The full stack config
- The conventions object
- The full inventory (all entries except CLAUDE.md)

The `reposition-equip:deliver` skill is responsible for generating all remaining files: git hooks, test config, .gitignore, .mcp.json, and all `.reposition/equip/` documents.

**Step 4c — Save stack config**
Write the stack config to `.reposition/equip/.stack-config.json` with:
- All stack fields (language, framework, app_type, template, test_runner, services)
- An `equipped_date` field set to today's date in ISO 8601 format (YYYY-MM-DD)
- A `conventions` object containing all Q&A answers from Stage 2

Example structure:
```json
{
  "language": "TypeScript",
  "framework": "Next.js",
  "app_type": "web",
  "template": "nextjs",
  "test_runner": "jest",
  "services": ["postgres", "redis"],
  "equipped_date": "2026-04-15",
  "conventions": {
    "naming": "camelCase functions, PascalCase classes, kebab-case files",
    "error_handling": "Custom exception classes",
    "pr_review": "2 approvals required",
    "logging": "Errors + warnings",
    "other": "default"
  }
}
```

---

## Stage 5: Summary

Print the delivery manifest showing the outcome for every deliverable. Use this exact format, substituting values in brackets with actuals:

```
Toolkit delivered:

  Created:
    ✓ CLAUDE.md ([stack], [N] convention sections filled)
    ✓ .githooks/pre-commit ([lint tool] + [test runner] + .env check)
    ✓ [test config file] (e.g., jest.config.ts)
    ✓ [test directory] (e.g., __tests__/)
    ✓ .reposition/equip/methodology.md
    ✓ .reposition/equip/prompt-library.md
    ✓ .reposition/equip/code-review-checklist.md
    ✓ .reposition/equip/skills-reference.md
    ✓ .reposition/equip/agentic-workflows.md
    ✓ .reposition/equip/worktree-strategy.md

  Merged:
    ✓ .mcp.json (added [server name], kept existing [N] servers)

  Skipped (already configured):
    ✗ [file path] ([reason])
```

Rules for the manifest:
- Only list files under "Created" if their action was create and they were successfully written
- Only list files under "Merged" if their action was merge and content was added
- Only list files under "Skipped" if the file already existed and was left unchanged
- If a section (Created / Merged / Skipped) has no entries, omit that section header entirely
- Replace all bracketed placeholders with real values derived from the stack config and inventory

After printing the manifest, end with this exact line:

```
Your toolkit is ready. Run /reposition-inventory anytime to check status.
```
