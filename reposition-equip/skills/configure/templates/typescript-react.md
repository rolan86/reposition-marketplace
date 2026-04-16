# CLAUDE.md — TypeScript + React Project

## About This Project
<!-- Replace with your project description -->
This is a TypeScript SPA or Next.js application with strict type checking enabled.
Key technologies: React 18+, TypeScript strict mode, component-driven architecture.

## Development Workflow

### Planning First (Non-Negotiable)
- Use `/brainstorming` before starting any feature — explore approaches before committing to one
- Use `/writing-plans` for architecture decisions — produce a written plan and get approval before coding
- Use `/frontend-design` for UI components — generate component tree and data flow before implementation
- NEVER start coding without a component hierarchy, data flow diagram, and acceptance criteria
- Ask clarifying questions about UX requirements before proposing solutions

### Test-Driven Development
- Write tests BEFORE implementation code — tests define the component contract
- Use Vitest or Jest with React Testing Library for component tests
- Use MSW (Mock Service Worker) for API mocking — no hardcoded fetch stubs
- Test behavior, not implementation: query by role/label, never by className or test-id unless necessary
- Integration tests for user flows (page-level), unit tests for utilities and hooks
- Run `npm test` after every change to verify correctness
- Invoke `/test-driven-development` when starting any feature or bugfix

### Worktree Strategy
- Use `/using-git-worktrees` for every feature — one feature per worktree, main always deployable
- Create a worktree before starting work: `git worktree add ../feature-name feature/feature-name`
- Never commit WIP to main — worktrees eliminate the temptation
- Clean up merged worktrees promptly

### Code Review Checklist
- Run `/requesting-code-review` before every pull request — no exceptions
- Accessibility: all interactive elements have aria labels, keyboard navigation works, color contrast passes
- Bundle size: no new dependencies without justification, check impact with `npm run build`
- Type safety: zero uses of `any` — use `unknown` and narrow, or define proper types
- No `console.log` in committed code (use a logger utility or remove)
- No TODO comments without a linked ticket number

### Code Patterns
- Functional components only — no class components
- Hooks for all state and side effects (useState, useEffect, useMemo, useCallback)
- Export Props interfaces alongside components: `export interface ButtonProps { ... }`
- Collocate tests next to components: `Button.tsx` and `Button.test.tsx` in the same directory
- Error boundaries at route level to catch rendering failures gracefully
- Suspense boundaries for async data loading (React.lazy, data fetching)
- CSS Modules or Tailwind for styling — no inline styles for reusable components
- Prefer composition over prop drilling — use context or state management for deep data

### Error Handling & Security
- Sanitize all user input before rendering — never render raw HTML from external sources
- Use a sanitizer library (e.g., DOMPurify) any time HTML content must be rendered dynamically
- API keys and secrets in environment variables only (`.env.local`, never committed)
- Include CSRF tokens for all mutation requests
- Configure Content Security Policy (CSP) headers in production
- Validate and sanitize URL parameters before use

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
- Framework: React 18+ / Next.js 14+
- Testing: Vitest or Jest + React Testing Library + MSW
- Styling: Tailwind CSS or CSS Modules
- State: React Query (server state), Zustand (client state)
- Linting: ESLint + Prettier
- Build: Vite or Next.js bundler

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
