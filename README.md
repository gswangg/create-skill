# create-skill

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that helps you create new Claude Code skills. Invoke it with `/create-skill [name] [description]` and it will scaffold a well-structured `SKILL.md` with the correct frontmatter, file layout, and patterns.

## Purpose

Creating Claude Code skills involves getting several things right: YAML frontmatter fields, variable substitution syntax, tool permissions, invocation control, and directory placement. This skill encodes all of those conventions so you can describe what you want and get a working skill created for you.

## Prerequisites

- **Claude Code** installed and working
- No additional dependencies — this skill is pure instructions, no scripts

## Usage

```
/create-skill my-skill-name Does something useful
```

Arguments:
- First argument: skill name (lowercase, hyphens, max 64 chars)
- Remaining arguments: description of what the skill should do

Claude will create the skill directory and `SKILL.md` at the appropriate location.

## What it covers

### Skill locations

| Location | Path | When to use |
|----------|------|-------------|
| **Personal** (default) | `~/.claude/skills/<name>/SKILL.md` | Available across all projects |
| **Project** | `.claude/skills/<name>/SKILL.md` | Project-specific skills |

### Frontmatter fields

| Field | Purpose |
|-------|---------|
| `name` | Slash command name |
| `description` | What the skill does (used for auto-invocation matching) |
| `allowed-tools` | Tools allowed without permission prompts |
| `model` | Model override when skill is active |
| `argument-hint` | Autocomplete hint for arguments |
| `disable-model-invocation` | Prevent Claude from auto-invoking (for destructive actions) |
| `user-invocable` | Set `false` for background knowledge only |
| `context` | Set to `fork` for isolated subagent execution |
| `agent` | Subagent type (`Explore`, `Plan`, `general-purpose`) |

### Variable substitutions

| Variable | Expands to |
|----------|------------|
| `$ARGUMENTS` | All arguments passed when invoking |
| `$0`, `$1`, `$2`, ... | Specific arguments by position |
| `${CLAUDE_SESSION_ID}` | Current session ID |

### Dynamic context injection

Shell commands can be executed at skill load time to inject dynamic context (e.g., current git branch, environment info).

### Patterns included

1. **Simple slash command** — basic instruction skill
2. **Command with arguments** — positional argument handling
3. **Background knowledge** — non-invocable context (architecture docs, conventions)
4. **Restricted tool access** — read-only exploration skills
5. **Isolated subagent task** — forked context with dynamic command output
6. **Multi-argument command** — file patterns, migrations

## Limitations

- Skills are a Claude Code feature — they don't work outside of Claude Code
- The `SKILL.md` file must be under 500 lines; use supporting files for longer content
- Dynamic context injection runs at skill load time, not at invocation time
- Variable substitution syntax uses `$ARGUMENTS` / `$0` etc. — these are Claude Code-specific, not shell variables

## License

MIT
