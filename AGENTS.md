# Repository Guidelines

Codex-style guidelines for agents working on Spec Mint Core HTML. The skill's full project context is in `CLAUDE.md`; this file is the contributor / agent-style guide.

For architectural context across the Mint family (core vs TDD, core-vs-core-html differences, distribution, evals), read and write to the **ngvoicu-sme** brain through kluris — `/kluris-ngvoicu-sme` (Claude Code skill: search, learn, remember, create) or `kluris search "<query>" --brain ngvoicu-sme` (CLI). Never edit brain files by hand.

## Project Structure & Module Organization

- `SKILL.md`: the universal, cross-tool skill — workflow, lifecycle rules, invariants, and OpenAPI behavior. The single behavioral contract.
- `references/`:
  - `researcher.md` — deep-research subagent brief, spawned via the Task tool during forge.
  - `spec-format.md` — canonical `SPEC.html` format reference (regions, `data-status`, JSON key order, conventions).
  - `html-template.html` — empty canonical template AI seeds from.
  - `edit-recipes.md` — before/after snippets for every common surgical edit.
  - `validate.md` — post-edit validation recipe (Python one-liner).
  - `wireframe-library.md` — wireframe mockup patterns built on `.wf-*` primitives.
  - `mockup-library.md` — hi-fi mockup patterns built on `.ui-*` components.
  - `command-contracts.md` — behavioral contract checklist for SKILL.md.
- `assets/`:
  - `spec-styles.css` — shared design system (gets copied into `.specs/assets/` on every forge in any consuming project).
  - `spec-runtime.js` — progress deriver + Mermaid/Prism init + SVG annotation arrows + diagram fullscreen modal + full-spec validator.

  The reference render of a generated `SPEC.html` lives at <https://specmint.io/#gallery> (instead of an embedded screenshot in this repo).
- `evals/evals.json`: 6 real eval scenarios with 33 verifiable expectations (tracked). Run via `/skill-creator improve` (Anthropic's official skill-creator plugin) in a fresh Claude Code session. Run outputs land in a gitignored `specmint-core-html-workspace/` sibling directory.
- `.specs/`: local dogfooding output for specs (gitignored).

## Build, Test, and Development Commands

- `rg --files`: fast inventory of repository files before editing.
- `sed -n '1,160p' SKILL.md`: inspect skill content in the terminal.
- `python3 -m http.server 8000` (run inside a consumer project's `.specs/<id>/`): serve a real generated `SPEC.html` at <http://localhost:8000/SPEC.html> to eyeball visual changes. The reference render lives at <https://specmint.io/#gallery>.
- `python3 -c "import re,json,sys; p=sys.argv[1]; h=open(p).read(); m=re.search(r'<script[^>]*id=\"spec-meta\"[^>]*>(.+?)</script>',h,re.S); json.loads(m.group(1)); o=re.findall(r'<!--\\s*region:(\\w+)\\s*-->',h); c=re.findall(r'<!--\\s*endregion:(\\w+)\\s*-->',h); assert sorted(o)==sorted(c); print('OK')" path/to/SPEC.html`: validate a generated SPEC.html (full recipe in `references/validate.md`).
- `npx skills add ngvoicu/specmint-core-html -g -a codex`: smoke-test universal-skill installation flow.
- `git log --oneline -n 10`: review recent commit style before committing.

This repository has no compile/build pipeline; Markdown, JSON, HTML, CSS, and JS are consumed directly by host tools or the browser.

## Coding Style & Naming Conventions

- Skill source (`SKILL.md`, references, README.md): ASCII Markdown with concise, imperative instructions.
- Use lowercase, hyphenated filenames for reference docs (for example `references/spec-format.md`).
- Keep instructions procedural (numbered steps, explicit file paths, deterministic behavior).
- Spec naming in examples and recipes:
  - Spec IDs are lowercase-hyphenated (`user-auth-system`).
  - Task codes are `[PREFIX-NN]` where prefix is a 2-4 letter uppercase abbreviation of the spec.
  - Phase status uses `data-status="pending" | "in-progress" | "completed" | "blocked"`.
- SPEC.html metadata uses **canonical JSON key order**: `id`, `title`, `status`, `created`, `updated`, `priority`, `tags`, `mockup-fidelity` (logical, not alphabetical).
- One attribute per line on state-bearing elements (task, phase, AC item) when the line would be long; one element per line for list rows. Keeps git diffs surgical.

## Testing Guidelines

- No automated test suite currently exists in this repository.
- Perform manual validation for each change:
  - Run the validate recipe on a generated `.specs/<id>/SPEC.html` after any format change.
  - Confirm referenced paths/files exist.
  - Dogfood the skill in a disposable consumer project after any visual change (CSS / runtime / template) and open the generated `SPEC.html` in a browser.
  - Smoke-test the install/use flow in a disposable project when skill behavior changes (e.g. `npx skills add ./. -g -a claude-code`, or copy `SKILL.md` into its skills dir, then exercise natural-language triggers — forge / resume / implement).
- If you change spec-format rules, update `SKILL.md`, `references/spec-format.md`, and `references/edit-recipes.md` in the same PR.

## Commit & Pull Request Guidelines

- Git history is mostly terse `update` commits, with occasional Conventional Commit messages (for example `feat: …`, `docs: …`).
- Prefer descriptive, scoped commit messages (for example `docs: tighten openapi command generation rules`).
- PRs should include purpose, affected files, behavior changes (before/after prompt or HTML snippets when relevant), and linked issue/context when available.
- Don't span sub-repos in a single commit — keep changes scoped to this skill.
