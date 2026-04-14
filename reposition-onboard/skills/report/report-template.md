---
description: "Template for the personal onboarding doc generated at .reposition/onboarding-YYYY-MM-DD.md"
---

# REPOSITION Onboarding Report Template

**Role:** This file defines the output format for the personal onboarding doc. It is not executed directly — the report skill reads this and uses it to generate `.reposition/onboarding-YYYY-MM-DD.md`.

**Inputs required:**
- `CODEBASE_MODEL` — from the scan
- `SESSION_DATA` — accumulated during the session: architecture findings, patterns, conventions, deep dive area, contribution details, Q&A log

**Template:**

```markdown
# Onboarding — [repo_name]
Generated: [YYYY-MM-DD]

## Architecture
- **System type:** [system_type] ([language] + [framework])
- **Directory map:**
  [For each key directory: `directory/` — one-line explanation]
- **Data flow:** [One traced request path, e.g., "POST /orders → OrderController → OrderService → OrderRepository → PostgreSQL"]
- **Key dependencies:** [Top 5-10 libraries and their roles]

## Patterns & Conventions
- **Code conventions:** [naming, file organization, import ordering]
- **Architecture patterns:** [detected patterns with examples]
- **Testing:** [framework, organization, mocking approach]
- **Git workflow:** [branch naming, commit format, PR process]
- **Unwritten rules:** [inferred patterns not documented elsewhere]

## Deep Dive: [area_explored]
- **Module structure:** [key files and their roles]
- **Execution paths:** [traced paths through this area]
- **Gotchas:** [unusual patterns, workarounds, known issues]
- **Recent activity:** [who's been working here, what's changed]

## My First Contribution
- **Task:** [description of what was worked on]
- **Files changed:** [list of files]
- **Patterns applied:** [conventions followed]
- **PR:** [link if available, or "not yet submitted"]

## Questions & Answers
[Top 10 most significant Q&A exchanges from the session.
Each entry: the dev's literal question + 1-3 sentence summary of the answer.
Omit clarifying back-and-forths. If fewer than 10 exchanges occurred, include all.]
```

**Sections to Omit:**
- If the dev did not do a deep dive: omit the "Deep Dive" section entirely
- If the dev did not contribute: omit the "My First Contribution" section
- If the dev asked no questions (guided mode, no follow-ups): omit "Questions & Answers"
- Never include empty sections with placeholder text
