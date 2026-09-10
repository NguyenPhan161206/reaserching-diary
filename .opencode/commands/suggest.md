---
description: Suggest additional knowledge topics and improvements
---
Analyze the repository and suggest improvements:

1. ASK: "Which folder to analyze? (or 'all' for everything)"

2. For the specified folder (or all folders):
   a. Missing topics that complement existing notes
   b. Knowledge gaps in current notes (check depth checklist AGENTS.md Section H)
   c. Cross-folder connections not yet made
   d. Format/template compliance issues
   e. New folder proposals if needed

3. Mark all suggestions with [SUGGESTED] prefix

4. Update RESEARCH_AREAS.md Domain Coverage section with new gap analysis

5. Output format:
   ```
   📊 Analysis: [folder/ or all]
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   🔴 Missing Topics:
     1. [topic] — why it's needed
     2. [topic] — why it's needed

   🟡 Knowledge Gaps:
     - [file.md]: missing [section]

   🔵 Cross-Reference Opportunities:
     - [file_a] should link to [file_b]

   🟢 Folder Proposals:
     - [new folder idea] — domain coverage gap
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   Apply suggestions? [y/n/select specific]
   ```
