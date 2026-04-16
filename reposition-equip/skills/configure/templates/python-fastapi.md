# CLAUDE.md — Python + FastAPI Project

## About This Project
<!-- Replace with your project description -->
This is a Python API service built with FastAPI, SQLAlchemy for data access, and Pydantic for validation.
Key technologies: FastAPI, SQLAlchemy 2.0, Pydantic v2, async-first architecture.

## Development Workflow

### Planning First (Non-Negotiable)
- Use `/brainstorming` before starting any feature — explore API design approaches before committing
- Use `/writing-plans` for architecture decisions — produce endpoint list, data models, and migration plan before coding
- NEVER start coding without a defined API contract: endpoints, request/response schemas, status codes
- Ask clarifying questions about business rules and edge cases before proposing solutions
- Get plan approval before proceeding to implementation

### Test-Driven Development
- Write tests BEFORE implementation code — tests define the API contract
- Use pytest with pytest-asyncio for async endpoint tests
- Use Factory Boy for test fixtures — no hardcoded test data scattered across files
- Use `httpx.AsyncClient` for integration tests against the running app
- Test per-endpoint: happy path, validation errors (422), authentication (401/403), edge cases
- Run `pytest` after every change to verify correctness
- Invoke `/test-driven-development` when starting any feature or bugfix

### Worktree Strategy
- Use `/using-git-worktrees` for every feature — one feature per worktree, main always deployable
- Create a worktree before starting work: `git worktree add ../feature-name feature/feature-name`
- Never commit WIP to main — worktrees eliminate the temptation
- Clean up merged worktrees promptly

### Code Review Checklist
- Run `/requesting-code-review` before every pull request — no exceptions
- SQL injection prevention: ORM queries only, no raw SQL string concatenation
- Input validation: all request data flows through Pydantic models, never access raw body directly
- Auth middleware: every protected endpoint uses dependency injection for auth verification
- Proper HTTP status codes: 201 for creation, 204 for deletion, 404 for not found, 422 for validation
- Error responses follow a consistent schema across all endpoints

### Code Patterns
- Pydantic models for ALL request and response schemas — no raw dicts in/out of endpoints
- SQLAlchemy models with Alembic migrations — never modify the database schema manually
- Dependency injection for services: use FastAPI's `Depends()` for database sessions, auth, and services
- Repository pattern for data access — endpoints never query the database directly
- Async where possible: use `async def` for endpoints and I/O-bound operations
- Type hints on every function — parameters and return types, no exceptions
- Separate schemas from models: `schemas/` for Pydantic, `models/` for SQLAlchemy
- Use `Annotated` types for dependency injection: `db: Annotated[AsyncSession, Depends(get_db)]`

### Error Handling & Security
- All user input validated via Pydantic — never trust or use raw input directly
- Use SQLAlchemy ORM exclusively — never concatenate strings into SQL queries
- Secrets in `.env` file loaded via python-dotenv — never committed to version control
- CORS configured strictly: explicit allowed origins, no wildcard `*` in production
- Rate limiting on all public endpoints (slowapi or similar middleware)
- Password hashing with bcrypt — never store plaintext passwords

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
- Language: Python 3.11+
- Framework: FastAPI
- ORM: SQLAlchemy 2.0 + Alembic (migrations)
- Validation: Pydantic v2
- Testing: pytest + pytest-asyncio + httpx + Factory Boy
- Linting: Ruff + Black + isort
- Local Dev: Docker Compose for database and services
- Auth: JWT or OAuth2 (via FastAPI security utilities)

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
