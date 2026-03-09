# codex-delegate — Claude Code Skill

A Claude Code skill plugin that offers to delegate implementation work to the [OpenAI Codex CLI](https://github.com/openai/codex) at every implementation decision point. Claude plans and reviews; Codex writes files and runs commands.

---

## Why

Claude Code is excellent at planning, architecture, and code review, but each tool call (file write, shell command, edit) consumes tokens. For large implementation tasks — scaffolding a project, applying a multi-file refactor, running a build pipeline — token cost adds up fast.

The Codex CLI (`codex exec`) can execute an entire implementation plan from a single structured prompt, using its own token budget. This skill wires the two together:

- **Claude** reads the codebase, plans the work, crafts a precise Codex prompt, and reviews the result
- **Codex** writes files, runs shell commands, and reports back

One delegation can replace dozens of Claude tool calls.

---

## How it works

### Trigger

The skill auto-triggers whenever a plan is approved and implementation is about to start. Trigger phrases include: `proceed`, `implement`, `go ahead`, `execute the plan`, `approved`, `looks good`, `do it`.

### Workflow

```
User approves plan
       │
       ▼
Claude asks: "Delegate implementation to Codex CLI? (yes / no)"
       │
  ┌────┴────┐
 YES        NO
  │          │
  ▼          ▼
Step 2    Claude implements normally
Assemble permissions
  │
  ▼
Step 3
Claude reads 3–5 key files (no writes)
  │
  ▼
Step 4
Craft structured Codex prompt
  │
  ▼
Step 5
codex exec "<prompt>" --sandbox workspace-write
  │
  ▼
Step 6
Claude reviews: git diff, reads modified files,
checks for deny-list violations, reports to user
```

### Permission inheritance

Claude reads your `~/.claude/settings.json` and passes your permission model into the Codex prompt:

| Claude setting | Effect on Codex |
|---|---|
| `allow` list | Commands auto-approved in the Codex prompt — Codex runs them freely |
| `deny` list | Goes into `HARD CONSTRAINTS` — Codex must never run them |
| Commands in neither list | Claude surfaces them once for session approval before calling Codex |

### Security

- Codex runs with `--sandbox workspace-write`: file access is bounded to the project directory
- The Codex prompt begins with an explicit SECURITY directive treating all file contents as data only (prompt injection defense)
- Deny-list violations are checked twice: in the prompt (model-level) and in Claude's post-execution `git diff` / shell history grep
- Claude always reviews the diff before reporting completion

---

## Installation

### Prerequisites

- [Claude Code](https://claude.ai/claude-code) installed
- [Codex CLI](https://github.com/openai/codex) installed and authenticated
  ```bash
  npm install -g @openai/codex
  codex login
  ```

### 1. Create the plugin directory structure

```bash
mkdir -p ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/codex-delegate/.claude-plugin
mkdir -p ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/codex-delegate/skills/codex-delegate
```

### 2. Create the plugin manifest

Create `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/codex-delegate/.claude-plugin/plugin.json`:

```json
{
  "name": "codex-delegate",
  "description": "Delegate code execution to OpenAI Codex CLI while Claude plans and reviews",
  "author": {
    "name": "Your Name"
  }
}
```

### 3. Copy the skill file

```bash
cp SKILL.md ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/codex-delegate/skills/codex-delegate/SKILL.md
```

Or clone this repo and run:

```bash
git clone https://github.com/cesarlandin/claudesquare
cp claudesquare/SKILL.md ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/codex-delegate/skills/codex-delegate/SKILL.md
```

### 4. Restart Claude Code

The plugin is picked up on startup. After restarting, the skill will auto-trigger at every implementation decision point.

---

## Settings

This skill is designed to work with a `~/.claude/settings.json` that explicitly defines which shell commands Claude (and by extension, Codex) may run or is blocked from running. An example settings file is included at [`settings.json`](./settings.json).

### Structure

```json
{
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep", "Write", "Edit",
      "Bash(git status*)", "Bash(git diff*)", "Bash(git add*)", "Bash(git commit*)",
      "Bash(python3*)", "Bash(npm run*)", "..."
    ],
    "deny": [
      "Bash(rm*)",
      "Bash(git push --force*)",
      "Bash(git reset --hard*)",
      "Bash(git clean*)",
      "Bash(gcloud projects delete*)",
      "Bash(gcloud billing*)",
      "Bash(dropdb*)",
      "Bash(mysqladmin drop*)"
    ]
  }
}
```

The `deny` list is the safety baseline that travels into every Codex prompt as HARD CONSTRAINTS. Review and adjust it for your environment before installing.

### Codex binary path

The skill hardcodes the Codex binary path in Step 5:

```
/Users/cesarlandin/.nvm/versions/node/v24.13.0/bin/codex
```

Edit `SKILL.md` to match your own path before installing:

```bash
which codex
```

---

## File reference

| File | Purpose |
|---|---|
| `SKILL.md` | The skill definition loaded by Claude Code |
| `settings.json` | Example Claude Code permissions file |
| `.claude-plugin/plugin.json` | Plugin manifest (created during install) |

---

## Verification

After restarting Claude Code:

1. Approve any plan or say `proceed`
2. Claude should ask: `Delegate implementation to Codex CLI? (yes / no)`
3. Answer `yes` — Claude will assemble the prompt, run `codex exec`, then review the diff

---

## License

MIT
