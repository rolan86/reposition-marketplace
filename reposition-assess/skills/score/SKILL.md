---
name: score
description: "Quick AI readiness rescan — updates scores from current project state without asking questions"
---

# REPOSITION Quick Rescan

Run a quick rescan of the project to update AI readiness scores. This skips questions and only uses scan-based signals.

## Flow

1. **Check for existing assessment:**
   - Look for `.reposition/assessment-*.md` files in the project
   - If none found: "No previous assessment found. Run `/reposition-assess` for a full assessment with targeted questions."
   - If found: read the most recent one (by date in filename) and extract the Raw Signals YAML block

2. **Run the scan engine:**
   - Follow the scan instructions from `../assess/scan-engine.md`
   - Use only automated signals (no questions)
   - For scores that previously had question-based input, keep the old question scores and only update scan-based signals

3. **Compare results:**
   - For each domain, calculate the new score using scan signals only
   - Compare with the previous assessment's scores
   - Calculate deltas for each domain

4. **Display updated scorecard:**
   Show a comparison table:
   ```
   ═══════════════════════════════════════════════════════
     REPOSITION Quick Rescan — Score Update
   ═══════════════════════════════════════════════════════

     Domain           Previous    Current     Delta
     ───────────────  ──────────  ──────────  ─────
     Product Mgmt     X/10        X/10        [↑X/↓X/→]
     Architecture     X/10        X/10        [↑X/↓X/→]
     Design           X/10        X/10        [↑X/↓X/→]
     Development      X/10        X/10        [↑X/↓X/→]

     Overall AI Readiness: X.XX/10 (was X.XX/10)
   ═══════════════════════════════════════════════════════
   ```

   Use ↑ for improvement, ↓ for regression, → for no change. Show the numeric delta.

5. **Offer to save:**
   - "Save this as a new assessment? This preserves your previous assessment for comparison. [Y/n]"
   - If yes: write `.reposition/assessment-YYYY-MM-DD.md` using `../assess/report-template.md` Format 2
   - Note in the file that this was a quick rescan (no questions) vs a full assessment
