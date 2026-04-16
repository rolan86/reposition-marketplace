# REPOSITION Marketplace

Free Claude Code plugins for AI readiness assessment and developer onboarding.

## Installation

Add this marketplace to Claude Code (one-time setup):

```bash
claude plugin marketplace add rolan86/reposition-marketplace
```

Then install the plugins you want:

```bash
claude plugin install reposition-assess
claude plugin install reposition-onboard
claude plugin install reposition-equip
```

## Plugins

### reposition-assess

Scan your project and measure your AI integration maturity across 4 domains: Development, Architecture, Design, and Product Management. Get a two-axis scorecard showing Competence vs AI Integration — the gap between them is your opportunity.

**Time:** 15-30 minutes

| Command | Description |
|---------|-------------|
| `/reposition-assess` | Full assessment: project scan, targeted questions, scorecard |
| `/reposition-score` | Quick rescore from current project state |
| `/reposition-export` | Re-export last assessment as markdown |

### reposition-onboard

Learn any codebase and ship your first contribution on day 1. The plugin scans the project (tech stack, structure, patterns, conventions, git history, tests, CI/CD), then offers two ways to learn:

- **Guided Tour** — Walk through architecture, patterns, conventions, then ship a contribution
- **Open Explore** — Quick orientation, then you drive with free-form Q&A

**Time:** 30-60 minutes

| Command | Description |
|---------|-------------|
| `/reposition-onboard` | Full onboarding: scan, learn, contribute |
| `/reposition-ask` | Quick Q&A about the codebase (reuses previous scan) |
| `/reposition-contribute` | Jump to guided contribution |

### reposition-equip

Deliver a complete AI-assisted development toolkit configured for your stack. Detects your tech stack, asks a few questions about your team's conventions, then generates a configured CLAUDE.md, methodology guide, prompt library, code review checklists, git hooks, test scaffolding, and workflow guides. No assessment, no scores — just practical tools.

**Time:** 5-10 minutes

| Command | Description |
|---------|-------------|
| `/reposition-equip` | Full toolkit delivery: stack detection, configure conventions, deliver everything |
| `/reposition-inventory` | Check what was delivered, verify status, regenerate missing pieces |

## Uninstall

```bash
claude plugin uninstall reposition-assess
claude plugin uninstall reposition-onboard
claude plugin uninstall reposition-equip
```

To remove the marketplace entirely:

```bash
claude plugin marketplace remove reposition
```

## License

Proprietary. Part of the REPOSITION Framework by Merryl DMello.
