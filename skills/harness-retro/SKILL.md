---
name: harness-retro
description: Use after multiple sessions to convert past session data into a smarter harness. Reads the last N sessions, mines evaluations, blocked sub-tasks, decisions-rejected, and verification failures, then proposes targeted updates to rules.md, map files, and clarifying questions. This is the loop that makes the harness compound in value over time — each session makes the next one faster and more accurate. Invoke with /harness-retro.
---

# harness-retro

Turn the last N sessions into a smarter harness. **Without this, every session is isolated. With this, the harness gets sharper every time you use it.**

This skill closes the feedback loop. It reads what went wrong, what got rejected, what failed verification, and what decisions kept recurring — then proposes concrete updates to `rules.md`, `map/architecture.md`, and the Phase 0 clarifying questions in `/harness-task`.

---

## When to Run

- **After every 5–10 completed sessions** — converts accumulated noise into compounding signal
- **When `/harness-task` keeps getting blocked on the same kind of failure** — promote the failure into a rule
- **After a painful debug session** — encode the lesson learned
- **Before starting a new major feature** — make sure rules are current

---

## Step 1 — Identify project and load sessions

- Get current project name from current working directory
- Read `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state`
- Ask: *"How many recent sessions should I analyze? (default: last 10)"*
- List all session folders, sort by date descending, take the top N

---

## Step 2 — Mine session data in parallel

For each session folder, read in parallel:
- `progress.md` — find sub-tasks marked `blocked` and the reason
- `decisions.md` — collect all "Decisions Rejected" rows and "Open Questions"
- `verification-notes.md` — collect entries in "Failures Log" section
- `evaluations/*-eval.md` — collect evaluator scores below 4.0 and listed issues
- `sprint-contract.md` — extract feature descriptions and hard constraints

---

## Step 3 — Pattern detection

Identify these patterns:

### 3a — Recurring failure patterns
Group failures by error signature. If the same verification command failed for the same reason in 3+ sessions, that's a **promotable failure** — should become a rule or forbidden pattern.

Output table:
```markdown
| Failure pattern | Times seen | Sessions | Recommended fix |
|----------------|------------|---------|----------------|
| Missing migration before model change | 4 | MilkyWay, Voyager, Nebula, Aurora | Add rule: "Always write migration before editing model file" |
```

### 3b — Recurring decision patterns
If the same decision was made (or rejected) in 3+ sessions, automate it.

Output table:
```markdown
| Decision pattern | Times made | Outcome | Recommended automation |
|------------------|-----------|---------|----------------------|
| Use bcrypt for password hashing | 5 | Always taken | Pre-fill in Phase 0 — stop asking |
| Reject in-memory session store | 4 | Always rejected | Add to forbidden patterns in rules.md |
```

### 3c — Recurring clarifying gaps
If the same clarifying question got the same answer 3+ times, it's a project convention, not a question.

```markdown
| Question repeatedly asked | Always answered | Recommendation |
|--------------------------|-----------------|----------------|
| "What database?" | "PostgreSQL" | Add to map/architecture.md, skip asking |
| "Should this use JWT?" | "Yes, HttpOnly cookie" | Promote to default in rules.md |
```

### 3d — Low-scoring evaluator dimensions
If "Test coverage" averaged < 4.0 across N sessions, the worker prompts may need a sharper testing requirement.

### 3e — Repeatedly modified files (drift signal)
Find files modified in 5+ sessions. These are hot zones that may indicate:
- Missing abstraction (file should be split)
- Unclear ownership (multiple features touch the same code)
- Architectural smell

---

## Step 4 — Generate retrospective report

Write to `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/retrospectives/retro-<YYYY-MM-DD>.md`:

```markdown
# Retrospective — <PROJECT_NAME>
Date: <YYYY-MM-DD>
Sessions analyzed: <N> (from <oldest_date> to <newest_date>)

## Summary
- Total sub-tasks: <N>
- Blocked sub-tasks: <N> (<%>)
- Average evaluator score: <N>/5
- Recurring failure patterns: <N>
- Recurring decision patterns: <N>

## Top Recurring Failures
<table from 3a>

## Top Recurring Decisions
<table from 3b>

## Clarifying Questions to Promote
<table from 3c>

## Hot Files (modified in N+ sessions)
| File | Sessions | Avg evaluator score | Concern |
|------|---------|--------------------|---------| 

## Evaluator Dimension Trends
| Dimension | Avg | Trend (last 3 sessions) | Concern |
|-----------|-----|------------------------|---------|

## Architectural Smells Detected
<observations>

## Velocity Trends
| Session | Date | Sub-tasks | Avg score | Duration | Cost (est) |
|---------|------|-----------|-----------|----------|------------|
```

---

## Step 5 — Propose harness updates

Generate a separate patch file: `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/retrospectives/retro-<YYYY-MM-DD>-patches.md`

```markdown
# Proposed Harness Patches — retro-<YYYY-MM-DD>

## 1. New rules to ADD to rules.md
### To Section 12 — Forbidden Patterns
- [ ] Add: "<exact rule text>" — Source: failed in <N> sessions
- [ ] Add: "..."

### To Section 3 — WIP=1
- [ ] Strengthen: "<new constraint>" — Source: <evidence>

## 2. Rules to REMOVE from rules.md (unused)
- [ ] Remove from Section 8: "<rule>" — Source: never triggered in last N sessions

## 3. Map file updates
### map/architecture.md
- [ ] Document: "<convention>" — Source: implicit in N sessions, should be explicit
- [ ] Update layer constraint: <details>

### map/db-structure.md
- [ ] Add note about <pattern> — Source: <evidence>

## 4. Phase 0 clarifying question changes
### Promote to map/product-info.md (stop asking)
- [ ] "What database?" → already answered consistently
- [ ] "JWT or session?" → already established

### Add NEW conditional question
- [ ] When detecting <pattern>, ask: "<new question>"

## 5. Worker brief improvements
### Strengthen for low-scoring dimension
- [ ] "Test coverage" — add explicit instruction: "<text>"

## 6. Architectural smells to address
- [ ] <File> being modified in <N> sessions — consider <suggestion>

---

## How to Apply These Patches

For each item:
1. Check the box if you accept the patch
2. Run `/harness-retro-apply` (or manually edit the target files)
3. Updates are versioned — old rules.md preserved at rules-<DATE>.md.bak
```

---

## Step 6 — Ask user to approve

Show the user the retrospective report and patches:

*"I analyzed <N> sessions and found <X> recurring failures, <Y> recurring decisions, and <Z> hot files. The full report is at `retrospectives/retro-<DATE>.md` and proposed patches at `retro-<DATE>-patches.md`. Review the patches and check the boxes you want to apply. Run /harness-retro-apply when ready."*

---

## Step 7 — Update harness-state with retro signal

Add to `.harness-state`:
```
last_retro: <YYYY-MM-DD>
sessions_analyzed: <N>
proposed_patches: <count>
```

---

## Why This Matters

Without `/harness-retro`, every session is isolated noise. With it:
- Every recurring failure becomes a rule (won't happen again)
- Every recurring decision becomes a default (stop asking)
- Every hot file becomes a refactoring signal (preempt drift)
- Every low evaluator score becomes a sharper worker brief (better output)

**The harness gets smarter every time you use it. That is the compounding asset.**
