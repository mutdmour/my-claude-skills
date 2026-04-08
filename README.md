# my-skills

Personal Claude Code skills for learning and productivity.

## Setup

Symlink each skill into your Claude Code skills directory:

```bash
# Clone the repo
git clone https://github.com/mutdmour/my-skills.git

# Symlink individual skills
ln -s /path/to/my-skills/quizme ~/.claude/skills/quizme
```

Skills are available immediately in any Claude Code session.

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
