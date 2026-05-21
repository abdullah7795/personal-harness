# Evaluator Rubric — <SUB_TASK_ID>
Session: <SESSION_NAME>
Evaluator: (assigned agent)
Date: <DATE>

---

## What Was Implemented
<SUB_TASK_DESCRIPTION>

## Files Changed
<FILES_CHANGED>

---

## Three-Layer Verification (run independently — do not trust the worker's results)

| Layer | Command | Expected | Actual | Pass/Fail |
|-------|---------|---------|--------|-----------|
| 1. Lint | <LINT_CMD> | zero errors | | |
| 1. Type-check | <TYPE_CMD> | zero errors | | |
| 1. Build | <BUILD_CMD> | success | | |
| 2. Tests | <TEST_CMD> | all pass | | |
| 3. E2E | <E2E_CMD> | <EXPECTED> | | |

---

## Quality Scoring

Score each dimension 1–5:
- **5** = Excellent, no issues
- **4** = Good, minor issues only
- **3** = Acceptable, some issues to address
- **2** = Needs work, significant issues
- **1** = Fail, major problems

| Dimension | Score (1-5) | Evidence | Notes |
|-----------|-------------|----------|-------|
| **Correctness** — does it actually do what the sub-task describes? All edge cases handled? | | | |
| **Architecture compliance** — layer boundaries respected, no forbidden patterns? | | | |
| **Test coverage** — main flow + edge cases covered? | | | |
| **Code conventions** — matches exact codebase style (naming, spacing, patterns)? | | | |
| **Security** — no credentials, input validated, no SQL injection / XSS risks? | | | |
| **Error handling** — failures handled explicitly, no silent swallows? | | | |

**Average: <sum>/5**

---

## Sprint Contract Compliance

- [ ] All hard constraints respected
- [ ] No explicit exclusions violated
- [ ] DoD criteria met

---

## Issues Found

### Critical (blocks pass — must fix)
| # | Issue | File | Line | How to Fix |
|---|-------|------|------|-----------|

### Important (should fix before merge)
| # | Issue | File | Line | Suggestion |
|---|-------|------|------|-----------|

### Minor (note for later)
| # | Issue | File | Suggestion |
|---|-------|------|-----------|

---

## Verdict

- [ ] **PASS** — avg ≥ 4.0, all verification layers pass, no critical issues
- [ ] **NEEDS WORK** — avg 3.0–3.9, or any layer failing, or any critical issue
- [ ] **FAIL** — avg < 3.0, multiple layers failing

**Verdict: ___**

**Reason:**

---

## If NEEDS WORK or FAIL — Required Fixes

*List specific, actionable fixes the worker agent must address:*

1.
2.
3.
