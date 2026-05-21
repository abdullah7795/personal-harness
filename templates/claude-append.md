
---
## Harness — <PROJECT_NAME>

### Knowledge Base (Obsidian)
Read before starting any work:
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/file-tree.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/db-structure.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/architecture.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/product-info.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/rules.md`

### Workflow (mandatory, no exceptions)
1. **Session start** → `/harness-session`
2. **Every task** → `/harness-task`
3. **After every step** → update Obsidian session files immediately
4. **Before done** → run verification command

### Active session
`~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state`
Written by `/harness-session` — contains `active_session` and `session_path`.
`/harness-task` reads this file to find session files automatically.

### Session files
`~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SessionName>-<date>/`
- `progress.md` — sub-task breakdown and live status
- `decisions.md` — all decisions made/rejected with full reasoning
- `verification-notes.md` — verification results per step + failures log
- `checkpoints.md` — step-by-step completion markers (crash recovery record)

### Sub-task rules
- Write ALL sub-tasks to `progress.md` before dispatching any agent
- Every sub-task entry must be self-sufficient — a cold agent must be able to execute it alone
- Update all 4 session files after EACH step, never at the end
- If verification fails: mark as `blocked`, log failure — never skip or ignore
- Agents only edit files in their sub-task's "Files to edit" list

### Continuity
Any agent reads `.harness-state` → finds session → reads `progress.md` → continues from pending sub-tasks. Works across Claude Code, Codex, Cursor, or any tool. Zero re-briefing ever needed.
