---
description: "Core scan engine — instructs Claude exactly what signals to collect per domain and how to convert findings into 1-10 scores"
---

# REPOSITION Scan Engine

**Role:** You are the silent scanner for Stage 2 of the REPOSITION assessment. Read every section below and execute every listed scan command in sequence. Do not prompt the user. Do not explain what you are doing. Collect all findings, store them in a structured result object, and hand control back to the assessment flow.

**When to run:** Triggered automatically by the `reposition-assess:assess` skill and the `reposition-assess:score` skill at the start of the scan phase.

**Output contract:** Produce a JSON-compatible object called `SCAN_RESULTS` with one key per domain (`development`, `architecture`, `design`, `pm`) plus a top-level `claude_usage` key. Each domain key contains `signals` (raw findings), `structural_score` (1-10), and `ai_integration_score` (1-10). The `claude_usage` key contains raw cross-cutting findings used by all four domain scorers.

---

## Pre-Scan: Detect Project Root

Before running any domain scans, confirm the working directory is a meaningful project root:

```
Bash: git rev-parse --show-toplevel
```

- If this succeeds, use the returned path as `PROJECT_ROOT` for all subsequent scans.
- If git is not initialised, use the current working directory and note `no_git: true` in `SCAN_RESULTS`. Git-dependent signals score 0 for that session.

---

## Signal 0: Project Type Classification

Before running domain-specific scans, classify the project type. This determines which structural signals are meaningful and adjusts scoring weights accordingly.

### Step 0.1 — Count source files

```
Glob: **/*.js  **/*.ts  **/*.jsx  **/*.tsx  **/*.py  **/*.go  **/*.rb  **/*.java  **/*.kt  **/*.rs  **/*.cs  (search path: PROJECT_ROOT, exclude node_modules, vendor, .venv, dist, build)
```

→ `total_source_files`.

### Step 0.2 — Count content files

```
Glob: **/*.md  **/*.mdx  **/*.rst  **/*.txt  (search path: PROJECT_ROOT, exclude node_modules, vendor, .venv, dist, build)
```

→ `total_content_files`.

### Step 0.3 — Check for application indicators

```
Glob: package.json  requirements.txt  pyproject.toml  go.mod  Cargo.toml  Gemfile  pom.xml  build.gradle  (search path: PROJECT_ROOT)
```

→ `has_package_manifest` (true if any found).

### Step 0.4 — Classify

Apply these rules in order (first match wins):

| Condition | `project_type` |
|-----------|---------------|
| `total_source_files >= 10` AND `has_package_manifest` | `"application"` |
| `total_source_files >= 5` AND `has_package_manifest` | `"library"` |
| `total_content_files >= 5` AND `total_source_files < 5` | `"docs"` |
| Multiple `package.json` files in distinct subdirectories | `"monorepo"` |
| None of the above | `"unknown"` |

### Step 0.5 — Record and apply

Store `project_type` in `SCAN_RESULTS.scan_metadata.project_type`.

**Scoring impact by project type:**

| Project type | Effect |
|-------------|--------|
| `"application"` | All domain signals scored normally. |
| `"library"` | All domain signals scored normally. Design signals that depend on UI files score 0 without penalty (noted, not penalised). |
| `"docs"` | Structural signals that require source code (test ratio, CI/CD, components, DI, API contracts) are **excluded** from structural score averages — they are recorded as `null` (not 0). Only signals that can meaningfully apply to docs projects contribute to the average. See Section 6 for adjusted averaging. |
| `"monorepo"` | All domain signals scored normally. Note `project_type: monorepo` for context. |
| `"unknown"` | All domain signals scored normally. Note `stack_unknown: true`. |

**For `project_type = "docs"`, the following signals are excluded from structural score averaging** (still collected for the record, but marked `excluded_for_docs: true`):

- Development: Signal 1.2 (test ratio), Signal 1.5 (CI/CD)
- Architecture: Signal 2.2 (modular structure), Signal 2.3 (API contracts), Signal 2.4 (DI patterns)
- Design: Signal 3.1 (component library), Signal 3.2 (CSS framework), Signal 3.4 (visual regression)
- PM: (no exclusions — all PM signals apply to any project type)

**Signals that still apply to docs projects:**
- Development: CLAUDE.md (1.1), git patterns (1.3), worktrees (1.4), Claude history (1.6)
- Architecture: ADRs and design docs (2.1), plan-before-code commits (2.5), Claude history (2.6)
- Design: Accessibility in HTML files (3.3), Claude history (3.5)
- PM: All signals (4.1–4.5)

---

## Section 1: Development Domain Signals

### Signal 1.1 — CLAUDE.md Existence and Quality

**Check 1 — File exists:**
```
Glob: CLAUDE.md  (search path: PROJECT_ROOT)
Glob: .claude/CLAUDE.md  (search path: PROJECT_ROOT)
```

- File found at root → +1 point toward existence sub-score.
- File found at `.claude/CLAUDE.md` → +1 point (counts same as root).
- Not found → existence sub-score = 0.

**Graceful handling:** If neither path returns a result, record `claude_md: null` and continue. Do not abort.

**Check 2 — Line count (when file exists):**
```
Bash: wc -l < PROJECT_ROOT/CLAUDE.md
```

- 0-10 lines → quality = 1 (stub).
- 11-30 lines → quality = 2 (minimal).
- 31-60 lines → quality = 3 (basic config).
- 61-100 lines → quality = 4 (solid config).
- 101+ lines → quality = 5 (comprehensive config).

**Check 3 — Planning-first rules:**
```
Grep: pattern="plan|spec|design.first|think.before|PLAN_MODE|plan.mode" file=PROJECT_ROOT/CLAUDE.md (case-insensitive)
```

- 1+ matches → planning_first = true (+1 point).
- No matches → planning_first = false.

**Check 4 — TDD rules:**
```
Grep: pattern="tdd|test.first|test.driven|write.test|failing.test" file=PROJECT_ROOT/CLAUDE.md (case-insensitive)
```

- 1+ matches → tdd_rules = true (+1 point).
- No matches → tdd_rules = false.

**Check 5 — Architecture rules:**
```
Grep: pattern="architect|module|layer|domain|separation|boundary|dependency" file=PROJECT_ROOT/CLAUDE.md (case-insensitive)
```

- 1+ matches → arch_rules = true (+1 point).
- No matches → arch_rules = false.

**Development signal 1.1 raw score:** existence sub-score (0-1) + quality (1-5, scaled to 0-3 by dividing by 1.67) + planning_first (0-1) + tdd_rules (0-1) + arch_rules (0-1). Max = 7 raw points. Normalise to 0-10 by multiplying by (10/7).

---

### Signal 1.2 — Test File Coverage

**Check 1 — Count test files:**
```
Glob: **/*.test.js  **/*.test.ts  **/*.test.jsx  **/*.test.tsx  **/*.test.py  **/*.test.go  **/*.test.rb  (search path: PROJECT_ROOT)
Glob: **/*.spec.js  **/*.spec.ts  **/*.spec.jsx  **/*.spec.tsx  **/*.spec.py  **/*.spec.rb  (search path: PROJECT_ROOT)
Glob: **/test_*.py  **/test_*.go  **/test_*.rb  (search path: PROJECT_ROOT)
Glob: **/*_test.go  **/*_test.py  (search path: PROJECT_ROOT)
```

Sum all results → `test_file_count`.

**Check 2 — Count source files:**
```
Glob: **/*.js  **/*.ts  **/*.jsx  **/*.tsx  **/*.py  **/*.go  **/*.rb  (search path: PROJECT_ROOT, exclude node_modules, vendor, .venv, dist, build)
```

→ `source_file_count`.

**Check 3 — Test ratio:**
`test_ratio = test_file_count / max(source_file_count, 1)`

- test_ratio < 0.05 → ratio_score = 0.
- 0.05-0.15 → ratio_score = 2.
- 0.16-0.30 → ratio_score = 4.
- 0.31-0.50 → ratio_score = 6.
- 0.51-0.80 → ratio_score = 8.
- > 0.80 → ratio_score = 10.

**Check 4 — Detect test framework:**
```
Bash: cat PROJECT_ROOT/package.json 2>/dev/null | grep -E '"jest"|"vitest"|"mocha"|"jasmine"|"@testing-library"'
Bash: cat PROJECT_ROOT/pyproject.toml PROJECT_ROOT/setup.cfg PROJECT_ROOT/requirements*.txt 2>/dev/null | grep -E 'pytest|unittest'
Bash: cat PROJECT_ROOT/go.mod 2>/dev/null | grep -E 'testing|testify'
```

- Any framework detected → framework_detected = true (+1 point modifier applied later).
- None detected → framework_detected = false.

**Graceful handling:** If `source_file_count` is 0 (e.g. pure config repo), set ratio_score = 0 and note `no_source_files: true`.

**Development signal 1.2 raw score:** ratio_score (0-10). If framework_detected = true, add 0.5 (cap at 10).

---

### Signal 1.3 — Git Commit Patterns

**Graceful handling for all git signals:** If `no_git: true` was set in pre-scan, skip this entire signal and record `git_signals: null`.

**Check 1 — Conventional commits:**
```
Bash: git -C PROJECT_ROOT log --oneline -100
```

Count lines matching `^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?:` → `conventional_count`.

- 0 matching → conventional_score = 0.
- 1-10 → conventional_score = 3.
- 11-30 → conventional_score = 6.
- 31-60 → conventional_score = 8.
- 61+ → conventional_score = 10.

**Check 2 — AI co-author signal:**
```
Bash: git -C PROJECT_ROOT log --all -100 --format="%B" | grep -c "Co-Authored-By: Claude"
```

→ `ai_coauthor_count`.

- 0 → ai_coauthor_score = 0.
- 1-5 → ai_coauthor_score = 4.
- 6-20 → ai_coauthor_score = 7.
- 21+ → ai_coauthor_score = 10.

**Check 3 — TDD commit ordering (test before implementation):**
```
Bash: git -C PROJECT_ROOT log --oneline -100 | grep -iE "^[a-f0-9]+ test:|^[a-f0-9]+ feat\(test\)|^[a-f0-9]+ add.*test|^[a-f0-9]+ tdd"
```

Count matching lines → `tdd_commit_count`.

- 0 → tdd_commit_score = 0.
- 1-3 → tdd_commit_score = 4.
- 4-10 → tdd_commit_score = 7.
- 11+ → tdd_commit_score = 10.

**Development signal 1.3 raw score:** average of (conventional_score, ai_coauthor_score, tdd_commit_score). Range 0-10.

---

### Signal 1.4 — Worktrees

```
Bash: git -C PROJECT_ROOT worktree list
```

Count lines returned → `worktree_count` (subtract 1 for the main worktree).

- 0 active secondary worktrees → worktree_score = 0.
- 1 → worktree_score = 6.
- 2-3 → worktree_score = 8.
- 4+ → worktree_score = 10.

**Graceful handling:** If `no_git: true`, set worktree_score = 0.

---

### Signal 1.5 — CI/CD Configuration

**Check 1:**
```
Glob: .github/workflows/*.yml  .github/workflows/*.yaml  (search path: PROJECT_ROOT)
```

Any found → ci_github = true.

**Check 2:**
```
Glob: .gitlab-ci.yml  .circleci/config.yml  Jenkinsfile  .travis.yml  bitbucket-pipelines.yml  (search path: PROJECT_ROOT)
```

Any found → ci_other = true.

**Check 3:**
```
Glob: Dockerfile  docker-compose.yml  docker-compose.yaml  docker-compose.*.yml  (search path: PROJECT_ROOT)
```

Any found → docker_present = true.

**Check 4:**
```
Glob: vercel.json  netlify.toml  .fly/  fly.toml  render.yaml  railway.toml  (search path: PROJECT_ROOT)
```

Any found → deploy_config = true.

- 0 of the above → cicd_score = 0.
- 1 true → cicd_score = 4.
- 2 true → cicd_score = 6.
- 3 true → cicd_score = 8.
- 4 true → cicd_score = 10.

**Graceful handling:** Missing files return no glob results — treat as false, do not error.

---

### Signal 1.6 — Claude History (Development)

See **Section 5** for the shared Claude usage scan. From those cross-cutting results, extract for development scoring:

- `/commit` command usage count → dev_commit_skill_count.
- `/tdd` command usage count → dev_tdd_skill_count.
- `/review-pr` command usage count → dev_review_skill_count.

**Development AI signal from history:**
- Total of above three counts < 5 → dev_history_score = 0.
- 5-15 → dev_history_score = 3.
- 16-30 → dev_history_score = 6.
- 31-60 → dev_history_score = 8.
- 61+ → dev_history_score = 10.

---

### Development Domain Scoring

**Structural score:**
Average of signal 1.1 normalised score, signal 1.2 raw score, signal 1.3 raw score, signal 1.4 worktree_score, signal 1.5 cicd_score. All on 0-10 scale. If `project_type = "docs"`, exclude signals 1.2 and 1.5 from the average (divide by 3 instead of 5).

**AI Integration score (70% weight from scan, 30% self-report applied later):**
`dev_history_score` from Signal 1.6, plus ai_coauthor_score from Signal 1.3. Average of those two. Range 0-10.

**Scoring Rubric:**

| Band | Structural Score | Description |
|------|-----------------|-------------|
| 0-2  | No CLAUDE.md; 0 test files; no conventional commits; no CI. Raw ad-hoc development. |
| 3-4  | CLAUDE.md exists but < 30 lines; test ratio < 0.10; some conventional commits (< 15 of 100). |
| 5-6  | CLAUDE.md 30-60 lines with some rules; test ratio 0.15-0.35; 15-40 conventional commits; CI pipeline present. |
| 7-8  | CLAUDE.md 60+ lines with planning/TDD rules; test ratio 0.35-0.60; 40+ conventional commits; AI co-author commits detected; worktrees in use. |
| 9-10 | CLAUDE.md 100+ lines with planning, TDD, and arch rules; test ratio > 0.60; 60+ conventional commits; regular AI co-author commits; multiple worktrees; full CI/CD. |

---

## Section 2: Architecture Domain Signals

### Signal 2.1 — ADR and Design Docs

**Check 1:**
```
Glob: **/adr/**  **/ADR/**  **/decisions/**  **/architecture/**  (search path: PROJECT_ROOT)
```

Count matching files → `adr_file_count`.

- 0 → adr_score = 0.
- 1-2 → adr_score = 4.
- 3-7 → adr_score = 7.
- 8+ → adr_score = 10.

**Check 2:**
```
Glob: **/docs/arch*  **/docs/design*  **/docs/system*  **/docs/rfc*  (search path: PROJECT_ROOT)
```

Any found → arch_docs = true (+2 bonus, cap at 10).

**Graceful handling:** No matches → score component = 0. Continue.

---

### Signal 2.2 — Modular Structure

**Check 1:**
```
Bash: ls PROJECT_ROOT/src/ 2>/dev/null
Bash: ls PROJECT_ROOT/pkg/ 2>/dev/null
Bash: ls PROJECT_ROOT/internal/ 2>/dev/null
```

Count how many of these directories exist → `module_root_count`.

**Check 2:**
```
Glob: src/modules/**  src/features/**  src/domains/**  src/services/**  (search path: PROJECT_ROOT)
```

Count distinct top-level subdirectories found → `domain_boundary_count`.

- `module_root_count` = 0 and `domain_boundary_count` = 0 → modular_score = 0.
- `module_root_count` >= 1 → modular_score += 3.
- `domain_boundary_count` 1-3 → modular_score += 3.
- `domain_boundary_count` 4-7 → modular_score += 5.
- `domain_boundary_count` 8+ → modular_score += 7. Cap at 10.

**Graceful handling:** If `ls` fails (directory does not exist), count = 0 for that directory.

---

### Signal 2.3 — API Contracts

**Check 1:**
```
Glob: **/*.openapi.json  **/*.openapi.yaml  **/*.openapi.yml  **/openapi.json  **/openapi.yaml  **/openapi.yml  (search path: PROJECT_ROOT)
```

**Check 2:**
```
Glob: **/swagger.json  **/swagger.yaml  **/swagger.yml  **/*.swagger.*  (search path: PROJECT_ROOT)
```

**Check 3:**
```
Glob: **/schema.graphql  **/*.graphql  **/*.gql  (search path: PROJECT_ROOT)
```

**Check 4:**
```
Glob: **/proto/**  **/*.proto  (search path: PROJECT_ROOT)
```

Count how many of checks 1-4 return at least one file → `contract_type_count`.

- 0 → contract_score = 0.
- 1 → contract_score = 5.
- 2 → contract_score = 7.
- 3+ → contract_score = 10.

**Graceful handling:** No matches for any check → contract_score = 0.

---

### Signal 2.4 — Dependency Injection Patterns

Run the appropriate Grep based on the detected stack (detect stack via package.json, go.mod, requirements.txt, Gemfile presence in PROJECT_ROOT):

**Node/TypeScript:**
```
Grep: pattern="@injectable|@inject|InversifyContainer|tsyringe|typedi|NestFactory|@Module\(" path=PROJECT_ROOT (case-sensitive, glob=**/*.ts,**/*.js)
```

**Python:**
```
Grep: pattern="@inject|inject\(|dependency_injector|Depends\(|fastapi.*Depends" path=PROJECT_ROOT (case-insensitive, glob=**/*.py)
```

**Go:**
```
Grep: pattern="wire\.Build|dig\.New|fx\.New|google/wire" path=PROJECT_ROOT (glob=**/*.go,**/go.mod)
```

**Java/Kotlin:**
```
Grep: pattern="@Autowired|@Inject|@Component|@Service|@Bean" path=PROJECT_ROOT (glob=**/*.java,**/*.kt)
```

Any Grep returning 1+ matches → di_detected = true.

- di_detected = false → di_score = 0.
- di_detected = true and match count 1-5 → di_score = 5.
- di_detected = true and match count 6-20 → di_score = 7.
- di_detected = true and match count 21+ → di_score = 10.

**Graceful handling:** If no stack is detectable, skip this signal and set di_score = 0, note `stack_unknown: true`.

---

### Signal 2.5 — Plan-Before-Code Git Pattern

**Check 1 — Prefix-style plan commits:**
```
Bash: git -C PROJECT_ROOT log --oneline -200 | grep -iE "plan:|spec:|design:|rfc:|adr:|proposal:"
```

**Check 2 — Scoped conventional commits with plan/spec/design scope:**
```
Bash: git -C PROJECT_ROOT log --oneline -200 | grep -iE "(feat|docs|chore)\((plan|spec|design|rfc|adr)\)"
```

**Check 3 — Keyword matches in commit messages (broader net):**
```
Bash: git -C PROJECT_ROOT log --format="%s" -200 | grep -icE "design spec|implementation plan|architecture decision|planning doc"
```

Sum unique matches across all three checks → `plan_commit_count`. (If the same commit matches multiple checks, count it once.)

- 0 → plan_commit_score = 0.
- 1-3 → plan_commit_score = 4.
- 4-10 → plan_commit_score = 7.
- 11+ → plan_commit_score = 10.

**Graceful handling:** If `no_git: true`, set plan_commit_score = 0.

---

### Signal 2.6 — Claude History (Architecture)

From cross-cutting Claude usage results (Section 5):

- Plan mode usage count (`plan mode` prompts or `--plan` flags detected in history) → arch_plan_count.
- Architecture-related prompts (Grep `history.jsonl` for `architect|ADR|design doc|system design|module boundary|API contract`) → arch_prompt_count.

**Architecture AI signal from history:**
- Total of arch_plan_count + arch_prompt_count < 3 → arch_history_score = 0.
- 3-10 → arch_history_score = 3.
- 11-25 → arch_history_score = 6.
- 26-50 → arch_history_score = 8.
- 51+ → arch_history_score = 10.

---

### Architecture Domain Scoring

**Structural score:**
Average of: adr_score, modular_score, contract_score, di_score, plan_commit_score. All 0-10. If `project_type = "docs"`, exclude modular_score, contract_score, and di_score (divide by 2 instead of 5).

**AI Integration score (70% weight from scan, 30% self-report applied later):**
arch_history_score from Signal 2.6. Range 0-10.

**Scoring Rubric:**

| Band | Structural Score | Description |
|------|-----------------|-------------|
| 0-2  | No ADRs; no domain separation; no API contracts; no DI; no plan commits. Ad-hoc structure. |
| 3-4  | Some directory grouping (src/); no ADRs; 1 API contract type; no DI; 1-2 plan commits. |
| 5-6  | Clear domain modules; 1-4 ADRs or arch docs; 1-2 API contract types; basic DI in 1-5 places. |
| 7-8  | 3-8 ADRs; modular domain structure with 4+ boundaries; 2+ API contract types; consistent DI pattern; 4-10 plan commits. |
| 9-10 | 8+ ADRs; strong domain separation; 3+ API contract types; pervasive DI; 11+ plan/spec commits; Claude-assisted design in history. |

---

## Section 3: Design Domain Signals

### Signal 3.1 — Component Library

**Check 1:**
```
Glob: **/components/**  **/ui/**  **/design-system/**  (search path: PROJECT_ROOT)
```

Count distinct top-level directories matched → `component_dir_count`.

- 0 → component_score = 0.
- 1 → component_score = 4.
- 2 → component_score = 6.
- 3+ → component_score = 8.

**Check 2 — Storybook:**
```
Glob: .storybook/**  **/*.stories.js  **/*.stories.ts  **/*.stories.jsx  **/*.stories.tsx  **/*.stories.mdx  (search path: PROJECT_ROOT)
```

Any found → storybook_present = true. Add +2 to component_score (cap at 10).

**Graceful handling:** No matches → component_score stays at whatever was calculated.

---

### Signal 3.2 — CSS / Styling Framework

**Check 1:**
```
Bash: cat PROJECT_ROOT/package.json 2>/dev/null | grep -E '"tailwindcss"|"@tailwindcss|"styled-components"|"@emotion|"sass"|"less"|"postcss"|"@mui|"@chakra-ui|"@radix-ui"'
```

**Check 2:**
```
Bash: cat PROJECT_ROOT/requirements.txt PROJECT_ROOT/pyproject.toml 2>/dev/null | grep -E 'tailwind|bootstrap|bulma'
```

**Check 3:**
```
Glob: tailwind.config.js  tailwind.config.ts  tailwind.config.cjs  (search path: PROJECT_ROOT)
Glob: **/*.module.css  **/*.module.scss  (search path: PROJECT_ROOT)
```

Count how many of checks 1-3 have positive results → `styling_signal_count`.

- 0 → styling_score = 0.
- 1 → styling_score = 5.
- 2 → styling_score = 7.
- 3 → styling_score = 10.

**Graceful handling:** Missing package.json or requirements.txt → skip that check, count as 0 for that check only.

---

### Signal 3.3 — Accessibility

**Check 1:**
```
Grep: pattern="aria-label|aria-describedby|aria-hidden|aria-live|role=\"" path=PROJECT_ROOT (glob=**/*.jsx,**/*.tsx,**/*.html,**/*.vue,**/*.svelte)
```

Count matches → `aria_attr_count`.

- 0 → aria_score = 0.
- 1-10 → aria_score = 3.
- 11-30 → aria_score = 5.
- 31-80 → aria_score = 7.
- 81+ → aria_score = 9.

**Check 2:**
```
Bash: cat PROJECT_ROOT/package.json 2>/dev/null | grep -E '"axe-core"|"jest-axe"|"@axe-core|"eslint-plugin-jsx-a11y"|"pa11y"'
Glob: **/*.a11y.test.*  **/a11y/**  **/accessibility/**  (search path: PROJECT_ROOT)
```

Any match → a11y_tooling = true → add +1 to aria_score (cap at 10).

**Graceful handling:** No JSX/TSX/HTML files found → aria_score = 0, note `no_ui_files: true`.

---

### Signal 3.4 — Visual Regression and UI Tests

**Check 1:**
```
Glob: **/*.stories.*  (search path: PROJECT_ROOT)
```

Count → `story_count`.

- 0 → story_score = 0.
- 1-5 → story_score = 4.
- 6-15 → story_score = 7.
- 16+ → story_score = 10.

**Check 2:**
```
Bash: cat PROJECT_ROOT/package.json 2>/dev/null | grep -E '"playwright"|"cypress"|"@playwright|"chromatic"|"percy"|"backstop"'
Glob: playwright.config.*  cypress.config.*  .percy.yml  backstop.json  (search path: PROJECT_ROOT)
```

Any match → visual_regression = true (+2 to story_score, cap at 10).

**Graceful handling:** If `no_ui_files: true` was noted in Signal 3.3, set story_score = 0.

---

### Signal 3.5 — Claude History (Design)

From cross-cutting Claude usage results (Section 5):

- `/frontend-design` command usage count → design_skill_count.
- Design-related prompts (Grep `history.jsonl` for `component|design system|storybook|tailwind|accessibility|a11y|UI`) → design_prompt_count.

**Design AI signal from history:**
- design_skill_count + design_prompt_count < 3 → design_history_score = 0.
- 3-10 → design_history_score = 3.
- 11-25 → design_history_score = 6.
- 26-50 → design_history_score = 8.
- 51+ → design_history_score = 10.

---

### Design Domain Scoring

**Structural score:**
Average of: component_score, styling_score, aria_score, story_score. All 0-10. If `project_type = "docs"`, exclude component_score, styling_score, and story_score (use aria_score alone).

**AI Integration score (70% weight from scan, 30% self-report applied later):**
design_history_score from Signal 3.5. Range 0-10.

**Scoring Rubric:**

| Band | Structural Score | Description |
|------|-----------------|-------------|
| 0-2  | No components directory; no CSS framework; 0 aria attributes; no stories. Raw HTML/CSS. |
| 3-4  | Some components (1 dir); 1 CSS framework detected; 1-10 aria attributes; 0-4 stories. |
| 5-6  | Component library with 2+ dirs; CSS framework + config file; 11-30 aria attributes; 5-15 stories. |
| 7-8  | Design system with Storybook; a11y tooling in package.json; 30-80 aria attributes; 15+ stories; E2E framework detected. |
| 9-10 | Full Storybook (16+ stories); pervasive a11y (80+ aria attrs + axe tooling); visual regression CI; 3 styling signals; Claude-assisted design in history. |

---

## Section 4: PM Domain Signals

### Signal 4.1 — Ticket References in Git Log

```
Bash: git -C PROJECT_ROOT log --oneline -200 | grep -cE "[A-Z]{2,10}-[0-9]+"
```

→ `jira_ref_count`.

```
Bash: git -C PROJECT_ROOT log --oneline -200 | grep -cE "#[0-9]+"
```

→ `github_ref_count`.

`total_ticket_refs = jira_ref_count + github_ref_count`.

- 0 → ticket_score = 0.
- 1-5 → ticket_score = 3.
- 6-15 → ticket_score = 5.
- 16-40 → ticket_score = 7.
- 41+ → ticket_score = 10.

**Graceful handling:** If `no_git: true`, set ticket_score = 0.

---

### Signal 4.2 — Requirements and Product Docs

**Check 1:**
```
Glob: **/PRD*  **/prd*  **/product-requirements*  (search path: PROJECT_ROOT)
```

**Check 2:**
```
Glob: **/requirements*.md  **/requirements*.txt  **/requirements*.rst  (search path: PROJECT_ROOT)
```

**Check 3:**
```
Glob: **/*-spec.md  **/*_spec.md  **/SPEC*.md  (search path: PROJECT_ROOT, exclude **/*.test.*, **/*.spec.ts, **/*.spec.js)
```

**Check 4:**
```
Glob: **/user-stories*  **/user_stories*  **/acceptance-criteria*  (search path: PROJECT_ROOT)
```

Count how many of checks 1-4 have at least one match → `req_doc_type_count`.

- 0 → req_doc_score = 0.
- 1 → req_doc_score = 4.
- 2 → req_doc_score = 6.
- 3 → req_doc_score = 8.
- 4 → req_doc_score = 10.

**Graceful handling:** Glob returning no results = 0 for that check. Continue.

---

### Signal 4.3 — Changelog and Release Tracking

```
Glob: CHANGELOG*  CHANGELOG.md  CHANGELOG.txt  RELEASES*  HISTORY*  HISTORY.md  (search path: PROJECT_ROOT)
```

Any found → changelog_present = true.

```
Bash: cat PROJECT_ROOT/CHANGELOG.md 2>/dev/null | grep -cE "^## \[|^## v[0-9]"
```

→ `changelog_entry_count`.

- changelog_present = false → changelog_score = 0.
- changelog_present = true and changelog_entry_count = 0 → changelog_score = 2 (stub).
- 1-5 entries → changelog_score = 5.
- 6-15 entries → changelog_score = 7.
- 16+ entries → changelog_score = 10.

**Graceful handling:** Missing CHANGELOG → changelog_score = 0. Non-standard format → changelog_entry_count may be 0; treat as stub.

---

### Signal 4.4 — MCP Integrations

```
Read: PROJECT_ROOT/.mcp.json  (if exists)
Read: ~/.claude/.mcp.json  (if exists)
```

Parse for server entries. Look for:
- `jira` in any key or value → jira_mcp = true.
- `linear` in any key or value → linear_mcp = true.
- `slack` in any key or value → slack_mcp = true.
- `github` in any key or value → github_mcp = true.
- `notion|confluence|asana|trello` in any key or value → pm_tool_mcp = true.

Count how many of the above are true → `mcp_pm_count`.

- 0 → mcp_score = 0.
- 1 → mcp_score = 5.
- 2 → mcp_score = 7.
- 3+ → mcp_score = 10.

**Graceful handling:** If `.mcp.json` does not exist at either path, set mcp_score = 0 and note `no_mcp: true`. Do not error.

---

### Signal 4.5 — Claude History (PM)

From cross-cutting Claude usage results (Section 5):

- Brainstorming prompts (Grep `history.jsonl` for `brainstorm|user story|acceptance criteria|PRD|product requirement|feature spec|roadmap`) → pm_prompt_count.
- MCP server interactions (Grep `history.jsonl` for `jira|linear|asana|notion|trello`) → pm_mcp_count.

**PM AI signal from history:**
- pm_prompt_count + pm_mcp_count < 3 → pm_history_score = 0.
- 3-10 → pm_history_score = 3.
- 11-25 → pm_history_score = 6.
- 26-50 → pm_history_score = 8.
- 51+ → pm_history_score = 10.

---

### PM Domain Scoring

**Structural score:**
Average of: ticket_score, req_doc_score, changelog_score, mcp_score. All 0-10. No exclusions for docs projects — all PM signals apply regardless of project type.

**AI Integration score (70% weight from scan, 30% self-report applied later):**
pm_history_score from Signal 4.5. Range 0-10.

**Scoring Rubric:**

| Band | Structural Score | Description |
|------|-----------------|-------------|
| 0-2  | 0 ticket refs; no requirements docs; no changelog; no MCP integrations. No process evidence. |
| 3-4  | 1-5 ticket refs; no requirements docs; changelog present but < 5 entries; no MCP. Informal process. |
| 5-6  | 6-15 ticket refs; 1 requirements doc type; 5-15 changelog entries; 0-1 MCP integration. |
| 7-8  | 16-40 ticket refs; 2 requirements doc types; 15+ changelog entries; 1-2 MCP integrations. |
| 9-10 | 41+ ticket refs; 3+ requirements doc types; 16+ changelog entries; 3+ MCP integrations; Claude-assisted PM prompts in history. |

---

## Section 5: Cross-Cutting Claude Usage Analysis

This section runs once and supplies findings to all four domain AI Integration scorers.

**Graceful handling for all checks in this section:** If any path does not exist, record `null` for that signal and continue. Different Claude Code versions may have different paths. A missing file is informative (new install or different version), not an error.

### Signal 5.1 — Claude History File

```
Read: ~/.claude/history.jsonl  (if exists)
```

If file exists and is non-empty:

**Total prompt count:**
Count newline-delimited JSON objects → `total_prompt_count`.

**Skill usage frequency:**
```
Grep: pattern="/commit" file=~/.claude/history.jsonl → commit_skill_count
Grep: pattern="/tdd" file=~/.claude/history.jsonl → tdd_skill_count
Grep: pattern="/review-pr" file=~/.claude/history.jsonl → review_pr_skill_count
Grep: pattern="/frontend-design" file=~/.claude/history.jsonl → frontend_design_skill_count
Grep: pattern="/reposition" file=~/.claude/history.jsonl → reposition_skill_count
```

**Plan mode usage:**
```
Grep: pattern="plan mode|--plan|PLAN_MODE" file=~/.claude/history.jsonl (case-insensitive) → plan_mode_count
```

**Architecture-related prompts:**
```
Grep: pattern="architect|ADR|design doc|system design|module boundary|API contract" file=~/.claude/history.jsonl (case-insensitive) → arch_prompt_count
```

**Design-related prompts:**
```
Grep: pattern="component|design system|storybook|tailwind|accessibility|a11y|UI" file=~/.claude/history.jsonl (case-insensitive) → design_prompt_count
```

**PM-related prompts:**
```
Grep: pattern="brainstorm|user story|acceptance criteria|PRD|product requirement|feature spec|roadmap" file=~/.claude/history.jsonl (case-insensitive) → pm_prompt_count
```

**MCP interactions:**
```
Grep: pattern="jira|linear|asana|notion|trello" file=~/.claude/history.jsonl (case-insensitive) → pm_mcp_count
```

**Unique projects (heuristic):**
```
Grep: pattern="\"project\":|\"cwd\":" file=~/.claude/history.jsonl → extract unique values → unique_project_count
```

If file does not exist: set all counts to 0 and record `history_file: null`.

---

### Signal 5.2 — Installed Plugins

```
Read: ~/.claude/plugins/installed_plugins.json  (if exists)
```

Parse JSON. Count entries → `installed_plugin_count`. Note plugin names in `installed_plugins` array.

If file does not exist: set `installed_plugin_count = 0`, record `plugins_file: null`.

---

### Signal 5.3 — Personal Custom Skills

```
Bash: ls ~/.claude/skills/ 2>/dev/null | wc -l
```

→ `personal_skill_count`.

If directory does not exist: set `personal_skill_count = 0`.

---

### Signal 5.4 — Project Memory Files

```
Bash: find ~/.claude/projects/ -name "*.md" -path "*/memory/*" 2>/dev/null | wc -l
```

→ `project_memory_count`.

If directory does not exist: set `project_memory_count = 0`.

---

### Signal 5.5 — Project-Level Claude Configuration

**Project skills:**
```
Glob: .claude/skills/**  (search path: PROJECT_ROOT)
```

Count files → `project_skill_count`.

**Project settings:**
```
Read: PROJECT_ROOT/.claude/settings.json  (if exists)
```

If exists and non-empty → project_settings_present = true.

If neither exists → record `project_claude_config: null`.

---

### Signal 5.6 — Custom Slash Commands

```
Glob: ~/.claude/commands/*.md  (if directory exists)
Glob: PROJECT_ROOT/.claude/commands/*.md  (if directory exists)
```

Count total files across both locations → `custom_commands_count`.

If neither directory exists → `custom_commands_count = 0`.

---

### Signal 5.7 — History Confidence

Derived from `total_prompt_count` and whether `history_file` was found:

- `HIGH` — `total_prompt_count >= 50` and `history_file` is not null
- `MEDIUM` — `total_prompt_count` is 10-49 and `history_file` is not null
- `LOW` — `total_prompt_count` is 1-9, or file found but very sparse
- `NONE` — `history_file` is null or `total_prompt_count = 0`

Record as `history_confidence`.

---

### Cross-Cutting Summary Object

Store all the above as `claude_usage`:

```json
{
  "history_file": "<path or null>",
  "total_prompt_count": 0,
  "commit_skill_count": 0,
  "tdd_skill_count": 0,
  "review_pr_skill_count": 0,
  "frontend_design_skill_count": 0,
  "reposition_skill_count": 0,
  "plan_mode_count": 0,
  "arch_prompt_count": 0,
  "design_prompt_count": 0,
  "pm_prompt_count": 0,
  "pm_mcp_count": 0,
  "unique_project_count": 0,
  "installed_plugin_count": 0,
  "installed_plugins": [],
  "personal_skill_count": 0,
  "custom_commands_count": 0,
  "project_memory_count": 0,
  "project_skill_count": 0,
  "project_settings_present": false,
  "history_confidence": "NONE"
}
```

---

## Section 6: Scoring Algorithm

### 6.1 Per-Domain Score Computation

For each domain `D` in `{development, architecture, design, pm}`:

**Structural Score (S_D):**
Compute as described in the domain's scoring section above. Range 0-10.

**Important — Adjusted averaging for docs projects:** When `project_type = "docs"`, some signals are marked `excluded_for_docs: true` (see Signal 0). When computing the structural score average for a domain, **exclude null signals from the denominator**. Only average the signals that meaningfully apply.

Example — Development domain in a docs project:
- Signal 1.1 (CLAUDE.md): 0 — included
- Signal 1.2 (Tests): null (excluded for docs) — skip
- Signal 1.3 (Git): 6 — included
- Signal 1.4 (Worktrees): 0 — included
- Signal 1.5 (CI/CD): null (excluded for docs) — skip
- Structural = average(0, 6, 0) = 2.0 (divide by 3, not 5)

**AI Integration Score from Scan (AI_scan_D):**
Compute from Claude usage signals as described in each domain's history signal. Range 0-10.

**Self-Report Score (SR_D):**
Applied later by the question bank (Task 3). Placeholder = null until question phase is complete.

**Final Competence Score:**

The weighting between structural and self-report depends on project type:

| `project_type` | Structural weight | Self-report weight | Rationale |
|----------------|------------------|--------------------|-----------|
| `"application"` | 0.60 | 0.40 | Code-heavy project — structural signals are strong evidence. |
| `"library"` | 0.60 | 0.40 | Same as application. |
| `"docs"` | 0.30 | 0.70 | Most structural signals don't apply — self-report carries the developer's actual competence. |
| `"monorepo"` | 0.60 | 0.40 | Same as application. |
| `"unknown"` | 0.50 | 0.50 | Uncertain — split evenly. |

```
Competence_D = (S_D × structural_weight) + (SR_D × self_report_weight)
```

If SR_D is null (scan-only mode, e.g. `/reposition-score`): `Competence_D = S_D`.

**Final AI Integration Score:**
```
AIIntegration_D = (AI_scan_D × 0.70) + (SR_AI_D × 0.30)
```
Where `SR_AI_D` is the self-reported AI usage portion from the question bank. If null: `AIIntegration_D = AI_scan_D`.

---

### 6.2 Gap Score and Opportunity Profile

For each domain, compute both the gap score and the opportunity profile:

```
Gap_D = clamp(Competence_D − AIIntegration_D, 0, 10)
Inverse_Gap_D = clamp(AIIntegration_D − Competence_D, 0, 10)
```

**Gap interpretation (standard case — Competence > AI Integration):**

- Gap >= 6: High REPOSITION opportunity. Structural capability exists but AI not yet leveraged.
- Gap 3-5: Moderate opportunity. Some AI usage but room to grow.
- Gap 0-2: Low gap. Competence and AI integration are close.

A high Gap with a low Competence score (< 4) = foundational work needed first, then AI integration.
A high Gap with Competence >= 6 = immediate REPOSITION candidate.

**Inverse gap interpretation (AI Integration > Competence):**

When `Inverse_Gap_D > 0`, AI integration exceeds structural competence. This is NOT "no opportunity" — it means the developer is AI-integrated but structural foundations in THIS project are weak.

- Inverse Gap >= 6: **Inverted profile.** Developer uses AI extensively but this project lacks the structural artifacts (tests, CLAUDE.md, ADRs, etc.) that make AI usage most effective. The REPOSITION opportunity is to build the structural foundation that AI can multiply.
- Inverse Gap 3-5: **AI-leading.** AI usage outpaces structural setup. Building structure will unlock more AI leverage.
- Inverse Gap 0-2: **Balanced.** Competence and AI integration are well-matched.

**Opportunity profile classification per domain:**

| Condition | Profile | Label |
|-----------|---------|-------|
| Gap >= 3 AND Competence >= 6 | `"classic"` | High competence, low AI — the textbook REPOSITION candidate |
| Gap >= 3 AND Competence < 6 | `"foundational"` | Low competence, low AI — needs skill-building before AI integration |
| Inverse_Gap >= 3 | `"inverted"` | High AI, low structural — needs structural foundations to maximise AI value |
| Gap < 3 AND Inverse_Gap < 3 | `"balanced"` | Competence and AI integration are well-matched |

Store `opportunity_profile` per domain in `SCAN_RESULTS.scores.[domain].opportunity_profile`.

---

### 6.3 Overall AI Readiness

```
AIReadiness_overall = average(AIIntegration_development, AIIntegration_architecture, AIIntegration_design, AIIntegration_pm)
```

Range 0-10. Bands:
- 0-2: Pre-AI. No meaningful integration detected across any domain.
- 3-4: Emerging. Isolated AI use in 1-2 domains.
- 5-6: Developing. AI used across most domains but inconsistently.
- 7-8: Practitioner. Regular structured AI use with skill and history evidence.
- 9-10: Advanced. Pervasive AI integration, custom skills, MCP, TDD, planning-first.

---

### 6.4 SCAN_RESULTS Output Structure

When the scan is complete, produce the following object and pass it to the calling skill:

```json
{
  "scan_metadata": {
    "project_root": "<absolute path>",
    "project_type": "application",
    "no_git": false,
    "stack_unknown": false,
    "no_ui_files": false,
    "total_source_files": 0,
    "total_content_files": 0,
    "scan_timestamp": "<ISO 8601>"
  },
  "claude_usage": { "<Signal 5 results>" },
  "domains": {
    "development": {
      "signals": {
        "claude_md_exists": false,
        "claude_md_lines": 0,
        "planning_first_rules": false,
        "tdd_rules": false,
        "arch_rules": false,
        "test_file_count": 0,
        "source_file_count": 0,
        "test_ratio": 0.0,
        "framework_detected": false,
        "conventional_commit_count": 0,
        "ai_coauthor_count": 0,
        "tdd_commit_count": 0,
        "worktree_count": 0,
        "ci_github": false,
        "ci_other": false,
        "docker_present": false,
        "deploy_config": false
      },
      "structural_score": null,
      "ai_integration_score": null
    },
    "architecture": {
      "signals": {
        "adr_file_count": 0,
        "arch_docs": false,
        "module_root_count": 0,
        "domain_boundary_count": 0,
        "contract_type_count": 0,
        "di_detected": false,
        "di_match_count": 0,
        "plan_commit_count": 0
      },
      "structural_score": null,
      "ai_integration_score": null
    },
    "design": {
      "signals": {
        "component_dir_count": 0,
        "storybook_present": false,
        "styling_signal_count": 0,
        "aria_attr_count": 0,
        "a11y_tooling": false,
        "story_count": 0,
        "visual_regression": false
      },
      "structural_score": null,
      "ai_integration_score": null
    },
    "pm": {
      "signals": {
        "jira_ref_count": 0,
        "github_ref_count": 0,
        "total_ticket_refs": 0,
        "req_doc_type_count": 0,
        "changelog_present": false,
        "changelog_entry_count": 0,
        "mcp_pm_count": 0
      },
      "structural_score": null,
      "ai_integration_score": null
    }
  },
  "scores": {
    "development":   { "structural": null, "ai_integration": null, "gap": null, "inverse_gap": null, "opportunity_profile": null },
    "architecture":  { "structural": null, "ai_integration": null, "gap": null, "inverse_gap": null, "opportunity_profile": null },
    "design":        { "structural": null, "ai_integration": null, "gap": null, "inverse_gap": null, "opportunity_profile": null },
    "pm":            { "structural": null, "ai_integration": null, "gap": null, "inverse_gap": null, "opportunity_profile": null },
    "overall_ai_readiness": null
  }
}
```

Populate all `null` values with computed results before returning. Pass `SCAN_RESULTS` to the next stage of the assessment flow.

**Field naming note:** The `scores` object above uses `"structural"` because this is the pre-question scan-only score. After the question bank applies self-report weighting (Section 6.1), the final output file (Format 2 in report-template.md) renames this field to `"competence"`:
- `SCAN_RESULTS.scores.[domain].structural` → saved as `scores.[domain].competence` in the assessment YAML
- The formula is: `competence = (structural × structural_weight) + (self_report × self_report_weight)`
- Plugin 2 files (`spiral-generator.md`, `claude-md-generator.md`) read `competence` from the saved assessment file, not `structural` from `SCAN_RESULTS` directly.
