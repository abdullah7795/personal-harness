---
name: harness-task
description: Use whenever a task is given during an active harness session. Runs a structured intake with clarifying questions to build an explicit Definition of Done, creates a Sprint Contract and Feature List before any coding begins, verifies the environment bootstraps cleanly, executes features one at a time (WIP=1) with parallel micro-agents, applies a three-layer termination check (syntax → runtime → E2E) per sub-task, uses a separate evaluator agent, updates all Obsidian session files after every step, and ends with a five-dimension clean state check. Production-ready output guaranteed. Invoke with /harness-task.
---

# harness-task

Execute any task to production-ready quality. Requires an active session — run `/harness-session` first.

**Core guarantee:** If you give this skill a task, it will ask questions until it knows exactly what "done" means, split the work into independently verifiable units, execute each with a dedicated agent, verify all three layers (syntax → runtime → E2E), and leave the codebase in clean state. Nothing is declared done until a machine-executable command proves it.

---

## PHASE 0 — Task Intake & Explicit Definition of Done

This phase produces a Sprint Contract and Feature List before a single line of code is written. **No coding begins until this phase is complete and you approve it.**

### 0a — Read active session and project context

Read `.harness-state`:
```bash
cat ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state
```
If missing: *"No active session. Run `/harness-session` first."* → stop.

Read in parallel:
- `<session_path>/progress.md` — check for unfinished work
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/architecture.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/file-tree.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/product-info.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/rules.md`

If progress.md has `in_progress` or `pending` sub-tasks from a prior run → show the user and ask: *"There is unfinished work from a previous run. Resume it, or start a new task?"* Resume skips to Phase 3.

### 0b — Triage: How big is this task?

**Don't apply heavy ceremony to small tasks.** Categorize the task first:

| Size | Description | Example | Ceremony |
|------|-------------|---------|----------|
| **trivial** | Typo, single-line fix, rename, format | "fix typo in README" | Skip Phase 0c (no questions), still run Phases 1–4 minimally |
| **small** | One file or one function, no architecture decisions | "add validation to /api/users endpoint" | Ask 2 questions max (what + verification command), light Sprint Contract |
| **medium** | Multiple files, one feature, minor architecture | "add password reset feature" | Full Phase 0c (clarifying questions) |
| **large** | Multiple features, architectural decisions | "create backend with auth + CRUD + DB" | Full Phase 0c + recommend splitting into multiple `/harness-task` calls |

Detect size by:
- Task description length and complexity
- Number of files likely affected (read file-tree.md)
- Whether new architectural patterns are introduced

If `trivial` or `small`: write a minimal Sprint Contract automatically, ask user to approve in one shot, proceed to Phase 1.

If `medium` or `large`: continue to 0c (full clarifying questions).

**Always ask the user to confirm the triage if you're not sure:**
> *"I'm sizing this as `<size>`. That means <ceremony level>. Sound right, or should we treat it as smaller/larger?"*

### 0c — Ask clarifying questions (medium/large tasks only)

**Purpose:** Turn a vague task (e.g., "create a backend") into a machine-verifiable Definition of Done.

Ask ONE question at a time. Wait for each answer. Stop asking when you can write a complete Sprint Contract.

**Always ask (in this order):**

1. *"What is the exact outcome you want? Describe what a user or developer can do once this is done."*
2. *"What tech stack should this use? (language, framework, database, auth library, etc.)"*
3. *"What are the 3–5 core features or endpoints this must have? List them concretely."*
4. *"What would you run to prove it works? (e.g., 'npm test passes', 'curl /api/users returns 200', 'the UI shows a login form')"*
5. *"What are the hard constraints? Things that must never happen — e.g., no plaintext passwords, must use TypeScript strict mode, must have 80% test coverage."*
6. *"What is explicitly OUT of scope? Things you do not want built right now."*

**Ask follow-ups based on task type — these are MANDATORY when the task matches:**

| If the task involves... | Ask (do not skip) |
|---|---|
| Backend / API | *"Should this include authentication? If yes — JWT, session, or OAuth?"* |
| Backend / API | *"Does this need pagination, filtering, or sorting on list endpoints?"* |
| Backend / API | *"What are the error response shapes (status code + body schema)?"* |
| Database | *"What database? Does it need migrations, seeds, or fixtures?"* |
| Database | *"Are there any required indexes for query performance?"* |
| Frontend | *"Should this work on mobile? What browser support is required?"* |
| Frontend | *"Should there be loading states, error states, and empty states?"* |
| Any feature | *"Are there existing tests I must not break?"* |
| Refactor | *"What is the pass condition — all existing tests still green?"* |
| Auth / security | *"What password hashing algorithm (bcrypt, argon2)?"* |
| File upload | *"What file types and max size limits?"* |

**Do not skip these — they prevent foreseeable rejections during evaluation.**

### 0c — Write Sprint Contract

Write to `<session_path>/sprint-contract.md`:

```markdown
# Sprint Contract — <SESSION_NAME>
Date: <YYYY-MM-DD>
Task: <original task description>
Status: approved

## Scope (what will be built)
<explicit list of things being built — referenced by name>

## Features
| # | Behavior Description | Verification Command | State |
|---|---------------------|---------------------|-------|
| F01 | <concrete observable behavior, e.g., "POST /api/users creates a user and returns 201 with {id, email}"> | <exact runnable command> | not_started |
| F02 | ... | ... | not_started |

## Definition of Done
All of the following must be true before the task is complete:
- [ ] All features in state: passing
- [ ] Layer 1 (lint + type-check + build) passes
- [ ] Layer 2 (unit + integration tests) passes
- [ ] Layer 3 (E2E / full flow) passes
- [ ] No debug code (console.log, debugger, TODO)
- [ ] Build passes from clean clone
- [ ] All existing tests still pass

## Hard Constraints
<list from user's answers — these are non-negotiable>

## Explicit Exclusions (out of scope)
<list from user — do not build these>

## Verification Commands
| Check | Command |
|-------|---------|
| Lint | <exact command> |
| Type-check | <exact command> |
| Tests | <exact command> |
| E2E / Feature | <exact curl or test command per feature> |
```

### 0d — Get approval before coding

Show the Sprint Contract to the user:

*"Here is the Sprint Contract for your review. Does this match what you want? Should I adjust anything before we start building?"*

**Do not proceed until the user approves.** If they request changes, update the contract and ask again.

---

## PHASE 1 — Bootstrap Check

Before writing any feature code, verify the environment works cleanly.

Run these checks:
1. *Does the project have a package manager / build tool? (package.json, pyproject.toml, Cargo.toml, etc.)*
2. *Can it be set up from scratch?* — run `make setup` or `npm install` or equivalent
3. *Does at least one test pass?* — run `make test` or `npm test` or equivalent

**If any check fails:**
- Create a sub-task `BOOTSTRAP-FIX` at the top of progress.md
- Fix the environment first before any feature work
- Verify bootstrap passes before proceeding

**Bootstrap Contract** — write to `<session_path>/bootstrap-contract.md`:
```markdown
# Bootstrap Contract — <SESSION_NAME>
Date: <YYYY-MM-DD>

## Can Start
- [ ] Setup command: `<command>` → Pass / Fail
- [ ] Dev server: `<command>` → Pass / Fail

## Can Test
- [ ] Test command: `<command>` → Pass / Fail
- [ ] At least one passing test: Yes / No

## Can Verify
- [ ] Lint command: `<command>` → Pass / Fail
- [ ] Type-check: `<command>` → Pass / Fail

## Notes
<anything unusual about the environment>
```

---

## PHASE 2 — Plan & Write Feature List to `progress.md`

### 2a — Decompose each sprint contract feature into micro sub-tasks

For each feature in the Sprint Contract:

**Decomposition rules (from WIP=1 principle):**
- Each sub-task touches a clearly bounded set of files
- Each sub-task has one verification command
- A developer could complete it in < 30 minutes
- If a sub-task can be split into "set up X" and "use X" — split it
- 50+ sub-tasks is normal for a complex task

**Common decomposition patterns:**
| Feature Type | Split into |
|---|---|
| API endpoint | schema migration → model → service method → route handler → input validation → error handling → unit test → integration test |
| Auth system | user model → password hashing → JWT generation → JWT validation middleware → login endpoint → register endpoint → protected route test |
| Database | migration file → model/ORM definition → repository layer → seed data → tests |
| Frontend feature | component shell → props/types → render logic → event handlers → API call → loading/error states → unit test |
| Refactor | map current code → change one call site → update imports → update types → update tests → verify all tests pass |

### 2b — Write ALL sub-tasks to `progress.md` BEFORE dispatching any agent

**This is the single most important rule. Writing first means zero work lost on crash.**

```markdown
## Task: <original task description>
Sprint Contract: <session_path>/sprint-contract.md
Status: in_progress
Started: <YYYY-MM-DD HH:MM>
Total features: <N>
Total sub-tasks: <N>
WIP limit: 1 feature active at a time

## Feature List
| # | Feature | Status | Verification Command |
|---|---------|--------|---------------------|
| F01 | <title> | not_started | <command> |
| F02 | <title> | not_started | <command> |

## Sub-tasks

### FEATURE F01: <feature title>

#### Sub-task F01-01: <short title>
- Status: pending
- Agent: (assigned on dispatch)
- Files to read:
    <path> — <why: e.g., "understand current schema">
- Files to edit:
    <path> — <what change>
- Context: <2-3 sentences — why this sub-task exists, what it achieves>
- What to do:
    1. <specific step>
    2. <specific step>
    3. <specific step>
- What done looks like: <observable outcome>
- Layer 1 check: <lint/typecheck command>
- Layer 2 check: <unit test command>
- Layer 3 check: <e2e / integration command, or "N/A — covered by F01-07">
- Blocked by: none / <sub-task id>
- Notes:

#### Sub-task F01-02: <short title>
...
```

Write Checkpoint 0 to `checkpoints.md`:
```markdown
## Checkpoint 0: Full plan written
- Date: <YYYY-MM-DD HH:MM>
- Task: <task>
- Features: <N>
- Sub-tasks: <N>
- Bootstrap: Pass
- Sprint Contract: approved
- WIP limit: 1 feature at a time
```

---

## PHASE 3 — Execute Features (WIP=1)

**Process one feature at a time. The next feature does not start until the current one has VCR = 1.0 (all sub-tasks verified passing).**

For each feature:

### 3a — Dispatch worker agents in parallel

Use `superpowers:dispatching-parallel-agents`.

Dispatch all sub-tasks for this feature that have `Blocked by: none` simultaneously.

**Agent brief template (fill every field — leave nothing blank):**

```
# Agent Brief — Sub-task <ID>: <TITLE>
Project: <PROJECT_NAME>
Session: <SESSION_NAME>
Feature: <F0N title>
Date: <YYYY-MM-DD>

---

## YOUR SUB-TASK

<paste the full sub-task entry from progress.md>

---

## HARD CONSTRAINTS (non-negotiable — from Sprint Contract)
<list from sprint contract>

---

## PROJECT RULES (follow all)
<full contents of rules.md>

---

## ARCHITECTURE CONTEXT
<relevant section of architecture.md for this sub-task>

---

## CODEBASE MAP
<relevant section of file-tree.md for the files this sub-task touches>

---

## PRIOR DECISIONS THIS SESSION
<relevant entries from decisions.md to avoid contradictions>

---

## THREE-LAYER TERMINATION CHECK
You must pass all three layers before marking done:

Layer 1 — Syntax/Static (run first):
  Command: <Layer 1 check from sub-task entry>
  Pass condition: zero errors, zero warnings

Layer 2 — Runtime/Tests (run only if Layer 1 passes):
  Command: <Layer 2 check from sub-task entry>
  Pass condition: all tests pass, no failures

Layer 3 — System/E2E (run only if Layer 2 passes):
  Command: <Layer 3 check from sub-task entry, or N/A>
  Pass condition: <expected output>

If Layer 1 FAILS: stop, do not run Layer 2. Mark blocked.
If Layer 2 FAILS: stop, do not run Layer 3. Mark blocked.
If Layer 3 FAILS: mark blocked.

---

## YOUR OBLIGATIONS AFTER COMPLETING WORK

1. Run all three verification layers in order. Document each result.
2. Update progress.md sub-task entry:
   - Status: completed (all 3 layers pass) OR blocked (any layer fails)
   - Layer 1 result: Pass/Fail + command run
   - Layer 2 result: Pass/Fail + command run
   - Layer 3 result: Pass/Fail + N/A + command run
   - Notes: what was done, surprises, edge cases
3. Update decisions.md — EVERY decision made (no matter how small):
   - Taken: what was decided, why, what alternatives were rejected
   - Rejected: what was not chosen, why
4. Update verification-notes.md — one row per layer per sub-task
5. Add checkpoint to checkpoints.md
6. If BLOCKED: write full error output to Failures Log in verification-notes.md

---

## AGENT SCOPE RULES
- Only edit files listed in "Files to edit" above
- Read full files before editing any part
- Match the exact code style of surrounding code
- Do not refactor outside scope of this sub-task
- Do not fix bugs discovered outside scope — log them in decisions.md
- Do not rename or delete files unless explicitly stated
- Write no comments unless the WHY is genuinely non-obvious
```

### 3b — Dispatch evaluator agent (after each sub-task completes)

**The agent that implemented the code must NOT evaluate it. A separate evaluator agent runs independently.**

Send this brief to the evaluator:

```
# Evaluator Brief — Sub-task <ID>: <TITLE>

You are an ADVERSARIAL EVALUATOR. Your goal is to find reasons this code is WRONG, not reasons it is right.

You did NOT write this code. You are skeptical of it. Your job is to:
1. Try to break it — actively find a counter-example, edge case, or failure mode
2. Run the verification commands fresh (do not trust the worker's reported results)
3. Score it only after attempting to find at least one real problem

**You must report at least one attempted attack/counter-example.** If you cannot find a single thing wrong, document what you tried and why it didn't find anything. A passing eval with no attempted counter-example is INVALID and must be rejected.

---

## WHAT WAS IMPLEMENTED
<paste the sub-task entry with completion notes>

## FILES THAT WERE CHANGED
<list files and brief description of changes>

## WHAT TO EVALUATE

1. CORRECTNESS
   - Does the code actually do what the sub-task describes?
   - Are all edge cases handled?
   - Are there any logical errors?

2. THREE-LAYER VERIFICATION (run independently)
   - Layer 1: <command> → did it pass? Show output.
   - Layer 2: <command> → did it pass? Show output.
   - Layer 3: <command> → did it pass? Show output.

3. SPRINT CONTRACT COMPLIANCE
   - Does this implementation respect all hard constraints?
   - Does it respect the explicit exclusions?

4. CODE QUALITY (from rules.md)
   - Naming conventions correct?
   - Architecture boundaries respected?
   - No forbidden patterns?
   - No debug code (console.log, TODO, commented-out code)?
   - Error handling present where needed?

5. SECURITY
   - No secrets or credentials in code?
   - User input validated at boundaries?
   - No SQL injection, XSS, or eval() risks?

---

## EVALUATOR RUBRIC

Score each dimension 1–5:

| Dimension | Score (1-5) | Notes |
|-----------|-------------|-------|
| Correctness — all verification layers pass | | |
| Architecture compliance — boundaries respected | | |
| Test coverage — main flow + edge cases | | |
| Code conventions — matches codebase style | | |
| Security — no obvious vulnerabilities | | |
| Error handling — failures handled explicitly | | |

**Overall: <average>/5**

## VERDICT
- PASS (≥ 4.0 average, all verification layers pass): ready for next sub-task
- NEEDS WORK (3.0–3.9 or any layer failing): list specific issues, block sub-task
- FAIL (< 3.0): list all issues, mark sub-task blocked

---

## REQUIRED: Counter-example attempt
Before scoring, attempt at least ONE of:
- Find an edge case the code does not handle (null, empty, boundary, very large input)
- Find a security weakness (injection, missing auth check, unvalidated input)
- Find a concurrency bug (race condition, missing lock, shared state)
- Find an architecture violation (layer crossing, forbidden import)
- Find a test that should exist but does not

Document what you tried, what you found (or did not find), and why.

## OUTPUT FORMAT
Write your evaluation to:
<session_path>/evaluations/<sub-task-id>-eval.md

Required sections:
1. Three-layer verification (run independently — show actual output)
2. Counter-example attempted (mandatory — what you tried to break)
3. Quality scores (1-5 per dimension)
4. Issues found (critical / important / minor)
5. Verdict (PASS / NEEDS WORK / FAIL)
```

**An evaluation without a documented counter-example attempt is INVALID.** Reject it and re-dispatch.

### 3c — Handle evaluation result

**If evaluator says PASS:**
- Mark sub-task `completed` in progress.md
- Update Obsidian files (see 3d)
- Dispatch blocked sub-tasks whose dependency just cleared

**If evaluator says NEEDS WORK or FAIL:**
- Mark sub-task `blocked` in progress.md
- Create a new sub-task `<ID>-fix` with the evaluator's specific issues
- Dispatch a new worker agent with the evaluation as brief context
- Do NOT move to the next sub-task until this passes evaluation

### 3d — Update Obsidian after EVERY sub-task (never batch)

After each sub-task completes evaluation:

**`progress.md`** — update the sub-task entry:
```markdown
#### Sub-task F01-01: <title>
- Status: completed
- Agent: <worker agent>
- Evaluator: <eval agent>
- Evaluator score: <N>/5
- Layer 1: Pass — `<command output summary>`
- Layer 2: Pass — `<N tests passing>`
- Layer 3: Pass — `<e2e result>`
- Notes: <what was done, decisions made>
```

**`decisions.md`** — add every decision from this sub-task:
```markdown
| <decision> | <why it was right> | <what was rejected and why> | F01-01 |
```

**`verification-notes.md`** — add rows:
```markdown
| F01-01 | <title> | Layer 1 | `<command>` | <expected> | <actual> | Pass |
| F01-01 | <title> | Layer 2 | `<command>` | <expected> | <actual> | Pass |
| F01-01 | <title> | Layer 3 | `<command>` | <expected> | <actual> | Pass |
```

**`checkpoints.md`** — add entry:
```markdown
## Checkpoint F01-01: Sub-task complete
- Date: <YYYY-MM-DD HH:MM>
- What was done: <one clear sentence>
- Files changed: <list>
- Evaluator score: <N>/5
- All 3 layers: Pass
- Next: <next sub-task title>
```

### 3e — Feature completion gate

After ALL sub-tasks for a feature reach `completed`:

**Run feature-level E2E verification:**
- Execute the verification command from the Sprint Contract Feature List
- This is the end-to-end proof that the feature works as a whole

**Update Feature List in progress.md:**
```markdown
| F01 | <title> | passing | `<command>` ← ran successfully |
```

**VCR check (Verified Completion Rate):** verified features / activated features must = 1.0 before activating the next feature.

If VCR < 1.0 → fix the incomplete feature before starting the next one.

---

## PHASE 4 — Post-Task Clean State Check

After ALL features reach `passing`:

### 4a — Five-dimension clean state verification

Run all checks and record results:

| Dimension | Command | Result |
|-----------|---------|--------|
| 1. Build | `<build command>` | Pass/Fail |
| 2. All tests | `<test command>` | Pass/Fail |
| 3. No debug artifacts | `git diff --name-only main \| xargs grep -nE "console\.log\|debugger\|TODO\|FIXME\|\.only\(" 2>/dev/null` (checks ONLY files changed in this task — avoids false positives from vendored code) | Clean/Issues |
| 4. Lint | `<lint command>` | Pass/Fail |
| 5. Startup | `<dev server start>` | Pass/Fail |

**If any dimension fails:** fix it before declaring done. Session is not complete until all 5 pass.

### 4b — Update Quality Document

Write/update `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/quality-document.md`:

```markdown
# Quality Document — <PROJECT_NAME>
Last updated: <YYYY-MM-DD>

## Module Health

| Module / Feature | Build | Tests | Agent-Readable | Architecture | Conventions |
|------------------|-------|-------|----------------|-------------|------------|
| <feature/module> | ✓/✗ | ✓/✗ | Easy/Difficult | Compliant/Violations | Followed/Partially |

## Recent Sessions
| Session | Date | Features Added | Final Score | Notes |
|---------|------|---------------|-------------|-------|
| <SESSION_NAME> | <date> | <list> | <N>/5 | |

## Known Issues
| Issue | Module | Severity | Logged |
|-------|--------|----------|--------|

## Architecture Boundary Violations Found
| Violation | File | Rule Violated | Status |
|-----------|------|--------------|--------|
```

### 4c — Update DECISIONS.md summary

Add a session-level summary to `decisions.md`:

```markdown
## Session Summary: <SESSION_NAME>
Features completed: <N>
Total sub-tasks: <N>
Evaluator average score: <N>/5
Key decisions: <list of most important decisions made>
Rejected approaches: <list of approaches considered but not taken>
Known trade-offs accepted: <list>
```

### 4d — Final checkpoint

Write to `checkpoints.md`:
```markdown
## Checkpoint FINAL: All features complete + clean state verified
- Date: <YYYY-MM-DD HH:MM>
- Task: <original task>
- Features: <N>/<N> passing
- Sub-tasks: <N>/<N> complete
- Evaluator avg score: <N>/5
- Five-dimension clean state: Pass
- Build: Pass
- Tests: Pass
- No debug artifacts: Pass
- Lint: Pass
- Startup: Pass
```

### 4e — Update `.harness-state`

```
active_session: <SESSION_NAME>-<date>
session_path: <path>
status: completed
task_completed: <YYYY-MM-DD HH:MM>
features_passed: <N>/<N>
```

### 4f — Print final summary + suggest next steps

```
✓ Task complete — <SESSION_NAME>

  Features:   <N>/<N> passing
  Sub-tasks:  <N>/<N> complete
  Evaluator:  avg <N>/5
  Clean state: ✓ all 5 dimensions pass

  Five-dimension check:
    ✓ Build passes
    ✓ All tests pass
    ✓ No debug artifacts
    ✓ Lint passes
    ✓ Startup works

  Session files:
    ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SESSION_NAME>-<date>/

  Quality document updated:
    ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/quality-document.md

What's next:
  → Run /harness-export to generate a PR-ready TASK-REPORT.md from this session
  → If you've completed 5+ sessions, run /harness-retro to learn from past patterns
  → Run /harness-session to start a new session for the next task
```

---

## Crash Resilience

At any point — mid-clarification, mid-dispatch, mid-evaluation, after switching tools:

1. Run `/harness-session` → auto-detects from `.harness-state`
2. Run `/harness-task` → reads `progress.md`, finds pending/in-progress work
3. If mid-Phase 0: sprint contract may be incomplete → resume clarifying questions
4. If mid-Phase 3: find sub-tasks with `pending`/`in_progress` → re-dispatch
5. Zero re-briefing — every entry is self-sufficient

Works identically in Claude Code, Codex, Cursor, or any agent tool.

---

## The Five Failure Layers (diagnose every failure here first)

| Layer | Question | Fix |
|-------|----------|-----|
| 1. Task specification | Was the DoD explicit and machine-verifiable? | Add clarifying questions, rebuild Sprint Contract |
| 2. Context provision | Did the agent have architecture rules and constraints? | Update map files and rules.md |
| 3. Execution environment | Were dependencies installed, versions correct? | Fix bootstrap contract |
| 4. Verification feedback | Did the agent have runnable verification commands? | Add Layer 1/2/3 commands to sub-tasks |
| 5. State management | Did the agent know where to pick up after a reset? | Update progress.md and checkpoints.md |

**Never attribute a failure to "the model isn't good enough" before checking all five layers.**

---

## Hard Rules — Never Violate

| Rule | Source | Why |
|------|--------|-----|
| No coding before Sprint Contract is approved | L07, L08 | Prevents scope creep and wasted work |
| Write ALL sub-tasks to progress.md before dispatching | L05 | Crash loses zero work |
| One feature active at a time (WIP=1) | L07 | 37% higher completion rate |
| Three-Layer check in order — never skip | L09 | Unit tests miss 5 defect categories |
| Worker ≠ Evaluator (separate agents) | L09 | Agents over-rate their own work |
| Update Obsidian after EVERY step — never batch | L05, L12 | Any crash leaves recoverable state |
| VCR must = 1.0 before next feature | L07 | Prevents compounding incomplete work |
| Never declare done without running verification command | L01, L09 | Verification Gap is real and systematic |
| Five-dimension clean state at end | L12 | Entropy grows by default — must be countered |
| Do not refactor until current feature passes all 3 layers | L09 | Refactor blurs verified/unverified boundary |
