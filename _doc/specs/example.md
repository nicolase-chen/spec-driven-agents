# example.md — Module Spec Example
# 正體中文說明：這是規格文件的範例，請依實際模組內容填寫。

## Overview
One sentence describing what this module does.

## Models / Data Structures

### ExampleModel
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | int | ✅ | Primary key |
| name | str | ✅ | Display name |

## API Endpoints

### GET /api/example/
- Auth: Required
- Response: `{ id, name }`
- Error: 404 if not found

### POST /api/example/
- Auth: Required
- Body: `{ name }`
- Response: created object

## Business Logic
- Describe rules, validations, state transitions here
- Each rule on its own line

## Quality Rubric (optional)
- Define quality standards for Auditor to check
- e.g. All API responses must include error codes
