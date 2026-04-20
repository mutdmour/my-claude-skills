# codekit

Personal Claude Code plugin with interactive skills for learning and quizzing on codebase architecture.

## Install

In Claude Code, run:

```
/plugin marketplace add mutdmour/codekit
/plugin install codekit@mutasem
```

Skills are then available as `/codekit:teachme` and `/codekit:quizme`.

## Skills

### teachme

Guided interactive walkthrough of any codebase area, feature, PR, or architectural concept.

**Usage:**
- `/codekit:teachme` -- infers topic from current branch and conversation context
- `/codekit:teachme workflow execution` -- teach a specific topic
- `/codekit:teachme #4521` -- walk through a PR

**How it works:**
1. Claude reads relevant code to understand the actual architecture
2. Asks what areas to cover, at what depth, and how code-specific explanations should be
3. Builds a syllabus and teaches section by section with check-ins
4. Adapts pace based on your responses; offers to go deeper or move on

### quizme

Interactive architecture quiz. Tests your understanding of a codebase through a mix of multiple-choice and free-write questions.

**Usage:**
- `/codekit:quizme` -- infers topic from current branch and conversation context
- `/codekit:quizme workflow execution` -- quiz on a specific topic
- `/codekit:quizme #4521` -- quiz on the area a PR touches

**How it works:**
1. Claude reads relevant code to understand the actual architecture
2. Asks what areas to cover, at what difficulty, and how code-specific questions should be
3. Asks questions one at a time (terminology, patterns, data flow)
4. If you get something wrong: explains why, teaches the correct answer, asks a follow-up to reinforce
5. When you say "stop", shows a scorecard by concept area with pointers to code

## Adding New Skills

Create a folder with a `SKILL.md` file:

```
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
