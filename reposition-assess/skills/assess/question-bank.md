---
description: "Question bank for Stage 4 of the REPOSITION assessment — targeted questions that fill gaps the scan could not determine"
---

# REPOSITION Question Bank

**Role:** You are the Stage 4 interviewer. By this point, `scan-engine.md` has already produced `SCAN_RESULTS`. Your job is to ask only the questions where the scan returned low-confidence or no data. Never ask about something the scan already resolved at high confidence.

**How to use this file:**
1. Evaluate each question's skip condition against `SCAN_RESULTS` before presenting it.
2. If the skip condition is met, record the scan score at 100% weight for that signal and move on.
3. Present exactly one question at a time. Wait for the user's answer before continuing.
4. Never exceed 12 questions in a single session. Aim for 5-8.
5. After all questions are answered (or skipped), hand results to the score skill via `QUESTION_RESULTS`.

---

## Section 1: Question Design Principles

| Principle | Rule |
|-----------|------|
| Scan-first | Only ask what the scan cannot determine. If a signal has HIGH confidence from the scan (contribution >= 7 / 10), skip the corresponding question. |
| One at a time | Present a single question, wait for the answer, then continue. Never batch questions. |
| Multiple choice preferred | Use A/B/C/D options wherever possible. Scale questions (1-5) are a last resort. |
| Hard cap | Maximum 12 questions per session. If the scan was thorough, expect 5-8. |
| Skip aggressively | When in doubt, trust the scan. A missed question is better than a redundant one. |
| No leading | Do not hint at which answer scores higher. Present options in neutral order. |

---

## Section 2: Competence Questions

These questions cover the four REPOSITION domains: PM, Architecture, Design, and Development. Each question targets a competence sub-signal that is invisible to static file scanning.

---

### PM Domain

#### Q-PM-1: Requirements Intake

**Question text:**
> "How do requirements typically reach you before you start work?"

**Options:**

| Option | Text |
|--------|------|
| A | Structured tickets in Jira, Linear, or a similar tracker — with acceptance criteria attached |
| B | Slack messages, emails, or verbal conversation — I have to extract the requirements myself |
| C | Meetings or calls where we discuss what's needed — I take my own notes |
| D | A mix of all of the above — depends on the team or project |

**Scores:**

| Option | Score |
|--------|-------|
| A | 8 |
| D | 6 |
| B | 4 |
| C | 3 |

**Skip condition:**
Skip this question if `SCAN_RESULTS` contains any of:
- `mcp_servers` includes a Jira or Linear MCP server (found in `.mcp.json`), AND that MCP signal score contribution >= 7
- `claude_usage.slash_commands` shows `/plan` or `/spec` used 5+ times (suggests structured requirements handling)

If skipped, use the scan's `pm.ai_integration_score` at 100% weight for this sub-signal.

---

#### Q-PM-2: Work Prioritisation

**Question text:**
> "When your plate is full and new work arrives, how do you decide what to work on next?"

**Options:**

| Option | Text |
|--------|------|
| A | A sprint planning process — the team decides together during a ceremony |
| B | I estimate effort and negotiate priority with stakeholders myself |
| C | The product manager or team lead decides and assigns it to me |
| D | Ad-hoc — whoever asks loudest or whatever arrived first |

**Scores:**

| Option | Score |
|--------|-------|
| A | 8 |
| B | 7 |
| C | 5 |
| D | 3 |

**Skip condition:**
Skip this question if:
- `SCAN_RESULTS.pm.structural_score` >= 7 (high-confidence PM signal from scan artifacts)
- OR `claude_usage.slash_commands` includes `/plan` used 5+ times combined with structured commit messages referencing sprint/milestone

If skipped, use scan's `pm.structural_score` at 100% weight.

---

### Architecture Domain

#### Q-ARCH-1: Architecture Decision Process

**Question text:**
> "When your team needs to make a significant architecture decision — e.g. choosing a database, splitting a service — what does that process look like?"

**Options:**

| Option | Text |
|--------|------|
| A | Formal ADR (Architecture Decision Record) — written, reviewed, and committed to the repo |
| B | Team review or RFC process — written proposal, async or sync review, then decided |
| C | Solo judgment call — I decide based on my experience and implement |
| D | Ad-hoc — we discuss briefly and start building, no written record |

**Scores:**

| Option | Score |
|--------|-------|
| A | 9 |
| B | 7 |
| C | 5 |
| D | 3 |

**Skip condition:**
Skip this question if:
- `SCAN_RESULTS.architecture.signals.adr_file_count` >= 3 (scan found an ADR directory with 3+ files)

If skipped, use `architecture.structural_score` contribution from signal 2.1 at 100% weight.

---

#### Q-ARCH-2: Stack Experience

**Question text:**
> "How long have you been working with your current primary technology stack (language, framework, infrastructure)?"

**Options:**

| Option | Text |
|--------|------|
| A | Less than 1 year |
| B | 1-3 years |
| C | 3-5 years |
| D | 5 or more years |

**Scores:**

| Option | Score |
|--------|-------|
| D | 9 |
| C | 7 |
| B | 5 |
| A | 3 |

**Skip condition:**
Skip this question if:
- `SCAN_RESULTS.architecture.signals.stack_unknown` is false AND `architecture.structural_score` >= 7 (stack is clearly identified and the structural score is high-confidence)

If skipped, use `architecture.structural_score` at 100% weight for this sub-signal.

---

### Design Domain

#### Q-DESIGN-1: UI/UX Process

**Question text:**
> "When you build a user-facing feature, how does the UI/UX side get handled?"

**Options:**

| Option | Text |
|--------|------|
| A | We have a design system — components are pre-built and I assemble from the system |
| B | A designer hands off complete specs (Figma or similar) and I implement them |
| C | I design and build — no dedicated designer, I own the full user experience |
| D | Code-first — I build what works and refine the look later, no formal process |

**Scores:**

| Option | Score |
|--------|-------|
| A | 8 |
| B | 7 |
| C | 5 |
| D | 3 |

**Skip condition:**
Skip this question if any of:
- `SCAN_RESULTS.design.signals.storybook_present` = true (Storybook detected, implies design system usage)
- `claude_usage.slash_commands` shows `/frontend-design` used 1+ times

If skipped, use `design.structural_score` signal contribution at 100% weight.

---

#### Q-DESIGN-2: User Testing

**Question text:**
> "How do you validate that a feature actually works well for users before shipping?"

**Options:**

| Option | Text |
|--------|------|
| A | Automated tests (e.g. Playwright, Cypress) combined with manual review |
| B | I test every feature with real users or internal stakeholders before merging |
| C | I test big or risky features manually — smaller changes ship without user testing |
| D | I rarely do formal user testing — I rely on production feedback |

**Scores:**

| Option | Score |
|--------|-------|
| A | 9 |
| B | 7 |
| C | 5 |
| D | 2 |

**Skip condition:**
Skip this question if:
- `SCAN_RESULTS.design.signals.e2e_tests_present` = true AND e2e test file count >= 5 (scan found substantial end-to-end test suite)

If skipped, use the scan's e2e signal contribution at 100% weight for this sub-signal.

---

### Development Domain (ask only when scan had gaps)

These questions are only presented when the scan returned low-confidence results for the corresponding development signal. If the scan's structural score for the relevant sub-signal is >= 7, skip the question.

#### Q-DEV-1: Test Writing Habits

**Question text:**
> "What best describes how you write tests?"

**Options:**

| Option | Text |
|--------|------|
| A | TDD — I write a failing test first, then implement until it passes |
| B | After implementation — I write tests once the feature works |
| C | Sometimes — on complex or risky code, but not consistently |
| D | Rarely — I rely on manual testing or leave tests to QA |

**Scores:**

| Option | Score |
|--------|-------|
| A | 9 |
| B | 6 |
| C | 4 |
| D | 2 |

**Skip condition:**
Skip this question if ALL of:
- `SCAN_RESULTS.development.signals.claude_md.tdd_rules` = true (CLAUDE.md contains TDD rules)
- `SCAN_RESULTS.development.signals.tdd_commit_score` >= 7 (git history shows test-before-impl pattern)

Both conditions must hold. If only one holds, still ask the question to resolve ambiguity.

If skipped, use the combined scan TDD signal at 100% weight.

---

#### Q-DEV-2: Code Review Process

**Question text:**
> "How does code review work on your team or in your workflow?"

**Options:**

| Option | Text |
|--------|------|
| A | PR reviews are required before merge — human reviewers must approve |
| B | I use AI-assisted review (e.g. Claude, GitHub Copilot review) as the primary check |
| C | Reviews are optional — I or teammates sometimes review but it is not enforced |
| D | No code review — I merge directly after manual testing |

**Scores:**

| Option | Score |
|--------|-------|
| A | 8 |
| B | 7 |
| C | 5 |
| D | 2 |

**Skip condition:**
Skip this question if:
- `SCAN_RESULTS.development.signals.plugins_installed` includes `pr-review-toolkit` (scan found the PR review plugin installed)
- OR `claude_usage.slash_commands` shows `/review-pr` used 3+ times

If skipped, use dev AI integration signal contribution at 100% weight.

---

## Section 3: AI Integration Questions

These questions apply across all domains. Ask them when the scan's `claude_usage` signals were inconclusive — i.e. the `.claude/` directory was missing, command history was sparse, or MCP configuration was absent.

**When to ask Section 3 questions:** Present Section 3 questions only if `SCAN_RESULTS.claude_usage.history_confidence` is `LOW` or `NONE`. If `MEDIUM` or `HIGH`, skip all Section 3 questions and rely on scan data.

---

#### Q-AI-1: Requirements and Story Writing with AI

**Question text:**
> "Do you use AI (e.g. Claude, ChatGPT) to help write requirements, user stories, or acceptance criteria?"

**Options:**

| Option | Text |
|--------|------|
| A | Every time — AI is part of my standard requirements workflow |
| B | Sometimes — for complex features or when I need to structure my thinking |
| C | Never — I write requirements manually |

**Scores:**

| Option | Score |
|--------|-------|
| A | 9 |
| B | 5 |
| C | 1 |

**Skip condition:**
Skip if:
- `SCAN_RESULTS.claude_usage.slash_commands` shows `/spec` or `/brainstorm` used 5+ times

If skipped, use the scan's `pm.ai_integration_score` at 100% weight.

---

#### Q-AI-2: Planning Before Coding

**Question text:**
> "Before you start coding a feature, how do you plan it out?"

**Options:**

| Option | Text |
|--------|------|
| A | I already use AI for planning — I work through the design with AI before touching code |
| B | I plan by hand — diagrams, notes, or a written spec — then code |
| C | I sometimes plan for bigger features but often jump straight to code |
| D | I start coding and plan as I go |

**Scores:**

| Option | Score |
|--------|-------|
| A | 9 |
| B | 5 |
| C | 3 |
| D | 1 |

**Skip condition:**
Skip if any of:
- `SCAN_RESULTS.claude_usage.slash_commands` shows `/brainstorm` or `/plan` used 5+ times
- `SCAN_RESULTS.architecture.signals.plan_commit_count` >= 5 (git history shows plan/spec commits)
- `SCAN_RESULTS.development.signals.claude_md.planning_first` = true AND `plan_commit_count` >= 3

If skipped, use scan's `architecture.ai_integration_score` contribution at 100% weight.

---

#### Q-AI-3: AI-Assisted Code Review

**Question text:**
> "Do you use AI to review your code before submitting a PR or merging?"

**Options:**

| Option | Text |
|--------|------|
| A | Always — every PR gets an AI review pass |
| B | Sometimes — on larger or riskier changes |
| C | Never — I rely solely on human reviewers or manual inspection |

**Scores:**

| Option | Score |
|--------|-------|
| A | 9 |
| B | 5 |
| C | 1 |

**Skip condition:**
Skip if:
- `claude_usage.slash_commands` shows `/review-pr` used 3+ times
- OR `SCAN_RESULTS.development.signals.plugins_installed` includes `pr-review-toolkit`

If skipped, use the scan's dev AI integration score contribution at 100% weight.

---

#### Q-AI-4: Custom Prompts and Workflow Automation

**Question text:**
> "Have you built any custom prompts, slash commands, or AI scripts specifically for your own workflow?"

**Options:**

| Option | Text |
|--------|------|
| A | Yes — I have created custom skills, scripts, or slash commands that I use regularly |
| B | Yes — I have a set of go-to prompts I reuse, but nothing formally packaged |
| C | No — I use AI tools as-is, with standard prompts |

**Scores:**

| Option | Score |
|--------|-------|
| A | 9 |
| B | 6 |
| C | 1 |

**Skip condition:**
Skip if:
- `SCAN_RESULTS.claude_usage.custom_commands_count` >= 2 (scan found 2+ custom slash commands in `.claude/commands/`)
- OR scan found installed plugins beyond the standard set in `package.json` or `.claude/`

If skipped, use the scan's cross-cutting AI integration signal at 100% weight.

---

## Section 4: Skip Logic Reference

Use this table as a quick-reference checklist before presenting any question. Evaluate each row against `SCAN_RESULTS` in order.

| Question | Skip If (scan signal) | Scan condition threshold |
|----------|----------------------|--------------------------|
| Q-PM-1: Requirements intake | MCP config includes Jira/Linear server | Signal score contribution >= 7 |
| Q-PM-1: Requirements intake | `/plan` or `/spec` slash command usage | >= 5 uses in history |
| Q-PM-2: Prioritisation | `pm.structural_score` | >= 7 |
| Q-ARCH-1: Architecture decisions | ADR directory file count | >= 3 files found |
| Q-ARCH-2: Stack experience | Stack identified AND `architecture.structural_score` | >= 7 |
| Q-DESIGN-1: UI/UX process | Storybook detected | Any Storybook config found |
| Q-DESIGN-1: UI/UX process | `/frontend-design` slash command | >= 1 use in history |
| Q-DESIGN-2: User testing | E2E test suite detected | >= 5 e2e test files |
| Q-DEV-1: Test writing | CLAUDE.md TDD rules AND tdd_commit_score | Both must be >= 7 |
| Q-DEV-2: Code review | `pr-review-toolkit` plugin installed | Plugin present in scan |
| Q-DEV-2: Code review | `/review-pr` slash command | >= 3 uses in history |
| Q-AI-1: Requirements with AI | `/spec` or `/brainstorm` command | >= 5 uses in history |
| Q-AI-2: Planning with AI | `/brainstorm` or `/plan` command | >= 5 uses in history |
| Q-AI-2: Planning with AI | Plan/spec commits in git history | `plan_commit_count` >= 5 |
| Q-AI-2: Planning with AI | CLAUDE.md planning rules AND plan commits | planning_first=true AND plan_commit_count >= 3 |
| Q-AI-3: AI code review | `/review-pr` command OR pr-review-toolkit | >= 3 uses OR plugin present |
| Q-AI-4: Custom prompts | Custom slash commands count | >= 2 custom commands found |
| Q-AI-4: Custom prompts | Non-standard installed plugins | Any additional plugins detected |

**General rule:** If any scan signal's confidence for a sub-signal is HIGH (score contribution >= 7 out of 10), skip the corresponding question regardless of whether a specific row above matches.

---

## Section 5: Score Integration

### Weights by signal type

| Signal type | Scan weight | Self-report weight | Condition |
|-------------|-------------|-------------------|-----------|
| Competence questions | 60% | 40% | When scan data exists for that signal |
| AI integration questions | 70% | 30% | When scan data exists for that signal |
| AI integration questions | 0% | 100% | When scan has NO data for that signal |
| Skipped questions | 100% | 0% | Scan had high confidence — question was skipped |

### Integration formula

```
final_signal_score = (scan_score * scan_weight) + (question_score * question_weight)
```

**Example — test writing (Q-DEV-1), scan data exists:**
- `scan_score` = 6.5 (from signals 1.1 tdd_rules + 1.3 tdd_commit_score)
- `question_score` = 9 (user answered "TDD — failing test first")
- `scan_weight` = 0.60, `question_weight` = 0.40
- `final_signal_score` = (6.5 × 0.60) + (9 × 0.40) = 3.90 + 3.60 = **7.50**

**Example — AI for requirements (Q-AI-1), scan had no data:**
- `scan_score` = 0 (no `.claude/` directory found)
- `question_score` = 5 (user answered "Sometimes")
- `scan_weight` = 0.00, `question_weight` = 1.00
- `final_signal_score` = (0 × 0.00) + (5 × 1.00) = **5.00**

**Example — Q-ARCH-1 skipped (ADR directory found with 4 files):**
- `scan_score` = 8 (adr_file_count = 4, adr_score = 7 + arch_docs bonus)
- No question was asked
- `final_signal_score` = 8 × 1.00 = **8.00**

### Output contract

After all questions are answered or skipped, produce a `QUESTION_RESULTS` object with this shape:

```json
{
  "pm": {
    "requirements_intake": { "answered": true, "option": "A", "score": 8 },
    "prioritisation": { "answered": false, "skipped": true, "skip_reason": "pm.structural_score >= 7", "scan_score": 7.5 }
  },
  "architecture": {
    "decision_process": { "answered": true, "option": "B", "score": 7 },
    "stack_experience": { "answered": true, "option": "C", "score": 7 }
  },
  "design": {
    "ui_ux_process": { "answered": false, "skipped": true, "skip_reason": "storybook_present=true", "scan_score": 8.0 },
    "user_testing": { "answered": true, "option": "C", "score": 5 }
  },
  "development": {
    "test_writing": { "answered": false, "skipped": true, "skip_reason": "tdd_rules=true AND tdd_commit_score=8", "scan_score": 8.0 },
    "code_review": { "answered": true, "option": "A", "score": 8 }
  },
  "ai_integration": {
    "requirements_with_ai": { "answered": true, "option": "B", "score": 5 },
    "planning_with_ai": { "answered": false, "skipped": true, "skip_reason": "plan_commit_count=6", "scan_score": 7.0 },
    "ai_code_review": { "answered": true, "option": "A", "score": 9 },
    "custom_prompts": { "answered": true, "option": "B", "score": 6 }
  },
  "total_questions_asked": 6,
  "total_questions_skipped": 4
}
```

Pass `QUESTION_RESULTS` alongside `SCAN_RESULTS` to the score skill. The score skill applies the weights above to produce each domain's final score.
