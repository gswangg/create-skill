---
name: create-skill
description: Create a new Claude Code skill. Use when the user wants to create a custom slash command, add background knowledge, or define a reusable workflow.
argument-hint: "description of the skill to create"
---

# Create a New Claude Code Skill

Create a skill based on the user's request. `$ARGUMENTS` is a plain text description of what the skill should do.

## Skill Location

Create skills in the **personal** location by default unless the user specifies otherwise:

| Location | Path | When to Use |
|----------|------|-------------|
| **Personal** (default) | `~/.claude/skills/<name>/SKILL.md` | Available across all projects |
| **Project** | `.claude/skills/<name>/SKILL.md` | Only when user requests project-specific |

## Required Structure

Every skill needs a `SKILL.md` file with YAML frontmatter:

```yaml
---
name: skill-name
description: What this skill does and when Claude should use it
---

Instructions for Claude when this skill is invoked...
```

## Frontmatter Fields

Include only the fields needed for the skill's purpose:

### Core Fields

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Slash command name. Lowercase, hyphens only, max 64 chars. |
| `description` | string | What the skill does and when to use it. Helps Claude decide when to invoke automatically. |

### Invocation Control

By default, skills are invocable by both the user (via `/skill-name`) and Claude (automatically when relevant). Only change these when you have a specific reason:

| Field | Default | When to Change |
|-------|---------|----------------|
| `disable-model-invocation` | `false` | Set `true` only for destructive actions (deploys, commits, deletions) where you want explicit user control. |
| `user-invocable` | `true` | Set `false` only for pure background knowledge that should never appear in the `/` menu. |

### Tool and Model Control

| Field | Purpose |
|-------|---------|
| `allowed-tools` | Tools Claude can use without permission when skill is active. Example: `Read, Grep, Bash(git *)` |
| `model` | Model to use when skill is active. Example: `claude-sonnet-4-5-20250514` |

### Subagent Execution

| Field | Purpose |
|-------|---------|
| `context` | Set to `fork` to run in isolated subagent context |
| `agent` | Subagent type when using `context: fork`. Options: `Explore`, `Plan`, `general-purpose` |

### User Experience

| Field | Purpose |
|-------|---------|
| `argument-hint` | Autocomplete hint shown to user. Use a plain description string (e.g. `"description of what to do"`). Only use structured `[arg]` syntax if the user specifically requests positional arguments. |

## Variable Substitutions

Use these in skill content:

| Variable | Expands To |
|----------|------------|
| `$\ARGUMENTS` | All arguments passed when invoking (preferred — treat as plain text description) |
| `$\0`, `$\1`, `$\2`, ... | Specific arguments by position (only use when user explicitly requests structured args) |
| `$\{CLAUDE_SESSION_ID}` | Current session ID |

(Remove backslashes in actual skill files)

**Default to `$\ARGUMENTS` as a plain string.** Only use positional `$\0`, `$\1` etc. if the user specifically asks for structured arguments.

## Dynamic Context Injection

Inject shell command output into skill content at load time. Write an exclamation mark immediately followed by the command wrapped in backticks. Example: to inject the current git branch, write exclamation-backtick-git branch --show-current-backtick (all as one token, no spaces around the backticks).

## Supporting Files

For complex skills, keep `SKILL.md` under 500 lines and reference additional files:

```
my-skill/
├── SKILL.md           # Main instructions (required)
├── reference.md       # Detailed documentation
├── examples.md        # Example outputs
└── templates/
    └── component.md   # Templates for Claude to use
```

Reference them in SKILL.md:
```markdown
For API details, see [reference.md](reference.md)
```

## Common Patterns

### Pattern 1: Simple Slash Command

```yaml
---
name: explain-code
description: Explains code with diagrams and analogies
---

When explaining code:
1. Start with an analogy
2. Draw ASCII diagram of the flow
3. Walk through step-by-step
4. Highlight common gotchas
```

### Pattern 2: Command with Arguments (Plain String)

```yaml
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
argument-hint: "issue number or description"
---

Fix GitHub issue based on: $\ARGUMENTS
1. Find the relevant issue
2. Understand requirements
3. Implement the fix
4. Write tests
5. Create a commit referencing the issue
```

### Pattern 3: Background Knowledge Only (Exception Case)

Use this pattern only when the skill should never appear in the `/` menu - purely contextual knowledge that Claude applies automatically:

```yaml
---
name: project-architecture
description: Architecture context for this codebase
user-invocable: false
---

This project uses:
- TypeScript with strict mode
- React for frontend
- PostgreSQL for data
- Our custom ORM in src/db/

When making changes, follow these patterns...
```

### Pattern 4: Restricted Tool Access

```yaml
---
name: safe-explore
description: Explore codebase without modifications
allowed-tools: Read, Grep, Glob
---

You can read and search files but cannot modify anything.
Answer questions about the codebase structure and content.
```

### Pattern 5: Isolated Subagent Task

```yaml
---
name: analyze-pr
description: Analyze a pull request in isolated context
context: fork
agent: Explore
---

PR context:
\!\`gh pr view\`
\!\`gh pr diff --name-only\`

Analyze this PR for:
- Code quality issues
- Potential bugs
- Missing test coverage
- Documentation needs
```

(Remove backslashes in actual skill files)

### Pattern 6: Structured Arguments (Only When Explicitly Requested)

Most skills should use `$\ARGUMENTS` as a plain string. Only use positional args if the user specifically asks for them:

```yaml
---
name: migrate
description: Migrate code from one pattern to another
argument-hint: "[file-pattern] [from] [to]"
---

Migrate files matching `$\0` from $\1 to $\2.

1. Find all matching files
2. Apply the migration pattern
3. Update imports
4. Verify no regressions
```

## Creating the Skill

1. Create the directory at the chosen location
2. Write `SKILL.md` with appropriate frontmatter
3. Add supporting files if needed
4. Test by invoking with `/<skill-name>`

