---
name: assess
description: "Full AI readiness assessment — scans your project, asks targeted questions, and generates a two-axis scorecard across PM, Architecture, Design, and Development"
---

# REPOSITION AI Readiness Assessment

You are running the REPOSITION AI Readiness Assessment. This is a guided flow — follow each stage in order. Do not skip stages.

## Stage 1: Permission & Context

Greet the developer and explain what will happen:

"I'll scan your project files, git history, and Claude Code usage to assess your AI readiness across 4 domains: Product Management, Architecture, Design, and Development.

I can also look across your other Claude Code projects for a fuller picture. Allow cross-project scan? [Y/n]"

Collect:
- Project path (current working directory)
- Tech stack detection: Read package.json, requirements.txt, go.mod, Cargo.toml, or pyproject.toml to determine stack
- Git remote info: run `git remote -v`
- Cross-project consent (Y/n) — if yes, scan signals from `~/.claude/` apply; if no, limit to current project only

## Stage 2: Silent Scan

Follow the instructions in `scan-engine.md` to scan the project.

Show progress to the user as you complete each check:

```
Classifying project type... ✓ ([application/library/docs/monorepo/unknown])
Scanning project structure... ✓
Analyzing CLAUDE.md... ✓ (found, XX lines) [or ✗ (not found)]
Checking test coverage patterns... ✓
Reading git history (last 100 commits)... ✓
Analyzing Claude Code usage history... ✓
Detecting installed plugins... ✓
Checking skill configurations... ✓
```

Store all raw signal data in the SCAN_RESULTS structure defined in scan-engine.md for use in subsequent stages.

## Stage 3: Scan Results Preview

Present a compact summary of scan findings BEFORE asking any questions. This proves the plugin's value immediately — the developer sees real data about their project.

Format:
```
📊 Scan Complete — Here's what I found:

Project type: [application/library/docs/monorepo/unknown]
[If project_type = "docs": "Note: This is a docs/content project. Structural signals
that require source code (tests, CI, components) are excluded from scoring."]

Development: CLAUDE.md [✓/✗] | Tests [✓/✗] ([coverage]%) | Worktrees [✓/✗] | TDD pattern [✓/✗]
Architecture: ADRs [✓/✗] | Plan-before-code [✓/✗] | Modular structure [✓/✗]
Design: Component library [✓/✗] | Design system [✓/✗] | UI tests [✓/✗]
PM: Ticket refs in commits [✓/✗] | Requirements docs [✓/✗]

[If Claude history was accessible:]
Claude Usage: [N] prompts across [N] projects
Skills used: [list top skills with counts]
Planning pattern: [N]% of sessions start with planning prompts

I need to ask you a few questions to complete the picture...
```

Fill in actual values from SCAN_RESULTS. Use ✓ for detected, ✗ for not detected. For docs projects, mark excluded signals with `—` (dash) instead of ✗ to indicate they were not scored rather than failed.

## Stage 4: Targeted Questions

Follow `question-bank.md` to determine which questions to ask.

Rules:
- Apply skip logic from question-bank.md Section 4: do NOT ask questions the scan already answered with high confidence
- Ask ONE question at a time — present the question, wait for the user's answer, then ask the next
- Use multiple choice format (A/B/C/D) as defined in the question bank
- Track all answers in QUESTION_RESULTS for scoring
- Count questions asked — stop after 12 maximum (though typically 5-8)

After all questions: "Thanks! Let me calculate your scores..."

## Stage 5: Two-Axis Scorecard

Combine SCAN_RESULTS + QUESTION_RESULTS using the scoring algorithm from `scan-engine.md` Section 6 to calculate final scores.

Present the scorecard using `report-template.md` Format 1 (the terminal-formatted table with gap bars).

Highlight the biggest opportunity — the domain with the highest gap where competence is also >= 6. Explain why this represents the best ROI for REPOSITION.

## Stage 6: Evidence Breakdown

For each of the 4 domains, present:
- Competence score with evidence items marked 🔍 (from scan) or 💬 (from questions)
- AI Integration score with evidence items
- Gap analysis: 1-2 sentences about what's being left on the table
- Top 3 recommended actions for this domain

Use the evidence format from `report-template.md` Format 1.

Ask: "Would you like me to explain any of these scores in more detail?"

## Stage 7: Export & Next Steps

Offer four actions:

1. **Save report** — Write `.reposition/assessment-YYYY-MM-DD.md` using `report-template.md` Format 2. Create the `.reposition/` directory if it doesn't exist. Always recommend this.

2. **Export for facilitator** — Copy compact summary using `report-template.md` Format 3 to clipboard (or display for manual copy).

3. **Canvas 7 integration** (optional) — If the user wants to use the HTML Canvas 7 assessment grid, offer to display the scores in a format they can manually enter into Canvas 7. Only mention if relevant (workshop context).

4. **Next steps** — Check if reposition-transform plugin is installed (look for it in `~/.claude/plugins/installed_plugins.json`). If installed: "Run `/reposition-start` to begin closing these gaps." If not installed: "To start closing these gaps, the REPOSITION Transformation plugin provides guided CLAUDE.md generation, spiral path planning, and 3-3-3 review cycles. Ask your workshop facilitator for access."
