# Rules — <PROJECT_NAME>
Generated: <DATE>
Source: harness-proj-init — derived from codebase patterns + 12 harness engineering principles

---

## 1. Harness Workflow Rules (MANDATORY — no exceptions)

- Always run `/harness-session` at the start of every session
- Always run `/harness-task` when given any task — no exceptions
- Before coding begins: clarifying questions → Sprint Contract → user approval
- Update Obsidian session files after EVERY completed step — never batch at end
- Never declare a task or sub-task done without running its verification command
- Read rules.md and all map files before starting any task
- Read progress.md at the start of every task to check for incomplete work
- Write sub-task breakdown to progress.md BEFORE dispatching any agents
- Run clock-out routine at end of every session (update progress, verify clean state, commit)

**Source: L01 — Verification Gap; L05 — Continuity artifacts; L12 — Clean state mandatory**

---

## 2. Definition of Done (machine-verifiable, always)

Every task must have an explicit, machine-executable Definition of Done before work starts:

- Specify exact endpoints, behaviors, or outputs — not vague goals
- Include exact verification commands (not "tests pass" but `npm test -- auth.test.ts`)
- Include Layer 1 (lint/type/build) + Layer 2 (unit/integration) + Layer 3 (E2E) checks
- DoD is written in the Sprint Contract before any coding

Bad DoD: *"Add a search feature"*
Good DoD: *"GET /api/search?q=xxx returns paginated results with highlighted snippets. `npm test -- search.test.ts` passes. `mypy --strict` passes."*

**Source: L01 — Write explicit Definition of Done for every task**

---

## 3. WIP=1 — One Feature Active at a Time

- Only one feature in `active` status at any time
- Only start the next feature after the current one passes end-to-end verification
- VCR (Verified Completion Rate) = verified / activated must equal 1.0 before activating next
- Do not refactor feature B while implementing feature A
- Do not fix bugs discovered in other features mid-task — log them in decisions.md

**Source: L07 — WIP=1 shows 37% higher completion rate; lines of code is negatively correlated with feature completion**

---

## 4. Three-Layer Termination Check (in order — never skip a layer)

Every sub-task must pass all three layers before being marked complete:

**Layer 1 — Syntax/Static Analysis** (cheapest, run first):
- Lint: `<LINT_COMMAND>`
- Type-check: `<TYPECHECK_COMMAND>`
- Build: `<BUILD_COMMAND>`
- Pass condition: zero errors, zero warnings
- If Layer 1 fails → stop. Do not run Layer 2.

**Layer 2 — Runtime/Tests**:
- Tests: `<TEST_COMMAND>`
- Pass condition: all tests pass
- If Layer 2 fails → stop. Do not run Layer 3.

**Layer 3 — System/E2E**:
- E2E or integration: `<E2E_COMMAND>`
- Pass condition: full user flow works end-to-end
- This is the only true proof the feature is complete

Unit tests passing does NOT mean the task is complete. Unit tests are blind to:
- Interface mismatches between components
- State propagation errors (ORM cache, migrations)
- Resource lifecycle issues (file handles, connections)
- Environment dependencies (works with mocks, fails with real config)

**Source: L09 — Three-Layer Termination Check; L10 — E2E as true verification**

---

## 5. Worker ≠ Evaluator (always separate agents)

- The agent that writes code must NOT evaluate it
- A separate evaluator agent checks independently with an Evaluator Rubric
- Evaluator is explicitly prompted to be "picky and thorough"
- Agent self-evaluation is systematically over-positive — this is proven, not an assumption

Evaluator rubric dimensions:
1. Correctness — all verification layers pass
2. Architecture compliance — layer boundaries respected
3. Test coverage — main flow + edge cases
4. Code conventions — matches codebase style
5. Security — no obvious vulnerabilities
6. Error handling — failures handled explicitly

Pass threshold: ≥ 4.0 average, all verification layers passing.

**Source: L09 — Agents over-rate their own work; Anthropic: same model, 3-agent harness → fully playable vs. $9 broken**

---

## 6. Naming Conventions

<NAMING_CONVENTIONS>

Fill from codebase detection:
- File naming: <e.g., "kebab-case for .ts files, PascalCase for React components">
- Variable naming: <e.g., "camelCase for variables, SCREAMING_SNAKE for constants">
- Component naming: <e.g., "PascalCase + type suffix: UserCard, UserCardList">
- API routes: <e.g., "kebab-case plural nouns: /api/users, /api/blog-posts">
- Database tables: <e.g., "snake_case plural: users, blog_posts">
- Test files: <e.g., "co-located, same name + .test.ts">
- Branch names: <e.g., "feat/description, fix/issue-123">
- Commit messages: <e.g., "conventional commits: feat:, fix:, chore:, refactor:, test:">

---

## 7. Architecture Boundaries

<ARCHITECTURE_BOUNDARIES>

Fill from codebase detection:
- Controllers/routes must NOT directly access the database — route through services
- Services must NOT import from controllers/routes
- Models/schemas must NOT contain business logic
- Utility functions must NOT import from services or models
- Frontend components must NOT call backend directly

**Architectural boundaries must be machine-enforced, not just documented:**
```bash
# Example enforcement check (add to make check):
grep -r "require('fs')" src/renderer/ && exit 1 || echo "OK"
```

**Source: L10 — Architectural Boundary Enforcement Rules; enforce invariants, don't micromanage**

---

## 8. Testing Requirements

<TESTING_REQUIREMENTS>

Fill from detection:
- Test framework: <jest / vitest / pytest / rspec>
- Run command: `<exact command>`
- Coverage threshold: <percentage if configured>
- Every new function/method must have a unit test
- Every API endpoint must have an integration test
- Every feature must have an E2E / full-flow test
- Never delete existing tests — update when behavior changes
- Mock all external services in unit/integration tests
- One real E2E test per feature minimum

**Source: L10 — Four blind spots of unit tests**

---

## 9. Code Quality Rules

- Never use `any` type in TypeScript without a comment explaining why
- Never use `console.log` / `print` for debugging — use the project logger or remove before commit
- Never commit commented-out code — delete it (lives in git history)
- Never leave `TODO` or `FIXME` comments — implement now or create a tracked sub-task in progress.md
- Never use magic numbers — extract to named constants
- Functions must do one thing — split if description contains "and"
- Max function length: ~40 lines
- Never mutate function arguments
- Always handle errors explicitly — never swallow exceptions silently
- No refactoring until current feature passes all 3 verification layers

**Source: L09 — No refactoring until verification complete; L12 — No debug artifacts in clean state**

---

## 10. Security Rules

- NEVER commit API keys, passwords, tokens, or secrets to the repository
- NEVER hardcode credentials — use environment variables
- NEVER log sensitive data (passwords, tokens, PII, payment info)
- NEVER trust user input without validation at the API boundary
- NEVER use `eval()`, `exec()`, or dynamic code execution
- NEVER use string concatenation for SQL queries — use parameterized queries / ORM
- NEVER store plaintext passwords — always hash (bcrypt / argon2 / scrypt)
- Keep `.env.example` up to date when adding new environment variables
- Never add a dependency without verifying it is actively maintained

**Source: L01 — Security constraint compliance drops from 95% to 60% when buried in middle of instructions**

---

## 11. Agent Scope Rules

### Before editing
- Read the FULL file before making any edit
- Check if a similar function already exists before creating one
- Read the test file for any file you are about to edit
- Check what imports the file you are editing

### While editing
- Only edit files listed in your sub-task's "Files to edit"
- Match the exact code style of surrounding code (spacing, quotes, semicolons)
- Do not introduce new patterns when existing patterns serve the purpose
- Write no comments unless the WHY is genuinely non-obvious
- Do not refactor outside the scope of the sub-task

### After editing
- Run the three-layer verification — no exceptions
- Pass all 3 layers → update all 4 Obsidian session files immediately
- Any layer fails → mark as `blocked`, log failure in full, stop
- Log every decision in decisions.md (no decision is too small)
- If you discover a bug outside your scope → log in decisions.md, do NOT fix it

### Agent-oriented error messages

When writing verification scripts or test output, error messages must include:
- **What** went wrong
- **Why** it is wrong
- **Exactly how** to fix it

Bad: `"Test failed."`
Good: `"Test failed: POST /api/reset-password returned 500. Check that EMAIL_SERVICE_URL exists in .env. Email template must be at templates/reset-email.html. See docs/email-service.md."`

**Source: L09, L10 — Agent-oriented error messages turn failures into self-correcting loops**

---

## 12. Forbidden Patterns

<FORBIDDEN_PATTERNS>

Fill from codebase detection + existing AGENTS.md/CLAUDE.md:
- Never use `any` as a function parameter type
- Never import more than 2 levels deep (../../) — use path aliases
- Never use synchronous file I/O in request handlers
- Never call `process.exit()` outside the main entry file
- Never create new database connections directly — use the connection pool
- <Add project-specific patterns>

---

## 13. Git Rules

- Never commit directly to `main` or `master`
- Commit after each completed sub-task — one logical change per commit
- Commit message: `<type>(<scope>): <what> — <why>` (what AND why)
  - Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
  - Bad: `"fix bug"` / Good: `"fix: pagination returning 500 on empty sets — add null check before slice"`
- Never amend a pushed commit
- Never force-push to shared branches
- Stage specific files — never `git add .` without reviewing the diff

**Source: L05 — Git commits are free, automatically versioned continuity artifacts**

---

## 14. Context and Instruction Management

- AGENTS.md is a router, not an encyclopedia — keep it under 200 lines
- Critical rules go at the TOP or BOTTOM of instruction files — never the middle
- Instructions buried in the middle of long files are effectively ignored (Lost in the Middle effect)
- Every rule needs: source (why added), applicability (when needed), expiry (when removable)
- Audit and remove unused rules regularly — every rule costs context budget on every task
- Topic-specific rules live in separate docs linked from AGENTS.md, not inline

**Source: L04 — Instruction bloat at 10-15% context = performance degrades; Lost in the Middle proven**

---

## 15. Session Continuity Rules

- PROGRESS.md is the core continuity artifact — updated before every session ends
- DECISIONS.md preserves the "why" — compaction destroys reasoning, this file doesn't
- Clock-in: read progress.md → read decisions.md → run `make check` → start from "Next Steps"
- Clock-out: update progress.md → run `make check` → commit completed work
- Treat the agent like a brilliant engineer with amnesia — journal must be good enough for them to pick up in minutes
- Five-dimension clean state check at every session end (build, tests, progress, artifacts, startup)

**Source: L05 — 78% rebuild time reduction with continuity artifacts; L12 — clean state data**

---

## 16. Error Handling Rules

- All async operations must have explicit error handling
- All errors must be logged with context (what failed, with what inputs)
- User-facing errors must be sanitized — never expose stack traces
- External API calls must have timeout and retry logic
- Never fail silently — handle or propagate clearly

---

## 17. Performance Rules

- Never make N+1 database queries — use joins, eager loading, or batching
- Never load entire dataset into memory — use pagination or streaming
- Heavy computation goes to background jobs, not request handlers
- Cache expensive operations — document the cache key and TTL
- Never block the event loop (Node.js) with synchronous operations

---

## 18. The Five Failure Layers (diagnose every failure here first)

When something goes wrong, attribute it to one of these layers before considering a model upgrade:

| Layer | Question | Common Fix |
|-------|----------|-----------|
| 1. Task specification | Was DoD explicit and machine-verifiable? | Rebuild Sprint Contract with clearer criteria |
| 2. Context provision | Did agent have architecture rules and constraints? | Update map files and rules.md |
| 3. Execution environment | Dependencies installed? Versions correct? | Fix bootstrap contract |
| 4. Verification feedback | Did agent have runnable verification commands? | Add Layer 1/2/3 commands |
| 5. State management | Did agent know where to pick up after reset? | Update progress.md and checkpoints.md |

**Source: L01 — Harness-Induced Failure is the most common failure mode, not underpowered model**
