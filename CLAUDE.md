# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project overview

This is a documentation-first development repo for delegation skills. The root `SKILL.md` covers the Claude Code to OpenAI Codex handoff, and `gemini-delegate/SKILL.md` covers a Codex to Gemini CLI handoff for small, testable tasks. Most changes land in skill markdown files plus supporting docs; there is no build system or automated test suite.

## Repository structure

```
delegator/
├── SKILL.md                  # Claude Code skill for delegating to Codex
├── gemini-delegate/
│   └── SKILL.md             # Codex skill for delegating simple tasks to Gemini
├── README.md                # User-facing docs for the Claude skill
├── settings.json            # Example ~/.claude/settings.json permissions
├── AGENTS.md                # Contributor guide for human and AI collaborators
└── CLAUDE.md                # Claude-specific repo instructions
```

## Core workflow

The root `SKILL.md` defines the six-step Claude to Codex path:

1. Ask once whether to delegate to Codex
2. Read Claude permissions and assemble approved commands
3. Read a small set of relevant files only
4. Build a self-contained Codex prompt with a SECURITY directive
5. Run `codex exec` with `--sandbox workspace-write --full-auto`
6. Review the diff and check for deny-list violations

Keep shared safety language synchronized across `SKILL.md`, `gemini-delegate/SKILL.md`, `README.md`, `AGENTS.md`, and this file.

## Validation

Reinstall the skill after edits:

```bash
mkdir -p ~/.claude/skills/codex-delegate
cp SKILL.md ~/.claude/skills/codex-delegate/SKILL.md
```

Restart Claude Code, run `/skills`, then exercise the delegation prompt. If Step 5 changes, confirm the docs consistently mention `--full-auto` and `workspace-write`, and never recommend `danger-full-access` unless explicitly required.

## Editing guidance

Prefer narrow documentation edits over broad rewrites. Preserve exact flag names, command examples, and security language. The Codex binary path in `SKILL.md` is machine-specific; document it clearly but do not generalize it as a portable default. Commit messages should follow the existing pattern: short, imperative, and sentence case.

## Relationship to agent_skills

This repo remains canonical for `codex-delegate` and `gemini-delegate` — edit the skills here as before. `agent_skills/registry.yaml` lists this repo as an upstream, and `agent_skills/sync.sh --vendor` copies these skills into `agent_skills/core/`, which then distributes them to the global agent dirs. After changing a skill here, run `./sync.sh --vendor` from `agent_skills/` so the vendored copies and everything downstream pick up the change.
