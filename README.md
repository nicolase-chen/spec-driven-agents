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

Four workflow roles, plus an optional pre-planning thinking mode. Each with a clear mandate and hard boundaries.

```
Explore     — optional pre-Architect freeform thinking mode, produces no artifact by default
Architect   — plans, designs specs, breaks down tasks, leads execution
Controller  — drives a single task's implement→audit loop autonomously, reports DONE or BLOCKED
Implementer — executes per spec, writes tests first, stops when ambiguous
Auditor     — independently verifies implementation against spec, no fixes
```

The key rules:

- **Spec docs are frozen** while an Implementer is running — no mid-flight changes
- **Ambiguity = immediate stop** — Implementer writes to QUESTIONS.md and terminates; Architect resolves before re-invoking
- **Auditor is always a fresh session** — isolated view, no contamination from implementation context
- **Architect is a lossless relay** — sub-agent output is transcribed verbatim, never "improved"
- **TDD is vertical slices** — one test → one implementation, never all tests first
- **Review is tiered** — a lightweight per-task gate runs after every task; full independent audits and a final whole-feature audit fire only when risk warrants

---

## File Structure

```
your-project/
├── AGENTS.md              ← Master spec (start here)
├── EXPLORE.md             ← Explore mode (pre-Architect freeform thinking, no artifacts)
├── ARCHITECT.md           ← Architect role rules
├── CONTROLLER.md          ← Controller role rules
├── IMPLEMENTER.md         ← Implementer role rules
├── AUDITOR.md             ← Auditor role rules
├── CLAUDE.md              ← Claude Code entry point
├── GEMINI.md              ← Gemini CLI entry point
├── CONTEXT.md             ← Shared domain language / glossary
├── PROJECT_STRUCTURE.md   ← Project layout (fill per project)
├── _doc/
│   ├── specs/
│   │   ├── example.md         ← Module spec template
│   │   └── changes/           ← Delta documents (ADDED/MODIFIED/REMOVED); archived after merge
│   ├── tasks/
│   │   └── example.md         ← Task sheet template
│   ├── logs/
│   │   ├── CURRENT_STATE.md   ← Session continuity
│   │   ├── QUESTIONS.md       ← Ambiguity queue
│   │   └── task-XXX.md        ← Execution logs
│   └── audits/                ← Audit reports (Auditor produces)
└── .claude/
    └── agents/            ← Claude Code subagent configs (model + tools)
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

## Optional: Claude Code Subagents

If you use Claude Code, pre-configured subagent profiles are included in `.claude/agents/`.
Each agent is assigned an appropriate model to balance quality and cost:

| Agent | Model | Role |
|-------|-------|------|
| implementer | Haiku | Executes tasks — rule-following, no judgment needed |
| pre-auditor | Haiku | Spec consistency check before implementation |
| auditor | Sonnet | Full four-layer audit — requires reasoning |
| task-reviewer | Haiku | Post-task review gate — existence checks only |
| explore | Haiku | Codebase exploration, codegraph-aware |

Gemini CLI and Codex use prompt tags directly — subagent configs are Claude Code only.
All behavioral rules stay in AGENTS.md and role files, so all tools stay in sync automatically.

---

## Optional: CodeGraph Integration

[CodeGraph](https://github.com/colbymchenry/codegraph) pre-indexes your codebase into a
local SQLite knowledge graph. When initialized, the `explore` subagent uses it automatically.

**Install:**
```bash
npx @colbymchenry/codegraph
cd your-project
codegraph init -i
```

**What it does:**
- 90%+ fewer tool calls during codebase exploration
- Architect can run `codegraph_impact` before filling the `affects` field in task sheets
- Implementer uses `codegraph affected` to run only impacted tests

**Without CodeGraph:** everything works normally — explore agent falls back to grep/glob/read.

---

## Role Switching

All role switching uses explicit tags — no semantic guessing:

```
[EXPLORE]     — pre-Architect freeform thinking, no artifact by default
[ARCHITECT]   — planning, spec design, execution lead
[CONTROLLER]  — drive one task's implement→audit loop to DONE or BLOCKED
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

## Upgrading

To upgrade an existing project to a new framework version:

```bash
# Download new version into _ref/
mkdir -p _ref
curl -L https://github.com/nicolase-chen/spec-driven-agents/archive/vX.X.X.zip -o _ref/framework.zip
unzip _ref/framework.zip -d _ref/
rm _ref/framework.zip
```

Then run this migration prompt in Claude Code:

```
[ARCHITECT] Read AGENTS.md and all framework files in the project root,
then read _ref/spec-driven-agents-X.X.X/ for the new framework version.
Migrate the project's framework files to the new standard.
Preserve all project-specific content.
Do NOT touch _doc/specs/, _doc/tasks/, _doc/logs/, _doc/audits/
Report diff summary and wait for confirmation before making changes.
Respond in Traditional Chinese.
```

After confirming and applying changes:

```bash
rm -rf _ref/spec-driven-agents-X.X.X
git add AGENTS.md ARCHITECT.md IMPLEMENTER.md AUDITOR.md CLAUDE.md GEMINI.md
git commit -m "chore: migrate framework to spec-driven-agents vX.X.X"
git push
```

---

## License

MIT
