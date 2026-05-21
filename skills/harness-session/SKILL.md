---
name: harness-session
description: Use at the start of every work session before any task begins. Asks whether this is a new session or continuing an existing one. New sessions get a unique memorable name (cosmic/nature-themed) and fresh Obsidian session files. Existing sessions resume from the last checkpoint. Writes active session to .harness-state so harness-task can find it. Invoke with /harness-session.
---

# harness-session

Start or resume a work session. Run this **at the beginning of every work session**, before any task. No task should ever begin without running this first.

---

## Step 1 — Find the active project

- Get current working directory name → `<PROJECT_NAME>`
- Check Obsidian knowledge base exists: `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/`
- If not found: *"No harness found for `<PROJECT_NAME>`. Run `/harness-proj-init` first to set up the harness."* → stop.
- Read `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/rules.md` now to load project rules into context.

---

## Step 2 — Check for interrupted session

Before asking the user, check `.harness-state`:

```bash
cat ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state
```

If `.harness-state` exists and contains an `active_session`, it means a previous session was interrupted. Show:

```
⚠ Interrupted session detected: `<SESSION_NAME>` (started <date>)
  Last checkpoint: step <N>
  Pending sub-tasks: <count>

  Options:
  1. Resume this interrupted session
  2. Start a new session (interrupted session stays available)
```

If `.harness-state` does not exist, skip to Step 3.

---

## Step 3 — Ask: new or existing?

> "Is this a **new session** or are you **continuing an existing one**?"

If user says "existing" but no sessions folder exists yet:
*"No previous sessions found for `<PROJECT_NAME>`. Starting a new session instead."* → proceed with new session flow.

---

## If NEW session

### Generate session name

Pick a name from this list that has NOT been used yet. Check existing session folder names to avoid duplicates.

```
Cosmic:
  MilkyWay, NorthernLights, Voyager, Nebula, Andromeda, Pulsar, Quasar,
  Zenith, Aurora, Celestia, Solaris, Vega, Orion, Cassini, Halley,
  Pegasus, Lyra, Cygnus, Perseus, Phoenix, Polaris, Rigel, Altair,
  Betelgeuse, Arcturus, Capella, Procyon, Antares, Aldebaran, Canopus

Nature:
  ArcticFox, CrimsonDawn, TidalWave, BorealForest, SilverFrost,
  GoldenMesa, CoralReef, CobaltPeak, AmberDune, CinderGlade,
  CrystalBay, IronwoodRidge, MossValley, EmberCrest, ObsidianField,
  CopperCanyon, SapphireCoast, MarbleCliff, CinnabarPlain, IndigoMoor
```

If all 50 names are used, combine: pick one from each list → `AuroraFox`, `VoyagerDawn`, `NebulaMoss`.

### Create session folder

```bash
mkdir -p ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SESSION_NAME>-<YYYY-MM-DD>/evaluations
```

This creates both the session folder and the `evaluations/` subfolder used later by `/harness-task` for evaluator agent reports.

### Create `progress.md`

```markdown
# Progress — <SESSION_NAME>
Date: <YYYY-MM-DD>
Project: <PROJECT_NAME>
Status: active
Session path: ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SESSION_NAME>-<YYYY-MM-DD>/

## Task
(filled when /harness-task is run)
Total sub-tasks: (filled when /harness-task is run)

## Sub-task Breakdown

| # | Title | Status | Agent | Blocked By | Notes |
|---|-------|--------|-------|------------|-------|

## Status Legend
- pending   — not yet started
- in_progress — agent dispatched and working
- completed  — done and verified
- blocked    — verification failed or dependency not met
- skipped    — explicitly decided not to do
```

### Create `decisions.md`

```markdown
# Decisions — <SESSION_NAME>
Date: <YYYY-MM-DD>
Project: <PROJECT_NAME>

---

## Decisions Taken

Record every decision made during this session — no decision is too small.

| # | Decision Made | Why This Was Right | Alternatives Considered | Sub-task |
|---|---------------|-------------------|------------------------|---------|

---

## Decisions Rejected

Record everything that was considered but not chosen.

| # | Option Rejected | Why Rejected | What Was Chosen Instead | Sub-task |
|---|-----------------|-------------|------------------------|---------|

---

## Open Questions

Things that came up but could not be decided yet.

| # | Question | Context | Impact if Wrong |
|---|----------|---------|----------------|
```

### Create `verification-notes.md`

```markdown
# Verification Notes — <SESSION_NAME>
Date: <YYYY-MM-DD>
Project: <PROJECT_NAME>

---

## Sub-task Verifications

| # | Sub-task Title | Command Run | Expected Output | Actual Output | Pass/Fail | Notes |
|---|----------------|-------------|-----------------|---------------|-----------|-------|

---

## Failures Log

If any verification fails, document here in full detail.

| Sub-task # | Command | Error Output | Root Cause | Resolution |
|------------|---------|-------------|------------|-----------|

---

## Final Verification

(Filled after all sub-tasks complete)

- Date:
- End-to-end check:
- Result:
- Notes:
```

### Create `checkpoints.md`

```markdown
# Checkpoints — <SESSION_NAME>
Date: <YYYY-MM-DD>
Project: <PROJECT_NAME>

A checkpoint is written after EVERY step — including the initial breakdown.
This file is the recovery record. Any agent reading this knows exactly where to resume.

---

## Checkpoint 0 — Session started
- Date: <YYYY-MM-DD HH:MM>
- Status: session initialized, no task yet
- Next: run /harness-task with a task

---

(Further checkpoints added by /harness-task as steps complete)
```

### Write `.harness-state`

Write to `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state`:

```
active_session: <SESSION_NAME>-<YYYY-MM-DD>
session_path: ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SESSION_NAME>-<YYYY-MM-DD>
started: <YYYY-MM-DD HH:MM>
project: <PROJECT_NAME>
status: active
```

Note: `/harness-task` will add additional fields (`task_started`, `task_completed`, `features_passed`, `current_phase`) as it progresses through phases. Status transitions: `active` → `in_task` (during task) → `paused` (clock-out mid-task) → `completed` (task done).

### Clock-In Routine — Load context

Read in parallel:
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/file-tree.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/architecture.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/product-info.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/rules.md`

### Cold-Start Test (BLOCKING)

Answer these 5 questions from repo/Obsidian files alone — no verbal context needed:

1. **What is this system?** → must be answerable from `map/product-info.md`
2. **How is it organized?** → must be answerable from `map/architecture.md`
3. **How do I run it?** → must be answerable from `AGENTS.md` or `map/architecture.md`
4. **How do I verify it?** → must be answerable from `rules.md` or `AGENTS.md` (test/lint commands present)
5. **Where are we now?** → `quality-document.md` (created by `/harness-proj-init`)

**If any question cannot be answered: this is a BLOCKER, not a warning.**

Show user:
```
✗ Cold-start test FAILED on question(s) <N>: <question>

This means the next /harness-task agent will not have the context it needs and may make wrong decisions.

Options:
1. Update the map file now (recommended) — I'll help you fill in the gap
2. Re-run /harness-proj-init to regenerate map files from current codebase
3. Skip and continue anyway (NOT recommended — accept reduced output quality)
```

**Do not proceed to print summary until cold-start passes (or user explicitly accepts the risk).**

### Print summary

```
✓ Session `<SESSION_NAME>` started
  ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SESSION_NAME>-<date>/
  ├── progress.md
  ├── decisions.md
  ├── verification-notes.md
  └── checkpoints.md

Active session recorded in .harness-state.
Cold-start test: <Pass / N gaps found>
Context loaded: architecture + file-tree + product-info + rules

Ready. Run /harness-task with your task description.

Other available commands:
  /harness-retro   — analyze past sessions and propose harness improvements (run every 5–10 sessions)
  /harness-export  — generate PR-ready TASK-REPORT.md (run after completing a task)
```

---

## If EXISTING session

### List sessions

```bash
ls ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/
```

Sort by date, newest first. For each folder, read:
- `progress.md` → check `Status:` field (active / completed)
- `checkpoints.md` → last checkpoint entry number

Show the user:

```
Sessions for <PROJECT_NAME>:

  #  Session Name              Status     Last Checkpoint
  1. NorthernLights-2026-05-21 [active]   step 7 — "Sub-task 7 complete"
  2. MilkyWay-2026-05-20       [active]   step 3 — "Sub-task 3 complete"
  3. Voyager-2026-05-19        [completed]
  4. Nebula-2026-05-18         [completed]

Which session? (enter number, or 'new' to start fresh)
```

### User picks a session

### Read current state

Read in parallel:
- `progress.md` → find ALL sub-tasks, identify those with status `in_progress` or `pending`
- `checkpoints.md` → find the last checkpoint entry for resume context
- `decisions.md` → load all decisions already made (context for continuing)
- `verification-notes.md` → check if any prior failures need attention

### Update `.harness-state`

Update `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state`:

```
active_session: <SESSION_NAME>-<YYYY-MM-DD>
session_path: ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SESSION_NAME>-<YYYY-MM-DD>
resumed: <YYYY-MM-DD HH:MM>
project: <PROJECT_NAME>
status: active
```

### Add resume checkpoint to `checkpoints.md`

```markdown
## Checkpoint — Session resumed
- Date: <YYYY-MM-DD HH:MM>
- Resumed from: <N> completed checkpoints
- Pending sub-tasks: <list titles>
- Prior failures: <none / list>
```

### Print resume summary

```
✓ Resuming session `<SESSION_NAME>`
  Last checkpoint: step <N> — <checkpoint title>

  Pending sub-tasks (<count>):
    Sub-task <N>: <title>
      Files to edit: <files>
      What to do: <brief instruction>

    Sub-task <N+1>: <title> [blocked by <N>]
      ...

  Prior failures: <none / list with details>

Active session recorded in .harness-state.

Ready to continue. Run /harness-task to proceed from where you left off.
```

---

## Clock-Out Routine (run at end of every session)

When you are done working for the day or ending a session, run these steps in order:

1. **Update `progress.md`** — mark all completed sub-tasks, note any in-progress ones with current state
2. **Run verification** — `make check` or equivalent. Confirm build and tests pass
3. **Five-dimension clean state check:**
   - Build passes
   - All tests pass (including tests that existed before this session)
   - No debug artifacts (`console.log`, `debugger`, `TODO`, `FIXME`)
   - Lint passes
   - Standard startup works
4. **Commit all completed work** — one commit per logical unit, message explains what AND why
5. **Update `.harness-state`** — set `status: paused` if mid-task, `status: completed` if done
6. **Add clock-out checkpoint** to `checkpoints.md`:
   ```markdown
   ## Checkpoint — Session clock-out
   - Date: <YYYY-MM-DD HH:MM>
   - Completed sub-tasks this session: <N>
   - Pending sub-tasks remaining: <N>
   - Build: Pass/Fail
   - Tests: Pass/Fail
   - Clean state: Pass/Fail
   - Next session should start with: <description of next pending sub-task>
   ```

**"Clean up later" is a trap.** The next session will not know what was left behind and will start new work on top of the mess. Clean state at every exit is non-negotiable.

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Harness not found | Tell user to run `/harness-proj-init` first |
| User says "existing" but no sessions exist | Notify and start a new session instead |
| `.harness-state` shows active session | Alert user, offer to resume or start new |
| Session folder exists but files are missing | Recreate missing files with empty templates, log as note |
| All 50 session names used | Combine names from both lists |
| Cold-start test fails (question unanswerable) | Note gap, continue session, flag for map file update |
