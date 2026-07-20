# AGENTS.md — AI Agent Collaboration Framework
# 正體中文說明：本文件為所有 AI Agent 的協作主規範。如有疑問請直接詢問。

> This is the single source of truth for all agents.
> CLAUDE.md / GEMINI.md are AI-specific supplements — this file takes precedence.
> All roles (Architect / Controller / Implementer / Auditor) must read this file first.

---

## Quick Start — Copy the relevant line to begin

```
# Explore mode (freeform thinking, no artifacts by default — use before Architect when direction is undecided)
[EXPLORE] Read AGENTS.md and EXPLORE.md, then explore freely.

# Architect mode (planning + execution lead)
[ARCHITECT] Read AGENTS.md, confirm index, report current state briefly, then await instructions.

# Implementer mode (execute a task)
[IMPLEMENTER] Read AGENTS.md and _doc/logs/CURRENT_STATE.md, then execute task-XXX.

# Auditor mode (audit)
[AUDITOR] Read AGENTS.md and AUDITOR.md, then audit task-XXX.

# Controller mode (autonomous single-task implement→audit convergence)
[CONTROLLER] Read AGENTS.md and CONTROLLER.md, then drive the assigned task-XXX to DONE or BLOCKED.
```

---

## 0. Role Overview

| Role | Spec File | Core Responsibility |
|------|-----------|-------------------|
| Explore | `EXPLORE.md` | Freeform thinking mode before Architect's formal process — build context, lay out candidate directions, no binding output |
| Architect | `ARCHITECT.md` | Requirements, spec design, task breakdown, execution lead, audit trigger |
| Controller | `CONTROLLER.md` | Drive a single task's implement→audit convergence; decide DONE or BLOCKED |
| Implementer | `IMPLEMENTER.md` | Implement per task spec, write tests, maintain logs |
| Auditor | `AUDITOR.md` | Audit implementation against spec, produce objective report |

**Role exclusivity: one role per session. No dual roles.**

**Explore is not a workflow role** — it produces no artifacts by default (see
`EXPLORE.md` §2) and makes no binding decisions. Invoke it before Architect's
Mode A grilling interview when the requirement direction is still
undecided; skip it for requirements that are already well-defined.

---

## 1. Document Chain

```
AGENTS.md                  ← Master spec (this file)
├── EXPLORE.md             ← Explore mode spec (pre-Architect freeform thinking, no artifacts)
├── ARCHITECT.md           ← Architect role spec
├── CONTROLLER.md          ← Controller role spec
├── IMPLEMENTER.md         ← Implementer role spec
├── AUDITOR.md             ← Auditor role spec
├── DISPATCHER_CONTRACT.md ← Orchestrator ⇄ framework contract (external Dispatcher is out of scope; only its obligations are defined)
├── CLAUDE.md              ← Claude Code supplements
├── GEMINI.md              ← Gemini supplements
├── PROJECT_STRUCTURE.md   ← Project layout and module map (fill per project)
├── CONTEXT.md             ← Shared domain language / glossary (fill per project)
└── _doc/
    ├── specs/             ← Module specs (maintained by Architect) — merged baseline
    │   └── changes/       ← Delta documents (ADDED/MODIFIED/REMOVED); archived after merge
    ├── tasks/             ← Task sheets (maintained by Architect)
    ├── logs/
    │   ├── CURRENT_STATE.md   ← Current project state (update every session end)
    │   ├── QUESTIONS.md       ← Pending questions (Implementer writes, Architect resolves)
    │   └── task-XXX.md        ← Per-task execution log
    └── audits/            ← Audit reports (produced by Auditor)
```

---

## 2. Universal Working Principles

### 2.1 Action Logging (mandatory for all roles)

**All significant actions must be written to log. Nothing exists only in conversation memory.**

Required sequence:
1. **Before action**: log "what" and "why"
2. **Execute**
3. **After action**: log result (success / failure / key output)

```
[YYYY-MM-DD HH:MM] [PLAN] <action description>
  Reason: <why>
  Expected change: <scope>
[YYYY-MM-DD HH:MM] [DONE] <action description>
  Result: <success/failure + key output>
```

Applies to: file edits, command execution, design decisions, problem discovery, session start/end.

### 2.2 Session Continuity

On session start: read `_doc/logs/CURRENT_STATE.md` first. Confirm active task, last state, open questions.

On session end: update `CURRENT_STATE.md` so the next session can resume without context loss.

### 2.3 TDD Principles

- Write tests before implementation; tests must fail first, pass after
- Coverage thresholds defined in `PROJECT_STRUCTURE.md`
- All subprocess calls must be mocked (no real system commands in tests)
- All file I/O must be mocked or use `tmp_path` (no real paths in tests)

When codegraph is initialized (.codegraph/ exists):
- Use `git diff --name-only | codegraph affected --stdin` to identify
  impacted tests after each change
- Run only affected tests during development cycles
- Run full test suite only before session end

### 2.4 Language

- All docs, logs, and explanations: **Traditional Chinese (Taiwan usage)**
- Code, variable names, commit messages, shell commands: **English**

Prohibited terms (Mainland Chinese → Taiwan Chinese):

| ❌ Prohibited | ✅ Use instead |
|--------------|---------------|
| 文件 | 檔案 |
| 软件 | 軟體 |
| 信息 | 資訊 |
| 调用 | 呼叫 |
| 接口 | 介面 / API |
| 获取 | 取得 |
| 实现 | 實作 |
| 数据库 | 資料庫 |

### 2.5 Continuous Execution vs Ambiguity (precedence)

Inside a frozen spec with no open questions, execution proceeds without
pausing between tasks. The moment the spec does not cover a scenario, or any
ambiguity arises, ambiguity-terminate takes precedence over speed. This
precedence is absolute and applies to all roles.

### 2.6 Data Safety

During any session (implementation, audit, verification):
- Never modify production data without explicit user confirmation
- For login/auth testing: use .env credentials, fixtures, or test accounts first
- If no test account exists, inform the user and wait for explicit approval
  before using set_password or equivalent
- After any credential change, immediately report to user and prompt them to revert
- Never hardcode or log credentials in any file

### 2.7 Git-state authority (live-git-only)

Any claim about git state — a commit exists, the working tree is clean, a push
succeeded, what HEAD is — must be derived from a live git command at the moment
of the decision. Never treat an injected environment / `gitStatus` snapshot
(e.g. the status block in a session's opening system reminder) as current: it is
frozen when the session starts and goes stale the instant anything is committed.
Never assert git state from conversation memory.

Role division:
- **Auditor** is git-agnostic: it makes no git-state claim at all (AUDITOR.md
  §0). Its evidence is files on disk (read live) and the spec/task/log docs.
- **Controller** is the git-state authority within a task: it verifies commit,
  cleanliness, and HEAD with live git (CONTROLLER.md §6, CR-50/51/52).
- The **external Dispatcher / orchestrator** is out of framework scope, but the
  same rule binds it by contract for any git- or session-state it reports
  (see DISPATCHER_CONTRACT.md).

---

## 3. Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Python module / function / variable | `snake_case` | `get_user_by_id` |
| Python class | `PascalCase` | `UserProfile` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Test function | `test_<behavior>_<scenario>` | `test_login_with_invalid_password` |
| Task ID | `task-<3 digits>` | `task-007` |
| Audit report | `audit-<task-id>-<n>` | `audit-007-1` |

---

## 4. Document Lock Rules

**During Implementer execution, the task's assigned spec delta(s) are frozen.**

Spec changes are delta-based (see ARCHITECT.md § Spec Design Standards ›
Delta-based change management): `_doc/specs/<module>.md` is the merged
baseline, and each change is authored as a delta under
`_doc/specs/changes/<change-id>.md`. Freeze granularity is the delta a task
was dispatched against, not the whole module spec.

| Document | During Implementer run | Notes |
|----------|----------------------|-------|
| `_doc/tasks/<current>.md` | ❌ Frozen | Implementer's sole reference |
| `_doc/specs/changes/<assigned delta>.md` | ❌ Frozen | The delta(s) this task was dispatched against |
| `_doc/specs/<module>.md` (merged baseline) | ✅ Read-only reference | Source of truth for everything not covered by an open delta; not itself locked |
| `AGENTS.md` and role specs | ✅ Editable | Affects future tasks only |
| `_doc/logs/CURRENT_STATE.md` | ✅ Editable | Logging use |
| Other `_doc/specs/changes/*.md` (different, non-overlapping delta) | ✅ Editable | Parallel tasks on non-overlapping deltas of the same spec do not block each other |

Two open deltas touching the same spec section is a **conflict** — Architect
must decide merge order or re-split scope. Implementer must never resolve
this on its own judgment.

Emergency spec correction: wait for current session to end, fix in next session via audit mechanism.

---

## 5. Log Format Specs

### 5.1 CURRENT_STATE.md

```markdown
# CURRENT_STATE.md

## Status
- Last updated: YYYY-MM-DD
- Active task: task-XXX (or "none")
- Overall progress: <one sentence>
- Last verified by: <Auditor / Architect> after task-XXX (or "not yet verified")

## Active Context
- Spec files needed: _doc/specs/XXX.md, _doc/specs/YYY.md
- Open questions: Q-XXX (or "none")
- Depends on (unresolved): task-XXX (or "none")

## Next Session Should
<clear next action>

## Known Pitfalls
<extracted from past task logs — specific gotchas the next session should know>
(or "none")

## Decision Log
| Date | Decision | By |
|------|----------|----|
| YYYY-MM-DD | <decision> | Architect / Owner |
```

> CURRENT_STATE.md accuracy is verified by Auditor during post-audit. It is not guaranteed to be real-time correct. If in doubt, cross-check against the cited spec files directly.

### 5.2 Task Log (`_doc/logs/task-XXX.md`)

> Architect must confirm all dependencies are completed before invoking Implementer. A task with unresolved dependencies must not be started.

```markdown
# task-XXX Execution Log

## Info
- Task: task-XXX
- Affects: _doc/specs/XXX.md (or "none")
- Depends on: task-XXX, task-XXX (or "none")
- Consumes: <upstream interfaces this task relies on — exact signatures: function name, params, return type, from completed dependency tasks. Or "none">
- Produces: <interfaces later tasks will rely on — exact function names, params, return types. Or "none">
- Out of scope: <explicitly list what is NOT included in this task>
- Relevant files: <files Implementer should read, e.g. apps/auth/views.py, apps/auth/models.py>
- Entry point: <specific file or function to start from>
- Executor: <Claude / Gemini>
- Start: YYYY-MM-DD HH:MM
- End: YYYY-MM-DD HH:MM
- Base commit: <SHA at task start, or "n/a">
- Head commit: <SHA at task end, or "n/a">

## Objective
<one sentence>

## Execution Log
[per §2.1 format]

## Test Results
<paste actual test output>

## Decisions Made
<deviations from task sheet or self-judgments — must be filled honestly>

## Open Questions (→ QUESTIONS.md)
<or "none">
```

### 5.3 QUESTIONS.md

```markdown
# QUESTIONS.md — Pending Questions

## Open

### Q-<n>: <title>
- Raised by: <role>
- Date: YYYY-MM-DD
- Source task: task-XXX
- Description: <specific>
- Impact: <affected features or specs>

## Resolved

### Q-<n>: <title>
- Resolved: YYYY-MM-DD
- Resolution: <Architect's ruling>
```

### 5.4 Architect Session Log (`_doc/logs/architect-session-YYYY-MM-DD.md`)

```markdown
# Architect Session — YYYY-MM-DD

## Objective
## Completed
## Spec Decisions
## Next Steps
```

---

## 6. Audit Trigger Guidelines

Audits are tiered. Use the lightest tier that fits; escalate only when a
trigger fires.
These triggers are defined by task risk, not by who executes them. The executor — Architect in manual execution, Controller in autonomous execution — applies them.

### 6.1 Pre-implementation spec consistency check (Mode A)
Trigger a Mode A check before invoking Implementer when any of these hold:

| Situation | Trigger Mode A? |
|-----------|-----------------|
| Task sheet references 2+ spec files | ✅ |
| Task has depends_on entries | ✅ |
| Task adds or modifies any API interface | ✅ |
| Task modifies a shared structure used by multiple modules | ✅ |
| None of the above | ⛔ skip |

Mode A scope is spec consistency only — no code review. Prompt:
`[AUDITOR] Read AGENTS.md and AUDITOR.md, then perform a pre-implementation spec consistency check for task-XXX. Verify only: are all references in the task sheet consistent with the cited spec files? Do not review any code.`

### 6.2 Post-task review gate (Mode B) — runs after every task
After each Implementer task completes, the executor dispatches the
`task-reviewer` agent (AUDITOR.md Mode B). It is not optional and not
skipped for small tasks.
- CLEAN → proceed to the next task (continuous execution, AGENTS.md §2.5)
- ESCALATE → trigger a Mode C full audit (§6.3)

### 6.3 Full audit (Mode C)
Trigger a Mode C four-layer audit when any of these hold:

| Situation | Trigger Mode C? |
|-----------|-----------------|
| Mode B gate returned ESCALATE | ✅ |
| New module, model, or API | ✅ |
| Complex logic (state machine, async, permissions) | ✅ |
| Implementer task log shows unexpected decisions | ✅ |
| Pure bugfix or minor test addition | ⛔ skip |
| Fixing audit-reported implementation gaps | ⛔ skip |

If Implementer raised questions in QUESTIONS.md, resolve them first, then
decide.

### 6.4 Final audit (whole feature) — Mode D
After all tasks in a plan complete, the executor (Architect in manual
execution, the Dispatcher in autonomous execution) triggers AUDITOR.md
**Mode D — Final Audit (whole-feature)**:
- Baseline: all of `_doc/specs/` (latest signed spec), NOT a single module
- Target: all code in scope, NOT a single task
- Checks: Coverage (spec clause with no implementing code) and Drift (code that no longer conforms to the latest spec)
- Report: `_doc/audits/final-audit-<n>.md`

Audit report naming: `_doc/audits/audit-<task-id>-<n>.md` (Mode C),
`_doc/audits/review-<task-id>-<n>.md` (Mode B), `_doc/audits/pre-audit-<task-id>-<n>.md` (Mode A),
`_doc/audits/final-audit-<n>.md` (Mode D); n starts at 1.
All CRITICAL and WARNING items must be resolved before moving to the next task.
