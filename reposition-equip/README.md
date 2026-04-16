# reposition-equip

**REPOSITION Toolkit Accelerator** — deliver a complete AI-assisted development toolkit configured for your stack — no scores, just tools.

## What It Does

You just started a new project or joined a team without a development standard. Instead of figuring out conventions from scratch, you install this plugin and run it once. It detects your stack, asks a few quick questions about your team's conventions, then delivers a fully configured toolkit: CLAUDE.md (AI coding conventions), methodology, prompt library, code review checklists, git hooks, test scaffolding, and workflow guides. No assessment, no scores — just practical tools ready to use.

**Time:** 10-15 minutes for a full toolkit delivery. Use `/reposition-inventory` anytime to check status or regenerate missing pieces.

## Installation

First, add the REPOSITION marketplace (one-time setup):

```bash
claude plugin marketplace add /path/to/REPOSITION/developer-track/plugins
```

Then install the plugin:

```bash
claude plugin install reposition-equip
```

Uninstall:

```bash
claude plugin uninstall reposition-equip
```

## Commands

| Command | Description |
|---------|-------------|
| `/reposition-equip` | Full toolkit delivery: detect stack, configure conventions, deliver everything |
| `/reposition-inventory` | Check what was delivered, verify status, regenerate missing pieces |

## How It Works

### 1. Detect Your Stack (automatic)

The plugin identifies your tech stack from manifest files (package.json, requirements.txt, pom.xml, go.mod, Cargo.toml) or guides you through a quick Q&A if needed.

### 2. Configure Conventions (5 quick questions)

- Naming convention (camelCase, snake_case, etc.)
- Error handling approach (exceptions, error returns, etc.)
- PR review process (single approval, two reviewers, etc.)
- Logging strategy (structured logging, levels, etc.)
- Documentation preference (inline, separate docs, etc.)

### 3. Deliver Toolkit

Generates and places all toolkit files in your project:

| File | Location | Purpose |
|------|----------|---------|
| CLAUDE.md | Project root | AI coding conventions configured for your stack and team |
| .mcp.json | Project root | MCP server configuration (GitHub integration) |
| .githooks/pre-commit | Project root | Pre-commit hook with lint + test + .env guard |
| Test config | Project root | Test runner configuration + example test |
| methodology.md | .reposition/equip/ | AI-assisted development methodology |
| prompt-library.md | .reposition/equip/ | Curated prompts for your stack |
| code-review-checklist.md | .reposition/equip/ | Stack-specific code review checklist |
| skills-reference.md | .reposition/equip/ | Claude Code skills mapping for your workflow |
| agentic-workflows.md | .reposition/equip/ | Agent workflow guide and patterns |
| worktree-strategy.md | .reposition/equip/ | Git worktree workflow best practices |

### 4. Start Coding

All tools are ready. Use `/reposition-inventory` anytime to check status or regenerate missing pieces.

## Works Standalone

No setup required. No team lead configuration needed. Run `/reposition-equip` in any project and you get a complete toolkit.

If you later use `reposition-transform`, it will enhance and personalize these files further. But equip works perfectly on its own.

## Output

| File | Purpose | Committed to git? |
|------|---------|-------------------|
| `.reposition/equip-config.json` | Your stack and convention choices | No |
| `CLAUDE.md` | AI coding conventions | Yes |
| `.mcp.json` | MCP configuration | Yes |
| `.githooks/` | Git hooks | Yes |
| `.reposition/equip/` | Toolkit documentation | No |

## Part of the REPOSITION Ecosystem

| Plugin | Purpose | Distribution |
|--------|---------|-------------|
| **reposition-equip** | Get tools and methodology (this plugin) | Free |
| reposition-assess | Measure your team's AI integration maturity | Free |
| reposition-onboard | Learn any codebase and ship on day 1 | Free |
| reposition-transform | Deep skill building with personalized guidance | Paid (consulting) |

**The workflow:** Start with `reposition-equip` to get equipped → use `reposition-assess` to measure → use `reposition-onboard` for new team members → engage `reposition-transform` for advanced personalization.

## License

Proprietary. Part of the REPOSITION Framework by Merryl DMello.
