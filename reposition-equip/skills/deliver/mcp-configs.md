# MCP Server Configuration Templates

This file is read by the deliver skill orchestrator. The orchestrator checks the project
state and applies the appropriate configuration sections.

---

## GitHub MCP Server

Used when a `.git/` directory exists and a remote origin is configured.

### Configuration Block

Add to `.mcp.json` in the project root:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-token>"
      }
    }
  }
}
```

### Token Setup

1. Go to GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. Create a token scoped to this repository with permissions:
   - Contents: Read and write
   - Pull requests: Read and write
   - Issues: Read and write
   - Metadata: Read-only (required)
3. Replace `<your-token>` in the config with the generated token
4. Do not commit `.mcp.json` if it contains the token — add it to `.gitignore`
   or use an environment variable reference instead (see below)

### Token via Environment Variable (Recommended)

Set the token in your shell profile (`~/.zshrc` or `~/.bashrc`):

```bash
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_your_token_here"
```

Then reference it in `.mcp.json` without the literal value:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
      }
    }
  }
}
```

---

## Deliver Skill — Merge / Create Behaviour

The deliver skill orchestrator follows this logic when applying MCP configuration:

### Check 1: Does `.mcp.json` already exist?

- **Yes, and `github` key is already present** → Skip. Log:
  `[reposition-equip] .mcp.json already has github server configured — skipping.`
- **Yes, but `github` key is absent** → Merge the github block into the existing file.
  Preserve all other server configurations. Log:
  `[reposition-equip] Added github MCP server to existing .mcp.json`
- **No** → Create `.mcp.json` with the github server block only.
  Log: `[reposition-equip] Created .mcp.json with github MCP server config.`

### Check 2: Does `.git/config` have a remote origin?

- **No remote origin** → Skip GitHub MCP server config entirely. Log:
  `[reposition-equip] No git remote detected — skipping GitHub MCP server.`

### Check 3: Is `.mcp.json` in `.gitignore`?

- If the token is stored inline (not via env var), warn the user:
  `[reposition-equip] WARNING: .mcp.json may contain a token. Add it to .gitignore.`

---

## Other Useful MCP Servers

These are **not auto-configured** by the deliver skill. They are listed here for
developers to add manually based on their workflow needs. The equip skill's
`methodology.md` output may reference these when relevant.

### Filesystem MCP Server

Provides Claude with structured read/write access to local directories.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
    }
  }
}
```

Use when: working on large codebases where Claude needs to traverse directories
not already open in the editor context.

---

### PostgreSQL MCP Server

Provides Claude with read-only access to a PostgreSQL database for schema introspection
and query assistance.

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:password@localhost:5432/dbname"
      }
    }
  }
}
```

Use when: the project uses PostgreSQL and Claude needs to reason about schema,
write migrations, or debug query issues.

---

### Brave Search MCP Server

Provides Claude with real-time web search for documentation lookups and library research.

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "<your-brave-api-key>"
      }
    }
  }
}
```

Use when: the project uses rapidly evolving dependencies where Claude's training
cutoff may produce outdated API advice.

---

### Slack MCP Server

Provides Claude with access to Slack channel history for context on team decisions.

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "<your-slack-bot-token>",
        "SLACK_TEAM_ID": "<your-team-id>"
      }
    }
  }
}
```

Use when: team discussions about architectural decisions happen in Slack and should
inform Claude's recommendations.

---

### Memory MCP Server

Provides Claude with a persistent knowledge graph across sessions for long-running projects.

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

Use when: the project spans many Claude sessions and you want Claude to retain
context about decisions, patterns, and known issues across conversations.
