# CLAUDE.md — Go API Project

## About This Project
<!-- Replace with your project description -->
This is a Go HTTP API service using the standard library or a lightweight router (chi/gin).
Key technologies: Go 1.22+, net/http or chi router, interface-driven design, table-driven tests.

## Development Workflow

### Planning First (Non-Negotiable)
- Use `/brainstorming` before starting any feature — explore API design approaches before committing
- Use `/writing-plans` for architecture decisions — produce package structure, interface definitions, and API contract before coding
- NEVER start coding without defined interfaces, package boundaries, and endpoint contracts
- Ask clarifying questions about concurrency requirements and error semantics before proposing solutions
- Get plan approval before proceeding to implementation

### Test-Driven Development
- Write tests BEFORE implementation code — tests define the handler contract
- Use `go test` with table-driven tests as the default testing pattern
- Use `testify` for assertions and test suites where they improve readability
- Use `httptest` for handler tests — create real HTTP requests against handlers
- Test per-handler: valid input, invalid input, authentication, error paths
- Run `go test ./...` after every change to verify correctness
- Invoke `/test-driven-development` when starting any feature or bugfix

### Worktree Strategy
- Use `/using-git-worktrees` for every feature — one feature per worktree, main always deployable
- Create a worktree before starting work: `git worktree add ../feature-name feature/feature-name`
- Never commit WIP to main — worktrees eliminate the temptation
- Clean up merged worktrees promptly

### Code Review Checklist
- Run `/requesting-code-review` before every pull request — no exceptions
- Goroutine safety: no shared mutable state without `sync.Mutex`, `sync.RWMutex`, or channels
- Context propagation: every handler and service method accepts `context.Context` as first parameter
- Error wrapping: all errors wrapped with context via `fmt.Errorf("operation: %w", err)`
- Resource cleanup: every opened resource has a corresponding `defer Close()` call
- No goroutine leaks: every spawned goroutine has a clear termination path

### Code Patterns
- Interface-driven design: define interfaces where they are consumed, not where they are implemented
- Small packages with clear boundaries — one responsibility per package
- Accept interfaces, return structs — keep dependencies abstract, return concrete types
- Error wrapping with context: `fmt.Errorf("getUserByID(%d): %w", id, err)` at every call boundary
- Table-driven tests for all non-trivial functions — structured test cases with name, input, expected output
- Context propagation through all call chains — never drop context
- Graceful shutdown: listen for OS signals, drain connections, close resources in order
- Constructor functions: `NewService(deps) *Service` pattern for initializing structs with dependencies

### Error Handling & Security
- Use `sql.Named` or parameterized queries exclusively — never concatenate strings into SQL
- Validate all input at the handler boundary — never pass unvalidated data to service layer
- Secrets loaded from environment variables — never hardcoded or committed
- TLS required in production — no plaintext HTTP in deployed environments
- Rate limiting middleware on all public-facing endpoints
- No sensitive data in log output — redact tokens, passwords, PII before logging

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
- Language: Go 1.22+
- Router: net/http (stdlib) or chi router
- Database: sqlx or pgx for PostgreSQL access
- Testing: go test + testify + httptest
- Linting: golangci-lint (gofmt, govet, errcheck, staticcheck)
- Deployment: Docker (multi-stage build)
- Observability: structured logging (slog or zerolog)

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
