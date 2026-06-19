# PROJECT_STRUCTURE.md — Project Structure
# 正體中文說明：本文件為專案結構說明，各專案自行填寫。如有疑問請直接詢問。

> Fill this file per project. Keep the structure — replace the placeholders.

---

## Project Info

- **Name**: (fill in)
- **Stack**: (fill in, e.g. Python / Django / PostgreSQL)
- **Working directory**: (fill in, e.g. `/opt/project`)

## CONTEXT.md — Shared Domain Language

Every project should maintain a `CONTEXT.md` at the root. This is a shared glossary that eliminates ambiguity between the project owner and all agents.

Format:

```markdown
# CONTEXT.md — Project Domain Language

## Terms

**<Term>**:
<Definition — what it means in this project specifically>
*Avoid*: <common synonyms that mean something different here>

## Resolved Ambiguities

- "<vague phrase>" was ambiguous — resolved: it means <X>, not <Y>. (YYYY-MM-DD)
```

Rules:
- Only include terms meaningful to the domain, not implementation details
- Update inline during Architect grilling sessions, not in batch afterward
- When an Implementer encounters a term not in CONTEXT.md and meaning is unclear, write to QUESTIONS.md and terminate

---

## Directory Structure

Task sheets should declare dependencies via `depends_on`. Architect is responsible for dependency resolution before each invocation.

```
<project-root>/
├── src/
├── tests/
│   ├── conftest.py
│   └── unit/
├── _doc/
│   ├── specs/
│   ├── tasks/
│   ├── logs/
│   │   ├── CURRENT_STATE.md
│   │   ├── QUESTIONS.md
│   │   └── task-XXX.md
│   └── audits/
├── AGENTS.md
├── ARCHITECT.md
├── IMPLEMENTER.md
├── AUDITOR.md
├── CLAUDE.md
├── GEMINI.md
├── PROJECT_STRUCTURE.md
└── .claude/
    └── agents/
        ├── implementer.md   ← Claude Code subagent config (model + tools + pointer body)
        ├── pre-auditor.md   ← Claude Code subagent config
        ├── auditor.md       ← Claude Code subagent config
        ├── task-reviewer.md ← Claude Code subagent config (post-task review gate)
        └── explore.md       ← Claude Code subagent config (requires codegraph if used)
```

**Agent config authoring rule:** an `.claude/agents/*.md` body may only
(a) name the role, (b) point to the relevant role doc and section for its
behavior, and (c) state its conclusion vocabulary and response language.
It must not embed concrete tools or mechanisms. Mechanism is defined in
the role doc only; duplicating it in an agent body causes drift.

---

## .claude/agents/ — Claude Code Subagent Configs
Claude Code only. Gemini CLI and Codex use prompt tags directly.
These files specify model and tool permissions only.
All behavioral rules remain in AGENTS.md and role-specific files.
Changing rules in AGENTS.md / IMPLEMENTER.md / AUDITOR.md
automatically applies to all tools — no sync needed.

---

## .claudeignore

Create a `.claudeignore` file at the project root to exclude files
Claude should not read. Reduces unnecessary token consumption.

Recommended exclusions:
```
node_modules/
dist/
build/
.venv/
__pycache__/
*.min.js
*.lock
migrations/  (if auto-generated)
```

Add project-specific generated files and build artifacts as needed.

---

## Module Responsibilities

| Module | Responsibility |
|--------|---------------|
| (fill in) | (fill in) |

---

## Coverage Thresholds

| Scope | Threshold |
|-------|-----------|
| Overall | ≥ 80% (adjust per project) |
| (critical module) | ≥ 90% |

---

## Test Commands

```bash
# Standard run (required before every commit)
(fill in)

# With coverage report
(fill in)
```

---

## Spec File Map

| Module | Spec file |
|--------|-----------|
| (fill in) | `_doc/specs/<module>.md` |

---

## Common Commands

```bash
# Start dev environment
(fill in)

# Run service
(fill in)
```
