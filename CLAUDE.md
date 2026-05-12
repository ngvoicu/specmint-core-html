# CLAUDE.md — Spec Mint Core HTML

## Project Overview

Spec Mint Core HTML is a Claude Code plugin (no build step, no dependencies) that replaces ephemeral AI coding plans with persistent, resumable specs rendered as rich HTML documents (Mermaid diagrams, syntax-highlighted code diffs, wireframe and hi-fi UI mockups, derived progress scorecards). Plugin source is markdown + JSON; only the user-facing spec output is HTML. Also ships as a universal skill (`SKILL.md`) that works with Codex, Cursor, Windsurf, Cline, and Gemini CLI via `npx skills add`.

## Knowledge base

Architectural details and distribution context for the Mint family live in the **ngvoicu-sme** brain. Read and write through kluris (never edit brain files by hand — the skill enforces an approval protocol):

- `/kluris-ngvoicu-sme` — Claude Code skill (search, learn, remember, create)
- `kluris search "<query>" --brain ngvoicu-sme` — direct search

Topics relevant to this repo: specmint-core-html overview, architecture, core-vs-tdd differences, distribution, evals.

## Architecture

The plugin has two conceptual layers:

**Plugin layer** — `commands/*.md` (one file per slash command), `agents/researcher.md` (Opus-model deep research subagent), `.claude-plugin/` (metadata). Claude Code reads these markdown files as behavioral instructions.

**Data layer** — `.specs/` directory created in the *consuming* project root (not this repo). Contains a shared `.specs/assets/` (`spec-styles.css` + `spec-runtime.js`, copied once from `examples/`), the `registry.md` index (markdown), and one `<spec-id>/` folder per spec with `SPEC.html` + markdown research/interview notes. The `<script id="spec-meta">` JSON inside `SPEC.html` is authoritative for identity; `data-status` attributes are authoritative for lifecycle; `registry.md` is a denormalized index. See `references/spec-format.md` for the full format specification.

### File Relationships

These files must stay in sync — changing one without the other will cause behavioral drift:

| Source of truth | Must match |
|----------------|------------|
| `references/spec-format.md` | Spec format rules in `SKILL.md` |
| `examples/SPEC.html` + `spec-styles.css` + `spec-runtime.js` | Reference `html-template.html` and `edit-recipes.md` |
| `commands/*.md` | Behavioral contracts in `references/command-contracts.md` |

`skills/specmint-core-html/SKILL.md` is a symlink to `../../SKILL.md` — never replace it with a real file.

## Key Conventions

- `CLAUDE.md`, `AGENTS.md`, and `.specs/` are intentionally untracked in this repo
- `AGENTS.md` provides Codex-specific guidelines (see `SKILL.md` for details)
- `SKILL.md` must work for all AI tools — the Claude Code Plugin section at the top is tool-specific and kept to ~20 lines
- Spec format details (IDs, task codes, phase markers, sections) are in `references/spec-format.md` — that is the single source of truth
- Workflow details (forge phases, implement lifecycle) are in the respective `commands/*.md` files

## Working on This Codebase

- Edit `commands/*.md` to change slash command behavior
- Edit `SKILL.md` to change universal skill behavior
- Edit `references/spec-format.md` to change the SPEC.html format reference
- Edit `examples/spec-styles.css` / `spec-runtime.js` / `SPEC.html` to change the rendered visual / runtime behavior. After any visual change, eyeball `examples/SPEC.html` in a browser.
- After any spec-format change, run the validate recipe on `examples/SPEC.html` to confirm structure is still parseable.
- Validate `.claude-plugin/*.json` stays valid JSON after edits
- Smoke-test changes: `claude plugin add /path/to/specmint-core-html` in a disposable project, then run `/forge`, `/resume`, etc.
- Windsurf users must replace the symlink at `.windsurf/skills/specmint-core-html/SKILL.md` with a real file copy (Cascade doesn't follow symlinks)

## Eval Infrastructure

Eval workspace at `specmint-workspace/` (gitignored). Contains iterations 1-5 with evals covering forge workflow, resume, spec-then-stop, research depth, spec quality, implement, researcher spawn, acceptance criteria, and progress tracking. Eval definitions in `specmint-workspace/evals/evals.json`.
