---
name: orient
description: "Present architecture overview, patterns, and conventions from scan data"
---

# REPOSITION Onboarding — Orientation

**Input:** Expects CODEBASE_MODEL from the scan skill (loaded from `.reposition/scan-*.json` or passed in session).

---

## Mode: Full Orientation

Used by guided mode. Present Phase 1 and Phase 2 in sequence from the CODEBASE_MODEL data.

---

### Phase 1 — Architecture Overview

**System type**
Classify the system from `structure` and `stack` data. Write one sentence: e.g., "Spring Boot monolith with 3 bounded contexts", "Next.js app with separate API layer", "Django REST API with Celery workers". Use the actual tech stack; do not guess.

**Directory map**
Read top-level dirs from `structure.top_level_dirs`. For each directory, write one curated line explaining its role. Do not dump a raw tree. Example format:
```
src/         — application source code
tests/       — test suite (collocated by module)
infra/       — Terraform and deployment configuration
docs/        — architecture decision records and API specs
```

**Data flow**
Trace one concrete request path through the system:
1. Read `entry_points` from CODEBASE_MODEL to find a representative endpoint (prefer an endpoint with business logic, not a health check).
2. Read the entry point file and follow the call chain through controller → service → repository → database.
3. Present it as a single chain:
   ```
   POST /orders → OrderController → OrderService → OrderRepository → PostgreSQL
   ```
   Use the actual class and method names from the codebase. If the stack doesn't use that layering (e.g., serverless functions, scripts), adapt to what's actually there.

**Key dependencies**
List the top 10 dependencies from `stack.dependencies`. For each, write one line describing its role in this specific codebase:
```
express        — HTTP server framework, all routes defined in src/routes/
prisma         — ORM, schema in prisma/schema.prisma
zod            — request validation in middleware layer
```

**How to run it**
- If `environment.run_instructions` is present in CODEBASE_MODEL: present those instructions verbatim.
- Otherwise: read README.md, Dockerfile, and docker-compose.yml (whichever exist) and extract the run steps. Present them as numbered steps.

---

After presenting Phase 1, ask:

> "Any area you want me to go deeper on before we move on?"

Wait for the user's response. If they want to go deeper on something, answer it. When they're ready, proceed to Phase 2.

---

### Phase 2 — Patterns & Conventions

**Code conventions**
- If `claude_md.found` is true in CODEBASE_MODEL: extract naming conventions and file organization rules directly from `claude_md.rules`. Reference them by name: "Your CLAUDE.md says X — here's how that looks in practice."
- If no CLAUDE.md: invoke `pattern-detector.md` Detection 1 (Naming Conventions) and Detection 4 (Import/Module Patterns) to infer them.

Present:
- Naming: function naming, class naming, file naming, constant naming
- File organization: how source files are grouped (by feature, by layer, by type)
- Import style: absolute vs relative, barrel exports, any path aliases

**Architecture patterns**
Read the `abstractions` key from CODEBASE_MODEL. For each abstraction listed:
- State what pattern it represents in plain English: "This codebase uses the repository pattern for data access", "Error handling is centralized in GlobalExceptionHandler"
- Read the actual files referenced in the abstraction to confirm the pattern is implemented as described before presenting it

If no CLAUDE.md covers architecture patterns, also invoke `pattern-detector.md` Detection 2 (Structural Patterns).

**Testing conventions**
Read `test_patterns` from CODEBASE_MODEL. Present:
- Framework used (jest, pytest, JUnit, etc.)
- How tests are organized: collocated with source vs separate `tests/` directory
- File naming pattern (`*.test.ts`, `*_test.py`, `*Test.java`)
- What gets mocked vs integrated

If assessment data is available (see Assessment Integration below), weave in the testing score here.

**Git workflow**
Read `git` from CODEBASE_MODEL. Present:
- Branch naming convention (inferred from recent branch names)
- Commit message format (inferred from recent commit messages — conventional commits, Jira ticket prefix, free-form)
- Number of contributors, rough commit cadence

**Unwritten rules**
Invoke `pattern-detector.md` for any convention areas not covered by CLAUDE.md. Present the output as a numbered list of plain-English rules, each with file evidence:
```
1. Every service has a corresponding interface (UserService/IUserService, OrderService/IOrderService — seen in src/services/)
2. Date fields use Instant, never LocalDateTime (seen in User.java:45, Order.java:23)
```

---

After presenting Phase 2, ask:

> "Questions about any of these patterns?"

Wait for the user's response and answer any questions. When they're done, hand control back to the calling skill (guided or explore).

---

## Mode: Condensed Orientation

Used by explore mode. Present only the following — keep it tight, one screen:

1. **System type and tech stack** — one sentence: what it is and what it's built with
2. **Key directories** — bullet list, one line per directory explaining its role (read from `structure.top_level_dirs`)
3. **How to run it locally** — numbered steps from `environment.run_instructions` or README

Then hand control back to the explore skill without asking follow-up questions.

---

## CLAUDE.md Handling

- If `claude_md.found` is true in CODEBASE_MODEL: extract rules and conventions directly from `claude_md.rules`. Reference them by name when presenting each section ("Your CLAUDE.md says to use repository pattern — here's how that looks in practice"). Skip pattern-detector inference for any convention area that CLAUDE.md already covers.
- If no CLAUDE.md: infer everything from code patterns, git history, and existing docs via pattern-detector.md. Make this transparent to the developer: "There's no CLAUDE.md in this repo, so I'm inferring conventions from the code."

---

## Assessment Integration

Check whether CODEBASE_MODEL contains an `assessment` key with `found: true` and non-null `scores`.

**If assessment scores are present:** weave them into the relevant sections during Phase 2:
- Testing score alongside testing conventions: "Your team scored [X]/10 in testing — here's how that shows up in the code: [specific observation]"
- Development score alongside code conventions: "Development scored [X]/10 — [what that reflects about the codebase]"
- Architecture score alongside architecture patterns: "Architecture scored [X]/10 — [what that reflects]"

Keep these references brief and grounded in what you actually see in the code. Do not editorialize about whether the scores are good or bad.

**If no assessment data** (`found: false` or `scores: null`): skip entirely. Do not mention the assessment. This is purely opportunistic — orientation works the same with or without it.
