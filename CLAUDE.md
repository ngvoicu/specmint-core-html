# CLAUDE.md — Spec Mint Core HTML

Guidance for Claude Code (and other AI coding agents) when working on this skill's source.

## Project Overview

Spec Mint Core HTML is a universal skill (no build step, no dependencies) that replaces ephemeral AI coding plans with persistent, resumable specs rendered as rich HTML documents:

- **Mermaid diagrams** — flowchart, sequenceDiagram, erDiagram, stateDiagram-v2, timeline, journey, etc.
- **Syntax-highlighted code diffs** — PrismJS `diff-highlight` for red/green line backgrounds with language-aware highlighting underneath
- **Wireframe and hi-fi UI mockups** — bespoke `.wf-*` / `.ui-*` component libraries; zero CDN dependency for mockups; constrained palette to prevent bikeshedding
- **Derived progress scorecards** — `spec-runtime.js` counts `data-status` attributes at render time; one swap propagates to every progress display (no string drift)

Skill source is markdown. Only the user-facing spec output (`SPEC.html`) is HTML.

Ships as a universal skill (`SKILL.md`) for Claude Code, Codex, Cursor, Windsurf, Cline, and Gemini CLI via `npx skills add ngvoicu/specmint-core-html -a <tool>`.

## Knowledge base

Architectural context across the Mint family (core vs TDD, core-vs-core-html differences, distribution, evals) lives in the **ngvoicu-sme** brain. Read and write through kluris — never edit brain files by hand; the skill enforces an approval protocol:

- `/kluris-ngvoicu-sme` — Claude Code skill (search, learn, remember, create)
- `kluris search "<query>" --brain ngvoicu-sme` — direct CLI search

Topics relevant to this repo: specmint-core-html overview, HTML format rationale, distribution, evals.

## Architecture

The skill has two conceptual layers:

**Skill layer** (this repo) — `SKILL.md` (the universal skill behavior — workflow, lifecycle, invariants), `references/*` (format reference, edit recipes, validator, mockup libraries, and `researcher.md` — the deep-research subagent brief spawned via the Task tool during forge), `assets/*` (shared `spec-styles.css` + `spec-runtime.js` copied into consuming projects). The rendered preview lives at <https://specmint.io/#gallery>. AI tools read these markdown files as behavioral instructions.

**Data layer** (consuming project) — `.specs/` directory created in the consuming project root (not here). Layout:

```
.specs/
├── assets/
│   ├── spec-styles.css    # Shared design system — copied once from the skill's assets/
│   └── spec-runtime.js    # Progress deriver + Mermaid/Prism init + SVG annotation arrows
├── registry.md            # Markdown table — denormalized index across specs
└── <spec-id>/
    ├── SPEC.html          # The spec (rich HTML)
    ├── research-*.md      # Research notes (markdown)
    └── interview-*.md     # Interview notes (markdown)
```

**Source of truth split inside `SPEC.html`** (avoid duplicating state):
- `<script type="application/json" id="spec-meta">` in `<head>` — identity only (`id`, `title`, `status`, `created`, `updated`, `priority`, `tags`, `mockup-fidelity`). Single-line JSON, canonical key order.
- `data-status` attributes on `<li class="task">` / `<details class="phase">` / `<li class="ac-item">` — lifecycle state.
- Progress strings ("3/12", "Phase 2 of 4", "33%") — **never stored**; derived by `spec-runtime.js` at page load.

## File Relationships (must stay in sync)

| Source of truth | Must match |
|----------------|------------|
| `references/spec-format.md` | Spec format rules in `SKILL.md` |
| `assets/spec-styles.css` + `assets/spec-runtime.js` | `references/html-template.html` and `references/edit-recipes.md` |
| `SKILL.md` | Behavioral contracts in `references/command-contracts.md` |
| `references/researcher.md` | Research subagent spawn instructions in `SKILL.md` |

`skills/specmint-core-html/SKILL.md` is a symlink to `../../SKILL.md` — never replace it with a real file.

## Key Conventions

- `.specs/` is intentionally untracked (see `.gitignore`). `CLAUDE.md` and `AGENTS.md` ARE tracked in this repo (mirror of upstream `specmint-core` policy).
- `SKILL.md` must work for all AI tools — keep it tool-agnostic.
- Spec format details (regions, JSON key order, `data-status` values, task code format) are in `references/spec-format.md` — single source of truth.
- Workflow details (forge phases, implement lifecycle) live in `SKILL.md`.
- Pause/resume checkpoints at task boundaries only — there is no Resume Context section. Documented in SKILL.md.

## Working on This Codebase

### Behavioral changes
- Edit `SKILL.md` to change universal skill behavior. Mirror format-related changes into `references/spec-format.md` and `references/edit-recipes.md` in the same PR.
- Edit `references/researcher.md` to change the deep-research subagent brief.
- Edit `references/command-contracts.md` when you change behavioral contracts; this is the review checklist.

### Format changes
- Edit `assets/spec-styles.css` / `assets/spec-runtime.js` to change rendered visual / runtime behavior for every generated `SPEC.html`. To eyeball changes, dogfood the skill in a disposable consumer project — install it (e.g. `npx skills add ./. -g -a claude-code`, or copy `SKILL.md` into its skills dir), trigger the forge workflow with natural language, then open the generated `.specs/<id>/SPEC.html`. The reference render lives at <https://specmint.io/#gallery>.
- After any spec-format change, run the validate recipe on a generated `SPEC.html`:
  ```bash
  python3 -c "
  import re, sys, json
  h = open('.specs/<id>/SPEC.html').read()
  m = re.search(r'<script[^>]*id=\"spec-meta\"[^>]*>(.+?)</script>', h, re.S)
  assert m; json.loads(m.group(1))
  o = re.findall(r'<!--\s*region:(\w+)\s*-->', h); c = re.findall(r'<!--\s*endregion:(\w+)\s*-->', h)
  assert sorted(o) == sorted(c)
  print('OK')
  "
  ```

### Plumbing
- Smoke-test changes: install the skill into a disposable project (e.g. `npx skills add ./. -g -a claude-code`, or copy `SKILL.md` into its skills dir), then exercise natural-language triggers (forge / resume / implement).
- Windsurf users must replace the symlink at `.windsurf/skills/specmint-core-html/SKILL.md` with a real file copy (Cascade doesn't follow symlinks).

## Eval Infrastructure

Real evals live at `evals/evals.json` — 6 scenarios with 33 verifiable expectations covering: forge cold-start, resume from an existing spec, pause-and-switch, Mermaid always-quote rule, code-preview density, no-ASCII mockups. See the file for prompts and per-expectation grading criteria.

To run the full benchmark pipeline (with-skill vs baseline runs, grading, viewer):

```
/skill-creator improve                            # in a fresh session, point at this skill
```

skill-creator spawns parallel test runs, scores each expectation, and produces a benchmark + diff against any previous iteration. Run results land in a sibling `specmint-core-html-workspace/` directory (gitignored).

## Distribution

- **Universal skill**: `npx skills add ngvoicu/specmint-core-html -g` (installs `SKILL.md`; auto-triggers on natural language). Target a specific tool with `-a <claude-code|codex|cursor|windsurf|cline|gemini>`.
- **GitHub**: <https://github.com/ngvoicu/specmint-core-html>
