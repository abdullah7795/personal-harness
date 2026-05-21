# personal-harness
GitHub: https://github.com/abdullah7795/personal-harness

> **The only AI dev tool that refuses to write code until it knows what "done" means** — and builds a queryable knowledge graph of every decision in Obsidian.

Three things no other AI workflow tool gives you:

🛑 **The Sprint Contract gate.** Every task starts with clarifying questions and a machine-verifiable Definition of Done. AI does not touch your code until you approve.

🧠 **A growing memory in Obsidian.** Every decision, alternative rejected, evaluation, and verification result is preserved as a navigable knowledge graph. Months later you can still answer *"why did we choose this?"*

🔁 **A harness that gets smarter with use.** `/harness-retro` mines past sessions for recurring failures and decisions, then proposes patches to rules and questions. Use it more → it gets sharper.

Plus: crash-resilient across tools (switch Claude ↔ Codex ↔ Cursor mid-task), parallel agent execution with independent adversarial evaluators, three-layer verification (syntax → tests → E2E), and PR-ready exportable reports.

Built on 12 proven harness engineering principles from Anthropic and OpenAI research.

---

## Table of Contents

1. [What This Is](#what-this-is)
2. [The 5 Skills (Commands)](#the-5-skills-commands)
3. [Requirements](#requirements)
4. [Installation](#installation)
5. [Quick Start — Real Example](#quick-start--real-example)
6. [Command Reference — Full Details](#command-reference--full-details)
7. [Task Execution — The 5 Phases](#task-execution--the-5-phases)
8. [File Reference — What Each File Does](#file-reference--what-each-file-does)
9. [Plugin Directory Structure](#plugin-directory-structure)
10. [Obsidian Knowledge Base Structure](#obsidian-knowledge-base-structure)
11. [Built on 12 Harness Engineering Principles](#built-on-12-harness-engineering-principles)
12. [Crash Resilience — How Resume Works](#crash-resilience--how-resume-works)
13. [Troubleshooting](#troubleshooting)
14. [Uninstall](#uninstall)

---

## What This Is

A skill plugin that turns any project into a production-grade AI development environment.

**The problem it solves:** AI agents (Claude, Codex, Cursor) are inconsistent. They forget context, declare "done" before testing, take shortcuts when context runs low, and produce different output every run.

**What this gives you:**
- Every task gets clarified into an explicit, machine-verifiable Definition of Done
- Every task is split into micro sub-tasks executed in parallel
- Every sub-task is verified independently by a separate evaluator agent
- Every step is checked across 3 layers: syntax → tests → end-to-end
- Every decision is logged with full reasoning
- Every step is saved to Obsidian — you can stop and resume anywhere, anytime, with any agent

**Result:** Production-ready output every time. No "I think it works" — only "the verification command says it works."

---

## The 5 Skills (Commands)

| Command | When to run | What it does |
|---------|-------------|-------------|
| `/harness-proj-init` | **Once per project** | Reads codebase, generates Obsidian knowledge base, asks product questions, writes rules.md, updates AGENTS.md + CLAUDE.md |
| `/harness-session` | **Start of every work session** | Creates a new uniquely-named session (e.g., `NorthernLights`) or resumes an existing one, runs BLOCKING cold-start test |
| `/harness-task` | **Every time a task is given** | 5-phase execution: triage + clarify → bootstrap → plan → parallel workers + adversarial evaluators + 3-layer verification → clean state |
| `/harness-retro` | **After every 5–10 sessions** | Mines past sessions for recurring failures and decisions, proposes patches to rules.md. **This is the feedback loop that compounds value.** |
| `/harness-export` | **After completing a task** | Generates PR-ready TASK-REPORT.md with sprint contract + decisions + verification proof + evaluator scores |

---

## Requirements

- **Claude Code** installed — [claude.ai/code](https://claude.ai/code)
- **Obsidian** installed with a vault at `~/Documents/Obsidian/` — [obsidian.md](https://obsidian.md)
- **superpowers plugin** installed — needed for `superpowers:dispatching-parallel-agents`
  - Install: `/plugin marketplace add obra/superpowers && /plugin install superpowers@superpowers-dev`

---

## Installation

**Option A — via Claude Code plugin system (recommended):**
```bash
/plugin marketplace add abdullah7795/personal-harness
/plugin install personal-harness
```

**Option B — via npx skills CLI (multi-agent support):**
```bash
npx skills add abdullah7795/personal-harness
```

**Option C — direct git clone:**
```bash
git clone https://github.com/abdullah7795/personal-harness ~/.claude/plugins/personal-harness
```

Restart Claude Code after installation. The three skills are available immediately.

---

## Quick Start — Real Example

Say you want to build a backend API for a task management app.

### Step 1: Initialize the project (once)
```
cd ~/projects/task-manager
/harness-proj-init
```

The skill will:
1. Read your codebase (if any), detect tech stack
2. Ask you 3–5 product questions like *"What's the primary action a user comes to this product to do?"*
3. Create `~/Documents/Obsidian/creator/agent-memory/task-manager/` with all map files
4. Write a `rules.md` with 18 sections (workflow, security, code quality, architecture, etc.)
5. Append harness instructions to your `AGENTS.md` and `CLAUDE.md`

### Step 2: Start a session
```
/harness-session
```
Output: `✓ Session NorthernLights-2026-05-21 started`

### Step 3: Give your task
```
/harness-task

create a backend API with user auth, task CRUD, and PostgreSQL
```

The skill enters **Phase 0** and asks:
- *"What is the exact outcome you want?"*
- *"What tech stack? (Node.js, Python, etc.?)"*
- *"What endpoints exactly?"*
- *"What's your verification — npm test passing? curl returns 200?"*
- *"Hard constraints — no plaintext passwords, TypeScript strict mode?"*
- *"What's out of scope?"*
- *"Authentication style — JWT, session, OAuth?"*
- *"Pagination on list endpoints?"*

After your answers, it writes a **Sprint Contract** and shows it to you:

```
Sprint Contract for review:
Features:
F01 — POST /api/users creates user, returns 201 with {id, email}
      Verification: `npm test -- users.test.ts && curl -X POST...`
F02 — POST /api/auth/login returns JWT in HttpOnly cookie
F03 — POST/GET/PUT/DELETE /api/tasks (full CRUD)
F04 — All tasks require valid JWT in cookie

Hard constraints: bcrypt for passwords, TS strict, 80% coverage
Exclusions: no email verification, no password reset (next sprint)

Approve to proceed?
```

After you approve, **Phase 1** verifies the environment, then **Phase 2** writes ~30 micro sub-tasks to `progress.md` BEFORE dispatching anything.

**Phase 3** dispatches parallel agents — each gets its own sub-task brief. After each sub-task, a separate **evaluator agent** scores it 1–5 across 6 dimensions and verifies all 3 layers (lint → tests → E2E). The session files in Obsidian update after every single step.

**Phase 4** runs the five-dimension clean state check (build, tests, no debug code, lint, startup) and updates the Quality Document.

Final output: `✓ Task complete — 30/30 sub-tasks, avg evaluator 4.7/5, all 5 clean state dimensions pass`

---

## Command Reference — Full Details

### `/harness-proj-init` — Project Initialization

**When:** Run ONCE per project, before any other harness commands.

**What it does (8 steps):**

1. **Identify project** — gets current directory name as `<PROJECT_NAME>`. If monorepo, asks which workspace.
2. **Read codebase in parallel:**
   - Generate file tree (excludes `node_modules/`, `.git/`, `dist/`, `build/`, etc.)
   - Detect DB layer (Prisma, Mongoose, SQLAlchemy, Sequelize, TypeORM, etc.)
   - Detect architecture (Next.js, Express, FastAPI, Django, NestJS, Rails, etc.)
   - Detect auth, background jobs, API specs, testing stack, CI/CD, env vars
3. **Create Obsidian folders** at `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/`
4. **Write 4 map files:**
   - `file-tree.md` — full project tree + key entry points
   - `db-structure.md` — ORM, models with fields, migrations, indexes
   - `architecture.md` — tech stack, layers, request flow, external services
   - `product-info.md` — (filled in Step 5)
5. **Ask 3–5 product questions** — always 2 mandatory questions + conditional ones based on what was detected (e.g., if payment module found → "What payment providers?")
6. **Generate `rules.md`** with 18 sections covering harness workflow, DoD, WIP=1, three-layer check, worker≠evaluator, naming, architecture, testing, code quality, security, agent scope, forbidden patterns, git, context management, session continuity, error handling, performance, five failure layers
7. **Append to `AGENTS.md` and `CLAUDE.md`** — paths to Obsidian files, mandatory workflow, session file structure
8. **Initialize `quality-document.md`** — empty health tracker, will be updated by `/harness-task`

**Outputs:**
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/map/file-tree.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/map/db-structure.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/map/architecture.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/map/product-info.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/rules.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/quality-document.md`
- Updated `AGENTS.md` and `CLAUDE.md` in the project repo

**Safe to re-run:** Yes. Overwrites map files but checks for duplicate AGENTS.md / CLAUDE.md sections.

---

### `/harness-session` — Start or Resume a Session

**When:** At the beginning of EVERY work session, before any task.

**What it does:**

1. **Identify project** from current directory
2. **Verify harness exists** for this project (`/harness-proj-init` must have run first)
3. **Check `.harness-state`** for interrupted sessions
4. **Ask: new or existing session?**

**If NEW session:**
- Generates unique cosmic/nature name (50 options: `MilkyWay`, `NorthernLights`, `Voyager`, `ArcticFox`, `CrimsonDawn`, etc.)
- Creates session folder: `sessions/<SessionName>-<YYYY-MM-DD>/`
- Creates 4 session files: `progress.md`, `decisions.md`, `verification-notes.md`, `checkpoints.md`
- Creates `evaluations/` subfolder for evaluator agent reports
- Writes `.harness-state` with active session pointer
- Runs **Cold-Start Test** — verifies repo can answer 5 questions from files alone:
  1. What is this system?
  2. How is it organized?
  3. How do I run it?
  4. How do I verify it?
  5. Where are we now?
- Loads all map files + rules.md into context

**If EXISTING session:**
- Lists all sessions sorted by date with last checkpoint number
- User picks one
- Reads `progress.md`, `checkpoints.md`, `decisions.md`, `verification-notes.md`
- Identifies pending/in-progress sub-tasks
- Updates `.harness-state` with `resumed: <datetime>`
- Adds resume checkpoint

**Clock-out routine** (run before ending session):
- Update progress.md with current state
- Run verification commands
- Five-dimension clean state check
- Commit completed work
- Update `.harness-state` status
- Add clock-out checkpoint

---

### `/harness-task` — Execute a Task

**When:** EVERY time a task is given. Requires active session.

**What it does — 5 phases in strict order:**

#### Phase 0 — Task Intake & Definition of Done
- Reads `.harness-state` for active session
- Checks for unfinished work in `progress.md`
- Asks clarifying questions one at a time (mandatory + task-type-specific)
- Writes **Sprint Contract** with feature list (behavior + verification + state)
- **Stops and asks for your approval before any coding**

#### Phase 1 — Bootstrap Check
- Verifies environment works:
  - Setup command runs cleanly
  - Test runner executes
  - At least one passing test
  - Lint runs
  - Build runs
- Writes **Bootstrap Contract** to session folder
- If bootstrap fails → creates fix sub-task at top of progress.md, fixes before any feature work

#### Phase 2 — Plan & Write Feature List
- Decomposes each feature into micro sub-tasks (50+ for complex tasks is normal)
- Each sub-task has 11 fields: title, status, agent, files-to-read, files-to-edit, context, what-to-do (numbered steps), what-done-looks-like, layer-1/2/3 verification commands, blocked-by, notes
- **Writes ALL sub-tasks to `progress.md` BEFORE dispatching anything** (crash-resilient)
- Writes Checkpoint 0 to `checkpoints.md`

#### Phase 3 — Execute (WIP=1, one feature at a time)
For each feature in the Sprint Contract:
- **Dispatch parallel worker agents** for all sub-tasks with `Blocked by: none` (using `superpowers:dispatching-parallel-agents`)
- Each worker agent gets: full sub-task entry, hard constraints, project rules, architecture context, file tree, prior decisions, 3-layer verification commands
- After each worker completes, **dispatch a separate evaluator agent** to score 1–5 on 6 dimensions (correctness, architecture, test coverage, conventions, security, error handling)
- **Three-Layer Termination Check per sub-task:**
  - Layer 1: lint + type-check + build (zero errors)
  - Layer 2: unit + integration tests (all pass)
  - Layer 3: E2E / full flow (expected output)
  - If any layer fails → mark `blocked`, document fully, create fix sub-task
- **Update all 4 Obsidian session files IMMEDIATELY after each sub-task** (never batched)
- **Verified Completion Rate (VCR)** must equal 1.0 before activating next feature

#### Phase 4 — Clean State Check
- **Five-dimension verification:**
  1. Build passes
  2. All tests pass (including pre-existing tests)
  3. No debug artifacts (`console.log`, `debugger`, `TODO`, `FIXME`)
  4. Lint passes
  5. Standard startup works
- Updates Quality Document with session results
- Writes session-level summary to decisions.md
- Final checkpoint to checkpoints.md
- Updates `.harness-state` to `completed`
- Prints final summary with all metrics

**Hard rules (never violated):**
- No coding before Sprint Contract is approved
- All sub-tasks written to progress.md before dispatching
- One feature active at a time
- 3-layer check in order, never skip
- Worker ≠ Evaluator (different agents)
- Update Obsidian after EVERY step, never batch
- VCR = 1.0 before next feature
- Never declare done without running verification command

---

## Task Execution — The 5 Phases

| Phase | What happens | Output Files |
|-------|--------------|--------------|
| **Phase 0** — Intake | Clarifying questions → Sprint Contract → user approval | `sprint-contract.md` |
| **Phase 1** — Bootstrap | Environment verification (setup, test, lint, build) | `bootstrap-contract.md` |
| **Phase 2** — Plan | Decompose into micro sub-tasks, write ALL to progress.md FIRST | full sub-task list in `progress.md` |
| **Phase 3** — Execute | Parallel workers + independent evaluators + 3-layer verification per sub-task | code + `evaluations/*-eval.md` |
| **Phase 4** — Clean State | Five-dimension verification + Quality Document update | clean repo + updated `quality-document.md` |

---

## File Reference — What Each File Does

### Project-level files (in your repo)
| File | Purpose |
|------|---------|
| `AGENTS.md` | Universal agent instructions — appended with harness workflow during `/harness-proj-init` |
| `CLAUDE.md` | Claude Code instructions — same appended content as AGENTS.md |

### Obsidian — Project map files (per-project, persistent)
| File | Purpose | Updated by |
|------|---------|-----------|
| `map/file-tree.md` | Project file structure + entry points + notable directories | `/harness-proj-init` |
| `map/db-structure.md` | ORM, models, fields, migrations, indexes | `/harness-proj-init` |
| `map/architecture.md` | Tech stack, layers, request flow, external services, env vars | `/harness-proj-init` |
| `map/product-info.md` | Primary purpose, critical invariants, user roles, dependencies | `/harness-proj-init` (filled by your answers) |
| `rules.md` | 18 sections of rules: workflow, DoD, WIP=1, three-layer, security, etc. | `/harness-proj-init` |
| `quality-document.md` | Codebase health over time, module scores, known issues | `/harness-task` (Phase 4) |
| `.harness-state` | Active session pointer, status, paths | `/harness-session` and `/harness-task` |

### Obsidian — Per-session files (one set per session)
| File | Purpose | Updated by |
|------|---------|-----------|
| `progress.md` | Task description, feature list, sub-task breakdown with live status | `/harness-task` after every step |
| `decisions.md` | Every decision taken/rejected with full reasoning, open questions | `/harness-task` after every step |
| `verification-notes.md` | Three-layer results per sub-task, failures log, final verification | `/harness-task` after every step |
| `checkpoints.md` | Numbered completion markers (recovery record) | `/harness-task` after every step |
| `sprint-contract.md` | Scope, features, Definition of Done, hard constraints, exclusions | `/harness-task` Phase 0 |
| `bootstrap-contract.md` | Environment verification results | `/harness-task` Phase 1 |
| `evaluations/<sub-task>-eval.md` | One per sub-task: evaluator scores + verdict | Evaluator agent in Phase 3 |

---

## Plugin Directory Structure

What's inside this plugin:

```
personal-harness/
├── README.md                              ← this file
├── plugin.json                            ← plugin manifest
├── skills/
│   ├── harness-proj-init/
│   │   ├── SKILL.md                       ← /harness-proj-init definition (~700 lines)
│   │   └── metadata.json
│   ├── harness-session/
│   │   ├── SKILL.md                       ← /harness-session definition (~380 lines)
│   │   └── metadata.json
│   └── harness-task/
│       ├── SKILL.md                       ← /harness-task definition (~660 lines)
│       └── metadata.json
└── templates/
    ├── agents-append.md                   ← content appended to AGENTS.md
    ├── claude-append.md                   ← content appended to CLAUDE.md
    ├── rules-template.md                  ← 18-section rules.md template
    ├── sprint-contract.md                 ← Phase 0 contract template
    ├── bootstrap-contract.md              ← Phase 1 contract template
    ├── evaluator-rubric.md                ← evaluator agent scoring template
    └── quality-document.md                ← codebase health template
```

---

## Obsidian Knowledge Base Structure

What gets created in Obsidian per project:

```
~/Documents/Obsidian/
└── creator/
    └── agent-memory/
        └── <your-project>/                      ← one folder per project
            ├── .harness-state                   ← active session tracker
            ├── map/                             ← navigation pointers (not full content)
            │   ├── file-tree.md
            │   ├── db-structure.md
            │   ├── architecture.md
            │   └── product-info.md
            ├── rules.md                         ← 18 sections of rules
            ├── quality-document.md              ← health tracker
            ├── retrospectives/                  ← created by /harness-retro
            │   ├── retro-2026-05-21.md
            │   └── retro-2026-05-21-patches.md
            └── sessions/
                ├── NorthernLights-2026-05-21/   ← one session (memorable name)
                │   ├── progress.md
                │   ├── decisions.md
                │   ├── verification-notes.md
                │   ├── checkpoints.md
                │   ├── sprint-contract.md
                │   ├── bootstrap-contract.md
                │   └── evaluations/
                │       ├── F01-01-eval.md
                │       ├── F01-02-eval.md
                │       └── ...
                ├── MilkyWay-2026-05-22/         ← another session
                │   └── ...
                └── Voyager-2026-05-23/          ← another session
                    └── ...
```

**Why Obsidian?** You can open the vault and read all your project history, decisions, and progress as a navigable knowledge base. Every agent decision is preserved with full reasoning — useful for code reviews, post-mortems, or just remembering "why did we choose this approach?"

---

## Built on 12 Harness Engineering Principles

Each skill implements proven principles from harness engineering research (Anthropic, OpenAI experiments):

| Principle | Lecture | Implementation |
|-----------|---------|---------------|
| Five failure layers — diagnose before blaming model | L01 | Rules section 18, harness-task error handling |
| Explicit machine-verifiable Definition of Done | L01 | Phase 0 Sprint Contract |
| AGENTS.md as router, not encyclopedia | L02 | agents-append.md under 50 lines |
| Cold-start test — 5 questions from files alone | L03 | `/harness-session` step 4 |
| Critical rules at top/bottom (Lost-in-Middle effect) | L04 | rules.md sections 1–5 are mandatory workflow |
| PROGRESS.md + DECISIONS.md + clock-in/clock-out | L05 | All 4 session files, clock-in on session start, clock-out routine |
| Bootstrap Contract before feature code | L06 | Phase 1 of `/harness-task` |
| WIP=1 + Verified Completion Rate gating | L07 | One feature active at a time, VCR check |
| Feature list triple (behavior + verification + state) | L08 | Sprint Contract feature table |
| Three-Layer Termination Check (syntax → tests → E2E) | L09 | Per sub-task in Phase 3 |
| Worker ≠ Evaluator (separate agents) | L09 | Phase 3b dispatches evaluator after worker |
| E2E as true verification | L10 | Layer 3 mandatory per feature |
| Agent-oriented error messages (what/why/how-to-fix) | L10 | Rules section 11 |
| Sprint Contract + Evaluator Rubric + Task Traces | L11 | Phase 0 contract, evaluator-rubric.md, full session files |
| Five-dimension clean state | L12 | Phase 4 verification |
| Quality Document — codebase health tracking | L12 | Updated per task in Phase 4 |

---

## Crash Resilience — How Resume Works

The skill is designed so that AT ANY POINT — mid-clarification, mid-dispatch, mid-evaluation, after closing your laptop, after switching from Claude Code to Codex/Cursor — work can continue with zero re-briefing.

**How it works:**

1. **`.harness-state`** stores the active session pointer
2. **`progress.md`** stores every sub-task with full self-sufficient details (files to read, files to edit, what to do, verification command)
3. **`checkpoints.md`** stores numbered completion markers (recovery record)
4. **`decisions.md`** stores every decision made + alternatives rejected with reasoning

**Resume procedure:**
1. Open any agent tool (Claude Code, Codex, Cursor, Windsurf)
2. Run `/harness-session` → it auto-detects from `.harness-state` and offers to resume the active session
3. Run `/harness-task` → reads `progress.md`, finds sub-tasks with status `pending` or `in_progress`, continues from there

**No context loss:** Every sub-task entry is self-sufficient — a brand new agent with zero prior knowledge can execute it from the entry alone.

**Cross-tool guarantee:** Works identically in Claude Code, Codex, Cursor, Windsurf, or any agent tool that can read files from disk.

---

## Troubleshooting

| Problem | Cause | Solution |
|---------|-------|---------|
| `No harness found for <project>` | `/harness-proj-init` was not run | Run `/harness-proj-init` from project directory first |
| `Obsidian vault not found` | Vault not at `~/Documents/Obsidian/` | Install Obsidian and create a vault at that path |
| `No active session` (when running `/harness-task`) | `.harness-state` missing | Run `/harness-session` first |
| `superpowers:dispatching-parallel-agents not found` | superpowers plugin missing | Install: `/plugin install superpowers@superpowers-dev` |
| Sub-task marked `blocked` | Verification failed | Read `verification-notes.md` Failures Log; create fix sub-task |
| Want to skip Phase 0 questions | You can't | The Sprint Contract is mandatory for production-ready output. If you really want speed, manually write the contract first. |
| Sessions piling up | Old completed sessions never auto-delete | Manually delete from `sessions/` folder, or archive to a separate vault |
| Quality Document keeps showing old failures | Not auto-cleared | Manually edit `quality-document.md` to resolve old items |
| Re-running `/harness-proj-init` | Safe — overwrites map files, checks before appending to AGENTS.md | No special action needed |
| Two projects with same name | One folder gets shared in Obsidian | Rename one project directory before running `/harness-proj-init` |

---

## Uninstall

**To remove the plugin:**
```bash
# Via plugin system
/plugin uninstall personal-harness

# Or via direct removal
rm -rf ~/.claude/plugins/personal-harness
```

**To remove project harness data** (per project):
```bash
rm -rf ~/Documents/Obsidian/creator/agent-memory/<project-name>
```
*Note: Your AGENTS.md and CLAUDE.md will still have the appended harness section. Remove manually if desired.*

**To remove the harness section from AGENTS.md / CLAUDE.md:**
Look for the `## Harness — <PROJECT_NAME>` section (starts with `---`) and delete it through to end of file.

---

## License & Credits

MIT License — feel free to fork, modify, and redistribute.

Built on principles from:
- [Anthropic harness engineering research](https://www.anthropic.com/research)
- [OpenAI Codex experiments](https://openai.com)
- [Superpowers plugin](https://github.com/obra/superpowers) for parallel agent dispatch
