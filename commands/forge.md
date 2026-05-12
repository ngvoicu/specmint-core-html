---
description: Research deeply, interview the user, then forge a structured spec with phases and tasks. This is a persistent planning workflow.
disable-model-invocation: true
---

# Forge a Spec

You are about to run the Spec Mint Core HTML forge workflow. This bypasses plan mode
with something far more thorough: deep research → interview → more research
→ more interview → write spec → review.

The forge workflow never produces application code. Its only outputs are
`.specs/` files: research notes, interview notes, the shared `.specs/assets/`
folder (on first run), and the SPEC.html.

The user's request: $ARGUMENTS

## Preflight: Resolve Spec Identity

Before starting research, resolve spec identity:

1. Generate a spec ID from the user's request (lowercase, hyphenated)
2. Collision-check read-only:
   - Check `.specs/<spec-id>/SPEC.html`
   - Check whether `<spec-id>` exists in `.specs/registry.md`
3. If the ID already exists, stop and ask the user to choose one:
   - **Resume** existing spec
   - **Rename** new spec (suggest `<spec-id>-v2`)
   - **Archive** old spec then recreate
4. Use this resolved `<spec-id>` in all later phases.

## Plan Mode Check

Before starting, check if you're in plan mode (read-only).

- If in plan mode:
  - Do not run `/specmint-core-html:forge` in plan mode
  - Ask the user to exit plan mode (Shift+Tab), then rerun `/specmint-core-html:forge`
  - Stop here until plan mode is exited
- If NOT in plan mode:
  - Create/initialize `.specs/<spec-id>/` before the first write
  - If `.specs/assets/` does not exist, create it and copy `spec-styles.css`
    + `spec-runtime.js` from the plugin's `assets/` directory
  - Persist artifacts as each phase completes

## Phase 1: Deep Research

This is the most important phase. Be exhaustive. You are gathering every
piece of context needed to write a spec that won't need revision mid-build.

### 1a. Codebase Research

Scan the project thoroughly. Don't just grep for keywords — understand the
architecture:

- **Project structure**: Map the directory tree, identify patterns (monorepo?
  modules? packages?)
- **Tech stack**: Read package.json/Cargo.toml/go.mod/requirements.txt etc.
  Understand what's already in use
- **Related code**: Find every file, function, component, route, model, and
  test that touches the area the user wants to change
- **Patterns**: How does the existing code handle similar things? If adding
  auth, how is the existing middleware structured? If adding a feature, what
  patterns do similar features follow?
- **Tests**: What testing frameworks are used? What's the test coverage like
  in the relevant area?
- **Config**: Environment variables, build config, CI/CD pipelines that
  might be affected
- **Dependencies**: What libraries are relevant? Are there version
  constraints?

Use Glob, Grep, and Read aggressively. Read actual file contents, not just
file names. Open 10-20 files if needed.

**Always spawn the `specmint-core-html:researcher` agent** (Task tool) to run an
exhaustive parallel research pass. The researcher reads 15-30 files, runs
3+ web searches, compares library candidates, and assesses risks. Save
structured findings to `.specs/<id>/research-01.md`. Don't skip this —
thorough research is the foundation of a spec that won't need revision
mid-build.

### 1b. Context7 & Cross-Skill Research (in parallel with researcher)

While the researcher agent runs, do these yourself — they use MCP tools
that the researcher agent doesn't have access to:

- **Context7**: If available (resolve-library-id / query-docs tools), pull
  up-to-date documentation for 2-5 key libraries involved. Check API changes,
  deprecated features, and recommended patterns for the specific versions.
- **Cross-skill loading**: Load other available skills when relevant:
  - **frontend-design**: For UI-heavy features — creative, professional design
  - **datasmith-pg**: For database work — schema design, migrations, indexing
  - **webapp-testing**: For testing strategy — Playwright patterns
  - **vercel-react-best-practices**: For Next.js/React optimization
  - Any other relevant skill that's available

### 1c. UI Research (if applicable)

If the project has a UI and the changes affect it:

- Take screenshots of current state if browser tools are available
- Map the component hierarchy
- Understand the routing and state management
- Research modern UI patterns for the specific use case
- Look at design references for creative, professional approaches
- Note accessibility requirements (WCAG compliance)

### 1d. Merge & Save Research

When the researcher agent completes, read its output. Merge your Context7
and cross-skill findings into the research notes. Write the combined
findings to:
```
.specs/<spec-id>/research-01.md
```

Structure it clearly (markdown):

```markdown
# Research Notes — <Title>
## Date: <today>

## Project Architecture
<what you found about the structure>

## Relevant Code
<key files, functions, patterns found>

## Tech Stack & Dependencies
<what's in use, versions>

## Library Comparison
<comparison tables for any libraries evaluated, with recommended picks>

## External Research
<web findings, library docs, best practices, Context7 findings>

## UI Research (if applicable)
<screenshots, component map, design references, accessibility notes>

## Risk Assessment
<what could go wrong, security considerations, performance implications>

## Open Questions
<things you couldn't determine from research alone>
```

## Phase 2: Interview Round 1

Now present your findings and ask targeted questions. The goal is NOT to ask
generic questions — your research should inform very specific questions.

**Structure the interview like this:**

1. **Summarize what you found** (2-3 paragraphs, not a wall of text)
2. **State your assumptions** — "Based on the codebase, I'm assuming we'll
   use X pattern because that's what similar features use. Correct?"
3. **Ask specific questions** that your research couldn't answer:
   - Architecture decisions: "Should this be a new module or extend the
     existing one in src/features/?"
   - Scope boundaries: "Should this handle X edge case or is that a
     separate spec?"
   - Technical choices: "I see you're using Library A for similar things.
     Should we stick with that or is there a reason to try Library B?"
   - User-facing behavior: "What should happen when X fails?"
   - Acceptance criteria: "What does 'done' look like? Any specific
     conditions that must be true when this is complete?"
   - **Mockup fidelity (only if UI work is in scope)**: "Mockups in this
     spec should render as `wireframe` (clean grayscale boxes, structural,
     no design commitment), `hi-fi` (real-looking polished components), or
     `none` (prose + diagrams are enough)?" Record the answer — it goes
     into the spec's `mockup-fidelity` metadata field.
4. **Propose a rough approach** and ask for reactions — don't wait for the
   user to design everything

Keep it to 3-6 questions max per round. More than that overwhelms.

**STOP after asking your questions and wait for the user to answer.** Do not
answer your own questions, guess answers, or proceed to deeper research or
spec writing until the user responds. The interview is a real conversation —
the user's answers determine what gets built.

**Save the interview** (markdown):
```
.specs/<spec-id>/interview-01.md
```

```markdown
# Interview Round 1 — <Title>
## Date: <today>

## Questions Asked
1. <question>
   **Answer**: <user's response>

2. <question>
   **Answer**: <user's response>

## Key Decisions
- <decision made during this interview>

## New Research Needed
- <things to look into based on answers>
```

## Phase 3: Deeper Research (informed by interview)

Based on the user's answers, do another round of research:

- Explore the specific code paths they mentioned
- Look up the libraries or patterns they chose
- Check feasibility of the approach discussed
- Find potential issues with the chosen direction

Save to:
```
.specs/<spec-id>/research-02.md
```

## Phase 4: Interview Round 2+

Present your deeper findings. Ask about:

- Trade-offs you discovered
- Edge cases that emerged from the deeper research
- Implementation sequence — "I'd suggest building X first because Y depends
  on it. Does that sequence make sense?"
- Scope refinement — "This feels like it could be split into two specs.
  Want to keep it together or separate?"

Save each round to `interview-02.md`, `interview-03.md`, etc.

**Repeat phases 3-4 as many times as needed.** The loop ends when:

- You have enough clarity to write a spec with no ambiguous tasks
- The user says they're satisfied with the direction
- Every task in the spec can be described concretely (not "figure out X")

It's fine if this takes 2 rounds or 5 rounds. Don't rush it.

## Setup (before writing)

Before writing the spec, ensure the directory structure exists:

1. Reuse the already-resolved `<spec-id>` from Preflight.
2. Create the spec directory:
   ```
   mkdir -p .specs/<spec-id>
   ```
3. If `.specs/` doesn't exist yet, also create `registry.md` and the
   `.specs/assets/` directory (with `spec-styles.css` + `spec-runtime.js`
   copied from the plugin's `assets/`).

If directory creation fails because the environment is still read-only, ask
the user to exit plan mode (Shift+Tab) and rerun `/specmint-core-html:forge`.

## Phase 5: Write the Spec

Now synthesize everything — all research notes, all interview answers, all
decisions — into a `SPEC.html`.

**Start from the canonical template:** copy `references/html-template.html`
to `.specs/<spec-id>/SPEC.html` and fill in every placeholder. Then add
section content as described below. Use `references/edit-recipes.md` for
the exact HTML structure of each common element (tasks, AC items, diagrams,
code-diff figures, mockups, log rows). Use `references/wireframe-library.md`
and `references/mockup-library.md` for mockup patterns.

The spec must include:

1. **Metadata JSON** in `<script type="application/json" id="spec-meta">`:
   `id`, `title`, `status` (`active`), `created`, `updated`, `priority`,
   `tags`, `mockup-fidelity` (from the interview answer). Single line,
   sorted keys.
2. **Spec header card** — title, status pill (Active), priority chip,
   created / updated / tags / mockup-fidelity dl, scorecard with four cells.
3. **Overview**: 2-4 sentences capturing the goal and scope. Someone reading
   just this section should understand what's being built and why.
4. **Acceptance Criteria**: Each criterion is `<li class="ac-item" data-ac="N"
   data-status="pending">`. Must be specific and verifiable. Use
   `<span class="ac-flag">Needs clarification</span>` for unresolved questions.
5. **Architecture Diagram(s)**: Mermaid diagrams (`<pre class="mermaid">`).
   Every non-trivial spec should have at least one. Pick the right diagram
   type per case: `flowchart` for system flows, `sequenceDiagram` for
   request lifecycles, `erDiagram` for data models, `stateDiagram-v2` for
   state machines, `timeline`/`gantt` for milestones. Mermaid is the only
   recommended path — no ASCII required.
6. **Library Choices**: `<table class="table">` comparing evaluated
   libraries. Format: `Need | Library | Version | Alternatives | Rationale`.
7. **Phases & Tasks**: Major milestones (3-6 typical) in a `<div class="phases">`.
   Each phase is `<details class="phase" open data-phase="N" data-status="...">`.
   Tasks within are `<li class="task" data-task="CODE" data-status="pending">`.
   Task codes use `<PREFIX>-<NN>` continuous numbering across all phases.
   Tasks must be concrete (file paths, function names, expected behavior).
8. **Code Previews** (optional): Use `<figure class="code-diff">` blocks
   with PrismJS `language-diff-LANG diff-highlight` classes for illustrative
   changes. Unified by default; side-by-side via `data-view="split"` for
   changes >30 lines or spanning multiple files / non-contiguous hunks.
9. **UI Mockups**: One or more `<figure class="mockup">` blocks, using
   `mockup--wireframe` or `mockup--hifi` based on the `mockup-fidelity`
   decided in the interview. Compose with `.wf-*` or `.ui-*` components
   from the corresponding library reference. Omit entirely if
   `mockup-fidelity` is `none`.
10. **Decision Log**: `<table class="log-table">`. Add a row for every
    non-obvious decision from the interviews.
11. **Deviations**: Empty table. Filled during implementation.

(Resume Context is NOT a section in HTML specs. Pause/resume checkpoints
at task boundaries only — see SKILL.md.)

**Coherence and logic review (mandatory before presenting):**

Before presenting the spec to the user, review it for coherence and logic:

1. Read through the entire spec as a whole — does it tell a coherent story?
2. Check that phases are in logical dependency order — no phase requires
   work from a later phase
3. Verify every task is concrete and actionable (file paths, function names)
4. Confirm the architecture diagram(s) match the task descriptions
5. Verify library choices are consistent throughout (no conflicting picks)
6. Ensure the overview accurately summarizes what the phases will deliver
7. Look for gaps — is there anything the implementation would need that
   isn't covered by a task?
8. Verify acceptance criteria are specific, testable, and cover the key
   behaviors the user expects
9. **Placeholder check**: Search for "TBD", "TODO", "placeholder", "TBC",
   "to be determined", "will be decided", "figure out" — replace every
   instance with a concrete decision or remove the section
10. **Internal consistency**: All task code references valid, library versions
    in different sections don't conflict
11. **Scope check**: Does the spec deliver what was discussed in interviews?
    Nothing more, nothing less.
12. **Ambiguity check**: For each task, ask "could an implementer complete
    this without asking me a question?" If no, add detail until yes.
13. **Run the validate recipe** (`references/validate.md`) — confirm the
    file parses cleanly. Re-run after every subsequent edit.

**Quality check before presenting:**

- Every task should be concrete ("Add verifyToken() to src/auth/tokens.ts"),
  not vague ("implement token verification")
- Phases should have clear boundaries and dependencies
- The Decision Log should capture every non-obvious choice
- The Overview should be understandable without reading the interviews
- Diagrams should be clear and accurate
- UI mockups (if present) should match the chosen fidelity consistently
- Library choices should be the best available, modern, well-maintained

Save to:
```
.specs/<spec-id>/SPEC.html
```

Update `.specs/registry.md` (set status to `active`). Run the validate
recipe one final time.

**Present the spec to the user and wait for approval.** Walk through the
phases and ask: "Does this look right? Want to adjust anything before we
start?" Do not begin implementing until the user explicitly says to proceed.
The spec review is a gate — the user may want to add tasks, reorder phases,
change scope, or rename things. Respect this pause.

After user approval, implementation is handled by `/specmint-core-html:implement`.
Do not implement application code inside `/specmint-core-html:forge`.
