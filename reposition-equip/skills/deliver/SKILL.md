---
name: deliver
description: "Generate all non-CLAUDE.md toolkit deliverables configured for the developer's stack"
---

# Deliver Skill — Equip Orchestrator

## Role

This skill executes the final delivery phase of `reposition-equip`. It receives the stack configuration and pre-delivery inventory from the equip orchestrator, reads each template file in this directory, and writes the appropriate output files into the project.

---

## Input

Received from the equip orchestrator:

- `stack_config` — object parsed from `.reposition/equip/stack.json`, including:
  - `stack` — the template name (e.g. `TypeScript/React`, `Python/FastAPI`, `Go`, `Java/Spring`)
  - `multi_service` — boolean; true when the project has multiple stacks
  - `stacks` — array of template names (used when `multi_service` is true)
- `inventory` — pre-delivery scan results, including flags for:
  - `has_git` — `.git/` directory exists
  - `has_remote` — git remote origin is configured
  - `existing_githooks` — `.githooks/pre-commit` or `.git/hooks/pre-commit` already exists
  - `existing_test_config` — per-stack test config file already present
  - `has_gitignore` — `.gitignore` exists

---

## Reference Docs — Steps 1–6

These four templates are stack-agnostic. Read each file from the skill directory and write it verbatim to `.reposition/equip/`.

### Step 1 — methodology.md

- Read: `skills/deliver/methodology.md`
- Write: `.reposition/equip/methodology.md`
- Action: copy verbatim; no modification needed

### Step 2 — skills-reference.md

- Read: `skills/deliver/skills-reference.md`
- Write: `.reposition/equip/skills-reference.md`
- Action: copy verbatim; no modification needed

### Step 3 — agentic-workflows.md

- Read: `skills/deliver/agentic-workflows.md`
- Write: `.reposition/equip/agentic-workflows.md`
- Action: copy verbatim; no modification needed

### Step 4 — worktree-strategy.md

- Read: `skills/deliver/worktree-strategy.md`
- Write: `.reposition/equip/worktree-strategy.md`
- Action: copy verbatim; no modification needed

### Step 5 — prompt-library.md

- Read: `skills/deliver/prompt-library.md`
- Locate the section heading that matches `stack_config.template` (e.g. `## TypeScript/React`)
- Extract: that section and all content below it until the next `##` heading (or end of file)
- **Multi-service:** if `stack_config.multi_service` is true, extract and combine all sections matching each entry in `stack_config.stacks`; deduplicate any identical prompts
- Write: `.reposition/equip/prompt-library.md`
  - Include a brief header: `# Prompt Library — {stack name(s)}`
  - Append the extracted section(s) below the header

### Step 6 — code-review-checklist.md

- Read: `skills/deliver/code-review-checklist.md`
- Extract: the `## Universal Checks (All Stacks)` section (always included)
- Locate and extract: the section heading matching `stack_config.template`
- **Multi-service:** if `stack_config.multi_service` is true, extract all sections matching each entry in `stack_config.stacks`; deduplicate any identical checklist items
- Write: `.reposition/equip/code-review-checklist.md`
  - Include a brief header: `# Code Review Checklist — {stack name(s)}`
  - Append universal checks first, then the per-stack section(s)

---

## Functional Configs — Steps 7–9

These steps are conditional. Execute each only when the inventory flags permit it.

### Step 7 — .mcp.json (MCP server config)

- Read: `skills/deliver/mcp-configs.md`
- Condition: `inventory.has_git` is true
  - If `inventory.has_remote` is also true: include the GitHub MCP server block
  - Follow any merge/create instructions documented in `mcp-configs.md`
  - If `.mcp.json` already exists: read the existing file, merge the new server entries under `mcpServers` without removing existing entries, then write the merged result back
  - If `.mcp.json` does not exist: create it with the appropriate server blocks
- Condition: `inventory.has_git` is false
  - Skip this step; record `mcp.json: skipped (no git)` in the manifest

### Step 8 — pre-commit hook (git-hooks.md)

- Read: `skills/deliver/git-hooks.md`
- Locate the hook template section matching `stack_config.template`
- **Multi-service:** combine hook commands from all matching stack sections, deduplicate, maintain a logical run order (lint → test)
- Condition checks (in order):
  1. If `.githooks/pre-commit` already exists → skip; record `git-hooks: skipped (hook exists at .githooks/pre-commit)` in manifest
  2. If `.git/hooks/pre-commit` already exists → skip; record `git-hooks: skipped (hook exists at .git/hooks/pre-commit)` in manifest
  3. If `inventory.has_git` is false → skip; record `git-hooks: skipped (no git repository)` in manifest
  4. Otherwise → create `.githooks/pre-commit` with the selected hook template content; set the file as executable (`chmod +x`)
- If the hook was skipped due to an existing hook: write the generated template to `.reposition/equip/suggested-pre-commit-hook` so the developer can review it manually

### Step 9 — test scaffolding (test-scaffolding.md)

- Read: `skills/deliver/test-scaffolding.md`
- Locate the section matching `stack_config.template`
- Follow the skip conditions documented in `test-scaffolding.md` for the matched stack
- **Existence check first:** if `inventory.existing_test_config` is true for the matched stack → skip config creation; record `test-scaffolding: skipped (existing config detected)` in manifest
- If no existing config:
  - Create the test config file defined in the stack section
  - Create the test directory if it does not already exist
  - Create the example test file defined in the stack section
  - Record `test-scaffolding: created` in manifest with the list of files created

---

## .gitignore Handling — Step 10

### If `.gitignore` exists (`inventory.has_gitignore` is true):

- Read the existing `.gitignore`
- Check whether `.reposition/` is already listed (any line that is exactly `.reposition/` or `.reposition`)
- If not listed: append `.reposition/` on a new line at the end of the file; record `gitignore: appended .reposition/` in manifest
- If already listed: record `gitignore: already contains .reposition/ — no change` in manifest

### If `.gitignore` does not exist:

- Create `.gitignore` with a single entry: `.reposition/`
- Record `gitignore: created with .reposition/` in manifest

---

## Output

When all steps are complete, return a structured manifest to the equip orchestrator. The manifest must include one entry per step, recording one of three states:

- `created` — file(s) were written or created by this skill
- `merged` — existing file was read and new content was merged into it
- `skipped` — step was skipped; include the reason

**Manifest format (return as a structured object or markdown table):**

| Deliverable                        | Status  | Notes                                  |
|------------------------------------|---------|----------------------------------------|
| methodology.md                     | created |                                        |
| skills-reference.md                | created |                                        |
| agentic-workflows.md               | created |                                        |
| worktree-strategy.md               | created |                                        |
| prompt-library.md                  | created | Stack: {template name}                 |
| code-review-checklist.md           | created | Universal + {template name}            |
| .mcp.json                          | created / merged / skipped | Reason if skipped      |
| .githooks/pre-commit               | created / skipped | Reason if skipped             |
| suggested-pre-commit-hook          | created / skipped | Only if hook step was skipped |
| test config + directory + example  | created / skipped | Reason if skipped             |
| .gitignore                         | created / appended / no change |                       |

Pass this manifest back to the equip orchestrator for final reporting to the user.
