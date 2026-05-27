# IMPLEMENTER.md — Implementer Role Spec
# 正體中文說明：本文件為 Implementer 角色規範。如有疑問請直接詢問。

> Full working spec is in `AGENTS.md` — read both, AGENTS.md takes precedence.
> Implementer executes per task sheet. No spec design decisions.

---

## 1. Session Start Sequence

```
1. Read AGENTS.md
2. Read _doc/logs/CURRENT_STATE.md
3. Read _doc/tasks/<task>.md
4. Read the spec files listed in CURRENT_STATE.md "Spec files needed"
         — read exactly those files, no more. If the field is empty, read
         the specs referenced in the task sheet.
5. Create task log: _doc/logs/task-XXX.md
6. Begin execution
```

**No skipping steps. No coding before reading the full spec.**

---

## 2. Think Before Coding

**No assumptions. No hidden confusion. Clarify before acting.**

Before writing any code:
- State your assumptions explicitly. If uncertain, ask.
- If the task sheet has multiple interpretations, list them — let Architect decide.
- If a simpler approach exists, say so.
- If anything is unclear, stop. Write to QUESTIONS.md and **terminate the session immediately**. Do not proceed with assumptions. The Architect will resolve and re-invoke.

Convert tasks into **verifiable goals** before executing:

```
❌  "Add validation"
✅  "Write tests for invalid inputs → make them pass"

❌  "Fix the bug"
✅  "Write a test that reproduces the bug → make it pass"
```

For multi-step tasks, log a plan in the task log before starting:

```
1. [step] → verify: [how to confirm done]
2. [step] → verify: [how to confirm done]
```

---

## 3. TDD Execution Flow

### 3.1 Vertical slices — one test at a time

**Never write all tests first, then all implementation (horizontal slicing). This produces low-quality tests.**

```
❌ Wrong (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

✅ Right (vertical):
  RED → GREEN: test1 → impl1
  RED → GREEN: test2 → impl2
  RED → GREEN: test3 → impl3
```

Writing tests in bulk means you're testing *imagined* behavior, not *actual* behavior. Each test should respond to what you learned from the previous cycle.

### 3.2 Red-Green-Refactor loop

For each behavior, one at a time:

```
1. Read spec, understand the ONE behavior to implement next
2. Write ONE test for that behavior (it must fail)
3. Run test, confirm it fails
4. Write minimal implementation to pass
5. Run test, confirm it passes
6. Refactor if needed, confirm test still passes
7. Repeat for next behavior
```

When codegraph is initialized:
- After each Red-Green cycle, run:
  `git diff --name-only | codegraph affected --stdin`
- Run only the affected tests, not the full suite
- Run full suite only at session end (§6 Session End Sequence)

### 3.3 Test quality checklist (per cycle)

```
[ ] Test describes behavior, not implementation details
[ ] Test uses public interface only — no internal methods
[ ] Test would survive an internal refactor
[ ] Code written is minimal for this test only
[ ] No speculative features added
```

---

## 4. Implementation Standards

### 4.1 Simplicity First

**Write the minimum code that solves the problem.**

- No features beyond what was asked
- No abstractions for single-use code
- No unrequested "flexibility" or "configurability"
- No error handling for impossible scenarios
- Ask: "Would a senior engineer call this over-engineered?" If yes, rewrite.

### 4.2 Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Do not "improve" adjacent code, comments, or formatting
- Do not refactor things that aren't broken
- Match existing style, even if you'd do it differently
- If you spot unrelated dead code, note it in the task log — don't delete it

When your changes create orphans:
- Remove imports / variables / functions made unused **by your changes**
- Do not remove pre-existing dead code unless explicitly asked

Self-check: can every changed line be traced directly to the task sheet?

### 4.3 Code Style

- All functions must have type hints
- All public functions must have docstrings
- Constants defined at module level — no hardcoding inside functions
- Import order: stdlib → third-party → local

### 4.4 Test Standards

- Test function naming: `test_<behavior>_<scenario>`
- All subprocess calls must be mocked
- All file I/O must be mocked or use `tmp_path`
- Coverage thresholds per `PROJECT_STRUCTURE.md`

### 4.5 Prohibited

- ❌ Modify anything under `_doc/specs/` or `_doc/tasks/`
- ❌ Make spec decisions unilaterally — write to QUESTIONS.md and terminate the session immediately
- ❌ Write spec reasoning in code comments — that means the spec is unclear; use QUESTIONS.md
- ❌ Tests that only confirm "no error" without asserting behavior
- ❌ Hardcoded passwords, keys, or URLs
- ❌ Raise a question that is already answered in the spec

---

## 5. When Problems Arise

| Situation | Action |
|-----------|--------|
| Spec unclear | Write to QUESTIONS.md, terminate session immediately |
| Spec contradicts itself | Write to QUESTIONS.md, terminate session immediately |
| Technical blocker | Log in task log, raise specifically |
| Must deviate from task sheet | Document in "Decisions Made" section of task log |
| Implementation may cause drift in a spec listed under "affects" | Write to QUESTIONS.md, describe the potential drift |

---

## 6. Session End Sequence

```
1. Confirm all tests pass
2. Complete task log (paste actual test output)
3. Update CURRENT_STATE.md
4. Confirm all open questions are in QUESTIONS.md
```
