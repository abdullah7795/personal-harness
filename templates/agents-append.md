
---
## Harness — <PROJECT_NAME>

### Knowledge Base (Obsidian)
Read these files before starting ANY work. They are your navigation map for this codebase.

  File tree:    ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/file-tree.md
  DB structure: ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/db-structure.md
  Architecture: ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/architecture.md
  Product info: ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/product-info.md
  Rules:        ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/rules.md

### Mandatory Workflow — No Exceptions
1. Session start → run `/harness-session`
2. Task received → run `/harness-task`
3. After every completed step → update Obsidian session files IMMEDIATELY (not at the end)
4. Before declaring done → run the verification command from the sub-task

### Active Session Tracking
  ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state
  Written by /harness-session. Contains active session name and session_path.
  harness-task reads this to locate session files without asking the user.

### Session Files
  ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SessionName>-<date>/
    progress.md           — sub-task breakdown and live status
    decisions.md          — every decision made/rejected with full reasoning
    verification-notes.md — verification result per step + failures log
    checkpoints.md        — step-by-step completion markers (recovery record)

### Sub-task Rules
- Every sub-task entry in progress.md must be self-sufficient (a cold agent can execute from the entry alone)
- Write ALL sub-tasks to progress.md BEFORE dispatching any agents
- Update session files after EACH step — never batch at the end
- If verification fails: mark sub-task as "blocked", log failure, stop — do NOT skip
- Only edit files listed in the sub-task's "Files to edit" list

### Continuity Rule
Session files are the single source of truth. Any agent — Claude Code, Codex, Cursor, or other —
reads .harness-state to find the active session, reads progress.md to find pending sub-tasks,
and continues from there. Zero re-briefing needed. Works across tool switches mid-task.
