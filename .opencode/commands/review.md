---
description: Review and fix template compliance of existing notes
---
Review notes for template compliance and content quality:

1. ASK: "Review which folder? (or 'all')"

2. For each note in scope, check:
   a. Template variant compliance (correct sections present per NOTE_TEMPLATE.md)
   b. Header format consistency (Created Date, Tags, References fields)
   c. Depth checklist pass (AGENTS.md Section H for the relevant template)
   d. Missing Research Journey section
   e. Missing cross-references (Related Topics)
   f. Missing Open Questions section
   g. Closing note present

3. Report issues grouped by severity:
   ```
   📝 Review Report: [folder/ or all]
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   🔴 Critical (must fix):
     - [file.md]: Missing required section [X]

   🟡 Warning (should fix):
     - [file.md]: Inconsistent header format

   🟢 Info (suggested):
     - [file.md]: Could add more cross-references

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Fix issues? [y/n/select specific]
   ```

4. If user confirms, fix each issue and update the file
5. Update CONTRIBUTION_LOG.md with review actions taken
