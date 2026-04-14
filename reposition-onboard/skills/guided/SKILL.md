---
name: guided
description: "Guided tour of the codebase — architecture, patterns, deep dive, then first contribution"
---

# REPOSITION Onboarding — Guided Tour

You are running the guided tour mode of the REPOSITION Onboarding Accelerator. Walk the developer through the codebase step by step across 4 phases. Follow each phase in order.

**Input:** Expects CODEBASE_MODEL from the scan skill (loaded into session context by the scan skill).

---

## Phase 1 — Architecture Overview (5–10 min)

Invoke the `reposition-onboard:orient` skill in **full orientation mode** for Phase 1 content.

The orient skill will present:
- System type
- Directory map
- Data flow trace
- Key dependencies
- How to run it

After the orient skill presents Phase 1, ask:

> "Any area you want me to go deeper on before we move on?"

Wait for the developer's response.

- If they ask about a specific area: read the relevant files for that area and explain what you find. Then ask again: "Anything else before we move on to patterns?"
- When they're ready, proceed to Phase 2.

---

## Phase 2 — Patterns & Conventions (5–10 min)

Continue with the `reposition-onboard:orient` skill for **Phase 2 content**.

The orient skill will present:
- Code conventions
- Architecture patterns
- Testing conventions
- Git workflow
- Unwritten rules

After the orient skill presents Phase 2, ask:

> "Questions about any of these patterns?"

Wait for the developer's response. Answer any follow-up questions. When they're done, proceed to Phase 3.

---

## Phase 3 — Deep Dive (10–30 min, adaptive)

Ask the developer:

> "What area will you be working in? (e.g., 'payments', 'user auth', 'the CLI', or 'not sure yet')"

Wait for the answer, then branch:

### If they name a specific area

1. **Map the keyword to the codebase.** Search `structure.top_level_dirs` and `entry_points` in CODEBASE_MODEL for directories or files containing the keyword. Also glob for directories/files whose path contains the keyword.

2. **Read the key files.** Read the entry points and main service/controller files for that area.

3. **Trace execution paths.** Follow imports and function calls step by step through the area. Show the chain: entry point → service/handler → data layer → output.

4. **Explain the domain model.** Identify the key entities and data structures in this area. Describe how they relate to each other.

5. **Point out gotchas.** Flag any of the following as you read through the files:
   - Unusual patterns that differ from the rest of the codebase
   - Workarounds with comments explaining why
   - Complex or non-obvious logic
   - TODO/FIXME comments and what they signal

6. **Show recent changes.**
   ```
   Bash: git log --oneline -20 -- [area-path]
   ```
   Summarise who has been working here and what has changed recently.

### If they say "not sure yet"

Identify the most active area from git history:

```
Bash: git log --oneline -50 --diff-filter=M --name-only | sort | uniq -c | sort -rn | head -10
```

Pick the top directory from the output. Explain it as a representative example using the same deep-dive flow above (steps 1–6). Let the developer know this is the most actively changed area in the repo.

---

After the deep dive, ask:

> "Ready to make your first contribution, or want to explore another area?"

- If they want to explore another area: repeat Phase 3 for the new area.
- If they're ready to contribute: proceed to Phase 4.

---

## Phase 4 — Contribute

Invoke the `reposition-onboard:contribute` skill.

The contribute skill handles: task selection, guided implementation, PR readiness check, and report generation.

Do not add anything after invoking contribute — it handles the rest of the flow including report generation.
