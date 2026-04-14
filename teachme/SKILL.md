---
name: teachme
description: Use when the user wants to learn about a codebase area, feature, file, PR, or architectural concept through interactive guided exploration
---

# TeachMe - Interactive Codebase Exploration

## Overview

Guided, opinionated teaching through interactive conversation. Always starts top-down (big picture first), presents information in short bursts with branching options, and actively calls out what's strong, weak, unusual, or missing. The user steers depth and direction -- a "choose your adventure" through code.

## Invocation

- `/teachme` -- infer topic from current context (branch, conversation, files touched)
- `/teachme <concept>` -- e.g., `/teachme execution engine`
- `/teachme <file-path>` -- e.g., `/teachme packages/cli/src/services/execution.service.ts`
- `/teachme <PR>` -- e.g., `/teachme #4521` or a GitHub PR URL

## Input Detection

| Input | Detection | Entry Point |
|-------|-----------|-------------|
| No argument | Infer from branch/context, confirm with user | Big picture of inferred area |
| Concept | No path prefix, no `#` or URL | Big picture of that system/pattern |
| File path | Path-like string, verify exists | Big picture of the system the file belongs to, then narrow to the file |
| PR | `#number` or GitHub URL, fetch via `gh` | Big picture of area touched, then what changed and why |

## Process

```dot
digraph teachme {
    "Invoked" -> "Detect input type";
    "Detect input type" -> "No argument?" [shape=diamond];
    "No argument?" -> "Infer topic, confirm with user" [label="yes"];
    "No argument?" -> "Read relevant code" [label="no"];
    "Infer topic, confirm with user" -> "Read relevant code";
    "Read relevant code" -> "Generate syllabus + write to file";
    "Generate syllabus + write to file" -> "Present big picture + text graph";
    "Present big picture + text graph" -> "Offer 2-4 branches";
    "Offer 2-4 branches" -> "User picks branch or asks question";
    "User picks branch or asks question" -> "Read more code as needed";
    "Read more code as needed" -> "Teach chunk (2-3 paragraphs)";
    "Teach chunk (2-3 paragraphs)" -> "Update syllabus progress";
    "Update syllabus progress" -> "Deep in a subtopic?" [shape=diamond];
    "Deep in a subtopic?" -> "Suggest continuing syllabus" [label="yes, 3+ levels deep"];
    "Deep in a subtopic?" -> "Offer new branches" [label="no"];
    "Suggest continuing syllabus" -> "User decides" [shape=diamond];
    "User decides" -> "Offer new branches" [label="stay deep"];
    "User decides" -> "Next uncovered syllabus topic" [label="continue syllabus"];
    "Next uncovered syllabus topic" -> "Read more code as needed";
    "Offer new branches" -> "User picks branch or asks question";
}
```

### Step 1: Determine Topic

**No argument:** Check current branch name, recent files, discussion context. State inference: "It looks like you're working on **X**. Want me to teach you about that area?" Wait for confirmation.

**With argument:** Detect type (concept, file, PR) and proceed.

### Step 2: Read Code and Build Understanding

Read relevant packages, services, dependencies, and callers. For PRs, also fetch via `gh pr view` and `gh pr diff`. Build a mental model of:
- Component boundaries and responsibilities
- Key dependencies and data flow
- Design patterns in use
- Gaps in the design (missing validation, error handling, tests, etc.)

### Step 3: Generate Syllabus

After reading the code, generate a syllabus covering all key parts of the topic. Write it to `.claude/teachme/<topic-slug>.md` (e.g., `.claude/teachme/execution-engine.md`). Derive the slug from the topic name -- lowercase, hyphens, no special characters. The syllabus serves two purposes: ensuring comprehensive coverage, and letting the user resume across sessions.

**Syllabus format:**

```markdown
# TeachMe: [Topic Name]
Started: [date]

## Syllabus

- [ ] **1. Big Picture** -- architecture overview, where it fits, key components
- [ ] **2. Core Data Flow** -- how data moves through the system
- [ ] **3. Key Component: X** -- responsibilities, patterns, dependencies
- [ ] **4. Key Component: Y** -- responsibilities, patterns, dependencies
- [ ] **5. Error Handling & Edge Cases** -- recovery paths, failure modes
- [ ] **6. Design Gaps & Weaknesses** -- what's missing, what's fragile
- [ ] **7. Testing Strategy** -- what's covered, what's not

## Progress Notes
<!-- Updated as topics are covered -->
```

**Syllabus rules:**
- Generate 5-10 items based on actual code complexity. Don't pad with filler.
- Each item should be a meaningful learning unit, not a file or class name.
- Order items top-down: big picture first, details later, gaps/critique last.
- Check off items as they are covered during the session. Add brief notes about what was discussed.
- If a syllabus already exists for this topic (check `.claude/teachme/` for matching slug), read it and resume from where the user left off. Ask: "We have an existing session on **X** -- want to continue where you left off, or start fresh?"
- On first invocation, list any existing sessions in `.claude/teachme/` so the user knows what's available.

### Step 4: Present Big Picture

Always start top-down, regardless of input type. Include:
- **2-3 paragraphs** explaining what this area is, where it fits, and its key responsibilities
- **A text graph** showing key components and their relationships (5-8 nodes max)
- **Opinionated observations** -- call out what's notable right away

Check off the "Big Picture" syllabus item. Then offer 2-4 branches to explore deeper.

### Step 5: Interactive Exploration

After each chunk, offer branches like:

> Want to explore:
> - **A) How `ExecutionService` delegates to node runners** -- this is where the complexity lives
> - **B) The error handling strategy** -- there's a gap here, no recovery path for partial failures
> - **C) What calls this service** -- the controller layer above

Repeat: user picks (or asks a free-form question) -> read more code -> teach chunk -> update syllabus -> offer new branches.

### Step 6: Progress Indicator

After each teaching chunk, include a small progress line before the branches:

> `[3/7 covered]` -- you've explored the big picture, core data flow, and error handling so far.

**Progress rules:**
- Show as `[X/Y covered]` where X is checked-off syllabus items and Y is total.
- Keep it to one line -- informational, not a celebration.
- Include it every message so the user always knows where they stand.
- When most items are covered (e.g., 6/7), mention it: "Almost there -- just **Testing Strategy** left if you want to round it out."

### Step 7: Syllabus Nudges

When the user has gone 3+ exchanges deep into a subtopic, gently suggest returning to the syllabus:

> "We've gone deep into error handling internals. There are still a few uncovered areas on the syllabus -- **Core Data Flow** and **Testing Strategy**. Want to keep going here, or jump to one of those?"

**Nudge rules:**
- Only suggest, never force. If the user wants to stay deep, stay deep.
- Only nudge once per deep dive. Don't nag.
- Frame it as an option, not a redirect: "There's more on the syllabus when you're ready."
- After any syllabus item is substantially covered (even through tangential exploration), check it off.

## PR-Specific Behavior

When input is a PR:

1. **Big picture** -- what area this PR touches, why it matters, text graph of affected components
2. **What changed and why** -- walk through the diff by logical concept (not file-by-file), explaining intent behind each change
3. **Tradeoffs and alternatives** -- proactively surface: what's strong, what's risky, what alternatives existed, what's missing
4. Offer branches as normal (explore surrounding architecture, dig into a specific change, examine test coverage, etc.)

## Opinionated Guidance

The skill is not a neutral narrator. It actively calls out:

- **Strengths:** "This is particularly well-designed -- the separation between X and Y means..."
- **Weaknesses:** "This is fragile -- if Z changes, this breaks silently because..."
- **Unusual patterns:** "This is atypical for this codebase -- most services use X but this one uses Y because..."
- **Design gaps:** "There's no validation between these two layers", "This error path is unhandled", "No tests cover this branch"
- **Missing pieces:** "This works for the happy path but doesn't account for X", "There's no retry/recovery mechanism here"

When offering branches, mark what's worth attention: "this is where the complexity lives", "this has a known gap", "this is the interesting part".

## Misconception Correction

If the user states something incorrect during conversation, correct it directly but respectfully:

- **Name the error:** "That's actually a mediator pattern, not observer."
- **Explain the difference:** "The difference is that a mediator coordinates between components, while observer broadcasts to subscribers."
- **Ground it in the code:** "Here, the `EventBus` acts as a central coordinator -- components don't subscribe to each other directly."

Never let incorrect terminology or understanding pass uncorrected. Corrections are teaching moments -- use them, then continue from the corrected understanding.

## Diagrams

**Big picture = always include a text graph.** When first presenting an area's architecture, include a simple text graph (5-8 nodes max) showing key components and relationships. This anchors the mental model.

**Deeper dives = prose only.** Once past the big picture, use prose with real names. Only produce additional diagrams if the user asks or if the data flow is genuinely too complex for prose.

## Code References

Use real class names, method names, and package names inline. Don't dump code blocks unless the user explicitly asks ("show me the code", "let me see that method"). Example good reference: "The `ExecutionService.run()` method orchestrates this -- it creates a `WorkflowExecute` instance, passes it the node graph, and hooks into its events."

## Concrete Over Abstract

Explanations should answer "who does what, and where" — not just "what happens." Two specific rules:

1. **Label ownership at boundaries.** When code crosses between project code, frameworks, and external APIs, show the chain with ownership at each stage. Don't say "the schema gets sanitized" — say "n8n's `sanitizeInputSchema` flattens the union before Mastra's `schema-compat` converts it to JSON Schema for Anthropic." The user's first question at any boundary is "whose code is this?" — answer it before they ask.

2. **Name the actor, not just the action.** Don't say "input is validated" — say "`credentialService.list` is called with the filtered type." Don't say "this gets cached" — say "Mastra caches the tool schema at construction time in `createTool()`." Specificity prevents follow-up questions and builds a mental model the user can navigate on their own.

When a cross-boundary flow first comes up, use a short pipeline diagram with ownership labels:

```
project code (Zod schema)
    ↓
project code (sanitization shim)
    ↓
framework (Mastra converts Zod → JSON Schema)
    ↓
external API (Anthropic receives JSON Schema)
```

## Handling User Input Mid-Session

At any point the user can:
- **Pick a branch** -- follow the offered option
- **Ask a free-form question** -- answer it (2-3 paragraphs), then offer new branches from that point
- **Say "show me the code"** -- display the relevant code block with brief annotations
- **Say "alternatives?"** -- compare the current approach to alternatives with tradeoffs
- **Say "what's weak here?" / "what's missing?"** -- give an honest critique and gap analysis
- **Redirect entirely** -- "actually, teach me about Y instead" -- pivot without ceremony

User questions and redirects always take priority over any planned flow.

## Session End

No ceremony. When the user stops asking, the session is over. No proactive summary, no scorecard. The syllabus file persists at `.claude/teachme/<topic-slug>.md` with progress marked -- the user can resume in a future conversation by invoking `/teachme` on the same topic.

## Key Rules

1. **Always read actual code** before teaching. Never invent architecture or guess at implementations.
2. **Top-down always.** Even for a specific file, start with where it fits before explaining what it does.
3. **2-3 paragraphs max per chunk.** Then pause with branches. Never monologue.
4. **Real names, no code dumps.** Reference actual classes/methods/packages inline. Only show code blocks when asked.
5. **Be opinionated.** Call out strengths, weaknesses, gaps, and unusual patterns. Don't be neutral.
6. **Text graph for big picture.** Always include one when first presenting an area. Keep it simple (5-8 nodes).
7. **Correct misconceptions immediately.** Never let incorrect understanding pass. Frame corrections as teaching moments.
8. **Call out design gaps.** Missing validation, unhandled errors, absent tests, incomplete patterns -- surface these proactively.
9. **One message at a time.** Present one chunk, offer branches, wait.
10. **Questions welcome anytime.** User questions always take priority over the planned flow.
11. **Syllabus tracks coverage.** Generate on session start, persist to `.claude/teachme/<topic-slug>.md`, check off items as covered, nudge when deep. Resume from existing syllabus if one exists for the topic.
