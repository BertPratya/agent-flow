# Tool adapters

The workflow lives in `AGENTS.md` and `workflow/`. Adapters
only point each tool at it. The lead copies them to the paths below, and
keeps the copies the same as the files here.

| Tool | Copy | To | Does |
|---|---|---|---|
| Claude Code | `claude/CLAUDE.md` | `CLAUDE.md` (if it exists, add its one line instead of replacing it) | imports `AGENTS.md` |
| Claude Code | (link) | `.claude/skills/flow-setup` → `../../.agents/skills/flow-setup` | Claude Code reads skills only from `.claude/skills/` |
| Claude Code | `claude/agents/` | `.claude/agents/` | roles as subagents (Cursor reads these too) |
| Codex | `codex/agents/` | `.codex/agents/` | roles as subagents |

Codex and Cursor read `AGENTS.md` and `.agents/skills/` without adapters.

On Windows, git links need extra setup. Without it, copy the
`flow-setup` folder into `.claude/skills/` instead.

## Notes

- **Codex:** project agents load only when the project is trusted. Agents
  inherit the parent's sandbox, so they cannot be made read-only per agent;
  the role files tell reviewers to write only their review file.
- **Same names everywhere** (`flow-worker`, …), so Cursor, which reads both
  `.claude/agents/` and `.codex/agents/`, sees each role once.
