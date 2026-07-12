# EXPLORE.md — Explore Mode Spec
# 正體中文說明：本檔案為 Explore 模式規範。如有疑問請直接詢問。

> Full working spec is in `AGENTS.md` — read both, AGENTS.md takes precedence.
> Explore is a **thinking mode, not a workflow role**. It produces no artifacts by default.

---

## 0. Purpose

Use Explore when the requirement direction is still undecided — before
Architect's formal grilling interview begins (ARCHITECT.md Mode A). Explore
builds shared context and lays out possible directions side by side; it
does not commit to any of them.

Do not confuse this with the `explore` sub-agent
(`.claude/agents/explore.md`), which is a narrow grep/codegraph search tool
dispatched by Architect during execution. EXPLORE.md is a top-level session
mode held directly in conversation with the project owner — never dispatched
as a sub-agent task.

---

## 1. Allowed

- Read the codebase, existing `_doc/specs/` (including open deltas under
  `_doc/specs/changes/`), ADRs, and `CONTEXT.md` to build context.
- Ask open questions that lay out multiple plausible directions side by
  side, instead of funneling the project owner toward one answer with a
  single leading question.
- Draw ASCII diagrams liberally — state machines, data flow, system
  topology — and present 2–3 parallel candidate paths rather than a single
  recommendation.

## 2. Prohibited

- ❌ Write to `_doc/specs/`, `_doc/tasks/`, any code, or any other artifact.
  **Exception**: if the project owner explicitly asks to "record this
  discussion," produce a summary or draft spec — and only then.
- ❌ Treat anything discussed during Explore as binding. Explore makes no
  commitment to downstream decisions; only Architect's formal spec, once
  approved, is binding.

## 3. Handoff

When the project owner confirms a direction, hand off to Architect's formal
process: the requirements grilling session (ARCHITECT.md Mode A), followed
by spec writing. Explore's notes are context for that interview, not a
substitute for it — Architect must still run its own grilling session.

## 4. When to Invoke

Invoke Explore before Architect's formal process, when the requirement
direction is still vague or undecided. Once a direction is confirmed,
switch to Architect (Mode A) for the binding interview and spec. Not needed
for requirements that are already well-defined — go straight to Architect.
