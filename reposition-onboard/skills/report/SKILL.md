---
name: report
description: "Generate the personal onboarding doc from session data"
---

# REPOSITION Onboarding — Report Generator

**Role:** You generate the personal onboarding document that captures everything the developer learned during this session.

**Input:** Expects CODEBASE_MODEL and SESSION_DATA (architecture findings, patterns, conventions, deep dive area, contribution details, Q&A exchanges accumulated during the session).

## Generation Flow

1. Read `report-template.md` for the template structure
2. Fill in all `[bracketed]` values from CODEBASE_MODEL and SESSION_DATA:
   - `[repo_name]` — from CODEBASE_MODEL.repo
   - `[YYYY-MM-DD]` — today's date
   - Architecture section — from CODEBASE_MODEL.stack, structure, entry_points, environment
   - Patterns section — from CODEBASE_MODEL.claude_md, abstractions, test_patterns, git
   - Deep dive section — from SESSION_DATA.deep_dive_area (if set)
   - Contribution section — from SESSION_DATA.contribution (if set)
   - Q&A section — from SESSION_DATA.qa_log (if any)
3. Apply the "Sections to Omit" rules from the template — never include empty sections
4. Apply Q&A capping: maximum 10 exchanges. Select the most significant (not the most recent). Omit clarifying back-and-forths like "what do you mean?" → "I mean..."

## Save

- Write to `.reposition/onboarding-YYYY-MM-DD.md`
- If a file with today's date already exists, append a counter: `onboarding-YYYY-MM-DD-2.md`, `onboarding-YYYY-MM-DD-3.md`, etc.
- Create `.reposition/` directory if it doesn't exist (Bash: `mkdir -p .reposition`)

## Confirmation

Display: "Onboarding doc saved to `.reposition/onboarding-YYYY-MM-DD.md`. This is your personal reference — it's not committed to git."

## Update Existing (for `/reposition-contribute` re-entry)

- If an existing onboarding doc is found (glob for `.reposition/onboarding-*.md`) and a new contribution was made in this session:
  - Offer: "You have an existing onboarding doc from [date]. Add this contribution to it, or create a new one?"
  - If updating: append a new "Contribution: [YYYY-MM-DD]" section to the existing file (don't overwrite the original "My First Contribution" — add subsequent ones with dated headers)
  - If creating new: proceed with the normal save flow

**Important:** The report skill does NOT show the upsell CTA. That is handled by the scan skill after this skill returns.
