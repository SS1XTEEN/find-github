# find-github

A portable Markdown instruction file for AI coding agents. It makes the agent find open-source code to build on — either a repository to start from, or a single technique to study and adapt for one feature — instead of writing everything from scratch.

Works with any agent that can read a file or accept a system prompt — Claude Code, Cursor, Codex, Gemini CLI, or a home-grown loop. No scripts, no dependencies, no vendor lock-in.

## The problem it solves

Most agent work is not "here is a big idea, build the whole thing." It is a project already underway that hits one small thing needing to exist:

- the page needs a loading animation, and everyone keeps writing a worse one from scratch
- the sidebar needs a draggable resize handle and a 3-bar collapse, and the agent hand-rolls it badly
- a file tree, a command palette, virtual scrolling, OAuth, PDF parsing

In those moments the agent should look at how other people solved it, take the technique, and write 60 good lines. Too often it either reinvents it poorly or never thinks to look. That is what this file fixes.

It fires on its own when that moment arrives — you don't have to ask.

## What it does

The skill walks a fixed sequence:

1. **Frame the need** — the exact behavior, the host stack, and the hard part.
2. **Search** — a short query ladder over GitHub repository search, with code search when you need the technique rather than the repo.
3. **Verify** — drop archived, stale, unlicensed and docs-only results; weigh fit, activity, adoptability and footprint. Stars alone won't do.
4. **Read the technique** — one to three files, not the whole repo.
5. **Choose how to build on it** — reimplement, vendor, depend, or fork, plus license rules.
6. **Present** — working code in the project, not a search page.

Two modes share that sequence:

**Component** — one feature inside an existing codebase. This is the common case. The target is a *file*, not a repository: a 300-file app containing the exact 80-line component you need is a hit. For a self-contained piece, *read two implementations and write your own* is usually right — no dependency, fits the codebase.

**Greenfield** — a whole app, prototype or MVP. The target is a repository you can start from.

## Install

The file is plain Markdown. Pick whichever way your agent loads instructions:

**Always-on (recommended).** Append `SKILL.md` to the file your agent reads at the start of every session — `AGENTS.md` at the repo root, or your agent's equivalent instruction file or system prompt. This is what makes it trigger without being asked.

**Skill-aware agents.** Drop it in the skills directory the agent scans:

```bash
git clone https://github.com/SS1XTEEN/find-github <your-skills-dir>/find-github
```

The YAML frontmatter at the top is how skill-aware agents decide when to load it. If your agent ignores frontmatter, nothing is lost — the body reads as a standalone instruction set.

**One-off.** Paste the body into the chat or system prompt when you want it for a single task.

## Usage

Normally you don't invoke it. Mid-project, just name the feature:

> The sidebar needs a drag handle to resize, and collapse to a 3-bar icon on narrow screens.

The agent finds a couple of implementations, adapts the approach, and delivers — no approval prompt, because looking is cheap.

Starting fresh, the same works:

> Build me a small internal tool to OCR scanned invoices into a table.

To force it, or to get the wider list:

> Use find-github: what open-source projects could I base a self-hosted analytics dashboard on?
> 有没有现成的开源项目可以参考？

## What it won't do

- Fire on a typo, a rename, or a copy change.
- Clone a repository to answer a question one file read would cover.
- Add a dependency for something three file reads would have covered.
- Copy code it doesn't understand, or copyleft code without telling you.

## Limits

- Unauthenticated GitHub search allows 10 requests/min; the skill caps itself at 2–4 queries and prefers the `gh` CLI when a token exists. Code search requires auth and cannot filter by stars.
- Star counts and `pushed_at` are point-in-time. Re-check before adopting.
- License advice here is a heuristic, not legal advice.
- Does not evaluate code quality beyond README, file size and issue activity — read before committing to a large repo.

## Contributing

Keep it one file. Changes should survive the question: "would a script do this better than a sentence?" If yes, it probably belongs in the failure-modes list instead.

## License

MIT
