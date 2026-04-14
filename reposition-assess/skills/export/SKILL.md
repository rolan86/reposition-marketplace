---
name: export
description: "Re-export your last REPOSITION assessment as markdown or copy to clipboard for sharing"
---

# REPOSITION Export

Re-export the most recent assessment report.

## Flow

1. **Find the most recent assessment:**
   - Look for `.reposition/assessment-*.md` files
   - If none found: "No assessment found. Run `/reposition-assess` first."
   - If found: read the most recent one (by date in filename)

2. **Parse the assessment:**
   - Read the assessment file
   - Extract the scorecard data, evidence sections, and Raw Signals YAML

3. **Offer export options:**

   "How would you like to export your assessment?"
   
   A) **Full markdown** — Print the complete report to terminal (for copying)
   B) **Facilitator summary** — Compact 4-line summary for workshop sharing
   C) **Scores only** — Just the scorecard table

4. **Execute the chosen export:**
   - **Option A:** Display the full assessment markdown content
   - **Option B:** Generate the compact format from `../assess/report-template.md` Format 3 and display it
   - **Option C:** Display only the scorecard table section

5. **Copy assistance:**
   - After displaying, note: "You can select and copy the text above, or I can try to copy it to your clipboard."
