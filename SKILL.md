---
name: plan-driven-dev
description: "Enforces a disciplined plan-driven development workflow for building new apps and features with Claude Code. Based on the Research, Plan, Annotate, Implement methodology. Use this skill whenever the user wants to build a new feature, implement a change, create a new app, refactor a system, fix a complex bug, or do any non-trivial development work. Also trigger when the user says things like 'build', 'implement', 'create', 'add feature', 'new project', 'develop', 'let\'s make', or any request that would involve writing more than a few lines of code. This skill should be used BEFORE any code is written. If you find yourself about to write implementation code without a reviewed plan.md, STOP and use this skill first."
---

# Plan-Driven Development Workflow

> Based on the workflow described by Boris Tane in
> "How I Use Claude Code" (https://boristane.com/blog/how-i-use-claude-code/)

This skill enforces a strict separation between thinking and coding. The core rule:

**NEVER write implementation code until the user has reviewed and approved a written plan.**

This is non-negotiable. It prevents wasted effort, keeps the user in control of architecture
decisions, and produces significantly better results than jumping straight to code.

## Workflow Overview

```
Research --> Plan --> Annotate (repeat 1-6x) --> Todo List --> Implement --> Feedback
```

Each phase has a specific purpose and specific artifacts. Follow them in order.

---

## Phase 1: Research

**Goal:** Deeply understand the relevant codebase before proposing anything.

**What to do:**
1. Read the relevant parts of the codebase thoroughly -- not just function signatures,
   but actual implementations, data flows, edge cases, and conventions used.
2. Write all findings into a `research.md` file in the project root (or a docs/ folder
   if one exists).
3. Present the research.md to the user for review before moving to planning.

**How to prompt yourself internally:**
- Read deeply. Do not skim. Understand how things actually work, not just what they're called.
- Look for: existing patterns, caching layers, ORM conventions, shared utilities, API contracts,
  error handling patterns, test patterns, config conventions.
- The goal is to prevent the most expensive failure mode: implementations that work in isolation
  but break the surrounding system.

**What research.md should contain:**
- Overview of the relevant system/module
- Key files and their responsibilities
- Data flow (how data moves through the system)
- Existing patterns and conventions the codebase follows
- Dependencies and integrations
- Potential gotchas or constraints discovered
- Any existing similar functionality that could be reused or extended

**Tell the user:**
> "Before I write any plan, I need to deeply understand the relevant parts of the codebase.
> I'll write my findings into research.md for you to review. This prevents me from making
> ignorant changes later."

**Wait for user confirmation** that the research is accurate before proceeding to Phase 2.
If the user points out misunderstandings, update research.md and re-confirm.

---

## Phase 2: Planning

**Goal:** Produce a detailed, reviewable implementation plan before any code is written.

**What to do:**
1. Based on the approved research, write a detailed `plan.md` file.
2. Do NOT use Claude Code's built-in plan mode. Use a real markdown file that the user
   can open, edit, and annotate in their editor.
3. Present the plan to the user for review.

**What plan.md should contain:**
- Summary of what will be built and why
- Detailed approach with rationale for key decisions
- Code snippets showing the actual planned changes (not pseudocode -- real code shapes)
- File paths that will be created or modified
- Database/schema changes if applicable
- API changes if applicable
- Trade-offs considered and why this approach was chosen
- Anything that was intentionally left out and why

**Important guidelines:**
- Base the plan on the ACTUAL codebase (from research.md), not generic patterns
- If the user provides a reference implementation (e.g., from an open-source repo), use it
  as a concrete guide rather than designing from scratch
- Include considerations for error handling, edge cases, and testing
- Keep it practical -- no over-engineering

**Tell the user:**
> "Here's the implementation plan in plan.md. Please review it in your editor.
> You can add inline notes directly into the document -- corrections, rejections,
> constraints, or domain knowledge I'm missing. Then tell me to address your notes.
> Do NOT tell me to implement yet."

---

## Phase 3: The Annotation Cycle

**Goal:** Iteratively refine the plan until the user is fully satisfied.

This is the phase where the user adds the most value. Expect 1-6 rounds of annotations.

**How it works:**
1. User reviews plan.md in their editor
2. User adds inline notes (corrections, rejections, constraints, domain knowledge)
3. User tells you to address the notes
4. You read the updated plan.md, address every single note, and update the document
5. Repeat until user says the plan is approved

**Critical rules:**
- DO NOT implement anything during this phase
- Address EVERY note the user added -- do not skip any
- After updating, explicitly tell the user the plan is ready for another review round
- If the user says "implement" or "go" or "looks good", move to Phase 4
- If the user adds more notes, stay in Phase 3

**Types of annotations to expect:**
- Correcting wrong assumptions ("this should be PATCH, not PUT")
- Rejecting approaches ("remove this section entirely, we don't need caching here")
- Adding domain knowledge ("use drizzle:generate for migrations, not raw SQL")
- Redirecting sections ("the visibility field needs to be on the list, not individual items")
- Scope trimming ("remove the download feature, I don't want this now")
- Protecting interfaces ("these function signatures should not change")
- Technical overrides ("use this library's built-in method instead of writing a custom one")

**Tell the user after each update:**
> "I've addressed all your notes and updated plan.md. Please review again.
> When you're satisfied, I'll add a todo list and then implement."

---

## Phase 4: Todo List

**Goal:** Create a granular, trackable task breakdown before implementation starts.

**What to do:**
1. Once the user approves the plan, add a detailed todo list to plan.md
2. Organize by phases and individual tasks
3. Use checkboxes that you will mark as completed during implementation
4. Present to the user for final confirmation

**Format:**
```markdown
## Todo List

### Phase 1: [Name]
- [ ] Task 1 description
- [ ] Task 2 description

### Phase 2: [Name]
- [ ] Task 3 description
- [ ] Task 4 description
```

**Tell the user:**
> "I've added a detailed todo list to plan.md. This will serve as a progress tracker
> during implementation. Ready to implement?"

Wait for explicit "go" / "yes" / "implement" from the user.

---

## Phase 5: Implementation

**Goal:** Execute the approved plan mechanically. No creative decisions -- those were made in the annotation cycles.

**When the user gives the green light, execute with these principles:**
1. Implement everything in the plan -- don't cherry-pick
2. Mark each task as completed in plan.md as you go: `- [ ]` becomes `- [x]`
3. Do not stop for confirmation mid-flow unless you hit a genuine blocker
4. Follow the project's existing code style (no unnecessary comments, no jsdoc unless
   the project uses it, match existing patterns)
5. Maintain strict typing -- no `any` or `unknown` unless the codebase already uses them
6. Run typecheck / linter / tests continuously to catch problems early
7. If something in the plan turns out to be wrong or impossible, STOP and tell the user
   rather than improvising a different approach

**Implementation should be boring.** If it's exciting, something went wrong in planning.

---

## Phase 6: Feedback & Iteration

**Goal:** Handle corrections during and after implementation with minimal friction.

**During implementation:**
- Your role shifts from architect to supervisor
- Accept terse corrections: "wider", "still cropped", "wrong file"
- When the user references existing code ("make it look like the users table"), read
  that reference before making changes
- For visual/frontend work, expect rapid short corrections

**When things go wrong:**
- If implementation goes in a wrong direction, accept reverts gracefully
- After a revert, narrow scope rather than trying to patch a bad approach
- "I reverted everything. Now just do X" is a valid and good workflow

**Do NOT:**
- Try to patch a fundamentally wrong approach -- revert and redo
- Add scope beyond what's in the plan without user approval
- Make architectural decisions that weren't in the plan

---

## Quick Reference: What to Say at Each Phase

| Phase | Say this | Wait for |
|-------|----------|----------|
| Research | "I'll deeply study the codebase and write research.md for your review." | User confirms research is accurate |
| Planning | "Here's plan.md. Add your notes, don't tell me to implement yet." | User says they've added notes OR approves |
| Annotation | "I've addressed all notes. Review again?" | User adds more notes OR approves |
| Todo List | "Todo list added to plan.md. Ready to implement?" | Explicit go-ahead |
| Implementation | "Implementing now. I'll mark tasks complete as I go." | User feedback/corrections |
| Feedback | Address corrections, continue | User says done |

---

## When to Skip or Shorten Phases

Not every task needs the full workflow. Use judgment:

- **Trivial changes** (fix a typo, change a color, update a string): Skip everything, just do it
- **Small contained fixes** (fix a specific bug with obvious cause): Skip research, do a minimal plan
- **Medium features** (new endpoint, new component, new page): Full workflow, maybe 1-2 annotation rounds
- **Large features / new systems**: Full workflow, expect 3-6 annotation rounds
- **Refactors**: Research phase is extra important, plan needs extra detail

When in doubt, do more planning rather than less. The annotation cycle is where quality comes from.

---

## Anti-Patterns to Avoid

1. **Writing code before the plan is approved.** This is the #1 failure mode. Don't do it.
2. **Skipping research for unfamiliar codebases.** You WILL make ignorant changes.
3. **Treating chat messages as the plan.** The plan must be a persistent markdown file.
4. **Improvising during implementation.** If the plan doesn't cover it, stop and ask.
5. **Adding scope during implementation.** Nice-to-haves go into a future plan, not this one.
6. **Giving verbal summaries instead of writing artifacts.** Always write research.md and plan.md.
7. **Moving to implementation after one plan draft.** At minimum, ask the user to review once.
