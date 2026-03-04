# Plan-Driven Dev -- Claude Code Skill

A Claude Code skill that enforces a disciplined **Research > Plan > Annotate > Implement** workflow for building apps and features. Prevents Claude from writing code before you've reviewed and approved a written plan.

Based on the workflow described by [Boris Tane](https://boristane.com) in his excellent blog post: **[How I Use Claude Code](https://boristane.com/blog/how-i-use-claude-code/)**. Boris has been using Claude Code as his primary development tool for ~9 months and developed this methodology that separates planning from execution. This skill automates enforcement of that workflow so you don't have to remember the steps yourself.

## Why?

Most AI coding workflows go: prompt > code > fix errors > repeat. The results fall apart for anything non-trivial.

This skill enforces one core rule: **never let Claude write code until you've reviewed and approved a written plan.** This separation of thinking and typing prevents wasted effort, keeps you in control of architecture, and produces significantly better results.

## What It Does

When you ask Claude Code to build something, instead of jumping straight to code, it will:

1. **Research** -- deeply read the relevant codebase, write findings to `research.md` for your review
2. **Plan** -- write a detailed `plan.md` with approach, code snippets, file paths, trade-offs
3. **Annotate** (1-6 rounds) -- you add inline notes to `plan.md`, Claude addresses them, repeat until satisfied
4. **Todo List** -- granular task breakdown with checkboxes added to `plan.md`
5. **Implement** -- only after your explicit go-ahead, marking tasks complete as it goes
6. **Feedback** -- handle your corrections during implementation

The skill triggers automatically on keywords like "build", "implement", "create", "add feature", "new project", etc.

## Install

Skills in Claude Code are filesystem-based. Place the skill directory where Claude Code can discover it. Pick the scope that fits your needs:

### Option 1: Personal skill (global -- works across all your projects)

```bash
git clone https://github.com/makowskid/plan-driven-dev.git ~/.claude/skills/plan-driven-dev
```

### Option 2: Project skill (shared with team via version control)

```bash
git clone https://github.com/makowskid/plan-driven-dev.git .claude/skills/plan-driven-dev
```

Then commit `.claude/skills/plan-driven-dev/` to your repo so the whole team gets the skill automatically.

### Option 3: Manual

Download `SKILL.md` from this repo and place it at:

```
~/.claude/skills/plan-driven-dev/SKILL.md
```

## Verify Installation

Start Claude Code and type `/` to open the skill menu. You should see `plan-driven-dev` in the list. You can also invoke it directly:

```
/plan-driven-dev
```

Or just ask Claude to build something:

```
Build a REST API for user management
```

Claude should start with the research phase instead of jumping to code.

## The Workflow Visualized

```
Research --> Plan --> Annotate (repeat 1-6x) --> Todo List --> Implement --> Feedback
```

**Research:** Claude reads the codebase deeply, writes `research.md`. You review and confirm it's accurate.

**Plan:** Claude writes `plan.md` with detailed approach, actual code snippets, file paths. You open it in your editor.

**Annotate:** You add inline notes directly into `plan.md` -- corrections, rejections, constraints, domain knowledge. Claude addresses every note. Repeat 1-6 times until the plan is right.

**Todo List:** Granular task breakdown with checkboxes. Your final review before implementation.

**Implement:** Claude executes the approved plan mechanically. Marks tasks as done. No creative decisions -- those happened in annotation.

**Feedback:** Short corrections during implementation. "wider", "wrong file", "move it to the admin app".

## When to Skip or Shorten

Not every task needs the full workflow:

- **Trivial changes** (fix a typo, change a color): skip everything, just do it
- **Small contained fixes** (obvious bug): skip research, minimal plan
- **Medium features** (new endpoint, new page): full workflow, 1-2 annotation rounds
- **Large features / new systems**: full workflow, 3-6 annotation rounds

## Update

If installed via git clone:

```bash
cd ~/.claude/skills/plan-driven-dev && git pull
```

## Uninstall

```bash
rm -rf ~/.claude/skills/plan-driven-dev
```

For project-level installs, remove `.claude/skills/plan-driven-dev/` from your repo and commit.

## Credits

This skill is an implementation of the workflow described by [Boris Tane](https://boristane.com) in his blog post **[How I Use Claude Code](https://boristane.com/blog/how-i-use-claude-code/)** (Feb 2026). The core ideas -- research-first approach, plan.md as shared mutable state, the annotation cycle, and "don't implement yet" guards -- are all his. This skill just packages them into an auto-enforcing Claude Code skill so you get the workflow without having to remember the steps.

Go read the [original post](https://boristane.com/blog/how-i-use-claude-code/) -- it's worth your time. Also check out what Boris is building at [nominal.dev](https://nominal.dev).

## License

MIT License. See [LICENSE](LICENSE).
