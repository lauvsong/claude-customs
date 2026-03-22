# rules/03-task-management.md — TASKS & LESSONS (on request only)

## SUMMARY (READ FIRST)
- Task management (todo/lessons recording) is performed only "when explicitly requested".
- Record in `tasks/todo.md`, `tasks/lessons.md` when possible; otherwise output same format in chat.
- Checklist plan → progress marking → review section (results/verification/risk/rollback).
- Record lessons from user feedback corrections to prevent recurrence.

## RULES
### R1) Plan First (checklist)
- For non-trivial tasks, create a checklist plan (when requested).
- Checklist must include "verification items".

### R2) Plan Validation (conditional confirmation)
- Seek approval/confirmation before implementation if:
  - Destructive action / Prod impact / Contract breaking / Bulk data change / Security·permission change
- Otherwise: present plan and proceed.

### R3) Progress Tracking
- Check off completed items as you progress.
- Provide high-level summary per step (what/why/risk/next).

### R4) Review Section (result documentation)
- At task completion, record:
  - Change summary (scoped)
  - Verification results (Evidence)
  - Risks / rollback
  - Remaining TODO/FIXME with tracking method (link/issue)

### R5) Lessons (self-improvement loop)
- When receiving corrections/feedback from user:
  - Record a short rule (pattern) to avoid repeating the same mistake.
  - Use the `lesson` skill to record in `tasks/lessons.md`.

### R6) Repo access condition
- If `tasks/todo.md`, `tasks/lessons.md` exist and are writable → update files.
- Otherwise → output same format in chat.

## TEMPLATES
### tasks/todo.md (or chat)
- [ ] Goals / non-goals
- [ ] Change scope (files/modules)
- [ ] Step 1 …
- [ ] Step 2 …
- [ ] Verification: which tests/commands/reproductions
- [ ] Risks / rollback

### tasks/todo.md Review section
- Result summary:
- Verification (Evidence):
- Risks / rollback:
- Remaining work (TODO):

### tasks/lessons.md (or chat)
- Lesson:
- Trigger (recurrence condition):
- Prevention Rule (1-line short rule):
- Example (correct/incorrect):
