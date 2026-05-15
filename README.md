# spec-driven-agents

A governance framework for multi-agent AI collaboration.

Built for teams using Claude Code, Gemini CLI, or any agentic coding tool — where the hardest problem isn't getting the AI to write code, it's getting it to write *the right* code without going rogue.

---

## The Problem

You ask an AI agent to "add OTP login." It implements SSH key + OTP two-factor auth because that's "more secure." You didn't ask for that.

You ask it to fix a bug. It refactors three unrelated modules while it's at it.

You run multiple agents in parallel. One modifies a spec file mid-task. Another reads the stale version. Now your codebase reflects two conflicting realities.

These aren't hallucination problems. They're **governance problems** — no clear roles, no locked specs, no mechanism to surface ambiguity before it becomes bad code.

---

## How It Works

Three roles. Each with a clear mandate and hard boundaries.

```
Architect   — plans, designs specs, breaks down tasks, leads execution
Implementer — executes per spec, writes tests first, stops when ambiguous
Auditor     — independently verifies implementation against spec, no fixes
```

The key rules:

- **Spec docs are frozen** while an Implementer is running — no mid-flight changes
- **Ambiguity = immediate stop** — Implementer writes to QUESTIONS.md and terminates; Architect resolves before re-invoking
- **Auditor is always a fresh session** — isolated view, no contamination from implementation context
- **Architect is a lossless relay** — sub-agent output is transcribed verbatim, never "improved"
- **TDD is vertical slices** — one test → one implementation, never all tests first

---

## File Structure

```
your-project/
├── AGENTS.md              ← Master spec (start here)
├── ARCHITECT.md           ← Architect role rules
├── IMPLEMENTER.md         ← Implementer role rules
├── AUDITOR.md             ← Auditor role rules
├── CLAUDE.md              ← Claude Code entry point
├── GEMINI.md              ← Gemini CLI entry point
├── CONTEXT.md             ← Shared domain language / glossary
├── PROJECT_STRUCTURE.md   ← Project layout (fill per project)
└── _doc/
    ├── specs/             ← Module specs (Architect writes)
    ├── tasks/             ← Task sheets (Architect writes)
    ├── logs/
    │   ├── CURRENT_STATE.md   ← Session continuity
    │   ├── QUESTIONS.md       ← Ambiguity queue
    │   └── task-XXX.md        ← Execution logs
    └── audits/            ← Audit reports
```

---

## Quickstart

**Use this repo as a GitHub template** (click "Use this template" above), then:

```bash
# 1. Fill in your project details
vim PROJECT_STRUCTURE.md   # directory layout, test commands, coverage thresholds
vim CONTEXT.md             # domain glossary — add terms as you go

# 2. Start an Architect session (paste this into Claude Code or Gemini)
[ARCHITECT] Read AGENTS.md, confirm index, report current state briefly, then await instructions.

# 3. The Architect will grill you on requirements before writing any spec
# 4. Once specs exist, Implementer sessions run automatically via sub-agent calls
# 5. Auditor sessions are triggered by Architect when risk warrants it
```

---

## Role Switching

All role switching uses explicit tags — no semantic guessing:

```
[ARCHITECT]   — planning, spec design, execution lead
[IMPLEMENTER] — execute task-XXX per spec
[AUDITOR]     — audit task-XXX against spec
```

No tag = Implementer by default.

---

## What Makes This Different

| | This framework | mattpocock/skills | Kiro SDD |
|--|---------------|-------------------|----------|
| Approach | Governance rules | Slash command skills | Agentic IDE |
| Auditor role | ✅ Independent session | ❌ | ❌ |
| Spec locking | ✅ Frozen during impl | ❌ | ❌ |
| Ambiguity handling | ✅ Terminate + queue | ❌ | ❌ |
| Tool lock-in | None | None | Kiro only |
| Token overhead | Low (English, no redundancy) | Very low (on-demand) | N/A |

---

## Compatibility

Works with any AI agent that reads markdown files:

- ✅ Claude Code
- ✅ Gemini CLI  
- ✅ Codex CLI
- ✅ Any agent with file access

---

## License

MIT
