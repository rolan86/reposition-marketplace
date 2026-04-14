---
description: "Report template for the REPOSITION assessment — defines three output formats used at Stages 5, 6, and 7 of the assess skill"
---

# REPOSITION Report Template

**Role:** This file defines the three output formats produced by the `reposition-assess:assess` skill. It is not executed directly — it is a rendering guide. The assess/SKILL.md reads this file and uses it to format output at Stages 5, 6, and 7.

**Inputs required before rendering:**
- `SCAN_RESULTS` — produced by `scan-engine.md` (Stage 2)
- `QUESTION_RESULTS` — produced by `question-bank.md` (Stage 4)
- `FINAL_SCORES` — computed by the score skill (Stage 5)

`FINAL_SCORES` shape:

```json
{
  "pm":           { "competence": 7.2, "ai_integration": 4.1, "gap": 3.1, "inverse_gap": 0.0, "opportunity_profile": "classic" },
  "architecture": { "competence": 8.0, "ai_integration": 3.5, "gap": 4.5, "inverse_gap": 0.0, "opportunity_profile": "classic" },
  "design":       { "competence": 5.5, "ai_integration": 5.0, "gap": 0.5, "inverse_gap": 0.0, "opportunity_profile": "balanced" },
  "development":  { "competence": 7.8, "ai_integration": 6.2, "gap": 1.6, "inverse_gap": 0.0, "opportunity_profile": "balanced" },
  "overall_ai_readiness": 4.70,
  "biggest_opportunity": {
    "domain": "Architecture",
    "profile": "classic",
    "competence": 8.0,
    "ai_integration": 3.5,
    "gap": 4.5,
    "reason": "High structural capability (8.0/10) with low AI integration (3.5/10) — significant ROI available from AI-assisted architecture planning and design docs"
  }
}
```

---

## Biggest Opportunity Calculation

Before rendering any format, compute `biggest_opportunity`:

**Step 1 — Determine the dominant profile:**

Check the `opportunity_profile` for all four domains (from `SCAN_RESULTS.scores.[domain].opportunity_profile`):

- If ANY domain has profile `"classic"` → use the **standard** selection path (Step 2A).
- If NO domain has `"classic"` but ANY has `"foundational"` → use the **foundational** selection path (Step 2B).
- If ALL domains have `"inverted"` or `"balanced"` → use the **inverted** selection path (Step 2C).

**Step 2A — Standard (classic profile exists):**

1. Filter domains to those with `opportunity_profile = "classic"` (competence >= 6, gap >= 3).
2. Among filtered domains, select the one with the highest `gap` score.
3. Construct reason using the **standard reason templates** below.

**Step 2B — Foundational (no classic, but foundational exists):**

1. Among domains with `opportunity_profile = "foundational"`, select the one with the highest `gap`.
2. Note "foundational work needed before AI integration" in the reason.
3. Construct reason: `"[Domain] competence is [competence]/10 — foundational skills need strengthening before AI can multiply them. Focus on building the structural baseline first."`

**Step 2C — Inverted (all domains are inverted or balanced):**

1. Among domains with `opportunity_profile = "inverted"`, select the one with the highest `inverse_gap`.
2. If no inverted domains (all balanced), select the domain with the lowest competence (most room to grow).
3. Construct reason using the **inverted reason templates** below.

**Standard reason templates (classic profile):**

| Domain | Reason template |
|--------|-----------------|
| Product Management | "Strong PM process ([competence]/10) but AI not yet used for requirements, stories, or prioritisation ([ai_integration]/10) — AI-assisted PM could compress planning cycles significantly" |
| Architecture | "High structural capability ([competence]/10) with low AI integration ([ai_integration]/10) — AI-assisted architecture planning, ADR drafting, and design reviews are the immediate leverage point" |
| Design | "Solid design system in place ([competence]/10) but AI not integrated into the design workflow ([ai_integration]/10) — AI-assisted component generation and a11y review are underutilised" |
| Development | "Strong development practice ([competence]/10) with room to increase AI integration ([ai_integration]/10) — AI-assisted TDD, code review, and commit workflows can accelerate delivery" |

**Inverted reason templates (inverted profile):**

| Domain | Reason template |
|--------|-----------------|
| Product Management | "Heavy AI usage for PM ([ai_integration]/10) but project lacks structural PM artifacts ([competence]/10) — adding ticket references, requirements docs, and a changelog will make your AI-assisted PM more effective" |
| Architecture | "AI-assisted architecture is strong ([ai_integration]/10) but structural artifacts are sparse ([competence]/10) — formalising ADRs, design docs, and plan-before-code commits will give AI more structure to work with" |
| Design | "AI-assisted design is active ([ai_integration]/10) but design infrastructure is minimal ([competence]/10) — establishing design tokens, component libraries, and accessibility tooling will amplify your AI design workflow" |
| Development | "AI co-authoring is strong ([ai_integration]/10) but development scaffolding is missing ([competence]/10) — adding a CLAUDE.md, test suite, and CI pipeline will make your AI-augmented development significantly more effective" |

---

## Gap Bar Visualization

Used in Format 1 (terminal) only.

```
gap_blocks = round(gap_score / 10 * 5)   # maps gap 0-10 to 0-5 filled blocks
bar = "▓" * gap_blocks + "░" * (5 - gap_blocks)
```

Examples:
- Gap 0.0 → `░░░░░`
- Gap 2.0 → `▓░░░░`
- Gap 5.0 → `▓▓▓░░`
- Gap 8.0 → `▓▓▓▓░`
- Gap 10.0 → `▓▓▓▓▓`

---

## Format 1: Terminal Display

**When to use:** Stage 5 of the assess skill — immediately after scores are computed. Display inline in the terminal conversation.

**Color suggestions** (note: these may not render in all terminals — treat as optional enhancement, never rely on them for readability):

| Domain | Suggested color |
|--------|----------------|
| Product Management | Blue |
| Architecture | Purple |
| Design | Pink / Magenta |
| Development | Green |

### 1A — Scorecard Banner

```
═══════════════════════════════════════════════════════
  REPOSITION AI Readiness Assessment
═══════════════════════════════════════════════════════

  Domain           Competence  AI Integration  Gap
  ───────────────  ──────────  ──────────────  ─────
  Product Mgmt     X.X/10      X.X/10          ▓▓▓░░
  Architecture     X.X/10      X.X/10          ▓▓▓▓▓
  Design           X.X/10      X.X/10          ▓░░░░
  Development      X.X/10      X.X/10          ▓░░░░

  ⚡ Biggest opportunity: [domain] ([profile])
     [reason — context-appropriate based on profile]

  Overall AI Readiness: X.XX/10
═══════════════════════════════════════════════════════
```

**Rendering rules:**
- Round all scores to one decimal place in the table (e.g., `7.2/10`).
- Round `overall_ai_readiness` to two decimal places (e.g., `4.70/10`).
- Gap bar: For standard profiles, compute `gap_blocks = round(gap / 10 * 5)`. For inverted profiles where `inverse_gap > gap`, use `gap_blocks = round(inverse_gap / 10 * 5)` and render with `◆` instead of `▓` to visually distinguish inverted gaps: `◆◆◆░░` (reads as "structural foundation needed" rather than "AI integration needed").
- Biggest opportunity: print domain name, profile label in parentheses, then the reason string on a new indented line.
- Profile labels in parentheses: `"classic"` → omit (default), `"foundational"` → `(foundational)`, `"inverted"` → `(inverted — build structure)`, `"balanced"` → `(balanced)`.

### 1B — Evidence Breakdown (per domain)

Display after the scorecard banner, one section per domain. Use the following structure for each domain:

```
━━━ [Domain Name] ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Competence: X.X/10
  Evidence:
    🔍 [scan finding label]: [detail from SCAN_RESULTS]
    🔍 [scan finding label]: [detail from SCAN_RESULTS]
    💬 [self-reported label]: [answer summary from QUESTION_RESULTS]

  AI Integration: X.X/10
  Evidence:
    🔍 [scan finding label]: [detail from SCAN_RESULTS.claude_usage]
    💬 [self-reported label]: [answer summary from QUESTION_RESULTS.ai_integration]

  Gap: X.X  [gap bar]
  [1-2 sentence gap analysis — what is being left on the table]

  Recommended Actions:
    1. [specific, concrete action]
    2. [specific, concrete action]
    3. [specific, concrete action]
```

**Evidence icons:**
- `🔍` — finding came from a static file scan (automated signal).
- `💬` — finding came from a self-reported answer in the question bank.

**Evidence rendering per domain:**

*Product Management:*
- Competence evidence: ticket_score → "Ticket references in git log: [total_ticket_refs] refs found"; req_doc_score → "Requirements docs: [req_doc_type_count] type(s) detected"; changelog_score → "Changelog: [changelog_entry_count] entries"; mcp_score → "MCP integrations: [mcp_pm_count] PM tool(s) connected".
- AI integration evidence: pm_history_score → "PM-related AI prompts in history: [pm_prompt_count] found"; pm_mcp_count → "MCP tool interactions in history: [pm_mcp_count]". Self-reported Q-PM-1 and Q-PM-2 answers summarised in one line each.

*Architecture:*
- Competence evidence: adr_score → "ADR files found: [adr_file_count]"; arch_docs → "Architecture docs: [present/not found]"; modular_score → "Domain boundaries: [domain_boundary_count] top-level module dir(s)"; contract_score → "[contract_type_count] API contract type(s) detected"; plan_commit_score → "Plan/spec commits: [plan_commit_count] in last 200".
- AI integration evidence: arch_history_score → "Architecture-related AI prompts: [arch_prompt_count]"; plan_mode_count → "Plan mode usage: [plan_mode_count] sessions".

*Design:*
- Competence evidence: component_score → "Component directories: [component_dir_count]"; storybook_present → "Storybook: [present/not found]"; styling_score → "Styling signals: [styling_signal_count]/3 detected"; aria_score → "ARIA attributes: [aria_attr_count] found"; story_score → "Story files: [story_count]".
- AI integration evidence: design_history_score → "Design-related AI prompts: [design_prompt_count]"; frontend_design_skill_count → "/frontend-design skill usage: [count]".

*Development:*
- Competence evidence: claude_md signal → "CLAUDE.md: [exists/missing], [claude_md_lines] lines, TDD rules=[tdd_rules], planning=[planning_first_rules]"; test ratio → "Test coverage: [test_file_count] test files / [source_file_count] source files (ratio [test_ratio])"; commit patterns → "Conventional commits: [conventional_commit_count]/100; AI co-author commits: [ai_coauthor_count]; TDD commits: [tdd_commit_count]"; cicd_score → "CI/CD: [which configs found]".
- AI integration evidence: dev_history_score → "Dev-related skill usage: /commit=[commit_skill_count], /tdd=[tdd_skill_count], /review-pr=[review_pr_skill_count]"; ai_coauthor_count → "AI co-author commits: [ai_coauthor_count]".

---

## Format 2: Markdown Export

**When to use:** Stage 7 of the assess skill — when the user requests to save the report. Write to `.reposition/assessment-YYYY-MM-DD.md` in the project root (create `.reposition/` directory if it does not exist).

**File name:** Use the `scan_date` field from SCAN_RESULTS for the date component (format: `YYYY-MM-DD`).

**Project name:** Read from `package.json` `name` field if it exists, otherwise use the basename of `PROJECT_ROOT`.

**Tech stack:** Summarise from scan signals — detected language(s), framework(s), and key dependencies. Example: `"TypeScript, React, Vite"`.

The markdown document to write:

```
# REPOSITION AI Readiness Assessment
**Date:** YYYY-MM-DD
**Project:** [project name]
**Tech Stack:** [detected stack]

## Scorecard

| Domain | Competence | AI Integration | Gap |
|--------|-----------|----------------|-----|
| Product Management | X.X/10 | X.X/10 | X.X |
| Architecture | X.X/10 | X.X/10 | X.X |
| Design | X.X/10 | X.X/10 | X.X |
| Development | X.X/10 | X.X/10 | X.X |

**Overall AI Readiness:** X.XX/10
**Biggest Opportunity:** [domain] — [reason]

---

## Evidence: Product Management

### Competence (X.X/10)
- 🔍 Ticket references in git log: [total_ticket_refs] refs found
- 🔍 Requirements doc types: [req_doc_type_count] type(s) ([names if known])
- 🔍 Changelog entries: [changelog_entry_count]
- 🔍 MCP PM integrations: [mcp_pm_count] tool(s) ([names if known])
- 💬 Requirements intake: [Q-PM-1 answer summary]
- 💬 Prioritisation process: [Q-PM-2 answer summary]

### AI Integration (X.X/10)
- 🔍 PM-related AI prompts in history: [pm_prompt_count]
- 🔍 MCP tool interactions in history: [pm_mcp_count]
- 💬 AI for requirements: [Q-AI-1 answer summary]
- 💬 AI for planning: [Q-AI-2 answer summary]

### Gap Analysis
[1-2 sentences describing what PM capability exists that AI is not yet augmenting, and what is being left on the table as a result.]

### Recommended Actions
1. [specific action — see recommended action map below]
2. [specific action]
3. [specific action]

---

## Evidence: Architecture

### Competence (X.X/10)
- 🔍 ADR files found: [adr_file_count]
- 🔍 Architecture / design docs: [present or not found]
- 🔍 Domain boundaries: [domain_boundary_count] top-level module directories
- 🔍 API contract types: [contract_type_count] ([openapi/graphql/proto/swagger])
- 🔍 Dependency injection: [di_detected] ([di_match_count] matches)
- 🔍 Plan/spec commits: [plan_commit_count] in last 200
- 💬 Architecture decision process: [Q-ARCH-1 answer summary]
- 💬 Stack experience: [Q-ARCH-2 answer summary]

### AI Integration (X.X/10)
- 🔍 Architecture-related AI prompts: [arch_prompt_count]
- 🔍 Plan mode usage: [plan_mode_count] sessions

### Gap Analysis
[1-2 sentences describing what architectural capability exists that AI is not yet augmenting.]

### Recommended Actions
1. [specific action]
2. [specific action]
3. [specific action]

---

## Evidence: Design

### Competence (X.X/10)
- 🔍 Component directories: [component_dir_count]
- 🔍 Storybook: [present / not found]
- 🔍 Styling signals: [styling_signal_count]/3 ([frameworks detected])
- 🔍 ARIA attributes: [aria_attr_count] found in UI files
- 🔍 A11y tooling: [present / not found]
- 🔍 Story files: [story_count]
- 🔍 Visual regression / E2E tooling: [present / not found]
- 💬 UI/UX process: [Q-DESIGN-1 answer summary]
- 💬 User testing: [Q-DESIGN-2 answer summary]

### AI Integration (X.X/10)
- 🔍 Design-related AI prompts: [design_prompt_count]
- 🔍 /frontend-design skill usage: [frontend_design_skill_count]

### Gap Analysis
[1-2 sentences describing what design capability exists that AI is not yet augmenting.]

### Recommended Actions
1. [specific action]
2. [specific action]
3. [specific action]

---

## Evidence: Development

### Competence (X.X/10)
- 🔍 CLAUDE.md: [exists/missing] — [claude_md_lines] lines; planning rules=[planning_first_rules]; TDD rules=[tdd_rules]; arch rules=[arch_rules]
- 🔍 Test coverage: [test_file_count] test files / [source_file_count] source files (ratio [test_ratio]); framework=[framework_detected]
- 🔍 Conventional commits: [conventional_commit_count] of last 100
- 🔍 AI co-author commits: [ai_coauthor_count] of last 100
- 🔍 TDD commit pattern: [tdd_commit_count] test-before-impl commits found
- 🔍 Worktrees in use: [worktree_count] active
- 🔍 CI/CD: GitHub Actions=[ci_github]; other CI=[ci_other]; Docker=[docker_present]; deploy config=[deploy_config]
- 💬 Test writing habits: [Q-DEV-1 answer summary]
- 💬 Code review process: [Q-DEV-2 answer summary]

### AI Integration (X.X/10)
- 🔍 /commit skill usage: [commit_skill_count]
- 🔍 /tdd skill usage: [tdd_skill_count]
- 🔍 /review-pr skill usage: [review_pr_skill_count]
- 🔍 AI co-author commits: [ai_coauthor_count]
- 💬 AI code review: [Q-AI-3 answer summary]
- 💬 Custom prompts / automation: [Q-AI-4 answer summary]

### Gap Analysis
[1-2 sentences describing what development capability exists that AI is not yet augmenting.]

### Recommended Actions
1. [specific action]
2. [specific action]
3. [specific action]

---

## Raw Signals

[yaml block starts here — use triple backticks with yaml language tag in the actual written file]
scan_date: YYYY-MM-DD
project_name: "[name from package.json or directory basename]"
project_type: "[application|library|docs|monorepo|unknown]"
tech_stack: [list]
scores:
  pm:
    competence: X.X
    ai_integration: X.X
    gap: X.X
    inverse_gap: X.X
    opportunity_profile: "[classic|foundational|inverted|balanced]"
  architecture:
    competence: X.X
    ai_integration: X.X
    gap: X.X
    inverse_gap: X.X
    opportunity_profile: "[classic|foundational|inverted|balanced]"
  design:
    competence: X.X
    ai_integration: X.X
    gap: X.X
    inverse_gap: X.X
    opportunity_profile: "[classic|foundational|inverted|balanced]"
  development:
    competence: X.X
    ai_integration: X.X
    gap: X.X
    inverse_gap: X.X
    opportunity_profile: "[classic|foundational|inverted|balanced]"
overall_ai_readiness: X.XX
biggest_opportunity:
  domain: "[domain name]"
  profile: "[classic|foundational|inverted|balanced]"
  reason: "[reason string]"
signals:
  claude_md:
    exists: bool
    lines: N
    has_planning: bool
    has_tdd: bool
    has_arch_rules: bool
  tests:
    exists: bool
    count: N
    ratio: 0.00
    framework: "[framework name or null]"
  plugins_installed: [list]
  installed_plugin_count: N
  skills:
    personal: N
    project: N
    project_settings_present: bool
  history:
    total_prompts: N
    unique_projects: N
    commit_skill_count: N
    tdd_skill_count: N
    review_pr_skill_count: N
    frontend_design_skill_count: N
    plan_mode_count: N
    arch_prompt_count: N
    design_prompt_count: N
    pm_prompt_count: N
    pm_mcp_count: N
    reposition_skill_count: N
    history_confidence: "[HIGH|MEDIUM|LOW|NONE]"
  git:
    no_git: bool
    conventional_commits_count: N
    conventional_commits_pct: 0.00
    ai_coauthor_count: N
    ai_coauthor_pct: 0.00
    tdd_commit_count: N
    tdd_pattern: bool
    plan_commit_count: N
    worktree_count: N
    ticket_refs_count: N
  mcp:
    configured: bool
    no_mcp: bool
    pm_tools: [list]
    mcp_pm_count: N
  development_signals:
    cicd_score: N
    ci_github: bool
    ci_other: bool
    docker_present: bool
    deploy_config: bool
  architecture_signals:
    adr_file_count: N
    arch_docs: bool
    domain_boundary_count: N
    module_root_count: N
    contract_type_count: N
    di_detected: bool
    di_match_count: N
    stack_unknown: bool
  design_signals:
    component_dir_count: N
    storybook_present: bool
    styling_signal_count: N
    aria_attr_count: N
    a11y_tooling: bool
    story_count: N
    visual_regression: bool
    no_ui_files: bool
  pm_signals:
    req_doc_type_count: N
    changelog_present: bool
    changelog_entry_count: N
question_results_summary:
  total_questions_asked: N
  total_questions_skipped: N
  pm:
    requirements_intake: { answered: bool, option: "[A-D or null]", score: N }
    prioritisation: { answered: bool, option: "[A-D or null]", score: N }
  architecture:
    decision_process: { answered: bool, option: "[A-D or null]", score: N }
    stack_experience: { answered: bool, option: "[A-D or null]", score: N }
  design:
    ui_ux_process: { answered: bool, option: "[A-D or null]", score: N }
    user_testing: { answered: bool, option: "[A-D or null]", score: N }
  development:
    test_writing: { answered: bool, option: "[A-D or null]", score: N }
    code_review: { answered: bool, option: "[A-D or null]", score: N }
  ai_integration:
    requirements_with_ai: { answered: bool, option: "[A-C or null]", score: N }
    planning_with_ai: { answered: bool, option: "[A-D or null]", score: N }
    ai_code_review: { answered: bool, option: "[A-C or null]", score: N }
    custom_prompts: { answered: bool, option: "[A-C or null]", score: N }
[yaml block ends here]
```

**IMPORTANT — Raw Signals YAML is the Plugin 1 to Plugin 2 contract.** Plugin 2 (`reposition-transform`) parses the `Raw Signals` YAML block from this file to drive the transformation flow. Every field listed above MUST be populated before saving. Do not omit fields — write `null` or `0` if the value is unknown, but never leave a field out of the output.

**`history_confidence` derivation:**
- `HIGH` — `total_prompts >= 50` and `~/.claude/history.jsonl` was found.
- `MEDIUM` — `total_prompts 10-49` and file was found.
- `LOW` — `total_prompts 1-9` or file found but very sparse.
- `NONE` — file not found or `total_prompts = 0`.

**`conventional_commits_pct`** = `conventional_commits_count / 100` (capped at 1.0).
**`ai_coauthor_pct`** = `ai_coauthor_count / 100` (capped at 1.0).
**`tdd_pattern`** = `true` if `tdd_commit_count >= 4` OR `(tdd_rules = true AND tdd_commit_count >= 1)`.

---

## Format 3: Facilitator Summary (Clipboard)

**When to use:** Stage 7 of the assess skill — when the user requests to copy the summary to clipboard. Output this block as plain text (no markdown formatting).

Template:

```
REPOSITION Assessment — [Project Name] — [YYYY-MM-DD]
PM: [comp]/10 → AI [ai]/10 (gap [gap]) | Arch: [comp]/10 → AI [ai]/10 (gap [gap])
Design: [comp]/10 → AI [ai]/10 (gap [gap]) | Dev: [comp]/10 → AI [ai]/10 (gap [gap])
Overall AI Readiness: [overall]/10 | Biggest Opportunity: [domain]
```

**Rendering rules:**
- All scores rounded to one decimal place.
- Overall AI Readiness rounded to two decimal places.
- Project Name: same source as Format 2 (package.json `name` or directory basename).
- Date: ISO format, from `scan_date`.
- Output as a fenced code block with no language tag so the user can copy it easily.

**Example rendered output:**

```
REPOSITION Assessment — my-app — 2026-04-09
PM: 7.2/10 → AI 4.1/10 (gap 3.1) | Arch: 8.0/10 → AI 3.5/10 (gap 4.5)
Design: 5.5/10 → AI 5.0/10 (gap 0.5) | Dev: 7.8/10 → AI 6.2/10 (gap 1.6)
Overall AI Readiness: 4.70/10 | Biggest Opportunity: Architecture
```

---

## Recommended Actions Map

Use this map to generate the three recommended actions per domain. Select the three that are most relevant given the domain's gap score and the specific signals that were weak.

### Product Management

| Condition | Recommended action |
|-----------|-------------------|
| `pm_history_score < 4` AND `Q-AI-1 score < 5` | "Integrate AI into requirements intake: use `/brainstorm` or Claude to draft user stories and acceptance criteria from raw feature requests before writing tickets." |
| `mcp_pm_count = 0` | "Connect your project management tool via MCP (Jira, Linear, or Notion server) so Claude can read and write tickets directly from the terminal." |
| `req_doc_type_count = 0` | "Create a lightweight PRD template in your repo (`docs/prd-template.md`) and use Claude to populate it before any new feature begins." |
| `ticket_score < 5` | "Adopt conventional commits with ticket references (`feat(PROJ-123): ...`) so your git log becomes a queryable PM audit trail." |
| `changelog_score < 3` | "Add a `CHANGELOG.md` and use the `/commit` skill's changelog update prompt after each sprint to keep it current automatically." |
| `Q-PM-2 score < 5` | "Use Claude to run weekly priority triage: paste your backlog and ask Claude to stack-rank items by effort/value ratio." |
| `plan_mode_count < 3` | "Adopt plan mode (`/plan`) for all feature work — write the spec with AI before writing a single line of code." |

### Architecture

| Condition | Recommended action |
|-----------|-------------------|
| `adr_file_count < 3` | "Create an `adr/` directory and write your first three ADRs — use Claude to draft them from your existing architecture decisions. Target: one ADR per major system choice." |
| `arch_history_score < 4` | "Use Claude for architecture exploration before choosing libraries or patterns: paste your requirements and ask for a comparison of 2-3 approaches with trade-offs." |
| `plan_commit_count < 4` | "Adopt plan-before-code commits: write a `docs/design/[feature]-plan.md` with Claude, commit it with `docs(plan):`, then implement. This creates a searchable architecture log." |
| `contract_type_count = 0` | "Define your API contracts explicitly — use Claude to generate an OpenAPI spec from your existing routes. A contract-first approach enables AI to assist with client generation and validation." |
| `di_detected = false` AND stack is Node/Python/Go | "Introduce a dependency injection pattern: even a simple service registry reduces coupling and enables AI to reason about your module boundaries more accurately." |
| `domain_boundary_count < 3` | "Refactor toward domain-driven structure (`src/features/[name]/` or `src/domains/[name]/`). Use Claude to propose a domain map from your current file tree." |
| `arch_prompt_count < 5` | "Use Claude's plan mode for your next architectural decision: describe the problem, ask Claude to produce an ADR draft, then refine it before committing." |

### Design

| Condition | Recommended action |
|-----------|-------------------|
| `storybook_present = false` AND `component_dir_count >= 1` | "Add Storybook to your existing component library — even 5 stories per core component gives you a living design system and a visual regression baseline." |
| `design_history_score < 4` | "Use the `/frontend-design` skill for your next UI feature: describe the component or screen, let Claude draft the JSX and styles, then refine." |
| `aria_score < 5` | "Run an accessibility audit with `axe-core` or `jest-axe` — ask Claude to add `aria-label`, `aria-describedby`, and `role` attributes to your top 10 interactive components." |
| `visual_regression = false` | "Add Playwright or Chromatic for visual regression — start with screenshots of your 3 most critical pages. This catches UI regressions before users do." |
| `styling_signal_count < 2` | "Adopt a CSS framework or utility library (Tailwind, CSS Modules) and use Claude to migrate your most complex component's styles as a proof of concept." |
| `story_count < 5` | "Write Storybook stories for your top 5 components — use Claude to generate the story file from the component's props interface." |
| `Q-DESIGN-2 score < 5` | "Add Playwright end-to-end tests for your primary user flows — describe the happy path to Claude and ask it to generate the test file." |

### Development

| Condition | Recommended action |
|-----------|-------------------|
| `tdd_rules = false` OR `claude_md_lines < 30` | "Expand your `CLAUDE.md` with explicit TDD and planning-first rules. Use Claude to draft a CLAUDE.md from your team's working preferences — this directly controls AI behaviour in every session." |
| `tdd_commit_count < 4` AND `Q-DEV-1 score < 7` | "Adopt TDD with AI assistance: write a failing test with Claude's `/tdd` skill, implement until green, then commit the test first. Aim for 5 TDD commits in the next sprint." |
| `ai_coauthor_count < 5` | "Start using the `/commit` skill for every commit — AI-assisted commit messages improve conventional commit adherence and create a searchable co-authorship trail." |
| `review_pr_skill_count < 3` | "Add an AI review step to your PR workflow: run `/review-pr` on every PR before requesting human review. This catches issues earlier and reduces review round-trips." |
| `test_ratio < 0.15` | "Increase test coverage by asking Claude to generate test files for your top 5 most-used modules. Target a test ratio of 0.20 within the next two sprints." |
| `cicd_score < 4` | "Add a GitHub Actions workflow — use Claude to generate a `.github/workflows/ci.yml` that runs your test suite on every PR. Start with lint + test, add deploy later." |
| `worktree_count = 0` | "Use git worktrees for parallel feature work: `git worktree add ../[project]-feat-[name] -b feat/[name]`. Claude Code sessions can run concurrently across worktrees without context bleed." |
| `commit_skill_count < 10` | "Run `/commit` after every logical unit of work — not just at end of day. Frequent atomic commits with AI-generated messages improve your git history quality." |

**Selection rule:** For each domain, pick the three actions where the condition is true. If more than three conditions are true, prioritise those where the signal score was lowest (greatest gap). If fewer than three conditions are true, generate a contextual action based on the domain's biggest_opportunity reason.

---

## Instructions for assess/SKILL.md

### When to use each format

| Stage | Format | Trigger |
|-------|--------|---------|
| Stage 5 — Score display | Format 1A (scorecard banner) + Format 1B (evidence breakdown, one domain at a time) | Automatically after scores are computed — no user prompt needed |
| Stage 7 — Save report | Format 2 (markdown export) | User confirms save OR user ran `/reposition-assess` with `--save` flag |
| Stage 7 — Clipboard | Format 3 (facilitator summary) | User confirms clipboard copy OR user ran with `--clipboard` flag |

### How to populate template variables

**From `SCAN_RESULTS`:**
- `SCAN_RESULTS.scan_metadata.scan_timestamp` → `scan_date` (truncate to date portion `YYYY-MM-DD`).
- `SCAN_RESULTS.domains.development.signals.*` → development evidence fields.
- `SCAN_RESULTS.domains.architecture.signals.*` → architecture evidence fields.
- `SCAN_RESULTS.domains.design.signals.*` → design evidence fields.
- `SCAN_RESULTS.domains.pm.signals.*` → PM evidence fields.
- `SCAN_RESULTS.claude_usage.*` → all history, plugin, and skill usage fields.
- `SCAN_RESULTS.scores.*` → intermediate structural and AI integration scores (pre-question-weighting).

**From `QUESTION_RESULTS`:**
- `QUESTION_RESULTS.pm.requirements_intake.option` → Q-PM-1 answer — map option letter to answer text for the evidence summary.
- `QUESTION_RESULTS.pm.prioritisation.option` → Q-PM-2 answer.
- `QUESTION_RESULTS.architecture.decision_process.option` → Q-ARCH-1.
- `QUESTION_RESULTS.architecture.stack_experience.option` → Q-ARCH-2.
- `QUESTION_RESULTS.design.ui_ux_process.option` → Q-DESIGN-1.
- `QUESTION_RESULTS.design.user_testing.option` → Q-DESIGN-2.
- `QUESTION_RESULTS.development.test_writing.option` → Q-DEV-1.
- `QUESTION_RESULTS.development.code_review.option` → Q-DEV-2.
- `QUESTION_RESULTS.ai_integration.requirements_with_ai.option` → Q-AI-1.
- `QUESTION_RESULTS.ai_integration.planning_with_ai.option` → Q-AI-2.
- `QUESTION_RESULTS.ai_integration.ai_code_review.option` → Q-AI-3.
- `QUESTION_RESULTS.ai_integration.custom_prompts.option` → Q-AI-4.
- For skipped questions: note `"scan data used (skipped)"` in the evidence line instead of showing an option.

**From `FINAL_SCORES`:**
- `FINAL_SCORES.pm.competence` → PM Competence column.
- `FINAL_SCORES.pm.ai_integration` → PM AI Integration column.
- `FINAL_SCORES.pm.gap` → PM Gap column and gap bar.
- Repeat for architecture, design, development.
- `FINAL_SCORES.overall_ai_readiness` → Overall AI Readiness line.
- `FINAL_SCORES.biggest_opportunity` → Biggest opportunity block.

### How to generate recommended actions

1. For each domain, evaluate the conditions in the Recommended Actions Map above against the signal values in `SCAN_RESULTS` and question scores in `QUESTION_RESULTS`.
2. Collect all true conditions.
3. Sort true conditions by signal score ascending (weakest signal first = highest priority action).
4. Take the top three.
5. If a question was skipped (scan data used), treat the scan score for that signal as the "question score" when evaluating conditions.
6. Fill in concrete values from the scan where the action template uses `[N]` or `[name]` placeholders (e.g., replace `[project management tool]` with the actual tool name if `mcp_pm_count > 0` and tool names are known from `.mcp.json`).

### Format 2 directory creation

Before writing `.reposition/assessment-YYYY-MM-DD.md`:
```
Bash: mkdir -p PROJECT_ROOT/.reposition
```
Then write the file. If the directory already exists, `mkdir -p` is safe.

### Format 2 YAML rendering

The Raw Signals YAML block is fenced with triple backticks and `yaml` language tag inside the markdown file. When rendering, substitute every placeholder value with the actual computed value from `SCAN_RESULTS`, `QUESTION_RESULTS`, and `FINAL_SCORES`. Use `null` (YAML null, not the string "null") for any field where no data was collected.
