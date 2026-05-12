---
description: Resume the active spec — read progress, load context, pick up where you left off
disable-model-invocation: true
---

# Resume Spec

Read the specmint-core-html skill and follow the "Resuming a Spec" workflow.

1. Read `.specs/registry.md` to find the spec with `active` status
2. If none is active, show the user their specs so they can choose one
3. Load `.specs/<id>/SPEC.html`
4. Parse progress — count `<li class="task">` elements grouped by
   `data-status` per phase
5. Find the current phase (first `<details class="phase">` with
   `data-status="in-progress"`) and the current task (first
   `<li class="task" data-status="pending">` in that phase)
6. Check if there are research notes (research-*.md, interview-*.md) in
   `.specs/<id>/` that provide additional context for the task
7. Present a compact summary and begin working on the current task

If there are no specs at all, suggest running `/specmint-core-html:forge` to
create one.

There is no separate Resume Context section in HTML specs — the first
pending task in the active phase is implicitly the next task. Look at
the most recent Decision Log entry and the task description itself for
context.
