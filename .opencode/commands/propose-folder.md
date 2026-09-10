---
description: Propose creating a new folder for unclassified content
---
Propose a new folder when content doesn't fit existing structure:

1. ASK: "What topic needs a new home?"

2. Analyze the topic:
   - What domain does it belong to?
   - Does it fit any existing folder? Check RESEARCH_AREAS.md
   - If no match found, propose options

3. Present proposal:
   ```
   📂 Folder Proposal
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Topic: [topic name]
   Current folders: [list existing]

   Options:
   a) Create new folder: [suggested_name]/
      - Domain: [domain]
      - Template: [suggested template]
      - Reason: [why this needs its own folder]

   b) Add to closest existing folder: [folder]/
      - Reason: [why it fits here]
      - Sub-topic potential: [yes/no]

   c) Create sub-folder: [parent]/[child]/
      - Reason: [why sub-folder is better]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Which option? (a/b/c or type custom name)
   ```

4. If user confirms new folder (option a):
   a. Create the folder
   b. Create README.md in new folder (describe purpose)
   c. Create first note using appropriate template
   d. Update AGENTS.md Section C (add mapping)
   e. Update README.md root (add description)
   f. Update RESEARCH_AREAS.md (add row + update Recently Added)
   g. Update CONTRIBUTION_LOG.md with folder creation event
   h. Update CONTRIBUTION_CALENDAR.md
