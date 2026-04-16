# Prompt Library — Stack-Specific Curated Prompts

This file is read by the deliver skill orchestrator. The orchestrator selects the section
matching the `stack` value from `.reposition/equip/stack.json` and delivers it to the project.

---

## TypeScript/React

### Feature Development

```
Implement [feature name] in [component/module]. Use TypeScript strict mode.
Define props with an interface, derive state types from the interface where possible.
Write a Vitest unit test for each exported function and a React Testing Library test
for each user-facing behaviour before touching implementation code.
```

> Why this works: Forces type-first thinking before any implementation. Tests written
> against the interface catch regressions when internals change and make Claude's
> output verifiable immediately.

### Bug Fixing

```
Debug [symptom] in [component/hook/module]. Check [likely areas: prop drilling,
stale closures, missing dependency arrays, or incorrect key props].
Write a failing Vitest/RTL test that reproduces the bug, then fix it so the test passes.
Do not change any behaviour outside the scope of this bug.
```

> Why this works: The regression test requirement stops Claude from applying broad
> "cleanup" changes that can introduce new issues. Naming likely areas narrows the
> search space to common React/TS failure modes.

### Refactoring

```
Refactor [target: component/hook/utility] to [goal: e.g., extract custom hook,
split into smaller components, remove prop drilling via context].
Do not change any observable behaviour. Run the existing test suite after each
individual change and fix failures before continuing.
```

> Why this works: The incremental test-check loop keeps refactors safe. Claude tends
> to make sweeping changes; this prompt enforces a step-by-step discipline.

### Code Review

```
Review this diff against our CLAUDE.md conventions. Flag:
- Missing or incorrect TypeScript types (especially `any` usage)
- Hooks rule violations (conditional calls, missing deps)
- Components that cross single-responsibility boundaries
- Missing accessibility attributes (aria-*, role, alt)
- Tests that mock implementation details instead of behaviour
Provide a severity (error / warning / suggestion) for each finding.
```

> Why this works: Severity tiers let the author triage findings. Stack-specific
> checks surface issues that generic linters miss (e.g. hooks-order problems).

### Test Writing

```
Write Vitest unit tests for [module/function] and React Testing Library tests for
[component]. Cover: happy path, edge cases (empty/null/boundary values), and error
states. Use `userEvent` for interactions, query by role or label (not test-ids unless
unavoidable). Mock external dependencies at the module boundary.
```

> Why this works: RTL best-practice guidance (role queries, userEvent) produces tests
> that survive implementation refactors. Module-boundary mocking prevents brittle
> internal coupling.

---

## Python/FastAPI

### Feature Development

```
Implement a new endpoint at [path] with method [GET/POST/PUT/DELETE].
Define Pydantic v2 request and response models in [models module].
Write tests first using httpx.AsyncClient against the FastAPI test client.
Include input validation, appropriate HTTP status codes, and docstrings for
OpenAPI schema generation.
```

> Why this works: Pydantic-first design catches schema issues before any business
> logic is written. httpx.AsyncClient tests exercise the full request/response cycle
> including middleware.

### Bug Fixing

```
Debug [symptom] in [module/router]. Check [likely areas: async/await missing,
Pydantic validation not triggered, dependency injection order, ORM session handling].
Write a failing pytest test that reproduces the bug before fixing it.
Do not change behaviour outside this scope.
```

> Why this works: Python async bugs are subtle; naming the common failure modes helps
> Claude look in the right places. The regression-first approach prevents silent
> re-introduction.

### Refactoring

```
Refactor [target] to [goal: e.g., extract service layer, replace raw SQL with
SQLAlchemy ORM, move validation into Pydantic model].
Do not change API contracts or observable behaviour.
Run `pytest --tb=short -q` after each step and fix failures before continuing.
```

> Why this works: pytest after each step prevents accumulated breakage. API contract
> preservation is explicit so Claude does not silently rename fields or change status
> codes.

### Code Review

```
Review this diff against our CLAUDE.md conventions. Flag:
- Missing `async`/`await` on I/O-bound functions
- Pydantic models not used for request/response validation
- Raw SQL strings (SQL injection risk)
- Incorrect HTTP status codes (e.g., 200 on creation, missing 422 on validation error)
- Broad `except Exception` blocks hiding errors
- Missing type annotations on function signatures
Provide severity (error / warning / suggestion) for each finding.
```

> Why this works: FastAPI-specific checks (async correctness, status codes) catch
> issues that Python linters do not surface. Explicit SQL-injection flag keeps
> security top of mind.

### Test Writing

```
Write pytest tests for [module] using Factory Boy fixtures for test data.
Cover: happy path, Pydantic validation errors (expect 422), business logic edge cases,
and database rollback on failure. Use `pytest-asyncio` for async routes.
Mark slow tests with `@pytest.mark.slow` so they can be skipped in fast runs.
```

> Why this works: Factory Boy fixtures produce realistic data and decouple tests from
> schema changes. Explicit 422 coverage ensures validation is actually enforced.

---

## Go

### Feature Development

```
Implement [feature] in package [pkg]. Define types in [types.go or domain file].
Write table-driven tests in [feature]_test.go before writing implementation.
Return errors as values (never panic in library code). Add godoc comments to all
exported identifiers.
```

> Why this works: Table-driven tests are idiomatic Go and scale easily to new cases.
> Error-as-value reminder prevents Claude from using panic where errors belong.

### Bug Fixing

```
Debug [symptom] in [package/function]. Check [likely areas: nil pointer dereference,
goroutine leak, incorrect error wrapping with %w, off-by-one in slice operations].
Write a failing test that reproduces the bug. Fix it. Verify with `go test ./... -run TestFoo`.
```

> Why this works: Go bugs concentrate in a small set of patterns; naming them focuses
> diagnosis. `-run TestFoo` verifies exactly the regression case was fixed.

### Refactoring

```
Refactor [target] to [goal: e.g., extract interface, reduce package coupling,
replace global state with dependency injection].
Do not change exported API signatures without explicit approval.
Run `go test ./...` and `go vet ./...` after each step.
```

> Why this works: `go vet` catches issues the compiler misses (printf format
> mismatches, unreachable code). Exported-API freeze prevents accidental breaking
> changes.

### Code Review

```
Review this diff against our CLAUDE.md conventions. Flag:
- Ignored errors (err assigned to _ or not checked)
- Goroutines started without a clear exit path (potential leak)
- `defer` inside loops (deferred until function return, not loop iteration)
- Missing context propagation on I/O calls
- Unexported types used in exported function signatures
Provide severity (error / warning / suggestion) for each finding.
```

> Why this works: These are the Go-specific issues most likely to cause production
> incidents. Generic linters catch style; this prompt catches correctness.

### Test Writing

```
Write table-driven tests for [function/package] in [file]_test.go.
Each table entry: name, input, expected output, expected error.
Use testify/assert for assertions. Cover nil inputs, zero values, boundary conditions.
Add a benchmark ([BenchmarkFoo]) if the function is performance-sensitive.
```

> Why this works: Table-driven tests are reviewable and extensible. Explicit nil/zero
> coverage addresses Go's most common edge-case class.

---

## Java/Spring

### Feature Development

```
Implement [feature] as a Spring service + REST controller. Define the request/response
DTOs using Java records. Annotate the controller with proper @RequestMapping,
@Valid on request bodies, and @ResponseStatus on creation endpoints.
Write JUnit 5 tests using @SpringBootTest (integration) and Mockito (unit) before
implementing the service.
```

> Why this works: Records as DTOs reduce boilerplate and are immutable by default.
> @Valid enforcement ensures Bean Validation runs. Test-first at both layers catches
> wiring errors and logic errors separately.

### Bug Fixing

```
Debug [symptom] in [class/layer]. Check [likely areas: missing @Transactional,
LazyInitializationException from detached entity, bean injection order, @Async
without executor config].
Write a failing test that reproduces the issue before fixing it.
```

> Why this works: Spring bugs concentrate in transaction and DI wiring; naming them
> focuses Claude's diagnosis. A failing test proves the fix without relying on
> manual testing.

### Refactoring

```
Refactor [target] to [goal: e.g., extract interface, move business logic from
controller to service, replace field injection with constructor injection].
Do not change public API signatures or REST contracts.
Run `./mvnw test -q` after each step.
```

> Why this works: Constructor injection reminder addresses a common Spring anti-pattern.
> Explicit contract-freeze prevents silent REST breakage.

### Code Review

```
Review this diff against our CLAUDE.md conventions. Flag:
- Field injection (@Autowired on fields) instead of constructor injection
- @Transactional missing on service methods that modify state
- Catching generic Exception or RuntimeException and swallowing it
- Missing @Valid on controller request body parameters
- Business logic implemented in controller methods
Provide severity (error / warning / suggestion) for each finding.
```

> Why this works: These are the Spring anti-patterns that cause the most production
> incidents. Catching them in review prevents them from becoming technical debt.

### Test Writing

```
Write JUnit 5 tests for [class].
- Unit tests: use Mockito to mock dependencies, test each public method independently.
- Integration tests: use @SpringBootTest + @AutoConfigureMockMvc, test full request
  cycle for each endpoint.
Cover: happy path, validation failure (expect 400), not-found (expect 404),
and any business rule violations.
```

> Why this works: Separating unit and integration tests by annotation keeps the fast
> feedback loop intact. Explicit 400/404 coverage ensures error handling is real.

---

## Angular

### Feature Development

```
Implement [feature] as a standalone Angular component with signals for state.
Define the component's public API (inputs/outputs) before writing the template.
Use OnPush change detection. Write a Jasmine/Karma spec for each input/output
interaction and each computed signal before implementing the component body.
```

> Why this works: Signals-first state keeps change detection efficient. OnPush forces
> explicit state management. Test-first against the public API prevents testing
> implementation internals.

### Bug Fixing

```
Debug [symptom] in [component/service/pipe]. Check [likely areas: signal not
updating view because mutation used instead of set(), change detection not running,
async pipe subscription not cleaned up, missing inject() in constructor].
Write a failing spec that reproduces the bug. Fix it.
```

> Why this works: Angular 17+ signal bugs are a new failure class; naming them helps
> Claude look in the right place. A spec proves the fix.

### Refactoring

```
Refactor [target] to [goal: e.g., convert to standalone component, replace
Subject+BehaviorSubject with signals, extract shared logic into inject()-based
service].
Do not change the component's public selector or input/output names.
Run `ng test --watch=false` after each step.
```

> Why this works: Selector/input freeze prevents accidental template breakage in
> parent components. Test after each step catches Angular DI wiring errors early.

### Code Review

```
Review this diff against our CLAUDE.md conventions. Flag:
- Components not declared as standalone: true
- State managed with BehaviorSubject when a signal would suffice
- Change detection not set to OnPush for leaf components
- Subscriptions not cleaned up (missing takeUntilDestroyed or async pipe)
- Template logic that belongs in the component class (complex expressions in *ngIf/[class])
Provide severity (error / warning / suggestion) for each finding.
```

> Why this works: Modern Angular best practices (standalone, signals, OnPush) are not
> enforced by the compiler yet; this review prompt fills that gap.

### Test Writing

```
Write Jasmine specs for [component/service/pipe].
- Component specs: use TestBed, query by role or aria-label (not CSS class),
  trigger signals via fixture.detectChanges().
- Service specs: inject via TestBed, mock HTTP with HttpClientTestingModule.
Cover: initial render, each input combination, each output emission,
and error/loading states.
```

> Why this works: Role-based queries survive template refactors. Signal-aware
> detectChanges() pattern is required for correct Angular 17+ testing.

---

## Generic

### Feature Development

```
Implement [feature] in [file/module]. Before writing any code:
1. Define the function/method signature and document its contract (inputs, outputs,
   side effects, error conditions) in a comment.
2. Write at least one test that will fail before implementation.
3. Implement the minimum code to pass the test.
4. Refactor if needed, keeping tests green.
```

> Why this works: Contract-first comment forces explicit thinking about edge cases.
> TDD loop prevents over-engineering.

### Bug Fixing

```
Debug [symptom] in [file/module]. 
1. Write a test that reproduces the bug (it should fail now).
2. Identify the root cause with a one-line explanation in a comment.
3. Fix the minimal code needed to make the test pass.
4. Verify no other tests break.
```

> Why this works: Steps 1-4 create an auditable fix trail. Root-cause comment
> prevents the same bug re-appearing under a different symptom.

### Refactoring

```
Refactor [target] to [goal].
Do not change observable behaviour (same inputs → same outputs, same side effects).
Run the test suite after each logical change and fix failures before continuing.
Stop and ask if any change would require modifying a public interface.
```

> Why this works: "Stop and ask" on interface changes prevents silent API breakage
> that downstream callers won't discover until runtime.

### Code Review

```
Review this diff. Flag:
- Missing tests for new behaviour
- Functions longer than 40 lines (single responsibility violation)
- Hardcoded values that should be constants or config
- Credentials, tokens, or secrets in code or comments
- Dead code (unreachable branches, unused variables)
Provide severity (error / warning / suggestion) for each finding.
```

> Why this works: Language-agnostic checks that apply universally. The 40-line
> heuristic is a practical proxy for complexity.

### Test Writing

```
Write tests for [module/function] using the project's existing test framework.
Cover: happy path with representative inputs, boundary values (zero, null, empty,
maximum), and at least one error/exception path.
Each test should have a descriptive name that reads as a sentence: "should [behaviour]
when [condition]".
```

> Why this works: Descriptive names double as living documentation. Boundary + error
> coverage catches the cases that happy-path tests miss.
