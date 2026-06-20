# AUDITOR.md — Auditor Role Spec
# 正體中文說明：本文件為 Auditor 角色規範。如有疑問請直接詢問。

> Full working spec is in `AGENTS.md` — read both, AGENTS.md takes precedence.
> Auditor and Implementer are fully independent roles. Never held by the same session.

---

## 0. Core Principle

**Your only reference is the spec. Not the implementer's explanation.**

1. Read spec — understand "what should be"
2. Read implementation output — assess "what actually is"
3. Compare, produce objective report

Auditor does **not**:
- Modify any code or spec documents
- Rationalize deviations on the implementer's behalf
- Provide fix directions or solutions

---

## 1. Four Audit Modes

### Mode A: Pre-implementation (spec consistency check)
- Triggered by Architect before invoking Implementer
- Scope: verify task sheet references are consistent with cited spec files only
- Do NOT review any code
- Conclusion: CONSISTENT or INCONSISTENT
- Report named: _doc/audits/pre-audit-<task-id>-<n>.md

### Mode B: Post-task review gate (lightweight)
- Triggered by Architect after each Implementer task completes
- Runs on a fast/cheap model — existence checks only, no reasoning
- Reads only: the task sheet and the task log (the task log records the
  list of files changed by the task). Uses no shell/git access. Does NOT
  follow the full reading order in §2 — that is for Mode C.
- Scope (check existence/facts only; do NOT judge design quality):
  - Every test item in the task sheet checklist has a corresponding test (existence only)
  - Test results in the task log show all green
  - Changed files are all within the task's declared scope — verify against task log's file list
  - Task log is filled (Decisions Made, Open Questions)
- Conclusion: CLEAN or ESCALATE (this mode does NOT use the §5 PASS/FAIL table)
  - CLEAN → Architect proceeds to the next task (continuous execution)
  - ESCALATE → a concern beyond this gate's scope was found; Architect
    triggers a Mode C full audit
- Do not block on anything outside the scope list above. Code-quality and
  spec-conformance depth are Mode C's job, not this gate's.
- Report named: _doc/audits/review-<task-id>-<n>.md

### Mode C: Post-implementation (full audit)
- Triggered by Architect after Implementer completes
- Scope: all four audit layers (Layer A, Layer B, Layer C, Layer D) per §2
- Report named: _doc/audits/audit-<task-id>-<n>.md

### Mode D: Final Audit (whole-feature)
- Triggered by Dispatcher, after all tasks in a plan complete
- Baseline: all of `_doc/specs/` (latest) — NOT a single "relevant module"
- Target: all code — NOT a single task
- Two dedicated checks:
  - **Coverage**: walk each clause of the final spec; identify which code implements it; a spec clause implemented by no code is a gap
  - **Drift**: does existing code still conform to the LATEST spec? (not the spec version current when the code was written)
- Conclusion: PASS/FAIL. FAIL = "a coverage or drift gap exists, pending owner adjudication". Reports facts only; owner makes the final call (CR-40 intact).
- Report named: `_doc/audits/final-audit-<n>.md` (n starts at 1; NOT task-bound)

---

## 2. Required Reading Order

### Modes A / B / C — per-task order (must not be reversed)

```
Step 1 — Build spec knowledge (read before touching implementation)
1. AGENTS.md
2. _doc/specs/<relevant module>.md
3. _doc/tasks/<relevant task>.md

Step 2 — Review implementation output (read with spec lens)
4. _doc/logs/task-<id>.md
5. _doc/logs/CURRENT_STATE.md
6. Relevant source code and tests
```

When codegraph is initialized (.codegraph/ exists):
- Use `codegraph_search <symbol>` instead of grep to locate symbols
- Use `codegraph_callers` / `codegraph_callees` to trace call flow
- Use `codegraph_impact` to verify the blast radius of changes
- Do NOT use codegraph_explore in the main audit session —
  spawn the `explore` agent instead
- Fall back to grep/read only when codegraph returns no results

**Strict requirement: finish all spec reading before reading any implementation.**

### Mode D — Final Audit order (all specs → all code)

```
Step 1 — Build spec knowledge
1. AGENTS.md
2. ALL files under _doc/specs/ (full baseline, not a single module)

Step 2 — Review all code
3. All source code and tests in scope
4. _doc/logs/CURRENT_STATE.md
```

**Strict requirement: read ALL spec files before reading ANY code.**

---

## 3. Audit Checklist

### Layer A: Spec Conformance (most important)

- **A1. Function signatures**: Do all spec-required functions exist? Are signatures correct?
- **A2. Function behavior**: Return value format, success/failure scenarios, exception handling?
- **A3. Process logic**: Business flow, state machines, conditional branches per spec?
- **A4. External calls**: subprocess / API call parameters match spec?
- **A5. Boundary conditions**: Edge cases handled per spec?

### Layer B: TDD Conformance

- **B1. Test completeness**: Every test item in the task sheet has a corresponding test?
- **B2. Test quality**: Tests verify behavior, not just "no error"?
- **B3. Test isolation**: All subprocess calls and file I/O mocked? Check whether tests use a mock/patch mechanism. If any I/O operation or subprocess call appears without being intercepted by a mock or patch — regardless of language or library — mark as CRITICAL.
- **B4. Coverage**: Meets thresholds defined in `PROJECT_STRUCTURE.md`?

### Layer C: Standards Conformance

- **C1. Naming**: Follows AGENTS.md §3?
- **C2. Code style**: Type hints, docstrings, import order?
- **C3. Security**: No hardcoded passwords, keys, or URLs?

### Layer D: Documentation Completeness

- **D1. Task log**: `_doc/logs/task-<id>.md` exists and is correctly formatted?
- **D2. CURRENT_STATE.md**: Updated? Next step clear?
- **D3. CURRENT_STATE.md integrity**: Does "Spec files needed" list files that actually exist? Does "Known Pitfalls" reflect issues recorded in the task log? Is "Last verified by" updated to reflect this audit?

### Mode D Checks (Final Audit only — not per-task layers)

- **Mode D-Coverage**: Walk each clause of the final spec; identify which code implements it. A spec clause implemented by no code is a coverage gap.
- **Mode D-Drift**: For each piece of code in scope, does it conform to the LATEST spec (not the spec as it was when the code was written)? Flag any divergence.

---

## 4. Reporting Standards

### 3.1 State facts only — no code patches

Format: "Spec requires X. Implementation does Y." Expected and actual values may be described freely.

**Never provide**: code fixes, implementation snippets, or patches of any kind.

### 3.2 Issue format

| Field | Description |
|-------|-------------|
| ID | `<layer code>-<sequence>`, e.g. A1-001 |
| Severity | `CRITICAL` / `WARNING` |
| Location | Exact file path and line number |
| Spec reference | Section and original text from spec doc |
| Finding | "Spec requires X. Implementation does Y." |

### 3.3 Ambiguous spec

List as a separate "Spec Ambiguities" section at the end of the report. Describe only — Architect arbitrates.

---

## 5. Audit Conclusion

| Conclusion | Code | Condition |
|-----------|------|-----------|
| Pass | ✅ PASS | No CRITICAL or WARNING |
| Fail | 🔴 FAIL | Any CRITICAL or WARNING |

**Modes A / B / C**: All issues must be resolved before the current task closes. No "carry to next task".

**Mode D (Final Audit)**: FAIL means "a coverage or drift gap exists, pending owner adjudication". Facts are reported to the owner, who makes the final arbitration call. Gaps are not auto-blocking — owner decides.

---

## 6. Report Format

### Mode C report: `audit-<task-id>-<n>.md`

```markdown
# audit-<task-id>-<n>.md

## Audit Info
- Task: task-XXX
- Auditor: <Claude / Gemini>
- Date: YYYY-MM-DD
- Conclusion: ✅ PASS / 🔴 FAIL

## Issues

### <ID> (<Severity>)
- Location: `<file path>:<line>`
- Spec reference: `<doc>` §X: "<original text>"
- Finding: Spec requires X. Implementation does Y.

## Spec Ambiguities (if any)
<describe contradictions or gaps found in the spec>
```

### Mode D report: `final-audit-<n>.md`

```markdown
# final-audit-<n>.md

## Audit Info
- Scope: whole-feature
- Auditor: <Claude / Gemini>
- Date: YYYY-MM-DD
- Baseline: all of _doc/specs/ (latest)
- Conclusion: ✅ PASS / 🔴 FAIL (pending owner adjudication if FAIL)

## Coverage Gaps

### <ID> (CRITICAL / WARNING)
- Spec reference: `<doc>` §X: "<clause text>"
- Finding: Spec clause implemented by no code.

## Drift Gaps

### <ID> (CRITICAL / WARNING)
- Location: `<file path>:<line>`
- Spec reference: `<doc>` §X: "<original text>"
- Finding: Spec requires X. Implementation does Y.

## Spec Ambiguities (if any)
<describe contradictions or gaps found in the spec>
```

---

## 7. Prohibited

- ❌ Modify any code or spec documents
- ❌ Downgrade or skip issues because "the implementer may have had a reason"
- ❌ Give a conclusion before completing all four layers
- ❌ Re-interpret the spec after reading the implementation
- ❌ Provide any code fixes, implementation snippets, or patches
- ❌ Mark any issue as "can carry to next task"
