# Repository Guidelines

Codex-style guidelines for agents working on Spec Mint Core HTML. The plugin's full project context is in `CLAUDE.md`; this file is the contributor / agent-style guide.

For architectural context across the Mint family (core vs TDD, core-vs-core-html differences, distribution, evals), read and write to the **ngvoicu-sme** brain through kluris — `/kluris-ngvoicu-sme` (Claude Code skill: search, learn, remember, create) or `kluris search "<query>" --brain ngvoicu-sme` (CLI). Never edit brain files by hand.

## Project Structure & Module Organization

- `.claude-plugin/`: plugin metadata for Claude Code distribution (`plugin.json`, `marketplace.json`).
- `.cursor-plugin/`: plugin metadata for Cursor distribution.
- `commands/`: one Markdown file per slash command (`forge.md`, `implement.md`, `resume.md`, `pause.md`, `switch.md`, `list.md`, `status.md`, `openapi.md`). Each file is the behavioral contract for that command.
- `agents/researcher.md`: subagent prompt used for deep codebase research (Opus model).
- `references/`:
  - `spec-format.md` — canonical `SPEC.html` format reference (regions, `data-status`, JSON key order, conventions).
  - `html-template.html` — empty canonical template AI seeds from.
  - `edit-recipes.md` — before/after snippets for every common surgical edit.
  - `validate.md` — post-edit validation recipe (Python one-liner).
  - `wireframe-library.md` — wireframe mockup patterns built on `.wf-*` primitives.
  - `mockup-library.md` — hi-fi mockup patterns built on `.ui-*` components.
  - `command-contracts.md` — behavioral contract checklist for commands and SKILL.md.
- `examples/`:
  - `SPEC.html` — ground-truth UI-rich exemplar (team-invites). Open in a browser to see what specs render as.
  - `spec-styles.css` — shared design system (gets copied into `.specs/assets/` on first forge in any consuming project).
  - `spec-runtime.js` — progress deriver + Mermaid/Prism init + SVG annotation arrows.
- `SKILL.md`: universal, cross-tool skill instructions (Codex, Cursor, Windsurf, Cline, Gemini CLI).
- `specmint-workspace/`: eval scaffold (gitignored). Contains `evals/evals.json` with placeholder TODO assertions — not yet runnable.
- `.specs/`: local dogfooding output for specs (gitignored).

## Build, Test, and Development Commands

- `rg --files`: fast inventory of repository files before editing.
- `sed -n '1,160p' commands/forge.md`: inspect command content in the terminal.
- `python3 -m http.server 8000` (run inside `examples/`): serve the exemplar at <http://localhost:8000/SPEC.html> to eyeball visual changes.
- `python3 -c "import re,json; h=open('examples/SPEC.html').read(); m=re.search(r'<script[^>]*id=\"spec-meta\"[^>]*>(.+?)</script>',h,re.S); json.loads(m.group(1)); o=re.findall(r'<!--\\s*region:(\\w+)\\s*-->',h); c=re.findall(r'<!--\\s*endregion:(\\w+)\\s*-->',h); assert sorted(o)==sorted(c); print('OK')"`: validate exemplar structure (full recipe in `references/validate.md`).
- `npx skills add ngvoicu/specmint-core-html -g -a codex`: smoke-test universal-skill installation flow.
- `git log --oneline -n 10`: review recent commit style before committing.

This repository has no compile/build pipeline; Markdown, JSON, HTML, CSS, and JS are consumed directly by host tools or the browser.

## Coding Style & Naming Conventions

- Plugin source (commands, references, SKILL.md, README.md): ASCII Markdown / JSON with concise, imperative instructions.
- Use lowercase, hyphenated filenames for command docs (for example `commands/openapi.md`).
- Keep command docs procedural (numbered steps, explicit file paths, deterministic behavior).
- Spec naming in examples and recipes:
  - Spec IDs are lowercase-hyphenated (`user-auth-system`).
  - Task codes are `[PREFIX-NN]` where prefix is a 2-4 letter uppercase abbreviation of the spec.
  - Phase status uses `data-status="pending" | "in-progress" | "completed" | "blocked"`.
- SPEC.html metadata uses **canonical JSON key order**: `id`, `title`, `status`, `created`, `updated`, `priority`, `tags`, `mockup-fidelity` (logical, not alphabetical).
- One attribute per line on state-bearing elements (task, phase, AC item) when the line would be long; one element per line for list rows. Keeps git diffs surgical.

## Testing Guidelines

- No automated test suite currently exists in this repository.
- Perform manual validation for each change:
  - Run the validate recipe on `examples/SPEC.html` after any format change.
  - Verify `.claude-plugin/*.json` and `.cursor-plugin/*.json` stay valid JSON.
  - Confirm referenced paths/files exist.
  - Open `examples/SPEC.html` in a browser after any visual change (CSS / runtime / template).
  - Smoke-test install/use flow in a disposable project when command behavior changes.
- If you change spec-format rules, update `SKILL.md`, `references/spec-format.md`, and `references/edit-recipes.md` in the same PR.

## Commit & Pull Request Guidelines

- Git history is mostly terse `update` commits, with occasional Conventional Commit messages (for example `feat: …`, `docs: …`).
- Prefer descriptive, scoped commit messages (for example `docs: tighten openapi command generation rules`).
- PRs should include purpose, affected files, behavior changes (before/after prompt or HTML snippets when relevant), and linked issue/context when available.
- Don't span sub-repos in a single commit — keep changes scoped to this plugin.
