# find-github

A Claude Code skill that finds an existing open-source GitHub project and adopts it, instead of having the agent design everything from scratch.

It fires on its own when you start a new project, prototype or MVP and the hard part probably already exists somewhere — you don't have to ask for a search.

## What it does

1. Identifies the *hard part* of what you're building (not the whole product).
2. Runs a short query ladder against the GitHub search API — `open source X alternative`, capability phrases, `topic:`, starters.
3. Gates out archived, stale, unlicensed and docs-only repos; scores the rest on fit, activity, adoptability and footprint.
4. Reads the README of the top 3 only.
5. Recommends one, names the runner-up, and states the exit condition ("nothing fits, build it yourself").
6. Adopts it in the cheapest mode that works: **depend → vendor → fork → reference**, with license handling.

## Install

Personal (all projects):

```bash
git clone https://github.com/SS1XTEEN/find-github ~/.claude/skills/find-github
```

Project-only:

```bash
git clone https://github.com/SS1XTEEN/find-github .claude/skills/find-github
```

Windows (PowerShell): clone into `$HOME\.claude\skills\find-github`.

Restart Claude Code. `/skills` should list `find-github`.

## Usage

Normally you don't invoke it — just start building:

> Build me a small internal tool to OCR scanned invoices into a table.

The agent searches, presents 2–3 candidates, and waits for you to pick before cloning.

To force it, or to get the wider list:

> Use find-github: what open-source projects could I base a self-hosted analytics dashboard on?
> 有没有现成的开源项目可以参考？

## Design notes

One Markdown file. No scripts, no dependencies.

The search itself is two curl calls, and the value isn't in the HTTP request — it's in the **query ladder** (what to search for), the **gating rules** (what to throw away), the **weighting judgments** (stars are a lagging indicator; a small active repo beats a big dead one), and the **adoption ladder** (depend before forking; never copy copyleft code silently). Those are model judgments, so a skill is the right form factor and a wrapper script would be dead weight.

The trigger policy is the other half. A skill that only fires when explicitly named solves nothing — that was the original problem. So the description carries "new project / MVP / is there a library for X" triggers plus an explicit do-not-fire list (bugfixes, refactors, small edits) to keep the noise down.

It deliberately stops before touching code. Searching is cheap and reversible; cloning is a decision, so the presentation step is a hard gate.

## Limits

- Unauthenticated GitHub search allows 10 requests/min; the skill caps itself at 2–4 queries and prefers `gh api` when a token exists.
- Star counts and `pushed_at` are point-in-time. Re-check before adopting.
- License advice here is a heuristic, not legal advice.
- Does not evaluate code quality beyond README, size and issue activity — clone and read before committing to a large repo.

## Contributing

Keep it one file. Changes should survive the question: "would a script do this better than a sentence?" If yes, it probably belongs in the failure-modes list instead.

## License

MIT
