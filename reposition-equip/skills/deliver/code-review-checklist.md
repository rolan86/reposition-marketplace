# Code Review Checklist — Stack-Specific

This file is read by the deliver skill orchestrator. The orchestrator outputs the universal
sections plus the per-stack section matching `.reposition/equip/stack.json`.

---

## Universal Checks (All Stacks)

Apply these checks to every pull request regardless of stack.

### Correctness & Safety

- [ ] All existing tests pass (CI green or local run confirmed)
- [ ] New behaviour is covered by new or updated tests
- [ ] No secrets, tokens, API keys, or passwords committed (in code, comments, or config)
- [ ] No hardcoded environment-specific values (URLs, ports, credentials) in source code
- [ ] No dependencies added without a clear justification in the PR description

### Scope & Focus

- [ ] PR addresses a single concern (feature, bug, or refactor — not mixed)
- [ ] Changed files are all related to the stated purpose
- [ ] No unrelated formatting or whitespace changes mixed in

### Convention Compliance

- [ ] Code follows project CLAUDE.md conventions (naming, structure, patterns)
- [ ] Commit messages are clear and follow the project's commit style
- [ ] New public APIs or config options are documented

---

## AI-Specific Smells (All Stacks)

Flag these patterns that commonly appear in AI-generated code.

- [ ] **Overly verbose comments** — comments that restate what the code already says
  (e.g., `// increment i by 1` above `i++`). Remove or replace with "why" comments.
- [ ] **Unnecessary abstractions** — interfaces, base classes, or helper functions
  created for a single use case with no foreseeable reuse.
- [ ] **Hallucinated imports** — imports for packages that do not exist in the project
  or were never installed. Verify each new import exists in package manifest.
- [ ] **Dead code** — unreachable branches, unused variables, functions defined but
  never called, `console.log` / `print` / `fmt.Println` debug statements left in.
- [ ] **Inconsistent naming** — mixing conventions within the same file (camelCase and
  snake_case, or abbreviations mixed with full names) introduced by the AI session.
- [ ] **Overly defensive null-checks** — cascading nil/null guards on values that are
  guaranteed non-nil by the API contract, obscuring the real logic.
- [ ] **Copy-paste test duplication** — test cases that are identical except for a
  variable name, where a table-driven or parameterised approach would be cleaner.

---

## Per-Stack Checks

### TypeScript/React

- [ ] No `any` types — use `unknown` and narrow, or define a proper type/interface
- [ ] No `as Type` casts without a comment justifying why narrowing is not possible
- [ ] Hooks rules followed: no conditional calls, no hooks inside non-component functions
- [ ] `useEffect` dependency arrays are complete (ESLint exhaustive-deps would pass)
- [ ] Component props are typed with an interface, not an inline object type
- [ ] Components have a single clear responsibility (not mixing data-fetch + render logic)
- [ ] Interactive elements are keyboard accessible (onKeyDown alongside onClick where needed)
- [ ] Images have `alt` text; form inputs have associated `<label>` elements
- [ ] No direct DOM manipulation (`document.querySelector`) inside React components
- [ ] State updates use the functional form when new state depends on previous state

---

### Python/FastAPI

- [ ] All I/O-bound route handlers are `async def`; CPU-bound work uses `run_in_executor`
- [ ] All request bodies and response models are Pydantic models (no raw `dict` returns)
- [ ] No raw SQL strings — use parameterised queries or ORM; no f-string SQL
- [ ] HTTP status codes are semantically correct: 201 on creation, 404 on not-found,
  422 on validation failure, 204 on delete with no body
- [ ] `except Exception` not used as a catch-all; specific exception types are caught
- [ ] All functions and classes have type annotations on parameters and return values
- [ ] Dependency injection via `Depends()` used for shared resources (DB sessions, auth)
- [ ] No mutable default arguments in function signatures (`def foo(items=[])`)
- [ ] Environment variables accessed via a settings class (Pydantic BaseSettings), not
  `os.environ` scattered throughout the codebase

---

### Go

- [ ] No ignored errors — `_` on error return requires a comment explaining why it is safe
- [ ] Every goroutine started in the diff has a clear, documented exit path
- [ ] No `defer` inside a loop (deferred calls run at function return, not loop end)
- [ ] `context.Context` is the first parameter on all I/O-bound functions
- [ ] Errors are wrapped with `%w` (not `%v`) when they need to be unwrapped by callers
- [ ] No `panic` in library code; panics only acceptable in `main` or init paths
- [ ] Exported types, functions, and constants have godoc comments
- [ ] Interfaces are defined in the package that uses them (consumer-side), not producer-side
- [ ] Struct fields that should not be compared use pointer receivers or are unexported

---

### Java/Spring

- [ ] Dependencies injected via constructor, not `@Autowired` on fields
- [ ] Service methods that modify state are annotated with `@Transactional`
- [ ] Controller method parameters that accept request bodies are annotated with `@Valid`
- [ ] Business logic lives in the service layer, not in controller or repository methods
- [ ] Custom exceptions extend an appropriate base in the project's exception hierarchy
- [ ] No catching `Exception` or `RuntimeException` without re-throwing or logging
- [ ] `Optional` return types used for repository `findBy*` methods; not returning null
- [ ] DTOs are Java records or have a no-arg constructor + validation annotations
- [ ] `@Async` methods are in a separate Spring bean (cannot call self-async on same bean)
- [ ] Entity relationships are fetched lazily unless a join fetch is explicitly justified

---

### Angular

- [ ] All new components declare `standalone: true`
- [ ] Component uses `OnPush` change detection strategy
- [ ] Mutable state is managed with signals (`signal()`, `computed()`) not `BehaviorSubject`
  unless RxJS interop specifically requires it
- [ ] All subscriptions are cleaned up: `takeUntilDestroyed()`, `async` pipe, or explicit
  `ngOnDestroy` unsubscribe
- [ ] Templates contain no complex logic — ternaries and method calls with side effects
  belong in the component class or a `computed()`
- [ ] Inputs typed with `input<T>()` signal (Angular 17+) where possible; model inputs
  for two-way binding
- [ ] No direct DOM access via `ElementRef.nativeElement` except for focus management
  with a documented justification
- [ ] HTTP calls go through a service, never directly in a component

---

### Generic

- [ ] Language-appropriate naming conventions followed consistently throughout the diff
- [ ] Functions have a single responsibility and are under 40 lines
- [ ] Error paths are handled explicitly — no silent failures or swallowed exceptions
- [ ] No commented-out code blocks checked in (remove or open a ticket instead)
- [ ] Configuration values (timeouts, limits, feature flags) are in config files or
  constants, not magic numbers in logic
- [ ] New external dependencies are the minimal version needed and have an active
  maintenance record
