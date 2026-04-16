---
name: inventory
description: "Scan the project for REPOSITION Equip deliverables, report status, and offer to regenerate missing pieces"
---

You are the `reposition-equip:inventory` skill. When invoked via `/reposition-inventory`, execute the following 4 steps in order.

---

## Step 1 — Load Stack Config

Read `.reposition/equip/.stack-config.json` from the project root.

- If the file does not exist, output exactly:

  ```
  No equip data found. Run /reposition-equip first.
  ```

  Then stop. Do not proceed to Step 2.

- If the file exists, parse it and store the following fields for use in subsequent steps:
  - `language`
  - `framework`
  - `equipped_date`
  - `template`
  - `test_runner`

---

## Step 2 — Check Each Deliverable

Check for the presence of every deliverable listed below. Do not write, modify, or delete any files — this step is read-only.

For each item, determine whether it is **Present** or **Not installed** / **Missing**.

| Deliverable | How to check |
|---|---|
| `CLAUDE.md` | Read `PROJECT_ROOT/CLAUDE.md`. Present if the file exists and is non-empty. |
| `.mcp.json` | Read `PROJECT_ROOT/.mcp.json`. Present if the file exists. Count the number of server entries in `mcpServers`. |
| `.githooks/pre-commit` | Check `PROJECT_ROOT/.githooks/pre-commit`. Also check `PROJECT_ROOT/.git/hooks/pre-commit`. Present if either exists. |
| Test config | Use `stack_config.test_runner` to determine the expected file. Check for: `jest.config.*` (jest/vitest), `vitest.config.*` (vitest), `pytest.ini` or `pyproject.toml` with `[tool.pytest]` section (pytest), `pom.xml` with surefire plugin (junit). |
| `methodology.md` | Read `.reposition/equip/methodology.md`. Present if the file exists. |
| `prompt-library.md` | Read `.reposition/equip/prompt-library.md`. Present if the file exists. |
| `code-review-checklist.md` | Read `.reposition/equip/code-review-checklist.md`. Present if the file exists. |
| `skills-reference.md` | Read `.reposition/equip/skills-reference.md`. Present if the file exists. |
| `agentic-workflows.md` | Read `.reposition/equip/agentic-workflows.md`. Present if the file exists. |
| `worktree-strategy.md` | Read `.reposition/equip/worktree-strategy.md`. Present if the file exists. |

Store the result of each check internally before proceeding to Step 3.

---

## Step 3 — Display Inventory

Print the inventory using this exact format. Replace bracketed placeholders with actual values from the stack config and check results.

```
REPOSITION Equip Inventory:

  CLAUDE.md                    ✓ Present ([language]/[framework], equipped [equipped_date])
  .mcp.json                    ✓ Present ([N] server configured)
  .githooks/pre-commit         ✗ Not installed (suggested hook available)
  Test config                  ✓ [config filename] present
  methodology.md               ✓ Present
  prompt-library.md            ✓ Present
  code-review-checklist.md     ✓ Present
  skills-reference.md          ✓ Present
  agentic-workflows.md         ✓ Present
  worktree-strategy.md         ✓ Present
```

Rules for display:

- Use `✓ Present` for items that exist. Use `✗ Missing` or `✗ Not installed` for items that do not.
- For `CLAUDE.md`: include `([language]/[framework], equipped [equipped_date])` using values from the stack config.
- For `.mcp.json`: include the server count as `([N] server configured)` or `([N] servers configured)`. If missing, show `✗ Missing`.
- For `.githooks/pre-commit`: if both `.githooks/pre-commit` and `.git/hooks/pre-commit` are absent, show `✗ Not installed (suggested hook available)`. If either is present, show `✓ Present`.
- For Test config: if present, show the actual filename (e.g., `✓ pytest.ini present`, `✓ jest.config.ts present`). If missing, show `✗ Missing ([expected filename])`.
- For all `.reposition/equip/` docs: show `✓ Present` or `✗ Missing` with no extra annotation.
- Preserve the column alignment shown in the format above. Pad with spaces so the status column lines up.

---

## Step 4 — Offer Regeneration

After displaying the inventory, evaluate the results:

**If any items are missing:**

Ask the user:

```
Regenerate missing items? [y/N]
```

- If the user answers **y** or **yes**:
  - Invoke the `reposition-equip:deliver` skill.
  - Pass the full stack config loaded in Step 1.
  - Pass a filtered list of only the missing deliverables so the deliver skill targets only those items.
  - Do not re-generate items that are already present.

- If the user answers **n**, **N**, presses Enter, or provides any other response:
  - Output nothing further. Stop.

**If all items are present:**

Output exactly:

```
Everything looks good. Your toolkit is complete.
```

Then stop.
