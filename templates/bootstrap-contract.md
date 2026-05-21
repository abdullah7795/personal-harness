# Bootstrap Contract — <SESSION_NAME>
Date: <DATE>
Project: <PROJECT_NAME>

*This contract verifies the environment is ready before any feature code is written.
A session that fails this contract must fix it before implementing anything.*

---

## Can Start

| Check | Command | Result |
|-------|---------|--------|
| Setup from scratch | `<SETUP_COMMAND>` | Pass / Fail |
| Dev server starts | `<DEV_COMMAND>` | Pass / Fail |
| No environment errors | (manual check) | Pass / Fail |

---

## Can Test

| Check | Command | Result |
|-------|---------|--------|
| Test runner executes | `<TEST_COMMAND>` | Pass / Fail |
| At least one test passes | (from above) | Yes / No |
| Test framework configured | (from above) | Yes / No |

---

## Can Verify

| Check | Command | Result |
|-------|---------|--------|
| Lint runs | `<LINT_COMMAND>` | Pass / Fail |
| Type-check runs | `<TYPECHECK_COMMAND>` | Pass / Fail |
| Build runs | `<BUILD_COMMAND>` | Pass / Fail |

---

## Can See Progress

| Check | Status |
|-------|--------|
| AGENTS.md exists | Yes / No |
| CLAUDE.md exists | Yes / No |
| Obsidian map files exist | Yes / No |
| progress.md initialized | Yes / No |

---

## Bootstrap Status

- [ ] **READY** — all checks pass, proceed to Sprint Contract and feature work
- [ ] **BLOCKED** — one or more checks fail, fix before any feature work

**Blocker details:**

---

## Notes
<NOTES>
