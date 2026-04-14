---
name: scan
description: "Scan a codebase to build the onboarding knowledge model"
---

# REPOSITION Onboarding — Codebase Scan

You are running the REPOSITION Onboarding Accelerator. Follow each stage in order.

---

## Stage 1: Check for Existing Scan

Glob for `.reposition/scan-*.json` in the current working directory.

- If a scan file is found: present the following prompt to the user (always prompt regardless of the file's age):

  > "I have a scan from [date extracted from filename]. Use it, or re-scan?
  >
  > A) **Use existing scan** — load it and continue to mode selection
  > B) **Re-scan** — run a fresh scan of the codebase"

  - If the user chooses A: read the JSON file into CODEBASE_MODEL, then skip to Stage 3.
  - If the user chooses B: proceed to Stage 2.

- If no scan file is found: proceed to Stage 2.

---

## Stage 2: Run Scan

Follow the instructions in `scan-engine.md` to execute all signal detections in sequence.

Show progress to the user line by line as each signal completes. Do not wait until all signals are done — output each line as its signal finishes:

```
Detecting tech stack...          ✓ [result summary]
Mapping project structure...     ✓ [result summary]
Reading CLAUDE.md...             ✓ Found ([line_count] lines) / ✗ Not found
Analyzing git history...         ✓ [commit_count] commits, [contributor_count] contributors
Scanning documentation...        ✓ [what was found]
Detecting test patterns...       ✓ [framework], [coverage]%
Checking CI/CD pipeline...       ✓ [platform] / ✗ None found
Identifying entry points...      ✓ [count] found
Extracting key abstractions...   ✓ [count] interfaces, [count] base classes
Reading environment setup...     ✓ [what was found]

Scan complete. Ready to explore.
```

Fill in the bracketed values from the actual scan results. For the CLAUDE.md line, use ✓ if found or ✗ if not found. For CI/CD, use ✓ with the platform name if found or ✗ if none found.

Handle failure cases per `scan-engine.md`. The scan should never abort entirely unless the directory is empty.

### Persist the scan

After the scan completes:

1. Create the `.reposition/` directory if it does not exist:
   ```
   Bash: mkdir -p .reposition
   ```

2. Save the CODEBASE_MODEL to `.reposition/scan-YYYY-MM-DD.json` (use today's date).

3. Check whether `.reposition/` is already listed in `.gitignore`:
   - Read `.gitignore` if it exists.
   - If `.reposition/` is not present: ask the user:

     > "Add `.reposition/` to .gitignore? This keeps scan data and onboarding docs out of version control. [Y/n]"

   - If the user confirms (Y or blank): append `.reposition/` as a new line to `.gitignore`.
   - If the user declines (n): skip, proceed without modifying `.gitignore`.

---

## Stage 2b: Ecosystem Integration

After the scan is saved, check for an existing assessment from the reposition-assess plugin.

Glob for `.reposition/assessment-*.md` in the current working directory.

**If found:**
- Read the file and attempt to extract domain scores. Look for a table or section containing Competence and AI Integration scores for each of the 4 domains (PM, Architecture, Design, Development).
- If scores are successfully parsed: store in CODEBASE_MODEL under an `assessment` key:
  ```json
  "assessment": {
    "found": true,
    "scores": {
      "development": { "competence": 7, "ai_integration": 4 },
      "architecture": { "competence": 8, "ai_integration": 3 },
      "design": { "competence": 6, "ai_integration": 5 },
      "pm": { "competence": 5, "ai_integration": 2 }
    }
  }
  ```
- If the file exists but scores cannot be parsed: log a warning inline:
  > "Assessment report found but couldn't parse scores — proceeding without them"

  Store:
  ```json
  "assessment": { "found": true, "scores": null }
  ```

**If not found:**
```json
"assessment": { "found": false }
```

Update the saved JSON file with the `assessment` key before proceeding.

---

## Stage 3: Mode Selection

Present the following choice to the user:

> "How would you like to learn this codebase?
>
> A) **Guided tour** — I'll walk you through the architecture, patterns, and conventions step by step, then help you make your first contribution
>
> B) **Open explore** — I'll give you the lay of the land, then you drive. Ask me anything about the codebase, drill into areas that matter to you, and tell me when you're ready to contribute"

- If the user chooses A: invoke the `reposition-onboard:guided` skill.
- If the user chooses B: invoke the `reposition-onboard:explore` skill.

Pass CODEBASE_MODEL to the invoked skill so it has access to all scan data without re-reading the file.

---

## Stage 4: Post-Session — Upsell CTA

This skill is the session orchestrator. After the guided or explore flow — including the contribute and report phases — returns here, show the upsell CTA once.

**Check which REPOSITION plugins are installed:**

- Check for `reposition-assess`:
  - Glob for `~/.claude/plugins/reposition-assess/package.json`
  - Or glob for `.reposition/assessment-*.md` in the current working directory (indicates the plugin has been used)
  - Set `assess_installed = true` if either is found.

- Check for `reposition-transform`:
  - Glob for `~/.claude/plugins/reposition-transform/package.json`
  - Set `transform_installed = true` if found.

**Show CTAs based on what is missing:**

- If `assess_installed` is false:
  > "Want to see how your whole team's AI readiness stacks up? The free REPOSITION Assessment plugin (`reposition-assess`) scores your team across 4 domains."

- If `transform_installed` is false:
  > "To build on this with a personalized learning path and CLAUDE.md configuration, ask your team lead about the REPOSITION Transformation engagement."

- If both are already installed: skip — no upsell needed.

The report skill does NOT show these CTAs. Only this scan skill shows them, once, at the very end of the session.
