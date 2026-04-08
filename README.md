# my-skills

Personal Claude Code skills for learning and productivity.

## Setup

Add to your Claude Code settings to make these skills available:

```bash
claude mcp add skills -- npx -y @anthropic-ai/claude-code-mcp-skills /path/to/my-skills
```

Or add directly to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "skills": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/claude-code-mcp-skills", "/path/to/my-skills"]
    }
  }
}
```

## Skills

### quizme

Interactive architecture quiz. Tests your understanding of a codebase through a mix of multiple-choice and free-write questions.

**Usage:**
- `/quizme` -- infers topic from your current branch and conversation context
- `/quizme workflow execution` -- quiz on a specific topic

**How it works:**
1. Claude reads relevant code to understand the actual architecture
2. Asks questions one at a time (terminology, patterns, data flow)
3. If you get something wrong: explains why, teaches the correct answer, asks a follow-up to reinforce
4. When you say "stop", shows a scorecard by concept area with pointers to code

## Adding New Skills

Create a folder with a `SKILL.md` file:

```
my-skills/
  skill-name/
    SKILL.md
```

The `SKILL.md` needs YAML frontmatter with `name` and `description`:

```yaml
---
name: skill-name
description: Use when [triggering conditions]
---
```
