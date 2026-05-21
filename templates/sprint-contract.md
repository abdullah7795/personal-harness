# Sprint Contract — <SESSION_NAME>
Date: <DATE>
Task: <TASK_DESCRIPTION>
Status: draft → approved

---

## Scope (what will be built)
<SCOPE_LIST>

---

## Feature List

| # | Behavior Description | Verification Command | State |
|---|---------------------|---------------------|-------|
| F01 | <concrete observable behavior — e.g., "POST /api/users creates a user and returns 201 with {id, email}"> | <exact runnable command> | not_started |
| F02 | | | not_started |

*Behavior description must be concrete and observable. "The user can log in" is not concrete. "POST /api/auth/login with {email, password} returns 200 with {token} and sets HttpOnly cookie" is concrete.*

---

## Definition of Done

All of the following must be true before the task is declared complete:

- [ ] All features in state: **passing**
- [ ] Layer 1 (lint + type-check + build): **pass**
- [ ] Layer 2 (unit + integration tests): **pass**
- [ ] Layer 3 (E2E / full flow): **pass**
- [ ] No debug code (console.log, debugger, TODO, FIXME, .only())
- [ ] Build passes from clean install
- [ ] All tests that existed before this task still pass

---

## Hard Constraints (non-negotiable)
<HARD_CONSTRAINTS>

---

## Explicit Exclusions (out of scope — do NOT build)
<EXCLUSIONS>

---

## Verification Commands

| Check | Command | Expected |
|-------|---------|---------|
| Lint | <LINT_COMMAND> | zero errors |
| Type-check | <TYPECHECK_COMMAND> | zero errors |
| Tests | <TEST_COMMAND> | all pass |
| E2E F01 | <E2E_COMMAND_F01> | <EXPECTED_F01> |
| E2E F02 | <E2E_COMMAND_F02> | <EXPECTED_F02> |

---

## Approval
- [ ] User reviewed and approved this contract
- Date approved:
- Notes:
