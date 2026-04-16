# Git Hooks — Pre-Commit Hook Templates

This file is read by the deliver skill orchestrator. The orchestrator selects the
hook template matching `.reposition/equip/stack.json` and applies the install logic below.

---

## Pre-Commit Hook Templates

### TypeScript/React

```bash
#!/bin/sh
set -e

# Lint: fail on any warning
npx eslint --max-warnings 0 .

# Tests: run full suite verbosely so failures are clear
npx vitest run --reporter=verbose

# Secret guard: fail if any staged file looks like a .env file
if git diff --cached --name-only | grep -q '\.env'; then
  echo "[pre-commit] ERROR: Staged files include a .env file. Unstage it before committing."
  exit 1
fi
```

---

### Python/FastAPI

```bash
#!/bin/sh
set -e

# Lint + format check
ruff check .

# Tests: short traceback, quiet output
pytest --tb=short -q

# Secret guard
if git diff --cached --name-only | grep -q '\.env'; then
  echo "[pre-commit] ERROR: Staged files include a .env file. Unstage it before committing."
  exit 1
fi
```

---

### Go

```bash
#!/bin/sh
set -e

# Static analysis
go vet ./...

# Tests: short mode skips long-running integration tests in local commits
go test ./... -short

# Secret guard
if git diff --cached --name-only | grep -q '\.env'; then
  echo "[pre-commit] ERROR: Staged files include a .env file. Unstage it before committing."
  exit 1
fi
```

---

### Java/Spring

```bash
#!/bin/sh
set -e

# Style check (quiet mode suppresses passing output)
mvn checkstyle:check -q

# Tests
mvn test -q

# Secret guard
if git diff --cached --name-only | grep -q '\.env'; then
  echo "[pre-commit] ERROR: Staged files include a .env file. Unstage it before committing."
  exit 1
fi
```

---

### Angular

```bash
#!/bin/sh
set -e

# Lint
npx ng lint

# Unit tests (headless, single run)
npx ng test --watch=false --browsers=ChromeHeadless

# Secret guard
if git diff --cached --name-only | grep -q '\.env'; then
  echo "[pre-commit] ERROR: Staged files include a .env file. Unstage it before committing."
  exit 1
fi
```

---

### Generic

```bash
#!/bin/sh
set -e

# Secret guard (minimum safe hook for any stack)
if git diff --cached --name-only | grep -q '\.env'; then
  echo "[pre-commit] ERROR: Staged files include a .env file. Unstage it before committing."
  exit 1
fi
```

---

## Deliver Skill — Install Logic

The deliver skill orchestrator follows this sequence when applying a pre-commit hook.

### Step 1: Check for an Existing Hook

Check both conventional hook locations:

- `.githooks/pre-commit` (project-level custom hooks directory)
- `.git/hooks/pre-commit` (default git hooks directory)

### Step 2: Existing Hook Found → Suggest, Do Not Replace

If either location already has a `pre-commit` file:

1. Do **not** overwrite it.
2. Save the stack-appropriate template to:
   `.reposition/equip/suggested-pre-commit-hook`
3. Log to the user:

```
[reposition-equip] A pre-commit hook already exists at [path].
The stack-appropriate template has been saved to:
  .reposition/equip/suggested-pre-commit-hook

Review it and merge any checks you want into your existing hook manually.
```

### Step 3: No Existing Hook Found → Create

1. Create `.githooks/pre-commit` with the stack-appropriate template content.
2. Make it executable: `chmod +x .githooks/pre-commit`
3. Log to the user:

```
[reposition-equip] Pre-commit hook created at .githooks/pre-commit

To activate it, run:
  git config core.hooksPath .githooks

This tells git to look for hooks in .githooks/ instead of .git/hooks/.
Commit .githooks/pre-commit to share the hook with your team.
```

### Step 4: Add `.githooks/` to Version Control

Remind the user that `.githooks/` (unlike `.git/hooks/`) is tracked by git and
should be committed so teammates get the same hook automatically after running
`git config core.hooksPath .githooks`.

---

## Notes

- The secret guard (`.env` check) is included in every stack template. It is the
  minimum safety net. The other checks are stack-specific and can be removed if the
  project uses a different CI approach.
- For monorepos, scope the lint/test commands to the affected workspace
  (e.g., `npx eslint packages/my-app/` or `pytest packages/api/ --tb=short -q`).
- The Java/Spring `mvn test -q` step can be slow. Teams may choose to run only
  `mvn checkstyle:check -q` locally and rely on CI for tests. Document the decision
  in the project's CLAUDE.md.
