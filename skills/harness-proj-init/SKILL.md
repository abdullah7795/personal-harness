---
name: harness-proj-init
description: Use when setting up a new project with the personal harness system for the first time. Run once per project. Reads the codebase, creates an Obsidian knowledge base with map files (file tree, DB structure, architecture, product info), asks targeted product questions based on what it finds, generates a comprehensive rules.md, and appends harness workflow instructions to AGENTS.md and CLAUDE.md. Invoke with /harness-proj-init.
---

# harness-proj-init

Initialize a project's Obsidian-backed harness. Run **once per project** before any sessions begin. Re-running is safe — it overwrites map files but never duplicates AGENTS.md/CLAUDE.md entries.

---

## What This Does

1. Reads the codebase → generates map files (file tree, DB structure, architecture)
2. Asks targeted product questions based on what it finds
3. Creates Obsidian knowledge base at `~/Documents/Obsidian/creator/agent-memory/<PROJECT>/`
4. Writes a comprehensive `rules.md` from codebase patterns + project conventions
5. Appends harness workflow section to `AGENTS.md` and `CLAUDE.md`

---

## Step 1 — Identify the project (with disambiguation)

### 1a — Get base name
- Get the current working directory name → candidate `<PROJECT_NAME>`
- If inside a monorepo (root `package.json` with `workspaces`, or `lerna.json`, or `pnpm-workspace.yaml`): ask *"This looks like a monorepo. Use the root workspace name or the current sub-package name?"*

### 1b — Disambiguate via git remote (CRITICAL — prevents silent collisions)
Two separate repos named `api` would silently share Obsidian state without this step.

Run: `git remote get-url origin 2>/dev/null`

- If a remote URL exists: append a short hash → `<PROJECT_NAME>-<hash6>`
  - Example: `api` + `git@github.com:user/internal-api.git` → `api-a1b2c3`
- If no git remote: use the absolute path hash → `<PROJECT_NAME>-<path-hash6>`
- If no git repo at all: ask user *"No git remote found. Use plain name `<PROJECT_NAME>`? (Warning: another project with the same directory name will collide)"*

Final identifier: `<PROJECT_NAME>` (use the disambiguated form everywhere below)

### 1c — Confirm with user
*"Initializing harness for `<PROJECT_NAME>` (git remote: <remote-url>). This will create files in Obsidian and update AGENTS.md + CLAUDE.md. Proceed?"*

### 1d — Check Obsidian vault
- Check `~/Documents/Obsidian/` exists. If not: stop, tell user to install Obsidian and create vault there.
- (Future: read vault path from `plugin.json` config — currently hardcoded)

### 1e — Check if already initialized
- If `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/` exists: *"Harness already exists for `<PROJECT_NAME>`. Re-running will overwrite map files but keep sessions. Continue?"*

---

## Step 2 — Read the codebase

Run all of these in parallel.

### 2a — File tree
Generate full file tree. Exclude these paths: `node_modules/`, `.git/`, `dist/`, `build/`, `out/`, `__pycache__/`, `.next/`, `.nuxt/`, `coverage/`, `.turbo/`, `*.lock`, `*.log`, `.DS_Store`, `*.min.js`, `*.min.css`.

For large codebases (>500 files after exclusions): group by top-level directory, show counts not full tree for `src/` subdirs deeper than 2 levels.

### 2b — DB / data layer detection
Check for any of:

**ORM / Query builders:**
- `schema.prisma` → Prisma
- `sequelize` / `sequelize-cli` in package.json → Sequelize
- `mongoose` in package.json → Mongoose (MongoDB)
- `typeorm` in package.json → TypeORM
- `drizzle-orm` in package.json → Drizzle
- `sqlalchemy` / `alembic` in requirements.txt → SQLAlchemy
- `django.db` in any Python file → Django ORM
- `activerecord` / Gemfile → ActiveRecord (Rails)
- `ecto` in mix.exs → Ecto (Elixir)

**Raw DB:**
- `*.sql` files
- `migrations/` or `db/migrations/` directories
- `database/` directory

**NoSQL / other:**
- `firebase` / `firestore` → Firebase
- `supabase` → Supabase
- `redis` → Redis
- `elasticsearch` → Elasticsearch
- `mongodb` → MongoDB

### 2c — Architecture detection
Detect from the following. Read actual file content when needed, not just filename.

**Language & runtime:**
- `package.json` → Node.js / TypeScript / JavaScript
- `requirements.txt` / `pyproject.toml` / `setup.py` → Python
- `Cargo.toml` → Rust
- `go.mod` → Go
- `pom.xml` / `build.gradle` → Java / Kotlin
- `Gemfile` → Ruby
- `mix.exs` → Elixir
- `*.csproj` → C# / .NET

**Frameworks (read config files, not just detect filename):**
- `next.config.js` / `next.config.ts` → Next.js (check if app router or pages router)
- `vite.config.ts` + React imports → Vite + React
- `nuxt.config.ts` → Nuxt.js
- `angular.json` → Angular
- `svelte.config.js` → SvelteKit
- `nestjs` core in package.json → NestJS
- `express` in package.json → Express.js
- `fastapi` / `uvicorn` in requirements → FastAPI
- `django` in requirements → Django
- `flask` in requirements → Flask
- `rails` / `rack` in Gemfile → Ruby on Rails

**Architecture patterns (read directory structure):**
- `src/controllers/` + `src/services/` + `src/models/` → MVC / layered
- `src/features/` or `src/modules/` → feature-based
- `apps/` + `packages/` → monorepo
- `src/api/` + `src/lib/` → API + library split
- `functions/` or `api/` at root → serverless / edge functions
- `src/domain/` + `src/infrastructure/` + `src/application/` → DDD / clean architecture
- `cmd/` + `internal/` + `pkg/` → Go standard layout

**Authentication:**
- `next-auth`, `passport`, `clerk`, `supabase/auth`, `firebase/auth`, `auth0`, `jwt`, `bcrypt`

**Background jobs:**
- `bull`, `bullmq`, `celery`, `sidekiq`, `resque`, `temporal`, `inngest`

**API specification:**
- `swagger.yaml`, `openapi.yaml`, `openapi.json`, `api-docs/` → OpenAPI/Swagger

**Testing stack:**
- `jest.config.*`, `vitest.config.*`, `pytest.ini`, `rspec`, `mocha`, `.spec.ts`, `.test.ts`, `__tests__/`
- Look for test coverage config (thresholds, reporters)

**CI/CD:**
- `.github/workflows/` → GitHub Actions
- `.gitlab-ci.yml` → GitLab CI
- `Dockerfile`, `docker-compose.yml` → Docker

**Environment:**
- `.env.example` → list all environment variables and their purpose (read the file)
- `.env.sample`, `env.template` → same

---

## Step 3 — Create Obsidian folder structure

```bash
mkdir -p ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map
mkdir -p ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions
```

---

## Step 4 — Write map files

Map files are **navigation pointers, not full content**. They tell an agent where to look and what exists — they do not replace reading the actual source files.

### `map/file-tree.md`

```markdown
# File Tree — <PROJECT_NAME>
Generated: <YYYY-MM-DD>
Total files: <count> (excluding node_modules, .git, dist, build)

## Full Structure
<full tree output>

## Key Entry Points
| File | Purpose |
|------|---------|
| <path> | <what this file does — e.g., "Express app entry, registers routes and middleware"> |

## Notable Directories
| Directory | Purpose |
|-----------|---------|
| <path> | <one-line description> |

## Configuration Files
| File | Purpose |
|------|---------|
| <path> | <what it configures> |

## Where to Find Things
- API routes: <path>
- Business logic / services: <path>
- Data models / schemas: <path>
- Tests: <path>
- Static assets: <path>
- Environment config: <path>
```

### `map/db-structure.md`

If DB detected:
```markdown
# DB Structure — <PROJECT_NAME>
Generated: <YYYY-MM-DD>
ORM: <name and version>
Database: <type — PostgreSQL, MySQL, MongoDB, SQLite, etc.>

## Models / Schemas

### <ModelName>
File: <path to model file>
Table/Collection: <name>

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| <field> | <type> | yes/no | <default or —> | <constraints, relations, index> |

Relations:
- <relation description, e.g., "has many Orders, FK: orders.user_id">

### <next model...>

## Migrations
| File | Description |
|------|-------------|
| <filename> | <what it does> |

## Indexes
| Table | Fields | Type | Purpose |
|-------|--------|------|---------|

## Environment Variables (DB)
| Variable | Purpose |
|----------|---------|
| DATABASE_URL | Connection string |
| <others> | <purpose> |
```

If no DB detected:
```markdown
# DB Structure — <PROJECT_NAME>
No database layer detected. This project may use external APIs, files, or in-memory storage.
```

### `map/architecture.md`

```markdown
# Architecture — <PROJECT_NAME>
Generated: <YYYY-MM-DD>

## Tech Stack
| Component | Technology | Version |
|-----------|------------|---------|
| Language | <detected> | <version from package.json/etc> |
| Framework | <detected> | <version> |
| Runtime | <detected> | <version> |
| Database | <detected> | — |
| Auth | <detected or "none"> | — |
| Testing | <detected> | — |
| CI/CD | <detected or "none"> | — |

## Architecture Pattern
<Describe the overall pattern: MVC, feature-based, DDD, monorepo, serverless, etc.>

## Layer Structure
| Layer | Directory | Responsibility |
|-------|-----------|---------------|
| <layer> | <path> | <what it does> |

## Request Flow
<Describe how a typical request flows through the system, e.g.:
"HTTP request → src/api/routes → src/controllers → src/services → src/models → DB">

## Entry Points
| File | Starts | Notes |
|------|--------|-------|
| <path> | <what it starts> | <e.g., "main server process"> |

## External Services / Integrations
| Service | Purpose | Config |
|---------|---------|--------|
| <service> | <what it does> | <env var or config file> |

## Environment Variables
| Variable | Required | Purpose |
|----------|----------|---------|
| <var> | yes/no | <description> |

## Key Patterns Used
- <pattern and where it's used>

## Known Constraints / Notes
- <anything unusual about the architecture>
```

### `map/product-info.md`

(Filled in Step 5 below)

---

## Step 5 — Ask targeted product questions

Ask ONE question at a time. Wait for each answer before asking the next. Always ask the two mandatory questions first. Then ask conditional questions based on detection.

### Mandatory questions (always ask both):

1. *"What is the primary action a user comes to this product to do? Describe it in one sentence."*

2. *"What are the 2–3 things that must never break in this product — the features that, if broken, would be immediately noticed and cause serious problems?"*

### Conditional questions (ask only if relevant module detected):

| Detected | Question |
|----------|----------|
| Payment module | *"What payment providers does this product support? (e.g., Stripe, PayPal, Razorpay)"* |
| Auth system | *"What are the user roles and what can each role do? Walk me through the permission levels."* |
| Multi-tenant / org structure | *"Is this SaaS (multiple paying customers with separate data) or internal tooling (one organization)?"* |
| External API layer | *"Is this API consumed by external third-party clients, or only by this project's own frontend?"* |
| Background jobs | *"What are the most critical background jobs? How often do they run and what breaks if they fail?"* |
| Email / notification system | *"What triggers email or notification sends? What must always be delivered?"* |
| File upload / storage | *"What file types are uploaded? Where are they stored? What's the size limit?"* |
| Search functionality | *"What is searchable? How does search work — full-text, filters, or both?"* |
| Caching layer | *"What data is cached? What are the cache invalidation rules?"* |

### Write answers to `map/product-info.md`:

```markdown
# Product Info — <PROJECT_NAME>
Generated: <YYYY-MM-DD>

## Primary Purpose
<answer to "what is the primary action">

## Critical Invariants
These must never break:
1. <first thing>
2. <second thing>
3. <third thing if given>

## User Roles & Permissions
<answer if asked, or "Not applicable">

## External Dependencies
<payment providers, third-party APIs, etc.>

## Data Sensitivity
<any PII, financial data, regulated data that affects how agents handle files>

## Additional Context
### <topic from conditional question>
<answer>

### <next topic>
<answer>
```

---

## Step 6 — Generate `rules.md`

Read existing `AGENTS.md` and `CLAUDE.md` in the project root if they exist. Extract all rules, conventions, constraints, and patterns. Combine with detected codebase patterns. Write a comprehensive, actionable rules file.

Write to `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/rules.md`:

```markdown
# Rules — <PROJECT_NAME>
Generated: <YYYY-MM-DD>

---

## 1. Harness Workflow Rules (MANDATORY)

- Always run `/harness-session` at the start of every session — no exceptions
- Always run `/harness-task` when given any task — no exceptions
- Update Obsidian session files (progress.md, decisions.md, verification-notes.md, checkpoints.md) after EVERY completed step — never batch at end
- Never declare a task or sub-task done without running its verification command
- Read rules.md and all map files before starting any task
- Read progress.md at the start of every task to check for incomplete work from a prior session
- Write sub-task breakdown to progress.md BEFORE dispatching any agents

---

## 2. Naming Conventions

<Fill from codebase detection. Examples:>
- File naming: <detected pattern — e.g., "kebab-case for all .ts files", "PascalCase for React components">
- Variable naming: <e.g., "camelCase for variables and functions", "SCREAMING_SNAKE_CASE for constants">
- Component naming: <e.g., "PascalCase, suffix with component type: UserCard, UserCardList">
- API routes: <e.g., "kebab-case, plural nouns: /api/users, /api/blog-posts">
- Database tables/collections: <e.g., "snake_case plural: users, blog_posts">
- Test files: <e.g., "co-located with source, same name + .test.ts: user.service.test.ts">
- CSS classes: <e.g., "BEM: block__element--modifier">
- Branch names: <e.g., "feat/short-description, fix/short-description">
- Commit messages: <e.g., "conventional commits: feat:, fix:, chore:, docs:">

---

## 3. Architecture Boundaries

<Fill from detection. Examples:>
- Controllers/routes must NOT directly access the database — route through services
- Services must NOT import from controllers/routes
- Models/schemas must NOT contain business logic — only data shape and DB operations
- Utility functions in `utils/` or `lib/` must NOT import from `services/` or `models/`
- Frontend components must NOT call the database or Node.js APIs directly
- API routes must NOT contain business logic — delegate to service layer
- Background jobs must NOT be called synchronously from request handlers
- <Add any detected layer constraints>

---

## 4. Testing Requirements

<Fill from detection. Examples:>
- Test framework: <jest / vitest / pytest / rspec / etc.>
- Run tests with: `<exact command>`
- Coverage threshold: <if configured, e.g., "80% line coverage required">
- Every new function/method must have a unit test
- Every API endpoint must have an integration test
- Never delete existing tests — if behavior changes, update the test
- Test file location: <co-located OR in __tests__/ OR in spec/>
- Mock external services in tests — never call real APIs
- Fixtures/factories location: <path if detected>
- Snapshot tests: <allowed or not>

---

## 5. Code Quality Rules

- Never use `any` type in TypeScript without a comment explaining why
- Never use `console.log` / `print` / `puts` for debugging — use the project logger if it exists, or remove before commit
- Never commit commented-out code — delete it (it lives in git history)
- Never leave `TODO` comments — either implement it now or create a tracked sub-task
- Never use magic numbers — extract to named constants
- Functions must do one thing — if a function has "and" in its description, split it
- Max function length: 40 lines (soft limit) — if longer, consider splitting
- Never mutate function arguments — treat them as read-only
- Always handle errors explicitly — never swallow exceptions silently
- Never use `== null` in JavaScript/TypeScript — use `=== null` or `=== undefined` separately, or nullish coalescing `??`

---

## 6. Security Rules

- NEVER commit API keys, passwords, tokens, or secrets to the repository
- NEVER hardcode credentials — use environment variables
- NEVER log sensitive data (passwords, tokens, PII, payment info) — even at debug level
- NEVER trust user input without validation — validate at the API boundary
- NEVER use `eval()`, `exec()`, or equivalent dynamic code execution
- NEVER use string concatenation for SQL queries — use parameterized queries / ORM
- NEVER disable CORS globally in production configuration
- NEVER store plaintext passwords — always hash with bcrypt / argon2 / scrypt
- Environment variables: check `.env.example` exists and is kept up to date when adding new vars
- Dependencies: never add a new package without checking it's actively maintained

---

## 7. Agent Behavior Rules

These rules apply to every agent working on any sub-task:

### Scope
- Only modify files listed in your sub-task's "Files to edit" — if you need to edit an unlisted file, stop and document it in decisions.md as a blocker
- If you discover a bug OUTSIDE your sub-task scope, log it in decisions.md but do NOT fix it — create a note for the next session
- Never rename files or directories unless the sub-task explicitly says to
- Never delete files or directories unless the sub-task explicitly says to
- Never reorganize imports across multiple files unless the sub-task explicitly says to

### Before editing
- Read the FULL file before making any edit — never edit based on a partial view
- Check if a similar function/component already exists before creating a new one
- Read the test file for any file you are about to edit — understand expected behavior first
- Check what imports the file you are editing — avoid breaking consumers

### While editing
- Match the exact code style of the surrounding code (spacing, quotes, semicolons, etc.)
- Add new code in the same pattern as existing code
- If the codebase uses a certain abstraction consistently, use it — do not introduce a new pattern
- Write no comments unless the WHY is non-obvious
- Do not refactor code outside the scope of the sub-task

### After editing
- Run the verification command listed in the sub-task — no exceptions
- If verification PASSES: update progress.md, decisions.md, verification-notes.md, checkpoints.md
- If verification FAILS: mark sub-task as "blocked" in progress.md, write the failure details in verification-notes.md, do NOT mark as completed, stop
- Write decisions.md entry for EVERY decision made, including trivial ones

### Decision logging (mandatory)
Log every decision with this level of detail:
- What was the choice point?
- What options were available?
- Why was this option chosen?
- What were the trade-offs?
- What was explicitly rejected and why?

---

## 8. Forbidden Patterns

<Fill from codebase detection + existing AGENTS.md/CLAUDE.md. Examples:>
- Never use `any` as a function parameter type in TypeScript
- Never import from `../../../` more than 2 levels deep — use path aliases
- Never use synchronous file I/O (`fs.readFileSync`) in request handlers
- Never call `process.exit()` outside the main entry file
- Never use `setTimeout` for retry logic — use the existing retry utility at `<path>`
- Never create new database connections directly — use the connection pool at `<path>`
- Never write to stdout in library code — only in CLI entry points
- <Add any patterns found in existing AGENTS.md / CLAUDE.md>

---

## 9. Git Rules

- Never commit directly to `main` or `master` — always use a branch
- Commit after each completed sub-task (one logical change per commit)
- Commit message format: `<type>(<scope>): <description>` (conventional commits)
  - Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
  - Example: `feat(auth): add JWT refresh token rotation`
- Never amend a commit that has been pushed
- Never force-push to shared branches
- Stage specific files — never `git add .` without reviewing what was changed

---

## 10. Error Handling Rules

- All async operations must have explicit error handling — no unhandled promise rejections
- All errors must be logged with enough context to debug (what operation failed, with what inputs)
- User-facing errors must be sanitized — never expose stack traces or internal paths
- External API calls must have timeout and retry logic
- Database operations must handle connection errors and timeouts
- Never fail silently — if an operation fails, either handle it or let it propagate clearly

---

## 11. Performance Rules

- Never make N+1 database queries — use joins, eager loading, or batching
- Never load the entire dataset into memory — use pagination or streaming
- Never do heavy computation in the request handler — offload to background jobs
- Cache expensive operations where appropriate — document the cache key and TTL
- Never block the event loop (Node.js) with synchronous operations

---

## 12. Documentation Rules

- Every public API endpoint must have a JSDoc / docstring comment with: purpose, params, return type, errors
- Every non-obvious algorithm must have a brief explanation comment
- Update `map/` files in Obsidian whenever you add a new module, model, or entry point
- Keep `.env.example` in sync with actual environment variables used
```

---

## Step 7 — Append to `AGENTS.md` and `CLAUDE.md`

**Before appending:** search each file for `## Harness —` string. If found, skip that file (do not duplicate).

If file does not exist, create it with the section as the entire content.

### Append to `AGENTS.md`:

```markdown

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
3. After every completed step → update Obsidian session files IMMEDIATELY
4. Before declaring done → run the verification command from the sub-task

### Session Files
  ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SessionName>-<date>/
    progress.md           — sub-task breakdown and live status
    decisions.md          — every decision made/rejected with full reasoning
    verification-notes.md — verification result per step
    checkpoints.md        — step-by-step completion markers

### Active Session
  ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state
  (written by /harness-session — contains active session name and path)

### Continuity Rule
Session files are the single source of truth. Any agent — Claude Code, Codex, Cursor, or other —
can resume any session by reading .harness-state, then progress.md, and continuing from the
last pending sub-task. Zero re-briefing needed.
```

### Append to `CLAUDE.md`:

```markdown

---
## Harness — <PROJECT_NAME>

### Knowledge Base (Obsidian)
Read before starting any work:
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/file-tree.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/db-structure.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/architecture.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/map/product-info.md`
- `~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/rules.md`

### Workflow (mandatory)
1. **Session start** → `/harness-session`
2. **Every task** → `/harness-task`
3. **After every step** → update Obsidian session files immediately
4. **Before done** → run verification command

### Active session tracking
`~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/.harness-state`

### Session files
`~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/sessions/<SessionName>-<date>/`
Contains: `progress.md`, `decisions.md`, `verification-notes.md`, `checkpoints.md`

These files are the source of truth. Any session resumes by any agent at any time, zero re-briefing.
```

---

## Step 8 — Initialize Quality Document

Create the empty Quality Document at:
`~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/quality-document.md`

```markdown
# Quality Document — <PROJECT_NAME>
Last updated: <YYYY-MM-DD>
Status: initialized — no tasks completed yet

*This document tracks codebase health over time. Updated automatically by /harness-task after every completed task.*

---

## Overall Health
| Metric | Status | Last Checked |
|--------|--------|-------------|
| Build | not yet measured | — |
| All tests | not yet measured | — |
| Lint | not yet measured | — |
| No debug artifacts | not yet measured | — |
| Standard startup | not yet measured | — |

## Module Health
(filled by /harness-task as features are added)

## Session History
(filled as sessions complete)

## Known Issues
(none)

## Architecture Boundary Violations Found
(none)

## Harness Health
| Component | Status | Last Audited |
|-----------|--------|-------------|
| AGENTS.md | current | <YYYY-MM-DD> |
| CLAUDE.md | current | <YYYY-MM-DD> |
| map/file-tree.md | current | <YYYY-MM-DD> |
| map/architecture.md | current | <YYYY-MM-DD> |
| map/db-structure.md | current | <YYYY-MM-DD> |
| map/product-info.md | current | <YYYY-MM-DD> |
| rules.md | current | <YYYY-MM-DD> |
```

---

## Step 9 — Print summary

```
✓ Harness initialized for <PROJECT_NAME>

Obsidian knowledge base:
  ~/Documents/Obsidian/creator/agent-memory/<PROJECT_NAME>/
  ├── map/
  │   ├── file-tree.md       (<N> files mapped)
  │   ├── db-structure.md    (<N> models detected, or "no DB")
  │   ├── architecture.md    (<framework> / <pattern>)
  │   └── product-info.md    (<N> questions answered)
  ├── rules.md               (18 rule sections generated)
  └── quality-document.md    (initialized, updated by /harness-task)

Project files updated:
  ✓ AGENTS.md  (harness section appended)
  ✓ CLAUDE.md  (harness section appended)

Detected stack: <language> / <framework> / <DB if any>
Rules generated from: <N> files read

Workflow available:
  /harness-session → start a new session or resume existing
  /harness-task    → runs Phase 0 (clarify) → Phase 1 (bootstrap) → Phase 2 (plan) → Phase 3 (execute) → Phase 4 (clean state)

Next step: Run /harness-session to start your first session.
```

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Obsidian vault not at `~/Documents/Obsidian/` | Stop. Tell user to install Obsidian and create vault there |
| Harness already exists | Ask to confirm overwrite of map files |
| AGENTS.md already has harness section | Skip append, notify user |
| Project has no AGENTS.md or CLAUDE.md | Create both from scratch with harness section as content |
| Codebase is very large (>2000 files) | Use directory-level summary for deep paths, not file-by-file |
| Monorepo detected | Ask user which package/workspace to harness |
| `.env.example` missing | Note in architecture.md: "No .env.example found — environment variables unknown" |
