# CLAUDE.md — <!-- Replace with your project description -->

## About This Project
<!-- Replace with your project description -->
This project uses <!-- replace with your primary language and framework -->.
Key technologies: <!-- list your main stack, e.g. language, framework, database, testing tools -->.

## Development Workflow

### Planning First (Non-Negotiable)
- Use `/brainstorming` before starting any feature — explore approaches before committing to one
- Use `/writing-plans` for architecture decisions — produce a written plan and get approval before coding
- NEVER start coding without a clear problem statement, proposed approach, and acceptance criteria
- Ask clarifying questions about requirements before proposing solutions
- Document decisions in comments or a `decisions/` folder so future contributors understand context

### Test-Driven Development
- Write tests BEFORE implementation code — tests define the contract for your module or function
- Use your project's test framework — <!-- replace with: e.g. pytest, Vitest, go test, RSpec -->
- Mock external dependencies (databases, APIs, file system) in unit tests — no real I/O in fast tests
- Test behavior, not implementation: assert outcomes, not internal method calls
- Structure tests at two levels: unit tests for individual functions, integration tests for user flows
- Run your test suite after every change to verify correctness
- Invoke `/test-driven-development` when starting any feature or bugfix

### Worktree Strategy
- Use `/using-git-worktrees` for every feature — one feature per worktree, main always deployable
- Create a worktree before starting work: `git worktree add ../feature-name feature/feature-name`
- Never commit WIP to main — worktrees eliminate the temptation
- Clean up merged worktrees promptly

### Code Review Checklist
- Run `/requesting-code-review` before every pull request — no exceptions
- Check for unhandled errors and edge cases — fail loudly with clear messages, not silently
- Verify no secrets, credentials, or environment-specific values are hardcoded
- Confirm all new public functions and modules have documentation comments
- No TODO comments without a linked ticket number

### Code Patterns
- Structure code in clear modules with single responsibilities — one concept per file
- Keep functions small and focused: if a function needs a long comment to explain what it does, split it
- Prefer explicit over implicit — name variables and functions so the code reads like prose
- Collocate tests next to the code they test where your project convention allows
- Handle errors explicitly at each layer — don't let exceptions bubble unhandled to the top
- Avoid global mutable state; pass dependencies explicitly (dependency injection)
- Follow the conventions already established in this codebase

### Error Handling & Security
- Validate and sanitize all external input before processing — never trust data from users or external APIs
- Store secrets and API keys in environment variables only — never commit credentials to version control
- Log errors with enough context to diagnose without exposing sensitive data
- Fail fast: surface errors early in the call chain rather than returning partial or silently wrong results
- Review dependencies for known vulnerabilities before adding them to the project

### Code Conventions
<!-- CUSTOMIZE:naming -->
<!-- CUSTOMIZE:error_handling -->
<!-- CUSTOMIZE:pr_process -->
<!-- CUSTOMIZE:logging -->
<!-- CUSTOMIZE:custom -->

### Git Discipline
- Conventional commits required: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`
- One logical change per commit with a clear message
- Squash-merge feature branches into main
- Use feature branches: `feature/`, `fix/`, `refactor/` prefixes

## Tech Stack
<!-- Replace each line with your actual stack -->
- Language: <!-- e.g. TypeScript, Python, Go, Ruby -->
- Framework: <!-- e.g. React, FastAPI, Gin, Rails -->
- Testing: <!-- e.g. Vitest, pytest, go test -->
- Linting/Formatting: <!-- e.g. ESLint + Prettier, Ruff + Black, golangci-lint -->
- Build/Runtime: <!-- e.g. Vite, uv, Docker -->

## Skills Reference
| Skill | When to Use |
|---|---|
| `/brainstorming` | Before starting any feature — explore approaches |
| `/writing-plans` | For architecture decisions — produce a plan before code |
| `/subagent-driven-development` | When you have a plan ready to execute |
| `/test-driven-development` | Before writing implementation code — tests first |
| `/requesting-code-review` | Before every pull request — comprehensive review |
| `/using-git-worktrees` | Before starting feature work — isolate in a worktree |
| `/systematic-debugging` | When encountering bugs — diagnose before fixing |
| `/dispatching-parallel-agents` | When you have 2+ independent tasks |
