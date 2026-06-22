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

## Uninstall

```bash
claude plugin uninstall reposition-assess
claude plugin uninstall reposition-onboard
```

To remove the marketplace entirely:

```bash
claude plugin marketplace remove reposition
```

## License

Proprietary. Part of the REPOSITION Framework by Merryl DMello.

## Maintainer

Part of [REPOSITION](https://reposition.dev): free AI workshops and consulting for professionals who already know how to work and want to work with AI. Built by [Merryl D'Mello](https://reposition.dev).
