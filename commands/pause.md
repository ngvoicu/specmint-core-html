---
description: Pause the active spec — finalize state at the current task boundary
disable-model-invocation: true
---

# Pause Spec

Read the specmint-core-html skill and follow the "Pausing a Spec" workflow.

1. Read `.specs/registry.md` to find the spec with `active` status
2. If no active spec exists, report that there is nothing to pause and stop
3. Load the SPEC.html
4. Make sure every completed task has `data-status="completed"` and every
   phase transition is reflected. Pause at a clean task boundary — HTML
   specs do not carry mid-task state.
5. Add any session decisions to the Decision Log (`<table class="log-table">`
   under `region:decisions`)
6. Change spec status from `active` to `paused`:
   - In `<script id="spec-meta">`: `"status":"paused"`
   - Visible header pill: `pill--in-progress`/`Active` → `pill--pending`/`Paused`
7. Update the `"updated"` date in the JSON and the visible "Updated" `<dd>`
8. Mirror status + date in `.specs/registry.md`
9. Run the validate recipe (`references/validate.md`)
10. Confirm to the user that progress was saved

**If you are partway through a task and the user wants to pause**, finish
the task or split it into smaller subtasks before pausing — there is no
freeform Resume Context section to capture in-flight state. The Decision
Log can hold a brief in-flight note but is designed for durable decisions,
not running scratchpad.
