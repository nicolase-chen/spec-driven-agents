# CLAUDE.md — Claude Code Supplements
# 正體中文說明：本文件為 Claude Code 專屬補充。如有疑問請直接詢問。

> Full working spec is in `AGENTS.md` — read both, AGENTS.md takes precedence.
> This file covers Claude Code-specific behavior only.

---

## Default Role

**Claude Code defaults to Implementer.** See `IMPLEMENTER.md`.

## Required Reading on Session Start

```
1. CLAUDE.md (this file — auto-loaded)
2. AGENTS.md
3. _doc/logs/CURRENT_STATE.md
4. Role-specific file:
   - Implementer → IMPLEMENTER.md + task sheet
   - Architect   → ARCHITECT.md
   - Controller  → CONTROLLER.md
   - Auditor     → AUDITOR.md
   - Explore     → EXPLORE.md
```

## Role Detection — Explicit Tags Only

**Tags, not semantic inference.**

| Tag in prompt | Role |
|--------------|------|
| (none) | Implementer |
| `[EXPLORE]` | Explore |
| `[ARCHITECT]` | Architect |
| `[CONTROLLER]` | Controller |
| `[AUDITOR]` | Auditor |
| `[IMPLEMENTER]` | Implementer (explicit) |

No tag → Implementer. No guessing from wording.
`[EXPLORE]` → Explore mode (see `EXPLORE.md`): pre-Architect freeform
thinking, produces no artifact by default.

## Response Language
Always respond in Traditional Chinese (Taiwan usage), regardless of
the language of the documents you read. This applies to all roles
(Architect, Controller, Implementer, Auditor).
