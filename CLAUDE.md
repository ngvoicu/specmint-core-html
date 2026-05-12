# CLAUDE.md — Spec Mint Core HTML

Guidance for Claude Code (and other AI coding agents) when working on this plugin's source.

## Project Overview

Spec Mint Core HTML is a Claude Code plugin (no build step, no dependencies) that replaces ephemeral AI coding plans with persistent, resumable specs rendered as rich HTML documents:

- **Mermaid diagrams** — flowchart, sequenceDiagram, erDiagram, stateDiagram-v2, timeline, journey, etc.
- **Syntax-highlighted code diffs** — PrismJS `diff-highlight` for red/green line backgrounds with language-aware highlighting underneath
- **Wireframe and hi-fi UI mockups** — bespoke `.wf-*` / `.ui-*` component libraries; zero CDN dependency for mockups; constrained palette to prevent bikeshedding
- **Derived progress scorecards** — `spec-runtime.js` counts `data-status` attributes at render time; one swap propagates to every progress display (no string drift)

Plugin source is markdown + JSON. Only the user-facing spec output (`SPEC.html`) is HTML.

Also ships as a universal skill (`SKILL.md`) for Codex, Cursor, Windsurf, Cline, and Gemini CLI via `npx skills add ngvoicu/specmint-core-html -a <tool>`.

## Knowledge base

Architectural context across the Mint family (core vs TDD, core-vs-core-html differences, distribution, evals) lives in the **ngvoicu-sme** brain. Read and write through kluris — never edit brain files by hand; the skill enforces an approval protocol:

- `/kluris-ngvoicu-sme` — Claude Code skill (search, learn, remember, create)
- `kluris search "<query>" --brain ngvoicu-sme` — direct CLI search

Topics relevant to this repo: specmint-core-html overview, HTML format rationale, distribution, evals.

## Architecture

The plugin has two conceptual layers:

**Plugin layer** (this repo) — `commands/*.md` (one file per slash command), `agents/researcher.md` (Opus-model deep research subagent), `.claude-plugin/` and `.cursor-plugin/` (metadata), `references/*` (format reference, edit recipes, validator, mockup libraries), `assets/*` (shared `spec-styles.css` + `spec-runtime.js` copied into consuming projects, plus the README `preview.png`). Claude Code reads these markdown files as behavioral instructions.

**Data layer** (consuming project) — `.specs/` directory created in the consuming project root (not here). Layout:

```
.specs/
├── assets/
│   ├── spec-styles.css    # Shared design system — copied once from plugin's assets/
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
| `commands/*.md` | Behavioral contracts in `references/command-contracts.md` |

`skills/specmint-core-html/SKILL.md` is a symlink to `../../SKILL.md` — never replace it with a real file.

## Key Conventions

- `.specs/` is intentionally untracked (see `.gitignore`). `CLAUDE.md` and `AGENTS.md` ARE tracked in this repo (mirror of upstream `specmint-core` policy).
- `SKILL.md` must work for all AI tools — the Claude Code Plugin section at the top is tool-specific and kept to ~20 lines.
- Spec format details (regions, JSON key order, `data-status` values, task code format) are in `references/spec-format.md` — single source of truth.
- Workflow details (forge phases, implement lifecycle) are in the respective `commands/*.md` files.
- Pause/resume checkpoints at task boundaries only — there is no Resume Context section. Documented in SKILL.md.

## Working on This Codebase

### Behavioral changes
- Edit `commands/*.md` to change slash command behavior.
- Edit `SKILL.md` to change universal skill behavior. Mirror format-related changes into `references/spec-format.md` and `references/edit-recipes.md` in the same PR.
- Edit `references/command-contracts.md` when you change command contracts; this is the review checklist.

### Format changes
- Edit `assets/spec-styles.css` / `assets/spec-runtime.js` to change rendered visual / runtime behavior for every generated `SPEC.html`. To eyeball changes, dogfood the plugin in a disposable consumer project — `claude plugin add /path/to/specmint-core-html`, run `/forge`, then open the generated `.specs/<id>/SPEC.html`. The README screenshot at `assets/preview.png` is the reference render.
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
- Validate `.claude-plugin/*.json` and `.cursor-plugin/*.json` stay valid JSON after edits (`python3 -c "import json; json.load(open(...))"`).
- Smoke-test changes: `claude plugin add /path/to/specmint-core-html` in a disposable project, then run `/forge`, `/resume`, etc.
- Windsurf users must replace the symlink at `.windsurf/skills/specmint-core-html/SKILL.md` with a real file copy (Cascade doesn't follow symlinks).

## Eval Infrastructure

Eval workspace at `specmint-workspace/` (gitignored). Currently contains a **placeholder scaffold** — `specmint-workspace/evals/evals.json` lists 6 evals (forge, resume, spec-quality, implement, asset-init, researcher-spawn) with TODO assertion bodies. The structure is in place; the assertion implementations are a follow-up.

## Distribution

- **Claude Code plugin**: `claude plugin add ngvoicu/specmint-core-html` (full feature set — all slash commands, researcher agent, auto-triggers).
- **Universal skill**: `npx skills add ngvoicu/specmint-core-html -a <codex|cursor|windsurf|cline|gemini>` (SKILL.md installs; no slash commands or researcher agent).
- **GitHub**: <https://github.com/ngvoicu/specmint-core-html>
