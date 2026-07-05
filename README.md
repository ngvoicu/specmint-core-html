# Spec Mint Core HTML

[![Benchmark +39%](https://img.shields.io/badge/benchmark-%2B39%25-brightgreen)](https://github.com/ngvoicu/specmint-core-html#evaluation-results)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Version](https://img.shields.io/badge/version-3.0.0-7c5cff)](https://github.com/ngvoicu/specmint-core-html)

**Plan mode, but actually good.**

Spec Mint Core HTML replaces ephemeral AI coding plans with persistent, resumable specs built through deep research and iterative interviews. Create a spec, work through it task by task, pause, switch to another spec, come back a week later and pick up exactly where you left off.

Works with Claude Code, Codex, Cursor, Windsurf, Cline, Gemini CLI, and any AI coding tool that can read files — installed as a universal skill.

**[→ See a rendered SPEC.html](https://specmint.ngvoicu.dev/#gallery)** on specmint.ngvoicu.dev — Team Invites exemplar with Mermaid diagrams, code diffs, and hi-fi UI mockups.

## The Problem

Every AI coding tool has some version of "plan mode" — think before you code. But these plans are ephemeral. They live in the conversation context. Close the terminal, start a new session, and the plan is gone. There's no way to:

- **Resume** a plan you were halfway through implementing
- **Switch** between multiple plans when juggling features
- **Track** which tasks are done and which are next
- **Persist** the research and decisions that informed the plan

Spec Mint Core HTML fixes all of this.

## How It Works

### The Forge Workflow

Say something like "forge a spec for adding user authentication with OAuth" and Spec Mint Core HTML takes over:

**1. Deep Research** — Exhaustive codebase scan (reads 10-20+ actual files, not just file names), web search for best practices, Context7 library docs, library comparisons, cross-skill research (frontend-design, datasmith-pg, etc.), UI inspection if applicable. Everything saved to `.specs/<id>/research-01.md`.

**2. Interview** — Presents findings, states assumptions, asks targeted questions informed by the research. Not generic questions — specific ones like "I see you're using Express middleware pattern X in `src/middleware/`. Should the auth middleware follow the same pattern?" Saves answers to `interview-01.md`.

**3. Deeper Research** — Investigates the specific directions from the interview. Checks feasibility, finds edge cases.

**4. More Interviews** — As many rounds as needed until every task in the spec can be described concretely. No ambiguous "figure out X" tasks.

**5. Write Spec** — Synthesizes all research and interviews into a comprehensive `SPEC.html` with Mermaid architecture diagrams, library comparison tables, phases, tasks, syntax-highlighted code diffs, wireframe or hi-fi UI mockups, a decision log, and a deviations table. Runs a coherence and logic review before presenting.

**6. Implement** — Works through the spec task by task during implementation, checking them off, updating progress, logging new decisions, writing tests as specified in the testing strategy.

### Specs Are Files

Specs live in `.specs/` at your project root. Each spec is a single `SPEC.html` file — rich rendering (Mermaid diagrams, syntax-highlighted code diffs, wireframe and hi-fi mockups, derived progress scorecards) backed by a strict template that AI tools can edit surgically. The `registry.md` index and the research/interview notes stay as markdown.

```
.specs/
├── assets/
│   ├── spec-styles.css             # Shared design system (written once)
│   └── spec-runtime.js             # Progress deriver + Mermaid/Prism init
├── registry.md                     # Denormalized index — markdown table
└── user-auth-system/
    ├── SPEC.html                   # The spec (rich HTML)
    ├── research-01.md              # Initial codebase + web research
    ├── interview-01.md             # First interview round
    ├── research-02.md              # Follow-up research
    └── interview-02.md             # Second interview round
```

**The `<script id="spec-meta">` JSON inside `SPEC.html` is authoritative for identity. `data-status` attributes on tasks/phases/AC items carry lifecycle state. Progress strings ("3/12", "33%") are derived at render time — never stored.** `.specs/registry.md` is a denormalized index for quick lookups across specs.

For this `specmint-core-html` repository, `.specs/` is intentionally gitignored for
local dogfooding. In consumer projects, you can choose to commit `.specs/`.

### A SPEC.html Looks Like This (Sketch)

The screenshot above is a real `SPEC.html` rendered in a browser. Below is the structure at a glance:

- **Header card**: title, status pill, priority chip, created/updated dates, tags, scorecard (Tasks / Phases / Acceptance / Blockers)
- **Acceptance Criteria**: checklist with custom-styled checkboxes, amber `Needs clarification` chips
- **Architecture**: one or more `<pre class="mermaid">` blocks rendered as diagrams (flowcharts, ER, sequence, state)
- **Library Choices**: clean table with version badges and rationale cells
- **Phases & Tasks**: each phase a collapsible `<details>` with status border, progress strip, and a list of `<li class="task" data-task="AUTH-03" data-status="completed">` rows
- **Code Previews**: `<figure class="code-diff">` blocks with red/green syntax-highlighted diffs (PrismJS `diff-highlight`)
- **UI Mockups**: wireframe primitives (`.wf-*`) or hi-fi components (`.ui-*`) inside a browser-chrome frame; annotation callouts with SVG arrows
- **Decision Log** + **Deviations**: styled tables

The skill ships a canonical empty template at `references/html-template.html`, before/after edit recipes for every common operation at `references/edit-recipes.md`, and pre-built mockup snippet libraries (`wireframe-library.md` + `mockup-library.md`).

## Installation

Spec Mint Core HTML installs as a **universal skill** — one command, and it works in Claude Code, Codex, Cursor, Windsurf, Cline, Gemini CLI, and any AI coding tool that reads files.

```bash
# Install globally (recommended)
npx skills add ngvoicu/specmint-core-html -g

# Or target a specific tool with -a
npx skills add ngvoicu/specmint-core-html -g -a claude-code
npx skills add ngvoicu/specmint-core-html -g -a codex
npx skills add ngvoicu/specmint-core-html -g -a cursor
npx skills add ngvoicu/specmint-core-html -g -a windsurf
npx skills add ngvoicu/specmint-core-html -g -a cline
npx skills add ngvoicu/specmint-core-html -g -a gemini
```

Once installed, the skill auto-triggers on natural language — "forge a spec for X", "what was I working on?", "implement the spec". No slash commands and no marketplace: the skill bundles the full spec workflow, the deep-research subagent brief (`references/researcher.md`), and the spec format reference.

> **Windsurf**: replace the symlink at `.windsurf/skills/specmint-core-html/SKILL.md` with a real file copy — Cascade doesn't follow symlinks.

## Usage

Talk to your AI tool in plain language — the skill recognizes the spec lifecycle and runs the right workflow:

| Goal | Say something like |
|------|--------------------|
| Start a new spec | "forge a spec for user authentication" |
| Implement it | "implement the spec" · "implement phase 2" · "implement all phases" |
| Resume where you left off | "what was I working on?" · "resume" |
| Pause and save context | "pause the spec" |
| Switch active spec | "switch to the api-refactor spec" |
| List all specs | "list my specs" |
| Show progress | "spec status" |
| Generate API docs | "generate openapi" |

Every action reads and writes plain files under `.specs/` — research notes, interviews, `SPEC.html`, and a `registry.md` index. Any tool that can read files shares the same `.specs/` directory, so you can forge in one tool and implement in another.

## The Forge Workflow (Detailed)

### Phase 1: Deep Research

Not a quick scan. The researcher reads 10-20+ files, following dependency chains, checking tests, examining config. Uses every available resource: web searches for best practices, Context7 for library docs, library comparisons, cross-skill research (frontend-design, datasmith-pg, etc.).

Output saved to `.specs/<id>/research-01.md`. Covers:
- Project architecture and directory structure
- Every file touching the area of change
- Tech stack versions (from lock files, not guesses)
- How similar features are currently implemented
- Library comparisons (2-3+ candidates per choice point)
- Test patterns and coverage
- Risk assessment
- UI/UX research and design references (if applicable)

### Phase 2-4: Interviews

Targeted questions based on what research found. Not generic "what do you want?" — specific questions like:

- "I see rate limiting middleware at `src/middleware/rateLimit.ts`. Should auth endpoints use the same limiter or a stricter one?"
- "The User model uses Prisma. Should OAuth tokens go in the same schema or a separate `AuthToken` model?"

Multiple rounds (typically 2-5) until every task can be described concretely. Each round saved to `interview-01.md`, `interview-02.md`, etc.

### Phase 5: Write Spec

Synthesizes everything into a comprehensive `SPEC.html`:
- Mermaid architecture diagrams (flowchart, sequence, ER, state, timeline)
- Library comparison table with alternatives and rationale
- 3-6 phases, each with concrete tasks (file paths, function names)
- Optional code-diff previews for illustrative changes (PrismJS syntax-highlighted)
- UI mockups in the chosen fidelity (wireframe with annotation arrows, or hi-fi with real-looking components)
- Decision log captures non-obvious technical choices
- Deviations table (empty at forge time, filled during implementation)
- Mandatory coherence and logic review + `validate.md` recipe before presenting

### Phase 6: Implement

Works through the spec task by task during implementation:
- Implements the task
- Swaps `data-status="pending"` → `data-status="completed"` on the task element
- When a phase finishes, transitions the phase and the next phase's status + pill
- Runs the validate recipe after every edit
- Updates registry progress/date
- Writes tests as specified in the spec
- Logs new decisions to the Decision Log
- Logs deviations when implementation diverges from spec

## Plan Mode

Spec Mint Core HTML **replaces** Claude Code's built-in plan mode. The forge workflow IS your planning phase — deep research, interviews, spec writing. You don't need plan mode at all.

If you happen to be in plan mode when you start the forge workflow, Spec Mint Core HTML
asks you to exit plan mode first (Shift+Tab), then start the forge workflow again.

## Project Structure

```
specmint-core-html/
├── SKILL.md                        # Universal skill (works with all tools)
├── skills/
│   └── specmint-core-html/
│       └── SKILL.md                # → ../../SKILL.md (symlink for skills-CLI discovery)
├── references/
│   ├── researcher.md               # Deep-research subagent brief
│   ├── spec-format.md              # SPEC.html format reference
│   ├── html-template.html          # Canonical empty SPEC.html template
│   ├── edit-recipes.md             # Before/after snippets for every surgical edit
│   ├── validate.md                 # Post-edit validation recipe (Python one-liner)
│   ├── wireframe-library.md        # Wireframe mockup patterns (.wf-* primitives)
│   ├── mockup-library.md           # Hi-fi mockup patterns (.ui-* components)
│   └── command-contracts.md        # Behavioral contract checklist for the skill
├── assets/
│   ├── spec-styles.css             # Shared design system — copied to .specs/assets/ on every forge
│   └── spec-runtime.js             # Progress deriver + Mermaid/Prism init + diagram modal + validator
├── README.md
└── LICENSE
```

## Spec Format

Full specification in [`references/spec-format.md`](references/spec-format.md).
Edit recipes for every common operation in [`references/edit-recipes.md`](references/edit-recipes.md).
Validation recipe in [`references/validate.md`](references/validate.md).
Behavioral guardrails in [`references/command-contracts.md`](references/command-contracts.md).

### Metadata (single-line JSON in `<script id="spec-meta">`)

| Field | Required | Description |
|-------|:---:|-------------|
| `id` | Yes | URL-safe slug (e.g., `user-auth-system`) |
| `title` | Yes | Human-readable name |
| `status` | Yes | `active`, `paused`, `completed`, `archived` |
| `created` | Yes | ISO date (YYYY-MM-DD) |
| `updated` | Yes | ISO date of last modification |
| `priority` | No | `high`, `medium`, `low` (default: medium) |
| `tags` | No | JSON array |
| `mockup-fidelity` | No | `wireframe`, `hi-fi`, `none` |

### Conventions

- **Phase status** (`data-status` on `<details class="phase">`): `pending`, `in-progress`, `completed`, `blocked`
- **Task status** (`data-status` on `<li class="task">`): same values
- **Task codes**: `[PREFIX-NN]` — unique per task, auto-incrementing across all phases
- **Region sentinels**: `<!-- region:NAME -->` / `<!-- endregion:NAME -->` around every top-level section — used as anchors for surgical edits
- **No current marker**: the first task with `data-status="pending"` in the active phase is implicitly current
- **Uncertainty**: `<span class="ac-flag">Needs clarification</span>` inline in an acceptance criterion
- **Architecture Diagrams**: Mermaid (`flowchart`, `sequenceDiagram`, `erDiagram`, `stateDiagram-v2`, etc.) inside `<pre class="mermaid">` blocks
- **Library Choices**: Comparison table with alternatives considered and rationale
- **Code Previews**: `<figure class="code-diff">` with PrismJS `diff-highlight` for syntax-highlighted red/green diffs
- **UI Mockups**: `mockup--wireframe` (grayscale `.wf-*` primitives) or `mockup--hifi` (real-looking `.ui-*` components) — both bespoke, zero CDN, constrained palette to prevent bikeshedding
- **Decision Log**: Table with date, decision, rationale
- **Deviations**: Table tracking where implementation diverged from spec
- **Progress strings**: Never authored. `spec-runtime.js` derives them from `data-status` counts at page load.

## Evaluation Results

Spec Mint Core HTML has been iteratively developed and evaluated using Anthropic's
[Skill Creator](https://github.com/anthropics/claude-plugins-official/blob/main/plugins/skill-creator/skills/skill-creator/SKILL.md)
— the official tool for building, testing, and benchmarking Claude Code skills.

Each iteration was validated through parallel eval runs (with-skill vs
without-skill baselines), automated assertion grading, and quantitative
benchmarking across multiple test scenarios — forge workflow fidelity,
interview gating, research depth, research subagent spawning, spec quality,
and implementation tracking.

**Latest benchmark (iteration 5):**

| Config | Pass Rate |
|--------|-----------|
| With Skill | **100%** (18/18 assertions) |
| Without Skill | 61% (11/18 assertions) |
| Delta | **+39%** |

For more on how Skill Creator works — evals, A/B comparisons, benchmarking,
and the iteration loop — see
[Improving skill-creator: Test, measure, and refine Agent Skills](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills).

## Why Not Just Use Plan Mode?

Plan mode is a good idea with a bad implementation. It restricts Claude to read-only tools and asks for a plan. That's it. No persistence, no research depth, no interviews, no progress tracking.

Spec Mint Core HTML's forge workflow does what plan mode should do:

- **Research depth**: Reads 10-20+ files, searches the web, pulls library docs. Not a quick scan.
- **Interviews**: Asks you targeted questions based on what it found. Multiple rounds until there's no ambiguity.
- **Persistence**: Everything is saved to files. Research notes, interviews, the spec itself. Nothing lives only in context.
- **Resumability**: Close the terminal, come back next week. The spec remembers exactly where you were.
- **Multi-spec**: Juggle multiple features. Switch between them with one command.

## Pair with Kluris

Spec Mint Core HTML reads your codebase. [Kluris](https://kluris.ngvoicu.dev) gives your agents the *other* half — the tribal knowledge that never made it into comments: architecture decisions, past incidents, vendor quirks, the "why" behind every weird choice.

Pair them and the forge workflow's Phase 1b (research) stops guessing. It consults the brain first.

**Inside your AI coding agent:**

```text
> write a spec for OAuth sign-in with GitHub — check the brain, the codebase, and the web

research · phase 1a · reading codebase ........ done
research · phase 1b · kluris wake-up ........... done
research · phase 1c · synthesis ............... done

> spec written to .specs/oauth-github.md — ready for interview.
```

The spec lands grounded in both the code *and* the knowledge your team already agreed to — no re-litigating decisions made six months ago.

**Why it works:**

- **Grounded research** — Phase 1b pulls from a curated brain instead of just the web.
- **Institutional memory** — new hires (and agents) inherit context instantly.
- **Spec reuse** — past specs and decisions surface automatically during research.

**Install Kluris:**

```bash
pipx install kluris
kluris wake-up
```

Full setup at [kluris.ngvoicu.dev](https://kluris.ngvoicu.dev).

## License

MIT

---

<!-- ngvoicu author section — identical across all ngvoicu repos, keep in sync -->
## AI-native toolkit

This project is part of a larger AI-native toolkit — and of a way of working your whole team can adopt: talks (["Becoming an AI Native Company"](https://ngvoicu.dev/becoming-an-ai-native-company/)), hands-on team training that teaches employees to use AI, and [AI adoption consulting for engineering teams](https://ngvoicu.dev/#consulting).

- Site: [ngvoicu.dev](https://ngvoicu.dev)
- Contact: [office@ngvoicu.dev](mailto:office@ngvoicu.dev) · +40 734 704 910

Toolkit: [Specmint](https://specmint.ngvoicu.dev) (durable AI coding specs) · [Kluris](https://kluris.ngvoicu.dev) (team knowledge brains) · [ConsensFlow](https://consensflow.ngvoicu.dev) (cross-agent second opinions)
