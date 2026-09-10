---
description: Log today's research activity
---
Log today's research activity to tracking files:

1. Check git log for today's changes:
   - `git log --since="midnight" --oneline`
   - `git diff --stat HEAD`
   - Any new/modified files

2. Check CONTRIBUTION_LOG.md for last entry date

3. For each activity since last log:
   Append to CONTRIBUTION_LOG.md:
   ```
   | YYYY-MM-DD | HH:MM | [category] | [action] | [files] | [summary] |
   ```

4. Update CONTRIBUTION_CALENDAR.md:
   - Count today's interactions
   - Update the grid cell:
     - 0 interactions → ░░
     - 1-2 interactions → ▓▓
     - 3+ interactions → ██
   - Update streak counter (consecutive days with activity)
   - Update monthly stats

5. Show summary:
   ```
   📊 Daily Log: YYYY-MM-DD
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Interactions today: X
   Files changed: [list]
   Current streak: 🔥 X days
   This month: X active days, Y total interactions
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ```

6. Ask: "Should I suggest a push? (meets push conditions: [check])"
