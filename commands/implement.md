---
description: Implement the active spec — work through tasks, mark them complete, update progress
disable-model-invocation: true
---

# Implement Spec

Implement tasks from the active spec. The argument can specify scope.

User's request: $ARGUMENTS

## Scope Detection

Parse the user's request to determine what to implement:

- **No argument / "the spec" / "implement"** → Start from the first task
  with `data-status="pending"` in the in-progress phase and work forward
- **"phase N" / "phase <name>"** → Implement all pending tasks in that
  specific phase only
- **"all phases" / "everything"** → Implement all remaining pending tasks
  across all phases, in order
- **"task CODE-NN"** → Implement just that specific task

## Implementation Workflow

1. Read `.specs/registry.md` to find the spec with `active` status
2. If none is active, show the user their specs so they can choose one
3. Load `.specs/<id>/SPEC.html`
4. Parse all phases and tasks — look at `data-status` on each
   `<li class="task">` and `<details class="phase">`. Identify which tasks
   are in scope.
5. Present a brief plan: "I'll implement N tasks across M phases. Starting
   with [TASK-CODE] — <task description>."
6. **TUI Progress**: Create a TaskCreate entry for each in-scope task so
   they appear as live checkboxes in the Claude Code TUI:
   - subject: `[TASK-CODE] <task description>`
   - activeForm: `Implementing [TASK-CODE]`
   The `data-status` attributes in `SPEC.html` remain the source of truth —
   TUI entries are a convenience view for real-time progress.

For each task in scope, in order:

### Execute

1. Set the task's TUI entry to `in_progress` via TaskUpdate
2. **Implement the task** — write the actual application code:
   - Follow the patterns and conventions identified in the research notes
   - Use the libraries and approaches specified in the spec
   - Write clean, maintainable, professional code
   - If the task has specific file paths or function names, use them
3. **Write tests** if the task has a corresponding test task or if the spec's
   testing strategy calls for it
4. **Run tests** to verify correctness after implementation

### Update Progress (sacred — never skip)

Progress tracking is the most important bookkeeping in specmint-core-html.
If you skip this, resume breaks, the registry lies, and the spec becomes
useless. After completing each task, immediately update the spec file:

1. In `SPEC.html`, change the task's `data-status="pending"` →
   `data-status="completed"`. See `references/edit-recipes.md` for the
   exact swap.
2. If all tasks in the current phase are now completed:
   - Phase `data-status="in-progress"` → `data-status="completed"`
   - Update the phase's pill class (`pill--in-progress` → `pill--completed`)
     and its label text
   - Next phase (if any): `data-status="pending"` → `data-status="in-progress"`
     and update its pill correspondingly
   - Review Acceptance Criteria — update `data-status="completed"` on any
     that are now satisfied
3. Update the `"updated"` field in `<script id="spec-meta">` JSON
4. Update the visible "Updated" `<dd>` in the spec header dl
5. **Run the validate recipe** (`references/validate.md`). If it fails,
   fix the broken JSON or sentinel pair before moving on.
6. Update progress and `updated` date in `.specs/registry.md` (count
   `data-status="completed"` task elements for the X/Y)
7. **Verify**: Re-read both `SPEC.html` and `registry.md` to confirm edits
   landed. If registry progress doesn't match the completed-task count,
   fix it before moving on.
8. Set the task's TUI entry to `completed` via TaskUpdate

If you realize you forgot to update after a previous task, stop and fix
it now before continuing with the next task.

### Phase Review (after completing a phase)

When all tasks in a phase are completed, review before moving to the next:

1. Dispatch the `superpowers:code-reviewer` agent (if available) with:
   - The phase requirements from the SPEC.html
   - The list of files created/modified during this phase
   - The acceptance criteria relevant to this phase
2. If no reviewer agent is available, do an inline review:
   - Re-read the phase's tasks and acceptance criteria
   - Verify each task's implementation matches what was specified
   - Check for missing edge cases, incomplete implementations, or spec drift
3. If the review finds issues, fix them before marking the phase complete
4. Log any review findings in the Decision Log

### Handle Issues

- If a task is more complex than expected, split it into subtasks and update
  the SPEC.html before continuing
- If implementation diverges from the spec (better approach found, errors in
  spec, etc.), log it in the **Deviations** section
- Log any new technical decisions in the **Decision Log**
- If blocked on a task:
  - Set the task's `data-status="blocked"`
  - Add a Decision Log entry noting the blocker
  - Set the phase `data-status="blocked"` only when the whole phase is blocked
  - Move to the next unblocked task if possible

## Parallel Task Execution (optional)

When multiple tasks within a phase are independent (no shared files, no
sequential dependencies), you may dispatch them in parallel using the
Agent tool:

1. Identify which tasks have no file-level or logical dependencies on
   each other
2. Dispatch an Agent for each independent task with:
   - The full task specification from the SPEC.html
   - The research notes and library choices for context
   - Clear instructions on which files to create/modify
3. After all agents complete, integrate results and run tests
4. Update all `data-status` attributes, the registry, and TUI entries in
   a single batch; run validate once

Default to sequential execution. Only parallelize when tasks are clearly
independent and the speedup is worth the coordination overhead. When in
doubt, execute sequentially.

## Verification Gate (mandatory before claiming completion)

Before reporting any phase or spec as complete, provide evidence:

1. Run the project's test suite (or the relevant subset) via Bash
2. Show the actual command and output in your response
3. If tests fail, fix the issues before claiming completion
4. Never use language like "should pass", "probably works", or "seems correct"

This applies at every completion boundary: task, phase, and spec. Evidence
first, then assertions. No exceptions.

## Completion

When all in-scope tasks are done:

- If all tasks in the spec are complete:
  - **Run the full test suite** and show the output (verification gate)
  - Verify all Acceptance Criteria have `data-status="completed"`. If any
    remain pending, report which ones and ask the user before marking
    the spec complete.
  - Set every phase `data-status="completed"` (and update their pills)
  - Set spec status to `completed` in JSON metadata + visible header pill
  - Update `.specs/registry.md` with `completed` status
  - Run the validate recipe
  - Present a summary: tasks completed, files created/modified, test output
  - Suggest next spec to activate if any are paused
- If only a phase was completed:
  - **Run tests** for the phase's scope and show the output (verification gate)
  - Report phase completion and remaining work
  - Set the next phase to `data-status="in-progress"` if applicable

## Quality Standards

During implementation:
- Write clean, simple, maintainable code — no over-engineering
- Follow existing codebase patterns and conventions
- Use the libraries specified in the spec's Library Choices section
- Write comprehensive tests as specified in the spec
- Handle edge cases identified in the spec
- Validate at system boundaries, trust internal code

## Pause Limitation

HTML specs checkpoint state at task boundaries. If you must pause
mid-task, finish or split the task first — there is no Resume Context
section to capture in-flight state.
