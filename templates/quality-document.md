# Quality Document — <PROJECT_NAME>
Last updated: <DATE>

*This document tracks codebase health over time. New sessions read this first to know where to prioritize.*

---

## Overall Health

| Metric | Status | Last Checked |
|--------|--------|-------------|
| Build | ✓ / ✗ | <DATE> |
| All tests | ✓ / ✗ | <DATE> |
| Lint | ✓ / ✗ | <DATE> |
| No debug artifacts | ✓ / ✗ | <DATE> |
| Standard startup | ✓ / ✗ | <DATE> |

---

## Module Health

Score each module on 5 dimensions: ✓ (good) / ~ (partial) / ✗ (issue)

| Module / Feature | Build | Tests | Agent-Readable | Arch Boundaries | Conventions |
|------------------|-------|-------|----------------|-----------------|------------|
| | | | | | |

**Agent-Readable**: can a new agent understand this module's purpose without reading all internals?
**Arch Boundaries**: does this module respect layer boundaries and not import across boundaries?

---

## Session History

| Session | Date | Features Added | Evaluator Avg Score | Final Clean State | Notes |
|---------|------|---------------|--------------------|--------------------|-------|
| | | | /5 | Pass/Fail | |

---

## Known Issues

| # | Issue | Module | Severity (High/Med/Low) | Status | Logged |
|---|-------|--------|------------------------|--------|--------|
| | | | | open/resolved | |

---

## Architecture Boundary Violations Found

| # | Violation | File | Rule Violated | Status |
|---|-----------|------|--------------|--------|
| | | | | open/fixed |

---

## Test Stability

| Test Suite | Status | Last Passing | Notes |
|------------|--------|-------------|-------|
| | stable/flaky | | |

---

## Stale Artifacts Count

| Type | Count | Action |
|------|-------|--------|
| console.log / debug statements | | |
| TODO / FIXME comments | | |
| Commented-out code blocks | | |
| Unused imports | | |
| Dead code | | |

---

## Harness Health

| Component | Status | Last Audited | Notes |
|-----------|--------|-------------|-------|
| AGENTS.md | current/stale | | |
| CLAUDE.md | current/stale | | |
| map/file-tree.md | current/stale | | |
| map/architecture.md | current/stale | | |
| map/db-structure.md | current/stale | | |
| rules.md | current/stale | | |

*Harness rots like code. Audit every map file after major changes.*
