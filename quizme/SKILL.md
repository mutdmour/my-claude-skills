---
name: quizme
description: Use when the user wants to test their understanding of codebase architecture, design patterns, or feature internals through an interactive quiz session
---

# QuizMe - Interactive Architecture Quiz

## Overview

Quiz the user on codebase architecture through interactive Q&A. Infer topic from current context or accept an explicit topic. Ask questions grounded in actual code, using the format the user prefers (multiple choice, free-write, or mixed). Act as a helpful guide -- when the user gets something wrong, explain why, teach the correct answer, and reinforce with a follow-up.

## Invocation

- `/quizme` -- infer topic from current branch, recent conversation, or files touched
- `/quizme <topic>` -- quiz on an explicit topic
- `/quizme <PR>` -- e.g., `/quizme #4521` or a GitHub PR URL

## Process

```
Invoked
  |
  v
Topic provided? --no--> Infer from context --> Confirm with user
  |yes                                              |
  v                                                 v
Read relevant code <--------------------------------+
  |
  v
Ask focus areas + difficulty
  |
  v
User responds
  |
  v
Generate quiz plan
  |
  v
Ask question <------------------------------------------+
  |                                                     |
  v                                                     |
Evaluate answer                                         |
  |                                                     |
  +--correct--> Acknowledge --> User says stop? --no----+
  |                                  |yes
  +--wrong----> Explain why          v
                  |             Show scorecard
                  v
              Reinforcement follow-up
                  |
                  v
              Evaluate again ----correct----> next question
```

### Step 1: Determine Topic

If no topic argument provided:
1. Look at current branch name, recent files in conversation, and discussion context
2. State what you think the user is working on: "It looks like you're working on **X**. Want me to quiz you on that?"
3. Wait for confirmation before proceeding

If topic argument is a PR (`#number` or GitHub URL):
1. Fetch PR info via `gh pr view` and `gh pr diff`
2. Ask the user: "Want me to check out the PR branch **`<branch-name>`** locally so we can explore the actual code?" If they confirm, run `gh pr checkout <number>`. If they decline, continue using the diff only.
3. Build quiz questions around the area the PR touches -- architecture, patterns, and design decisions in that area.

If topic argument provided (not a PR), proceed directly.

### Step 2: Read the Code

Read relevant packages, services, and key files to understand:
- Component boundaries and responsibilities
- Design patterns in use
- Data flow between layers
- Key terminology (class names, service names, pattern names)

**Important:** You are building questions about architecture, not about files. Read code to ensure accuracy, but quiz on concepts.

### Step 2.5: Ask Focus Areas, Difficulty, Abstraction Level, and Question Format

After reading the code, before generating the quiz plan, ask what areas the user wants to be tested on, at what difficulty, at what level of abstraction, and in what question format. Present this as a single combined question.

Based on your code reading, surface 4-6 meaningful focus area suggestions specific to the topic -- real areas from the actual code, not generic categories.

Example prompt (adapt to the actual topic):

> Before I build your quiz, a few quick questions:
>
> **What areas do you want to be tested on?** (pick one or more, or say "all")
> - **A) Component boundaries** -- what each piece owns and where responsibilities split
> - **B) Data flow** -- how data moves between layers end-to-end
> - **C) Design patterns** -- which patterns are in use and why
> - **D) Error handling** -- how failures propagate and what recovery looks like
> - **E) Integration points** -- how this connects to the rest of the system
>
> **What difficulty level?**
> - **1) Broad strokes** -- key concepts and terminology, no internals
> - **2) Standard** -- patterns, data flow, component interactions
> - **3) Rigorous** -- edge cases, design gaps, nuanced interactions, "why" questions
>
> **How code-specific should questions and explanations be?**
> - **I) Conceptual** -- concepts, responsibilities, and flows only; no class/function/file names
> - **II) Named** -- use and expect real class, service, and method names (no file paths or line numbers)
> - **III) Code-level** -- reference specific methods and implementation details; show code in explanations when it adds clarity
>
> **What question format do you prefer?**
> - **MC) Multiple choice only** -- pick from options; good for terminology and pattern recognition
> - **FW) Free-write only** -- explain in your own words; good for testing deeper understanding
> - **OD) Ordering** -- put steps or components in the correct sequence; good for data flow and pipelines
> - **SC) Scenario** -- reason through a failure or edge case ("what happens when X fails?"); good for resilience and deep understanding
> - **MX) Mixed** *(default)* -- combines all formats based on what best fits each concept

Wait for the user's response before generating the quiz plan.

**Scoping rules based on response:**
- **Focus areas:** Generate questions only from chosen areas unless the user says "all".
- **Broad strokes:** 4-6 areas, questions focus on naming and identifying components. Multiple choice preferred. No gotchas.
- **Standard:** 5-8 areas, mix of multiple choice and free-write. Covers main patterns and flows.
- **Rigorous:** 6-10 areas, heavier on free-write. Include edge cases, "what happens when X fails", design trade-offs, and gaps in the implementation.
- **Conceptual abstraction:** Questions and explanations use only concepts, responsibilities, and patterns -- no specific class, method, or file names. Good for testing architectural thinking.
- **Named abstraction:** Questions use and expect real class/service/method names. Correct terminology matters; wrong names get corrected. No file paths or line numbers.
- **Code-level abstraction:** Questions may reference specific methods or implementation details. Explanations for wrong answers may include code snippets when they clarify the concept.
- **Multiple choice only:** Every question uses multiple choice format with 3-4 plausible options.
- **Free-write only:** Every question asks the user to explain, describe, or reason in their own words.
- **Ordering only:** Every question presents a shuffled list of steps or components the user must sequence correctly.
- **Scenario only:** Every question presents a failure or edge-case situation and asks what happens next.
- **Mixed (default):** Combine all formats based on what best tests each concept -- MC for terminology, ordering for flows, scenario for resilience, free-write for architecture/reasoning.

### Step 3: Generate Quiz Plan

After reading the code, generate a quiz plan covering all key areas of the topic. Write it to `~/.claude/quizme/<topic-slug>.md` (e.g., `~/.claude/quizme/execution-engine.md`). Derive the slug from the topic name -- lowercase, hyphens, no special characters. The plan ensures comprehensive coverage and lets the user resume across sessions.

**Plan format:**

```markdown
# QuizMe: [Topic Name]
Started: [date]
Focus: [chosen areas, e.g. "Data Flow, Error Handling" or "All"]
Difficulty: [Broad strokes / Standard / Rigorous]
Abstraction: [Conceptual / Named / Code-level]
Format: [Multiple choice / Free-write / Mixed]

## Areas to Cover

- [ ] **1. Component Boundaries** -- what each piece owns, where responsibilities split
- [ ] **2. Data Flow** -- how data moves between layers and components
- [ ] **3. Design Patterns** -- patterns in use, why they were chosen
- [ ] **4. Key Terminology** -- correct names for components, services, concepts
- [ ] **5. Error Handling** -- how failures propagate, recovery strategies
- [ ] **6. Integration Points** -- how this connects to the rest of the system

## Progress
<!-- Updated as areas are covered -->
| Area | Questions Asked | Correct | Notes |
|------|----------------|---------|-------|
```

**Plan rules:**
- Scope areas to the user's chosen focus and difficulty from Step 2.5.
- Generate 4-6 areas for Broad strokes, 5-8 for Standard, 6-10 for Rigorous. Don't pad with filler.
- Each area should map to a meaningful concept cluster, not a file.
- Check off areas once 2+ questions in that area have been answered.
- If a plan already exists for this topic (check `~/.claude/quizme/` for matching slug), read it and resume. Ask: "We have an existing quiz on **X** (focus: [X], difficulty: [Y]) -- want to continue where you left off, or start fresh with different focus/difficulty?"
- On first invocation, list any existing sessions in `~/.claude/quizme/` so the user knows what's available.

### Step 4: Ask Questions

Alternate between two question types:

**Multiple choice** -- for terminology and concept identification:
- "What design pattern does the execution engine use for node processing?"
- Use 3-4 options. Make distractors plausible but distinguishable.

**Free-write** -- for architectural understanding:
- "In your own words, how does a workflow execution get triggered and flow through the system?"
- Expect the user to use correct terminology. If they use wrong terms, correct them gently.

**Ordering** -- for data flow and pipelines:
- "Put these steps in the correct order: [shuffled list of 4-6 steps]"
- Use for request lifecycles, execution pipelines, event propagation sequences.
- Accept loose correct answers; focus on whether they understand the sequence, not exact wording.

**Scenario** -- for resilience and edge cases:
- "The queue worker crashes mid-execution -- what happens to the in-flight job and why?"
- Use for error handling, failure modes, and "what happens when X" questions.
- Best reserved for Standard/Rigorous difficulty. Evaluate whether the user identifies the failure boundary and recovery path.

**Question principles:**
- Target component relationships, data flow, design patterns, layer responsibilities
- Use real class/service/package names from the codebase
- Never ask about specific line numbers or file paths
- Increase difficulty gradually -- start with broad architecture, move toward nuanced interactions
- One question per message. Wait for the answer.
- Draw questions from uncovered plan areas first. Once an area has 2+ answered questions, check it off and move to the next.

**Progress indicator:** Include a progress line before each question:

> `[2/6 areas covered]` -- you've nailed component boundaries and data flow. Next up: design patterns.

- Show as `[X/Y areas covered]` where X is checked-off areas and Y is total.
- Keep it to one line.
- Include it every question so the user always knows where they stand.
- When most areas are covered, mention it: "Almost done -- just **Error Handling** left."

### Step 5: Evaluate Answers

**If correct:**
- Brief acknowledgment ("Right!" or "Exactly.")
- Optionally add a small bonus insight the user might not know
- Move to the next question

**If wrong -- 3-step flow:**
1. **Explain why wrong:** "That's not quite right. [X] doesn't [do what you said] because [reason]."
2. **Explain the correct answer:** "What actually happens is [correct explanation]. The term for this is [correct terminology]."
3. **Reinforcement follow-up:** Ask a related question that tests the same concept from a different angle. This must be answered before moving on.

**Tone:** Helpful guide, not examiner. Frame corrections as "here's what's actually happening" not "you're wrong."

### Step 6: End Session and Scorecard

When the user says "stop", "done", "enough", or similar:

Produce a scorecard grouped by concept area:

```
## Quiz Summary

### Areas Covered

| Area | Rating | Notes |
|------|--------|-------|
| Data Flow | Strong | Correctly traced execution from trigger to completion |
| Terminology | Needs Work | Confused "controller" with "service" in several answers |
| Design Patterns | Decent | Got DI right, but missed event-driven communication pattern |

### Where to Dig Deeper
- **Terminology:** Look at `packages/cli/src/controllers/` vs `packages/cli/src/services/` to see the distinction in practice
- **Event patterns:** Check the event bus implementation in `packages/cli/src/events/`
```

**Scorecard principles:**
- Rate each area: Strong / Decent / Needs Work
- For weak areas, point to relevant code areas (package/directory level, not specific files)
- Keep tone encouraging -- highlight strengths alongside gaps
- Note which plan areas were covered and which remain
- End with a one-line overall assessment

The quiz plan persists at `~/.claude/quizme/<topic-slug>.md` with progress marked -- the user can resume in a future conversation by invoking `/quizme` on the same topic.

## Key Rules

- **One question per message.** Never batch questions.
- **Always read actual code** before asking questions. Don't invent architecture.
- **Ask focus areas, difficulty, abstraction level, and question format before generating the plan.** Step 2.5 is mandatory -- don't skip to a generic quiz. Suggestions must come from what you actually found in the code.
- **Scope questions to the chosen focus areas.** Don't ask about areas the user didn't select unless they said "all".
- **Respect the chosen question format.** If the user picked a single format (MC, FW, OD, SC), use only that type. In Mixed mode, let the concept guide the choice -- MC for terminology, ordering for flows/pipelines, scenario for failure modes, free-write for architecture and reasoning. Broad strokes leans MC; rigorous leans scenario and free-write.
- **Match code detail to chosen abstraction level.** Conceptual = no specific names in questions or explanations; Named = use and expect real class/method/service names, correct wrong terminology; Code-level = reference methods and show snippets in explanations when they clarify.
- **Care about terminology at Named and Code-level.** If the user says "handler" when they mean "controller", correct it. At Conceptual level, focus on whether the concept is right, not the name.
- **Free-write questions should require articulation**, not just yes/no recall.
- **Reinforcement follow-ups are mandatory** after wrong answers. Don't skip them.
- **Never quiz on file paths or line numbers.** Architecture, patterns, data flow, and terminology only.
