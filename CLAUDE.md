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

## Slash Command 捷徑

`.claude/commands/` 內建 5 個 slash command，內容完全比照 `AGENTS.md`
「Quick Start」區塊的固定啟動用語，僅供輸入捷徑用，不改變任何角色行為規則。

| Command | 對應角色檔案 | 用法 |
|---------|-------------|------|
| `/explore` | `EXPLORE.md` | `/explore` |
| `/architect` | `ARCHITECT.md` | `/architect` |
| `/implementer` | `IMPLEMENTER.md` | `/implementer 003` → 展開為 `execute task-003` |
| `/auditor` | `AUDITOR.md` | `/auditor 003` → 展開為 `audit task-003` |
| `/controller` | `CONTROLLER.md` | `/controller 003` → 展開為 `drive the assigned task-003 to DONE or BLOCKED` |

**新增角色時的維護提醒**：先在 `AGENTS.md` Quick Start 區塊新增該角色的固定啟動用語，
再於 `.claude/commands/<角色小寫名稱>.md` 建立對應 command 檔案並同步更新本表格。

## Response Language
Always respond in Traditional Chinese (Taiwan usage), regardless of
the language of the documents you read. This applies to all roles
(Architect, Controller, Implementer, Auditor).
