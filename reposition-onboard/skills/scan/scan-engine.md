---
description: "Core scan engine — instructs Claude exactly what signals to collect for onboarding and how to structure the CODEBASE_MODEL output"
---

# REPOSITION Onboard Scan Engine

**Role:** You are the scan engine for the REPOSITION Onboard plugin. Execute each signal detection step in sequence. Do not prompt the user. Do not explain what you are doing. Collect all findings into a CODEBASE_MODEL JSON structure and hand control back to the scan skill.

**When to run:** Triggered by the `reposition-onboard:scan` skill.

---

## Pre-Scan: Detect Project Root

Before running any signal scans, confirm the working directory is a meaningful project root.

```
Bash: git rev-parse --show-toplevel
```

- If this succeeds, use the returned path as `PROJECT_ROOT` for all subsequent scans.
- If git is not initialised, use the current working directory and note `no_git: true` in `CODEBASE_MODEL`. Signal 4 (Git History) will be skipped.

**Empty directory check:** Run a broad Glob for source files (`**/*.ts`, `**/*.js`, `**/*.py`, `**/*.java`, `**/*.go`, `**/*.rb`, `**/*.rs`), excluding `node_modules/`, `vendor/`, `.venv/`, `dist/`, `build/`, and `target/`. Also count markdown files (`**/*.md`, same exclusions). If zero source files AND zero markdown files are found, halt immediately with:

> "This directory has no source files. Point me at a project directory."

Do not proceed past this check on an empty directory.

---

## Signal 1: Tech Stack Detection

**Purpose:** Identify the language, framework, build tool, and key dependencies.

**Step 1 — Find manifest at the project root only:**

Look for these files in the project root directory only (not recursively — ignore manifests inside `node_modules/`, subdirectories, or nested packages):

```
Read: PROJECT_ROOT/package.json (if it exists)
Read: PROJECT_ROOT/pom.xml (if it exists)
Read: PROJECT_ROOT/build.gradle (if it exists)
Read: PROJECT_ROOT/build.gradle.kts (if it exists)
Read: PROJECT_ROOT/requirements.txt (if it exists)
Read: PROJECT_ROOT/go.mod (if it exists)
Read: PROJECT_ROOT/Cargo.toml (if it exists)
Read: PROJECT_ROOT/pyproject.toml (if it exists)
```

Read the first manifest found and extract information according to these rules:

**If `package.json`:** Read the file. Extract `dependencies` and `devDependencies`. List the top 10 packages by importance (prefer runtime dependencies over dev dependencies). Determine language as `"TypeScript"` if `typescript` appears in devDependencies, otherwise `"JavaScript"`. Detect framework from dependency names (e.g. `react`, `vue`, `angular`, `express`, `next`, `nuxt`, `nest`). Detect build tool from scripts or devDependencies (e.g. `vite`, `webpack`, `rollup`, `tsc`, `esbuild`).

**If `pom.xml` or `build.gradle` / `build.gradle.kts`:** Read the file. Set language to `"Java"` (or `"Kotlin"` for `.kts`). Extract groupId and artifactId of the top dependencies. Detect framework from dependency names (e.g. `spring-boot`, `quarkus`, `micronaut`). Detect build tool as `"Maven"` or `"Gradle"`.

**If `requirements.txt` or `pyproject.toml`:** Read the file. Set language to `"Python"`. List the first 10 package entries. Detect framework from package names (e.g. `django`, `flask`, `fastapi`, `starlette`). Detect build tool as `"pip"` or `"poetry"` / `"hatch"` depending on manifest type.

**If `go.mod`:** Read the file. Set language to `"Go"`. List the first 10 `require` entries. Detect framework from module names (e.g. `gin-gonic`, `echo`, `fiber`, `chi`). Set build tool to `"go"`.

**If `Cargo.toml`:** Read the file. Set language to `"Rust"`. List dependencies from `[dependencies]` section. Detect framework from crate names (e.g. `actix-web`, `axum`, `rocket`). Set build tool to `"cargo"`.

**Step 2 — Store result:**

```json
"stack": {
  "language": "TypeScript",
  "framework": "Next.js",
  "build": "vite",
  "dependencies": ["react", "next", "axios", ...]
}
```

**If no manifest found:** Store:

```json
"stack": { "language": "unknown", "framework": "unknown", "build": "unknown", "dependencies": [] }
```

---

## Signal 2: Project Structure

**Purpose:** Classify the project layout and enumerate top-level directories.

**Step 1 — List top level:**

```
Bash: ls PROJECT_ROOT
```

Collect the names of all top-level entries as `top_level_dirs`.

**Step 2 — Count source files by extension:**

```
Glob: **/*.ts  (exclude node_modules, vendor, .venv, dist, build)
Glob: **/*.tsx  (exclude node_modules, vendor, .venv, dist, build)
Glob: **/*.js  (exclude node_modules, vendor, .venv, dist, build)
Glob: **/*.py  (exclude node_modules, vendor, .venv, dist, build)
Glob: **/*.java  (exclude node_modules, vendor, .venv, dist, build)
Glob: **/*.go  (exclude node_modules, vendor, .venv, dist, build)
Glob: **/*.rs  (exclude node_modules, vendor, .venv, dist, build)
```

Sum all results → `file_count`.

**Step 3 — Classify project type.** Apply these rules in order (first match wins):

| Condition | `type` |
|-----------|--------|
| Multiple `package.json` files found in distinct subdirectories (excluding `node_modules/`) | `"monorepo"` |
| Multiple service directories (e.g. `services/`, `apps/`) each with their own manifest | `"microservice"` |
| Single manifest, has a main export (`index.*`) but no `server.*` or `app.*` entry | `"library"` |
| Single manifest, has `src/` or `app/` with significant source files | `"monolith"` |
| None of the above | `"unknown"` |

**Step 4 — Store result:**

```json
"structure": {
  "type": "monolith",
  "top_level_dirs": ["src", "tests", "docs", "scripts"],
  "file_count": 142
}
```

---

## Signal 3: CLAUDE.md

**Purpose:** Detect whether the project has AI working instructions and extract their contents.

**Step 1 — Locate file:**

```
Glob: CLAUDE.md  (search path: PROJECT_ROOT)
Glob: .claude/CLAUDE.md  (search path: PROJECT_ROOT)
```

**Step 2 — If found:** Read the file. Extract:
- `line_count`: total number of lines.
- `sections`: all lines beginning with `## ` (section headers), collected as an array of strings.
- `rules`: all lines that begin with an imperative verb (`Use`, `Never`, `Always`, `Avoid`, `Do`, `Don't`, `Prefer`, `Run`, `Set`, `Check`, `Require`). Collect up to 20 such lines.

**Step 3 — Store result:**

```json
"claude_md": {
  "found": true,
  "line_count": 74,
  "sections": ["## Stack", "## Commands", "## Conventions"],
  "rules": ["Always run tests before committing.", "Never mutate shared state directly.", ...]
}
```

**If not found:**

```json
"claude_md": { "found": false, "line_count": 0, "sections": [], "rules": [] }
```

---

## Signal 4: Git History

**Purpose:** Understand commit volume, contributors, branch naming, and commit message conventions.

**Graceful handling:** If `no_git: true` was set in pre-scan, skip this entire signal and store `"git": null`. Do not abort.

**Step 1 — Commit log:**

```
Bash: git log --oneline -200
```

Count the number of lines returned → `commit_count`.

**Step 2 — Contributors:**

```
Bash: git log --format="%ae" | sort -u
```

Collect unique email addresses → `contributors` array.

**Step 3 — Branch naming:**

```
Bash: git branch -r
```

Examine branch names. Classify `branch_convention`:
- `"feature-slash"` if most branches use `feature/` prefix.
- `"feat-slash"` if most branches use `feat/` prefix.
- `"ticket-ref"` if branch names contain ticket patterns (e.g. `PROJ-123`).
- `"freeform"` if no clear pattern.

**Step 4 — Commit convention:**

Sample the last 50 commit messages from Step 1. Classify `commit_convention`:
- `"conventional"` if most messages match `type(scope): description` pattern (feat:, fix:, chore:, docs:, etc.).
- `"ticket-ref"` if most messages begin with a ticket identifier (e.g. `JIRA-123`, `GH-45`).
- `"freeform"` if no clear pattern.

**Step 5 — Store result:**

```json
"git": {
  "commit_count": 183,
  "contributors": ["alice@example.com", "bob@example.com"],
  "branch_convention": "feature-slash",
  "commit_convention": "conventional"
}
```

---

## Signal 5: Documentation

**Purpose:** Inventory the documentation available to a new developer.

**Step 1 — Check for standard docs files at the project root only:**

```
Read: PROJECT_ROOT/README.md (check if it exists)
Read: PROJECT_ROOT/CONTRIBUTING.md (check if it exists)
Bash: ls PROJECT_ROOT/docs 2>/dev/null
Bash: ls PROJECT_ROOT/ADR 2>/dev/null || ls PROJECT_ROOT/adr 2>/dev/null
```

Set boolean flags: `readme`, `contributing`.

**Step 2 — If `docs/` exists:**

```
Bash: ls PROJECT_ROOT/docs
```

Collect top-level entries as `docs_dir` array.

**Step 3 — If ADR directory exists (`ADR/` or `adr/`):**

```
Glob: ADR/**/*.md  or  adr/**/*.md
```

Count the number of files → store as `adrs` array of filenames (up to 20).

**Step 4 — Store result:**

```json
"docs": {
  "readme": true,
  "contributing": false,
  "docs_dir": ["api.md", "setup.md", "architecture.md"],
  "adrs": ["0001-use-postgres.md", "0002-adopt-graphql.md"]
}
```

**If `docs/` does not exist:** `"docs_dir": null`. If no ADR directory: `"adrs": null`.

---

## Signal 6: Test Patterns

**Purpose:** Detect how the project is tested and whether coverage data is available.

**Step 1 — Find test files and directories:**

```
Bash: ls -d PROJECT_ROOT/test PROJECT_ROOT/tests PROJECT_ROOT/__tests__ 2>/dev/null
Glob: **/*_test.go  (exclude node_modules, vendor)
Glob: **/*_test.py  (exclude .venv)
Glob: **/*.test.ts  (exclude node_modules, dist)
Glob: **/*.test.js  (exclude node_modules, dist)
Glob: **/*.spec.ts  (exclude node_modules, dist)
Glob: **/*.spec.js  (exclude node_modules, dist)
Glob: **/*Test.java  (exclude build, target)
Glob: **/*Spec.java  (exclude build, target)
```

Collect all matching paths as `test_dirs`.

**Step 2 — Detect test framework from config files:**

```
Glob: jest.config.js  jest.config.ts  jest.config.cjs  (search path: PROJECT_ROOT)
Glob: vitest.config.ts  vitest.config.js  (search path: PROJECT_ROOT)
Glob: karma.conf.js  karma.conf.ts  (search path: PROJECT_ROOT)
Glob: pytest.ini  (search path: PROJECT_ROOT)
Glob: pyproject.toml  (search path: PROJECT_ROOT) — check for [tool.pytest.ini_options] section
```

Also check `pom.xml` for `junit` and `build.gradle` for `junit`. Set `framework` to the first detected name (e.g. `"jest"`, `"vitest"`, `"pytest"`, `"junit"`). If none detected, set `framework` to `"unknown"`.

**Step 3 — Check for coverage:**

```
Glob: .nycrc  .nycrc.json  (search path: PROJECT_ROOT)
Glob: .coveragerc  (search path: PROJECT_ROOT)
Glob: coverage/  (search path: PROJECT_ROOT)
Grep: pattern="jacoco" file=PROJECT_ROOT/pom.xml or PROJECT_ROOT/build.gradle (if they exist)
```

If a `coverage/` directory exists, attempt to read `coverage/coverage-summary.json`. If present, extract the `total.lines.pct` value → `coverage_pct`. Otherwise set `coverage_pct: null`.

**Step 4 — Store result:**

```json
"test_patterns": {
  "framework": "jest",
  "coverage_pct": 74.3,
  "test_dirs": ["src/__tests__", "tests"]
}
```

---

## Signal 7: CI/CD

**Purpose:** Identify the CI/CD platform and pipeline definitions.

**Step 1 — Find pipeline files:**

```
Glob: .github/workflows/*.yml  (search path: PROJECT_ROOT)
Glob: .github/workflows/*.yaml  (search path: PROJECT_ROOT)
Glob: Jenkinsfile  (search path: PROJECT_ROOT)
Glob: .gitlab-ci.yml  (search path: PROJECT_ROOT)
Glob: .circleci/config.yml  (search path: PROJECT_ROOT)
```

**Step 2 — If found:** Determine `platform`:
- Files under `.github/workflows/` → `"GitHub Actions"`
- `Jenkinsfile` → `"Jenkins"`
- `.gitlab-ci.yml` → `"GitLab CI"`
- `.circleci/config.yml` → `"CircleCI"`

For GitHub Actions files, read each workflow file and extract the `name:` field at the top level → collect as `workflows` array.

For other platforms, use the filename as the workflow identifier.

**Step 3 — Store result:**

```json
"ci_cd": {
  "platform": "GitHub Actions",
  "workflows": ["CI", "Deploy", "Lint"]
}
```

**If no pipeline files found:**

```json
"ci_cd": null
```

---

## Signal 8: Entry Points

**Purpose:** Locate where the application starts, its API surface, and its CLI commands.

**Step 1 — Stack-aware detection.** Use the `stack.language` value from Signal 1 to choose the right globs:

**Node / TypeScript:**

```
Glob: src/index.ts  src/index.js  src/app.ts  src/app.js  src/main.ts  src/main.js  (search path: PROJECT_ROOT)
Glob: **/routes/**  **/controllers/**  **/routers/**  (search path: PROJECT_ROOT, exclude node_modules, dist)
```

**Python:**

```
Glob: main.py  app.py  manage.py  run.py  wsgi.py  asgi.py  (search path: PROJECT_ROOT)
Glob: **/views.py  **/urls.py  **/routers.py  **/endpoints.py  (search path: PROJECT_ROOT, exclude .venv)
```

**Java:**

```
Glob: **/Application.java  **/*Application.java  (search path: PROJECT_ROOT, exclude target, build)
Glob: **/*Controller.java  **/*Resource.java  **/*Router.java  (search path: PROJECT_ROOT, exclude target, build)
```

**Go:**

```
Glob: cmd/  main.go  (search path: PROJECT_ROOT)
Glob: **/handler*.go  **/*handler.go  (search path: PROJECT_ROOT)
```

**Rust:**

```
Glob: src/main.rs  src/lib.rs  (search path: PROJECT_ROOT)
Glob: src/bin/**/*.rs  (search path: PROJECT_ROOT)
```

**Generic fallback (for unknown or other languages):**

```
Glob: main.*  index.*  app.*  server.*  (search path: PROJECT_ROOT, exclude node_modules, dist, build)
```

**Step 2 — Classify findings:**
- `main_files`: top-level entry files (e.g. `main.ts`, `app.py`, `Application.java`).
- `api_controllers`: route/controller/view files.
- `cli_commands`: any files under `cmd/`, `cli/`, or `commands/` directories.

**Step 3 — Store result:**

```json
"entry_points": {
  "main_files": ["src/main.ts"],
  "api_controllers": ["src/routes/users.ts", "src/routes/auth.ts"],
  "cli_commands": []
}
```

---

## Signal 9: Key Abstractions

**Purpose:** Identify the core interfaces, base classes, middleware, and shared utilities that a new developer must understand.

**Step 1 — Stack-aware grep for interfaces and base classes:**

**TypeScript / JavaScript:**

```
Grep: pattern="^(export )?(interface |abstract class )" (search path: PROJECT_ROOT, file types: *.ts *.tsx, exclude node_modules, dist)
Grep: pattern="(extends |implements )" (search path: PROJECT_ROOT, file types: *.ts *.tsx, exclude node_modules, dist)
```

**Python:**

```
Grep: pattern="class .*(ABC|Base)\b" (search path: PROJECT_ROOT, file types: *.py, exclude .venv)
Grep: pattern="@abstractmethod" (search path: PROJECT_ROOT, file types: *.py, exclude .venv)
```

**Java:**

```
Grep: pattern="^(public )?(interface |abstract class )" (search path: PROJECT_ROOT, file types: *.java, exclude target, build)
Grep: pattern="(extends |implements )" (search path: PROJECT_ROOT, file types: *.java, exclude target, build)
```

**Go:**

```
Grep: pattern="type \w+ interface" (search path: PROJECT_ROOT, file types: *.go)
```

Collect unique file paths containing matches. Limit to 20 entries per list.

**Step 2 — Detect middleware:**

```
Grep: pattern="middleware|Middleware|interceptor|Interceptor|filter|Filter" (search path: PROJECT_ROOT, exclude node_modules, vendor, .venv, dist, build)
```

Collect unique file paths → `middleware` array (limit 10).

**Step 3 — Detect shared utilities:**

```
Bash: ls -d PROJECT_ROOT/utils PROJECT_ROOT/shared PROJECT_ROOT/common PROJECT_ROOT/lib PROJECT_ROOT/helpers PROJECT_ROOT/core 2>/dev/null
```

Collect which of these directories exist → `utilities` array.

**Step 4 — Store result:**

```json
"abstractions": {
  "interfaces": ["src/types/IUserRepository.ts", "src/types/IAuthService.ts"],
  "base_classes": ["src/base/BaseController.ts"],
  "middleware": ["src/middleware/auth.ts", "src/middleware/logger.ts"],
  "utilities": ["src/utils", "src/shared"]
}
```

---

## Signal 10: Environment

**Purpose:** Determine how a developer sets up and runs the project locally.

**Step 1 — Check for environment scaffolding:**

```
Glob: .env.example  .env.sample  .env.template  (search path: PROJECT_ROOT)
Glob: docker-compose.yml  docker-compose.yaml  (search path: PROJECT_ROOT)
Glob: Dockerfile  (search path: PROJECT_ROOT)
```

Set `docker: true` if a Compose file or Dockerfile is found. Set `env_example: true` if an env template file is found.

**Step 2 — Extract run instructions from README:**

If a `README.md` was found in Signal 5, read it. Search for any of these section headers (case-insensitive): `## Getting Started`, `## Setup`, `## Installation`, `## Running`, `## Quick Start`.

If a matching section is found, extract the content of that section up to the next `##` header. Store as `run_instructions` (trim to 500 characters if longer).

If no README or no matching section: `run_instructions: null`.

**Step 3 — Store result:**

```json
"environment": {
  "docker": true,
  "env_example": true,
  "run_instructions": "## Getting Started\n\n1. Copy `.env.example` to `.env`\n2. Run `docker-compose up`\n3. Visit http://localhost:3000"
}
```

---

## Output Contract

After all signals complete, assemble the full `CODEBASE_MODEL` JSON structure and return it to the scan skill:

```json
{
  "repo": "repo-name",
  "scan_date": "YYYY-MM-DD",
  "stack": {
    "language": "...",
    "framework": "...",
    "build": "...",
    "dependencies": [...]
  },
  "structure": {
    "type": "...",
    "top_level_dirs": [...],
    "file_count": 0
  },
  "claude_md": {
    "found": true,
    "line_count": 0,
    "sections": [...],
    "rules": [...]
  },
  "git": {
    "commit_count": 0,
    "contributors": [...],
    "branch_convention": "...",
    "commit_convention": "..."
  },
  "docs": {
    "readme": true,
    "contributing": false,
    "docs_dir": [...],
    "adrs": [...]
  },
  "test_patterns": {
    "framework": "...",
    "coverage_pct": null,
    "test_dirs": [...]
  },
  "ci_cd": {
    "platform": "...",
    "workflows": [...]
  },
  "entry_points": {
    "main_files": [...],
    "api_controllers": [...],
    "cli_commands": [...]
  },
  "abstractions": {
    "interfaces": [...],
    "base_classes": [...],
    "middleware": [...],
    "utilities": [...]
  },
  "environment": {
    "docker": false,
    "env_example": false,
    "run_instructions": null
  }
}
```

Set `repo` to the basename of `PROJECT_ROOT`. Set `scan_date` to today's date in `YYYY-MM-DD` format.

---

## Failure Handling

Handle each failure mode as follows. Never abort entirely unless the directory is empty.

**Not a git repo:** Set `no_git: true`, skip Signal 4, store `"git": null`, proceed with all other signals. Append a note to output: "No git history available — some features will be limited."

**No package manifest found:** Set `stack` to the unknown values, proceed with structure-based analysis for all remaining signals.

**Empty directory:** Halt immediately with the message: "This directory has no source files. Point me at a project directory." Do not produce a `CODEBASE_MODEL`.

**Partial signal failure:** If an individual signal fails (e.g. a glob returns an error, a file cannot be read), skip that signal, set its key to `null` in `CODEBASE_MODEL`, and continue with the next signal. Log the failure reason in a top-level `"errors"` array in the output.

Example error entry:

```json
"errors": [
  { "signal": "Signal 6: Test Patterns", "reason": "coverage/coverage-summary.json could not be parsed" }
]
```
