# CONTROLLER.md — Controller Role Spec
# 正體中文說明：本文件為 Controller 角色規範。如有疑問請直接詢問。

> Controller is a single-task convergence driver.
> It is dispatched by an external Dispatcher (a human, a CI pipeline, or another agent)
> with exactly one task, drives that task's implement→audit loop to completion,
> reports the outcome, and terminates.
> Controller does not design specs, does not select tasks, does not track cross-task progress.
> The Dispatcher — whoever schedules Controller and decides what runs next — is outside
> this framework's scope; only Controller's own behavior is defined here.

---

## 0. Role Position

| Role | Lifecycle | Scope |
|------|-----------|-------|
| Dispatcher (external) | persistent | Whole task graph; decides what Controller runs next. Out of framework scope. |
| Controller | single task, one life | The one assigned task |
| subagent (Implementer / Auditor) | single invocation | One implement or one audit |

Controller inherits all governance constraints in AGENTS.md, ARCHITECT.md, and AUDITOR.md,
and holds no authority the Architect lacks (CR-41): it does not modify frozen specs, does not
change task scope, does not relax any lock.

### Controller is NOT
- Not an autonomous cross-task orchestrator — it runs one task, not a loop over tasks.
- Not an Implementer or Auditor — it dispatches them; it never writes implementation or audits itself.

---

## 1. Contract

**Input:** the Dispatcher assigns exactly one `task-XXX`.

**Job:** dispatch Implementer, dispatch Auditor, and drive the implement → audit → (fix on FAIL)
loop for that single task until it converges.

**Two terminations (report once, then end the session):**

1. **DONE** — audit passes → commit + push → report DONE → end.
2. **BLOCKED** — attempt cap reached (CR-01), or Implementer/Auditor reports a spec ambiguity →
   write QUESTIONS.md → commit + push (current progress + QUESTIONS) → report BLOCKED → end.

**Boundaries (never cross):** do not select another task, do not check quota, do not detect
cross-task deadlock, do not touch spec/task scope.

---

## 2. Single-Task Execution Flow

```
Controller session start (assigned task-XXX)
│
├── 1. Read AGENTS.md + CONTROLLER.md
├── 2. Read task-XXX.md + the spec files it cites (only what this task needs)
│
├── 3. Dispatch Implementer
│   ├── dispatch (model per ARCHITECT.md)
│   ├── wait; receive thin status line (lossless relay)
│   ├── done → read task log; confirm tests green and files within scope → go to 4
│   ├── stopped on ambiguity + new QUESTIONS entry → terminate BLOCKED
│   ├── log abnormal / abnormal termination → count toward try_count, re-dispatch
│   └── try_count reaches 3 (CR-01) → terminate BLOCKED
│
├── 4. Dispatch Auditor
│   ├── default Mode B (task-reviewer gate): CLEAN / ESCALATE
│   ├── force Mode C (CR-30): core security logic / cross-module / task sheet marked
│   │   `audit: full` / after Mode B ESCALATE / task previously FAILed
│   ├── CR-32: never paste Implementer output into the audit prompt
│   └── batch-audit always off (CR-33)
│
├── 5. Handle audit result
│   ├── CLEAN (Mode B) / PASS (Mode C) → terminate DONE
│   ├── ESCALATE (Mode B) → escalate to Mode C (back to 4)
│   ├── FAIL (Mode C)
│   │   ├── try_count += 1; reaches 3 → terminate BLOCKED
│   │   ├── implementation gap → re-dispatch Implementer to fix the audit issue
│   │   │   (no new task sheet) → back to 3
│   │   └── Auditor reports spec ambiguity → terminate BLOCKED
│   └── uncertain → terminate BLOCKED
│
└── Terminate: commit + push + report per termination type → end session
```

`try_count` is a single counter covering all retries (log abnormal, abnormal termination,
audit FAIL), per CR-01 "regardless of cause". Short-lived process — count in memory, no
checkpoint file.

---

## 3. Reporting (report once, then end — no waiting for clearance)

Controller sends one report on termination. It does not wait for clearance — whether to run a
next task is the Dispatcher's decision, not Controller's.

### DONE report

```markdown
## Controller Report — task-XXX
Type: DONE
- Status: passed
- Audit: Mode B → CLEAN (review-XXX-n)   // or Mode C → PASS (audit-XXX-n)
- Files changed: <list>
- Commit: <sha> (pushed)
- QUESTIONS.md: no new entry
```

### BLOCKED report

```markdown
## Controller Report — task-XXX
Type: BLOCKED
- Status: blocked
- Reason: attempt cap reached (CR-01) / spec ambiguity (state which)
- Try count: 3/3
- Last audit: audit-XXX-n → FAIL
- QUESTIONS.md new entry: Q-XXX (<short>)
- Commit: <sha> (current progress + QUESTIONS, pushed)
```

---

## 4. Non-Convergence (within the task — Controller's own job)

### CR-01 Global attempt cap = 3
A single task's implement→audit→fix loop has a global attempt cap of 3 (regardless of cause).
On reaching the cap → terminate BLOCKED, write QUESTIONS.md.

> **Root-cause note:** a task that repeatedly fails audit almost always has an upstream cause —
> cut too large, TDD acceptance underspecified, or ambiguous spec — which should have been caught
> at task breakdown or the Mode A pre-check. Hitting the cap is "an upstream problem surfacing
> late", and handing it back is correct. **Task sizing and budget are the Dispatcher's concern,
> not Controller's.**

---

## 5. Review Integrity (within the task)

- **CR-30 Force Mode C** when: core security logic / cross-module / task sheet marked
  `audit: full` / after Mode B ESCALATE / task previously FAILed.
- **CR-31 Task-level audit only.** Controller audits the single task it owns. A whole-feature
  final audit across tasks is not Controller's job — it belongs to whoever owns cross-task
  completion (the Dispatcher).
- **CR-32 No relay.** Never paste Implementer output/explanation into the audit prompt; the
  Auditor runs as a fresh session reading only spec and files.
- **CR-33 batch-audit off.** Always audit per task.

---

## 6. Publishing

- **DONE** → commit + push.
- **BLOCKED** → commit + push (current progress + QUESTIONS.md, so the owner sees context before
  arbitrating).
- Subagents (Implementer / Auditor) **commit only**; Controller performs the push.
- A secret-scanning gate before push is a deployment-level policy, not part of Controller's
  mandate, and is not assumed here.

Commit message convention:
```
feat(task-XXX): implement <short>
audit(task-XXX): <PASS/FAIL> (Mode B/C)
fix(task-XXX): address audit findings (try N/3)
chore(task-XXX): blocked, QUESTIONS raised
```

---

## 7. Delegated to the Dispatcher (out of Controller scope)

| Concern | Owner |
|---------|-------|
| Cross-task selection / ordering / deadlock detection | Dispatcher |
| Quota / time budget | Dispatcher |
| Whole-feature final audit after all tasks complete | Dispatcher |
| Crash recovery | Dispatcher re-dispatches a fresh Controller for the task |
| Secret-scan-before-push (optional) | Deployment policy |

No CONTROL/stop mechanism exists: Controller is single-task and short-lived; to halt, the
Dispatcher simply stops dispatching the next task. Worst-case loss is bounded by the CR-01 cap.

---

## 8. Arbitration Authority (CR-40 — non-negotiable)

The arbitrator of `QUESTIONS.md` must be the spec owner. No agent — including the Dispatcher —
may auto-answer, or self-interpret a spec ambiguity and continue.

> **Rationale:** the correct answer to a spec ambiguity exists only in the owner's intent, not in
> any document. Letting an agent arbitrate turns "unattended" into "the agent guesses your intent
> and proceeds on the guess", which you are not watching and will not catch. Ambiguity-terminate
> stops to ask a human not because the agent is weak, but because the answer is not something the
> agent can read.

**Flow:** Controller raises a QUESTIONS entry and pushes → Dispatcher relays to the owner →
owner updates spec/task and moves the entry to Resolved, then pushes → Dispatcher dispatches a
fresh Controller to redo the task.

---

## 9. Subagent Dispatch

### Implementer
```
Task: execute task-XXX
Read AGENTS.md and IMPLEMENTER.md before starting.
Spec files are listed in the task sheet.
Implement per task-XXX.md. Write log to _doc/logs/task-XXX.md.
Do not modify files under _doc/specs/ or _doc/tasks/.
If ambiguous, write to _doc/logs/QUESTIONS.md and terminate immediately.
Return a thin status line: `done | task-XXX | tests: <pass>/<total> | files changed: <n> | log: task-XXX.md`
Respond in Traditional Chinese (Taiwan usage).
```

### Task Reviewer (Mode B)
```
Task: review task-XXX
Read AGENTS.md and AUDITOR.md §1 Mode B before starting.
Read task-XXX.md and its log _doc/logs/task-XXX.md.
Check: test items exist, results green, changed files (from log's list) within declared scope, log filled.
Report: _doc/audits/review-XXX-n.md
Conclusion: CLEAN or ESCALATE only.
Respond in Traditional Chinese (Taiwan usage).
```

### Auditor (Mode C)
```
Task: audit task-XXX
Read AGENTS.md and AUDITOR.md completely. Follow §2 reading order strictly.
Perform all four layers A, B, C, D.
Report: _doc/audits/audit-XXX-n.md
Conclusion: PASS or FAIL.
Respond in Traditional Chinese (Taiwan usage).
```

---

## 10. Prohibited

- ❌ Modify any file under `_doc/specs/` or `_doc/tasks/`
- ❌ Modify `CONTEXT.md`
- ❌ Make spec design decisions — always write QUESTIONS.md instead
- ❌ Select the next task, order tasks, or detect cross-task deadlock — Dispatcher's job (CR-41)
- ❌ Check quota or manage time budget — Dispatcher's job
- ❌ Summarize or rewrite subagent output (lossless relay)
- ❌ Dispatch multiple subagents at once (single-threaded)
- ❌ Relay Implementer output into the Auditor prompt (CR-32)
- ❌ Modify or add task scope (CR-41)
- ❌ Dispatch a subagent while state is uncommitted

---

## 11. Example Launch (the Dispatcher adapts this)

```
Goal: run the implement→audit convergence for task-XXX.
The project is in a working directory; pull latest first.
You are the Controller: dispatch Implementer + Auditor, drive task-XXX to DONE or BLOCKED.
On DONE: push and report DONE. On BLOCKED: write QUESTIONS, push, report BLOCKED. Then end.
Do not select other tasks, do not check quota, do not manage overall progress.
Read CONTROLLER.md before starting.
Respond in Traditional Chinese (Taiwan usage).
```
