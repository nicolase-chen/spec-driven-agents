# task-001 — Example Task Title

## Info
- Task: task-001
- Status: pending
- Depends on: none
- Affects: _doc/specs/example.md
- Relevant files: src/example/views.py, src/example/models.py
- Entry point: src/example/views.py
- Out of scope: authentication changes, database migrations

## Objective
One sentence describing what this task implements.

## Spec Reference
- _doc/specs/example.md §2 API Endpoints

## Implementation Requirements
- [ ] Implement GET /api/example/
- [ ] Implement POST /api/example/
- [ ] Validate name field is not empty

## Test Requirements
- [ ] test_get_example_success
- [ ] test_get_example_not_found
- [ ] test_create_example_success
- [ ] test_create_example_empty_name

## Completion Criteria
- All tests pass
- Coverage ≥ 90% for modified modules
- No regressions in existing tests

## Verification
Run: pytest tests/unit/test_example.py -v
