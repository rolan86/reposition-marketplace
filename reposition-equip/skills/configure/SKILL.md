---
name: configure
description: "Generate or merge a stack-specific CLAUDE.md with team convention customizations"
---

# Configure Skill — CLAUDE.md Generation and Merge

You are the configure step of the reposition-equip plugin. Your job is to produce a fully populated `CLAUDE.md` file for the user's project — either by writing it fresh or merging new sections into one that already exists.

---

## Input

You receive two inputs from the equip orchestrator:

1. **`stack_config`** — the stack configuration object, which contains:
   - `template` (string): the template name to use for single-stack projects (e.g. `typescript-react`, `python-fastapi`, `go-api`, `java-spring`, `angular`, `generic`)
   - `conventions` (object): the user's Q&A answers, keyed by: `naming`, `error_handling`, `pr_process`, `logging`, `custom`
   - `services` (array, optional): for multi-service projects — each entry has `name` (service name) and `template` (template name)

2. **`pre_delivery_inventory`** — the result of the pre-delivery check, which contains:
   - `claude_md_exists` (boolean): whether a `CLAUDE.md` file already exists at `PROJECT_ROOT/CLAUDE.md`
   - `existing_sections` (array of strings, optional): section headers found in the existing `CLAUDE.md`, if it exists

---

## Template Selection

Templates live at `skills/configure/templates/` relative to this skill file. The available templates are:

| Template Name | File |
|---|---|
| `typescript-react` | `templates/typescript-react.md` |
| `python-fastapi` | `templates/python-fastapi.md` |
| `go-api` | `templates/go-api.md` |
| `java-spring` | `templates/java-spring.md` |
| `angular` | `templates/angular.md` |
| `generic` | `templates/generic.md` |

### Single-Stack Project

Read the template file matching `stack_config.template`. Use this single template as the base document.

### Multi-Service Project

When `stack_config.services` is present and contains more than one entry:

1. For each service in `stack_config.services`, load the template file matching `service.template`.
2. Combine all loaded templates into a single multi-service `CLAUDE.md` with the following structure:

```
# CLAUDE.md — Multi-Service Project

## System Architecture
[Overview paragraph naming all services, their roles, and how they interact. Infer from service names and stack types.]

## Shared Conventions
[Populate from Q&A answers in stack_config.conventions — these apply to all services.]

## Service: [service.name] — [stack label]
[Full content of that service's template, with CUSTOMIZE markers already replaced per the rules below.
 Remove the template's own "About This Project" header and merge its "Code Conventions" section here.]

## Service: [next service.name] — [stack label]
[Same treatment for each additional service.]

## Tech Stack
[Combined list of all technologies across all services, deduplicated. Group by service if they differ significantly.]

## Skills Reference
[Single unified table — same as any individual template's Skills Reference section.]
```

---

## Customize Injection

Each template contains `<!-- CUSTOMIZE:key -->` markers in the **Code Conventions** section. Replace every marker according to the rules below.

### Keys and replacement rules

**`<!-- CUSTOMIZE:naming -->`**
- If `stack_config.conventions.naming` is `"detect"`:
  - Inject the detected naming patterns. Include file evidence (e.g. filenames observed in the repo scan).
  - Format:
    ```
    **Naming Conventions (detected from codebase):**
    - Files: [detected pattern, e.g. kebab-case — observed: `auth-service.ts`, `user-router.ts`]
    - Variables/functions: [detected pattern, e.g. camelCase — observed: `getUserById`, `authToken`]
    - Classes/types: [detected pattern, e.g. PascalCase — observed: `UserService`, `AuthToken`]
    - Constants: [detected pattern, e.g. SCREAMING_SNAKE_CASE — observed: `MAX_RETRIES`, `DB_TIMEOUT`]
    ```
- If the answer is a non-empty string provided by the user: replace with that string verbatim as a bullet point or paragraph.
- If the answer is `"not sure"`, null, or empty: replace with:
  ```
  <!-- Default — update when your team decides -->
  Follow existing conventions in the codebase. Prefer consistency with adjacent files over any global rule.
  ```

**`<!-- CUSTOMIZE:error_handling -->`**
- If the user provided a non-empty answer: replace with that answer as a bullet point or paragraph.
- If the answer is `"not sure"`, null, or empty: replace with:
  ```
  <!-- Default — update when your team decides -->
  Throw typed exceptions at the boundary where an error is first detected. Log at the handler layer, not at every call site. Never swallow exceptions silently.
  ```

**`<!-- CUSTOMIZE:pr_process -->`**
- If the user provided a non-empty answer: replace with that answer as a bullet point or paragraph.
- If the answer is `"not sure"`, null, or empty: replace with:
  ```
  <!-- Default — update when your team decides -->
  PRs require at least one approval before merging. Link each PR to the relevant ticket or issue. Squash-merge to keep main history clean.
  ```

**`<!-- CUSTOMIZE:logging -->`**
- If the user provided a non-empty answer: replace with that answer as a bullet point or paragraph.
- If the answer is `"not sure"`, null, or empty: replace with:
  ```
  <!-- Default — update when your team decides -->
  Use structured logging (JSON format preferred). Log at INFO for normal operations, ERROR for failures, DEBUG for diagnostics (disabled in production). Never log secrets or PII.
  ```

**`<!-- CUSTOMIZE:custom -->`**
- If the user provided a non-empty, non-null answer: replace with that answer as a paragraph or bullet list.
- If the answer is null, `"not sure"`, or empty: **remove the marker entirely**. Do not leave an empty placeholder or comment.

### Injection formatting

When injecting a user-provided answer:
- If the answer is a single sentence or short phrase, format it as a bullet: `- [answer]`
- If the answer contains multiple lines or bullet points already, preserve the user's formatting
- Do not add extra headers inside the Code Conventions section — each injection is a flat block of text or bullets directly under the section

---

## Merge Logic (CLAUDE.md already exists)

When `pre_delivery_inventory.claude_md_exists` is `true`:

1. Read the existing `PROJECT_ROOT/CLAUDE.md`.
2. Extract the section headers from the existing file (lines starting with `##`).
3. Compare the section headers from the populated template against the existing file's headers.
4. For each `##` section in the populated template that is **not present** in the existing file:
   - Collect it as a section to append.
5. If there are sections to append:
   - Add the following block at the end of the existing `CLAUDE.md`:
     ```
     ---

     ## Added by REPOSITION Equip

     The following sections were added from the [stack] REPOSITION template because they were not found in your existing CLAUDE.md.
     Review and adjust to fit your team's actual conventions.

     [appended sections, each with their full content]
     ```
6. For sections already present in the existing file: **skip entirely**. Never modify existing content.
7. After writing the merged file, report:
   - Which sections were appended
   - Which sections were skipped (because they already existed)

**Important:** The merge is additive only. Do not reorder, reformat, or touch any line of the existing file above the appended block.

---

## Create Logic (no CLAUDE.md)

When `pre_delivery_inventory.claude_md_exists` is `false`:

1. Take the fully populated template (with all CUSTOMIZE markers replaced per the rules above).
2. Write it directly to `PROJECT_ROOT/CLAUDE.md`.
3. Report that the file was created and list the conventions that were injected.

---

## Output Report

After completing either create or merge, print a brief summary to the terminal:

```
REPOSITION Equip — CLAUDE.md [Created | Updated]

Template used:    [template name(s)]
Project root:     [PROJECT_ROOT path]

Conventions injected:
  naming:          [value used or "default applied"]
  error_handling:  [value used or "default applied"]
  pr_process:      [value used or "default applied"]
  logging:         [value used or "default applied"]
  custom:          [value used or "removed (empty)"]

[If merge:]
Sections added:   [list of section headers appended]
Sections skipped: [list of section headers already present]

Next step: Open CLAUDE.md and review the injected conventions.
Replace any "<!-- Default —" placeholders with your team's actual decisions.
```

---

## Error Handling

- If the template name in `stack_config.template` does not match any file in `templates/`: use `generic.md` and note the fallback in the output report.
- If a service in `stack_config.services` references an unknown template: use `generic.md` for that service and note it.
- If `PROJECT_ROOT` cannot be determined: ask the equip orchestrator to re-provide it before proceeding.
- If the existing `CLAUDE.md` cannot be read: report the error and stop. Do not attempt a partial write.
