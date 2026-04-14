# reposition-assess

**REPOSITION AI Readiness Assessment** — scan your project and measure your AI integration maturity across 4 domains.

## What It Does

This Claude Code plugin scans your project files, git history, and Claude Code usage patterns, then asks targeted questions to generate a two-axis scorecard. Each domain gets two scores: **Competence** (how well you do the work today) and **AI Integration** (how much AI is embedded in your workflow). The gap between them is your opportunity.

**Domains assessed:** Development, Architecture, Design, Product Management.

**Time:** 15-30 minutes for the full assessment, 2 minutes for a quick rescore.

## Installation

First, add the REPOSITION marketplace (one-time setup):

```bash
claude plugin marketplace add /path/to/REPOSITION/developer-track/plugins
```

Then install the plugin:

```bash
claude plugin install reposition-assess
```

Uninstall:

```bash
claude plugin uninstall reposition-assess
```

## Commands

| Command | Description |
|---------|-------------|
| `/reposition-assess` | Full guided assessment: project scan, targeted questions, two-axis scorecard with recommended actions |
| `/reposition-score` | Quick rescan: updates scores from current project state without re-asking questions |
| `/reposition-export` | Re-export your last assessment as markdown or copy to clipboard |

## How It Works

1. **Scan** — Automatically scans your project: CLAUDE.md, test coverage, git patterns, CI/CD, modular structure, design system, PM practices (30 signals total)
2. **Classify** — Detects your project type (application, library, docs, monorepo) and adjusts scoring accordingly
3. **Ask** — 12 targeted questions with skip logic based on what the scan already found
4. **Score** — Generates a two-axis scorecard: Competence vs AI Integration per domain
5. **Recommend** — Identifies opportunity profiles (classic, foundational, inverted, balanced) and suggests specific next steps

## Output

Assessment saved to `.reposition/assessment-YYYY-MM-DD.md`. This file is consumed by `reposition-transform` if you choose to engage with the full transformation.

## Part of the REPOSITION Ecosystem

| Plugin | Purpose | Distribution |
|--------|---------|-------------|
| **reposition-assess** | Measure where you are | Free |
| reposition-transform | Close the gaps | Paid (consulting) |
| reposition-onboard | Learn a new codebase on day 1 | Free |

## License

Proprietary. Part of the REPOSITION Framework by Merryl DMello.
