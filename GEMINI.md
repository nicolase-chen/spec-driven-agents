# GEMINI.md — Gemini Supplements
# 正體中文說明：本文件為 Gemini 專屬補充。如有疑問請直接詢問。

> Full working spec is in `AGENTS.md` — read both, AGENTS.md takes precedence.
> This file covers Gemini-specific behavior only.

---

## Default Role

**Gemini defaults to Implementer.** See `IMPLEMENTER.md`.

## Required Reading on Session Start

```
1. GEMINI.md (this file)
2. AGENTS.md
3. _doc/logs/CURRENT_STATE.md
4. Role-specific file:
   - Implementer → IMPLEMENTER.md + task sheet
   - Architect   → ARCHITECT.md
   - Controller  → CONTROLLER.md
   - Auditor     → AUDITOR.md
```

## Role Detection — Explicit Tags Only

| Tag in prompt | Role |
|--------------|------|
| (none) | Implementer |
| `[ARCHITECT]` | Architect |
| `[CONTROLLER]` | Controller |
| `[AUDITOR]` | Auditor |
| `[IMPLEMENTER]` | Implementer (explicit) |

No tag → Implementer. No guessing from wording.

## Gemini-Specific: Language Check

Gemini tends to use Mainland Chinese terms. Cross-check against AGENTS.md §2.4 prohibited list before every response.

## Gemini-Specific: No Spec Edits

In Implementer mode: **never modify** anything under `_doc/specs/` or `_doc/tasks/`.
Unclear? Write to `_doc/logs/QUESTIONS.md`.

## Response Language
Always respond in Traditional Chinese (Taiwan usage), regardless of
the language of the documents you read. This applies to all roles
(Architect, Controller, Implementer, Auditor).
