# Worktree Strategy

Git worktrees let you check out multiple branches of the same repository into separate directories simultaneously. With AI-assisted development, this is not a convenience — it is a discipline.

---

## Why Worktrees Matter With AI

When AI works inside your main checkout, it has access to everything: in-progress features, half-merged branches, files from other tasks. This creates context contamination — AI making decisions based on code that does not belong to the task at hand.

Worktrees solve this by giving each feature its own isolated directory. AI working in a worktree only sees what belongs to that branch. Main stays clean. Features stay separate.

A secondary benefit: you can run agents in different worktrees simultaneously without them interfering with each other.

---

## Setup

Create a worktree for a new feature:

```bash
# Create a new branch and worktree in one step
git worktree add ../feature-name feature/feature-name

# Or create from an existing branch
git worktree add ../feature-name feature/existing-branch
```

This creates `../feature-name` as a separate directory with its own working tree. Your original checkout is untouched.

List active worktrees:

```bash
git worktree list
```

---

## The Pattern

1. Start a feature: create a worktree
2. Do all work — AI sessions, commits, tests — inside the worktree directory
3. When the feature is complete and reviewed, merge it
4. Remove the worktree

Main always stays at a deployable state. You never have to stash, reset, or context-switch mid-task.

```
main/           ← always clean, always deployable
../auth-refactor/   ← worktree for auth-refactor branch
../payment-api/     ← worktree for payment-api branch
```

Run `/using-git-worktrees` to automate this setup.

---

## Parallel Work

Worktrees enable genuine parallelism when combined with parallel agents:

1. Create two worktrees for two independent features
2. Dispatch one agent per worktree (via `/dispatching-parallel-agents`)
3. Each agent operates in its isolated directory with no knowledge of the other
4. Review outputs independently, merge independently

This is the fastest path to working on multiple features simultaneously without coordination overhead or merge conflicts between in-progress work.

**Constraint:** this only works for truly independent features. If feature B needs code from feature A, you cannot parallelize them — feature B's worktree must branch from feature A's branch, and they must be sequenced.

---

## Cleanup

After merging a feature:

```bash
# Remove the worktree directory and deregister it from git
git worktree remove ../feature-name

# If the worktree has uncommitted changes git will refuse; force if you're sure:
git worktree remove --force ../feature-name

# Prune stale worktree entries (for worktrees removed manually)
git worktree prune
```

Do not leave old worktrees around. They accumulate, consume disk space, and create confusion about what is active.

---

## Integration With Claude Code

Run `/using-git-worktrees` to have Claude Code:
- Create the worktree with the correct branch name and directory structure
- Verify the branch is clean and branched from the right base
- Set up any project-specific configuration needed in the new worktree

This is the recommended starting point for every feature. Make it a habit before the first AI prompt, not an afterthought.

---

## Common Issues

**Worktree directory already exists:** git will refuse to create the worktree. Remove the directory or choose a different name.

```bash
rm -rf ../feature-name
git worktree add ../feature-name feature/feature-name
```

**Worktree shows as "locked":** another process is using it. If you are certain nothing is active, unlock it:

```bash
git worktree unlock ../feature-name
```

**Branch already checked out elsewhere:** git will not let you check out the same branch in two worktrees. Either use a different branch name, or remove the existing worktree first.

**Changes in worktree not visible in main checkout:** this is expected behavior, not a bug. Worktrees are isolated by design. To share code between branches, commit and merge normally.
