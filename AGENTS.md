# AGENTS.md — AI Agent Collaboration Framework
# 正體中文說明：本文件為所有 AI Agent 的協作主規範。如有疑問請直接詢問。

> This is the single source of truth for all agents.
> CLAUDE.md / GEMINI.md are AI-specific supplements — this file takes precedence.
> All roles (Architect / Implementer / Auditor) must read this file first.

---

## Quick Start — Copy the relevant line to begin

```
# Architect mode (planning + execution lead)
[ARCHITECT] Read AGENTS.md, confirm index, report current state briefly, then await instructions.

# Implementer mode (execute a task)
[IMPLEMENTER] Read AGENTS.md and _doc/logs/CURRENT_STATE.md, then execute task-XXX.

# Auditor mode (audit)
[AUDITOR] Read AGENTS.md and AUDITOR.md, then audit task-XXX.
```

---

## 0. Role Overview

| Role | Spec File | Core Responsibility |
|------|-----------|-------------------|
| Architect | `ARCHITECT.md` | Requirements, spec design, task breakdown, execution lead, audit trigger |
| Implementer | `IMPLEMENTER.md` | Implement per task spec, write tests, maintain logs |
| Auditor | `AUDITOR.md` | Audit implementation against spec, produce objective report |

**Role exclusivity: one role per session. No dual roles.**

---

## 1. Document Chain

```
AGENTS.md                  ← Master spec (this file)
├── ARCHITECT.md           ← Architect role spec
├── IMPLEMENTER.md         ← Implementer role spec
├── AUDITOR.md             ← Auditor role spec
├── CLAUDE.md              ← Claude Code supplements
├── GEMINI.md              ← Gemini supplements
├── PROJECT_STRUCTURE.md   ← Project layout and module map (fill per project)
├── CONTEXT.md             ← Shared domain language / glossary (fill per project)
└── _doc/
    ├── specs/             ← Module specs (maintained by Architect)
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

**During Implementer execution, the current task's spec docs are frozen.**

| Document | During Implementer run | Notes |
|----------|----------------------|-------|
| `_doc/tasks/<current>.md` | ❌ Frozen | Implementer's sole reference |
| `_doc/specs/<current module>.md` | ❌ Frozen | Same |
| `AGENTS.md` and role specs | ✅ Editable | Affects future tasks only |
| `_doc/logs/CURRENT_STATE.md` | ✅ Editable | Logging use |
| Other `_doc/specs/*.md` | ✅ Editable | Not a current task dependency |

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

## Next Session Should
<clear next action>

## Open Questions
<list from QUESTIONS.md, or "none">

## Decision Log
| Date | Decision | By |
|------|----------|----|
| YYYY-MM-DD | <decision> | Architect / Nico |
```

### 5.2 Task Log (`_doc/logs/task-XXX.md`)

```markdown
# task-XXX Execution Log

## Info
- Task: task-XXX
- Executor: <Claude / Gemini>
- Start: YYYY-MM-DD HH:MM
- End: YYYY-MM-DD HH:MM

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

Audits are **risk-driven**, not mandatory every round.

| Situation | Recommendation |
|-----------|---------------|
| New module, model, or API | ✅ Audit |
| Complex logic (state machine, async, permissions) | ✅ Audit |
| Implementer task log shows unexpected decisions | ✅ Audit |
| Implementer raised questions in QUESTIONS.md | Resolve first, then decide |
| Pure bugfix or minor test addition | ⛔ Skip |
| Fixing audit-reported implementation gaps | ⛔ Skip |

Audit report naming: `_doc/audits/audit-<task-id>-<n>.md` (n starts at 1).
All CRITICAL and WARNING items must be resolved before moving to the next task.
