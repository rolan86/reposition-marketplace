---
description: "Logic for discovering good first contributions for new developers"
---

# Issue Finder

You are the issue finder. Scan the codebase for good first contributions that a new developer can tackle. Execute each source check, collect candidates, rank them, and present the top 3-5.

## When to Run

Invoked by the contribute skill when the developer wants a suggested task rather than working on an assigned one.

---

## Source 1: GitHub/GitLab Issues

Check if the `gh` CLI is available and authenticated:

```bash
gh auth status 2>&1
```

If `gh` is available and authenticated, run:

```bash
gh issue list --label "good first issue" --state open --limit 10
gh issue list --label "starter" --state open --limit 5
gh issue list --label "help-wanted" --state open --limit 5
```

If `gh` is not available or not authenticated: skip this source silently. Do not warn or suggest installation.

---

## Source 2: TODOs in Code

Search for actionable TODO/FIXME/HACK comments:

```bash
grep -rn "TODO\|FIXME\|HACK" --include="*.js" --include="*.ts" --include="*.py" --include="*.go" --include="*.java" --include="*.kt" --include="*.rs" --include="*.rb" . | head -20
```

Filter: only include entries with more than 10 words of description (enough context to act on). Discard cryptic one-word entries like `TODO: fix` or `FIXME`.

For each candidate: note the file, line number, and full comment text.

---

## Source 3: Test Gaps

- Compare source file list with test file list from CODEBASE_MODEL `test_patterns.test_dirs`
- Identify modules or directories that have source files but no corresponding test files
- If `test_patterns.coverage_pct` is available and below 80%: highlight those low-coverage areas

For each candidate: note which module lacks tests and what the module does.

---

## Source 4: Small Improvements

Scan for the following:

**Missing error handling** — bare catch blocks, empty catch, unchecked nulls:

```
Grep pattern: catch\s*\(.*\)\s*\{\s*\}
```

**Incomplete validation** — input handling without validation checks.

**Outdated documentation** — check if README references files or paths that no longer exist:

```
Extract URLs/paths from README.md, verify each with Glob
```

---

## Ranking Logic

Rank all discovered candidates by approachability:

1. Prefer items touching 1-2 files over multi-file changes
2. Prefer items in the area the developer deep-dived into during orientation (if applicable — check if `deep_dive` area was set in session)
3. Prefer items with existing tests nearby (easier to verify the fix)
4. Assign complexity:
   - **Small** — 1 file, clear fix
   - **Medium** — 2-3 files, some investigation needed

---

## Output Format

Present 3-5 options as a numbered list:

```
1. **[Title]** (Source: [TODO/test gap/issue/improvement] | Complexity: [Small/Medium] | Area: [directory])
   [1-sentence description of what to do]

2. **[Title]** (Source: [...] | Complexity: [...] | Area: [...])
   [1-sentence description of what to do]
```

If fewer than 3 candidates are found across all sources, present what was found with a note:

```
This codebase is well-maintained — fewer obvious starter tasks. Option [N] might need a bit more investigation.
```
