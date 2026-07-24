# DISPATCHER_CONTRACT.md — Orchestrator ⇄ Framework Contract
# 正體中文：本文件定義框架對「外部 Dispatcher／Orchestrator」的契約要求。

> The Dispatcher / orchestrator (a human, a CI pipeline, a tmux/cron harness, a
> webhook service — a deployment's own driver) is OUT of framework scope: the
> framework never implements it. This file defines only the CONTRACT any
> orchestrator must honor so the shipped role contracts (Controller, Auditor,
> ...) hold. Implementation is the deployment's own concern.

---

## 1. Framework ships vs you provide

| The framework ships | You (the orchestrator) provide |
|---------------------|--------------------------------|
| Role behavioral contracts (Controller / Auditor / ...) | Session spawning, scheduling, lifecycle |
| A single task's convergence loop | Cross-task selection, ordering, deadlock, quota, budget |
| Outcome token contract (`Type: DONE/BLOCKED/RETRY`) + `Blocked-reason` sub-field | Parsing them; re-spawning a fresh Controller on RETRY; spawning a fresh Architect on `needs-replan`; relaying QUESTIONS on `spec-ambiguity`; deciding what runs next |
| BLOCKED-reason routing semantics + the replan cap (enforced inside Controller / Architect) | Spawning the fresh Architect / Controllers mechanically — WITHOUT counting attempts/replans or judging whether to retry |
| Git-state authority rules (AGENTS.md §2.7) | Honoring them for anything YOU report |

## 2. Session input contract (answers "do I pass git status in?")

- The only required input to a Controller is exactly one `task-XXX`
  (CONTROLLER.md §1). You do NOT pass authoritative git state in.
- The harness may inject an environment / `gitStatus` snapshot into a spawned
  session. That snapshot is NON-AUTHORITATIVE and frozen at spawn. Roles ignore
  it for decisions and self-verify with live git (AGENTS.md §2.7). You must not
  rely on it either.
- Roles that need git state derive it live themselves; you do not feed it in.

## 3. Git- and session-state you report must be live

Before you report a git- or session-state fact:
- Derive git state from a live git command.
- Derive session/process existence from a live query (the actual process /
  session table), not from a prior report or a pane's leftover prompt.
- Terminal state, next-task prompt, and session termination are three distinct
  moments — never infer one from another. A pane showing the next task's prompt
  does not mean that task has been dispatched or is running.

## 4. Triggers the framework delegates to you

- Trigger AUDITOR.md Mode D (whole-feature final audit) after all tasks complete
  (AGENTS.md §6.4).
- Re-dispatch a fresh Controller on crash recovery (CONTROLLER.md §7).
- Re-spawn a fresh Controller on `Type: RETRY` (CONTROLLER.md §1 / §4 CR-02): a RETRY report ends
  one Controller life but NOT the task — read the token, spawn a new Controller for the same task;
  it resumes from the checkpoint. Do not treat a RETRY session's termination as task completion.
- On a `Type: BLOCKED` report, route by its `Blocked-reason` field — mechanically, never by your
  own judgment (CONTROLLER.md §3):
  - `needs-replan` → spawn a fresh Architect session for that task (autonomous replan mode,
    ARCHITECT.md §9). The Architect re-decomposes and owns the `replan_count` cap. You do NOT count
    replans, do NOT decide re-split vs escalate, and do NOT plain-re-dispatch the same task to a
    Controller. When the Architect emits new / corrected task sheets, dispatch fresh Controllers for
    them as usual.
  - `spec-ambiguity` → relay QUESTIONS.md to the owner; never auto-answer (CR-40).
- Do not invent a "reset" action: re-running a BLOCKED task against the same task sheet without a
  structural change (re-decomposition or owner ruling) is not a Dispatcher power (CONTROLLER.md
  CR-03). The only bounded ways forward from BLOCKED are the two routes above.
- Relay QUESTIONS.md items to the spec owner; never auto-answer (CR-40).
