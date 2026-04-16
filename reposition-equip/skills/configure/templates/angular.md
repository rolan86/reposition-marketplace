# CLAUDE.md — Angular Project

## About This Project
<!-- Replace with your project description -->
This is an Angular single-page application with standalone components and strict type checking.
Key technologies: Angular 17+, TypeScript strict mode, standalone components, signals, RxJS.

## Development Workflow

### Planning First (Non-Negotiable)
- Use `/brainstorming` before starting any feature — explore component architecture before committing
- Use `/writing-plans` for architecture decisions — produce component tree, routing structure, and state management plan before coding
- Use `/frontend-design` for UI components — generate component hierarchy and data flow before implementation
- NEVER start coding without a component hierarchy, service boundaries, and route structure defined
- Ask clarifying questions about UX requirements and data flow before proposing solutions

### Test-Driven Development
- Write tests BEFORE implementation code — tests define the component contract
- Use Karma + Jasmine (Angular default) or Jest for unit and component tests
- Use `TestBed` for component tests — configure providers and imports for isolated testing
- Use Cypress or Playwright for end-to-end tests of critical user flows
- Use `HttpTestingController` for mocking HTTP calls — no hardcoded fetch stubs
- Test components: rendering, input/output bindings, user interactions, template conditionals
- Run `ng test` after every change to verify correctness
- Invoke `/test-driven-development` when starting any feature or bugfix

### Worktree Strategy
- Use `/using-git-worktrees` for every feature — one feature per worktree, main always deployable
- Create a worktree before starting work: `git worktree add ../feature-name feature/feature-name`
- Never commit WIP to main — worktrees eliminate the temptation
- Clean up merged worktrees promptly

### Code Review Checklist
- Run `/requesting-code-review` before every pull request — no exceptions
- Accessibility: all interactive elements have aria labels, keyboard navigation works, color contrast passes
- Bundle size: no new dependencies without justification, check impact with `ng build --stats-json`
- Type safety: zero uses of `any` — use `unknown` and narrow, or define proper interfaces
- Change detection: OnPush strategy on all components, signals preferred over manual change detection
- Memory leaks: all subscriptions cleaned up via `takeUntilDestroyed()`, `async` pipe, or `DestroyRef`
- No `console.log` in committed code — use a logging service or remove

### Code Patterns
- Standalone components exclusively — no NgModules for new components
- Signals for reactive state: `signal()`, `computed()`, `effect()` preferred over BehaviorSubject for local state
- RxJS for async streams (HTTP, WebSocket, complex event chains) — signals for synchronous reactive state
- Smart/dumb component pattern: smart components manage state and inject services, dumb components receive inputs and emit outputs
- Lazy-loaded routes for feature modules — never eager-load large feature areas
- Injectable services with `providedIn: 'root'` for singletons, component-level providers for scoped state
- Typed reactive forms: `FormGroup<T>` with explicit control types, never use `FormGroup` without type parameter
- Content projection (`ng-content`) and template outlets for reusable layout components
- Functional route guards and resolvers — no class-based guards

### Error Handling & Security
- Sanitize all user input before rendering — Angular's built-in sanitization handles most cases
- Never use `bypassSecurityTrustHtml()` unless content is from a trusted source and verified
- API keys and secrets in environment files only (`environment.ts` / `environment.prod.ts`, never committed)
- HTTP interceptors for auth tokens, error handling, and retry logic — centralized, not per-service
- Configure Content Security Policy (CSP) headers in production
- Validate and sanitize route parameters before use in service calls

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
- Language: TypeScript (strict mode enabled)
- Framework: Angular 17+ (standalone components)
- Testing: Karma + Jasmine or Jest + Angular Testing Library + Cypress/Playwright
- State: Signals (local), NgRx or NgRx SignalStore (global)
- Styling: Angular Material, Tailwind CSS, or SCSS Modules
- HTTP: Angular HttpClient with typed interceptors
- Linting: ESLint + @angular-eslint + Prettier
- Build: Angular CLI (ng build)

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
