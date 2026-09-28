# Repository Guidelines

## Project Structure & Module Organization

This is a development repository for delegation skills. The root `SKILL.md` defines the Claude-to-Codex workflow, and `gemini-delegate/SKILL.md` defines a Codex-to-Antigravity CLI (`agy`, Gemini 3.8 Flash) workflow for simple, verifiable tasks. `README.md` explains the Claude skill, while `settings.json` is the example permission file copied to `~/.claude/settings.json`. `CLAUDE.md` and this file should stay aligned when conventions change. Local-only Claude overrides belong under `.claude/` and should not drive repo-wide behavior.

## Build, Test, and Development Commands

There is no build system or automated test runner in this repo. The main development loop is edit, reinstall, verify:

```bash
cp SKILL.md ~/.claude/skills/codex-delegate/SKILL.md
```

Reinstalls the skill after changes.

```bash
git diff
```

Review documentation and workflow edits before committing.

```bash
rg -n "full-auto|workspace-write|danger-full-access" SKILL.md README.md CLAUDE.md
```

Sanity-check that security-sensitive flags stay consistent across docs.

## Coding Style & Naming Conventions

Use concise Markdown with clear `##` headings, short paragraphs, and fenced code blocks for commands or JSON. Keep terminology consistent: use `codex-delegate` as the skill name, `Codex CLI` for the executor, and `workspace-write` / `--full-auto` exactly as implemented. Prefer focused edits over broad rewrites, and update hardcoded paths explicitly when documenting installation steps.

## Testing Guidelines

Validation is manual. After editing `SKILL.md`, reinstall it, restart Claude Code, and run `/skills` to confirm the skill loads. Test the approval flow by advancing to implementation and verifying both the delegation prompt and the documented execution flags. When changing permissions or security language, confirm `settings.json`, `README.md`, `AGENTS.md`, and `CLAUDE.md` still match.

## Commit & Pull Request Guidelines

Recent commits use short, imperative, sentence-case subjects such as `Fix Step 5 invocation and add parallel execution guide`. Follow that pattern and keep each commit scoped to one behavior or documentation change. Pull requests should describe the workflow change, list the files touched, and note manual verification steps. Include screenshots only if terminal output or Claude UI behavior is relevant.

## Security & Configuration Tips

Do not document or recommend `--sandbox danger-full-access` unless explicitly required. Keep the deny list in `settings.json` conservative, and treat the hardcoded Codex binary path in `SKILL.md` as machine-specific setup that contributors must replace locally.
