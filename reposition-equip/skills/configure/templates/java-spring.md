# CLAUDE.md — Java + Spring Boot Project

## About This Project
<!-- Replace with your project description -->
This is a Java API service built with Spring Boot, Spring Data JPA for persistence, and Jakarta Validation.
Key technologies: Java 21+, Spring Boot 3.x, Spring Data JPA, Flyway migrations, constructor-based DI.

## Development Workflow

### Planning First (Non-Negotiable)
- Use `/brainstorming` before starting any feature — explore API design approaches before committing
- Use `/writing-plans` for architecture decisions — produce endpoint list, entity model, and migration plan before coding
- NEVER start coding without a defined API contract: endpoints, DTOs, status codes, entity relationships
- Ask clarifying questions about transactional boundaries and domain invariants before proposing solutions
- Get plan approval before proceeding to implementation

### Test-Driven Development
- Write tests BEFORE implementation code — tests define the service contract
- Use JUnit 5 with `@SpringBootTest` for integration tests and plain unit tests for services
- Use Mockito for mocking dependencies in unit tests — prefer constructor injection for testability
- Use `MockMvc` or `WebTestClient` for controller tests against the full Spring context
- Use Testcontainers for integration tests that need a real database
- Test per-endpoint: happy path, validation errors (400), authentication (401/403), not found (404)
- Run `./mvnw test` (or `./gradlew test`) after every change to verify correctness
- Invoke `/test-driven-development` when starting any feature or bugfix

### Worktree Strategy
- Use `/using-git-worktrees` for every feature — one feature per worktree, main always deployable
- Create a worktree before starting work: `git worktree add ../feature-name feature/feature-name`
- Never commit WIP to main — worktrees eliminate the temptation
- Clean up merged worktrees promptly

### Code Review Checklist
- Run `/requesting-code-review` before every pull request — no exceptions
- SQL injection prevention: Spring Data repositories and JPQL only, no raw string concatenation in queries
- Input validation: all request DTOs annotated with Jakarta Validation (`@NotNull`, `@Size`, `@Valid`)
- Transactional boundaries: `@Transactional` on service methods, never on controllers or repositories
- Proper HTTP status codes: 201 for creation, 204 for deletion, 404 for not found, 400 for validation
- No field injection (`@Autowired` on fields) — use constructor injection exclusively
- Exception handling via `@ControllerAdvice` with a consistent error response schema

### Code Patterns
- DTOs for ALL request and response payloads — never expose entities directly to controllers
- Entity classes with JPA annotations — Flyway for all schema migrations, never modify the schema manually
- Constructor injection for all Spring beans — no `@Autowired` on fields, final fields enforced
- Service layer for business logic — controllers delegate to services, services delegate to repositories
- Repository pattern via Spring Data JPA interfaces — custom queries use `@Query` with JPQL
- Records for immutable DTOs where applicable: `public record CreateUserRequest(@NotBlank String name, @Email String email) {}`
- Optional for nullable return values — never return null from service methods
- Separate packages by feature, not by layer: `user/`, `order/`, `payment/` each with their own controller, service, repository

### Error Handling & Security
- All user input validated via Jakarta Validation annotations on DTOs — never trust raw input
- Use Spring Data JPA or JPQL exclusively — never concatenate strings into SQL
- Secrets in `application.yml` loaded via environment variable placeholders: `${DB_PASSWORD}`
- Spring Security for authentication and authorization — configure SecurityFilterChain explicitly
- CORS configured strictly: explicit allowed origins, no wildcard `*` in production
- Rate limiting via Spring Cloud Gateway or Bucket4j middleware on public endpoints
- Password hashing with BCryptPasswordEncoder — never store plaintext passwords
- CSRF protection enabled for browser clients, disabled only for stateless API-only services

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
- Language: Java 21+
- Framework: Spring Boot 3.x
- Persistence: Spring Data JPA + Hibernate
- Migrations: Flyway
- Validation: Jakarta Validation (Bean Validation 3.0)
- Testing: JUnit 5 + Mockito + MockMvc + Testcontainers
- Linting: Checkstyle + SpotBugs + Error Prone
- Build: Maven (mvnw) or Gradle (gradlew)
- Auth: Spring Security (JWT or OAuth2)

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
