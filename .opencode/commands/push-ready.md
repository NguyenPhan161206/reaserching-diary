---
description: Check if ready to push and show summary
---
Check push readiness and display summary:

1. Run pre-push checklist:
   a. `git status` — list all changes
   b. Count new/modified files since last push
   c. Verify template compliance for all new notes
   d. Verify CONTRIBUTION_LOG.md is updated
   e. Verify CONTRIBUTION_CALENDAR.md is updated
   f. Check cross-references are added
   g. Check for any secrets/keys in diff

2. Display push summary:
   ```
   📋 Push Summary
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   New files:
     + [file1.md]
     + [file2.md]

   Modified files:
     ~ [file3.md]
     ~ CONTRIBUTION_LOG.md
     ~ CONTRIBUTION_CALENDAR.md

   Proposed commit messages:
     [category] add: [brief description]
     [category] update: [brief description]

   Template compliance: ✅/❌ [X/Y pass]
   Cross-references: ✅/❌ [X links added]
   Contribution streak: 🔥 [X days active]
   Secrets check: ✅ No secrets detected
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Proceed with push? [y/n]
   ```

3. If user confirms (y):
   a. `git add .`
   b. `git commit -m "[category] add: [description]"`
   c. `git push`
   d. Update CONTRIBUTION_LOG.md with push event
   e. Show: "✅ Pushed successfully! Repo updated."

4. If user declines (n):
   - Ask: "What would you like to change?"
   - Offer to modify commit message or hold changes
