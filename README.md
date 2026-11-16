# find-github

A skill for AI coding agents that finds open-source code to build on, instead of writing everything from scratch.

## Available Skills

### find-github

Finds an existing open-source implementation — a technique for one feature, or a repository for a whole project — then decides on its own whether to adapt it, use it as a reference, or drop it and write the code directly. The output is the working feature plus one line on where it came from, not a list of repositories to choose between.

**Use when:**

- Adding a common component or interaction — loading animation, drag-and-drop, resizable sidebar, command palette, virtual scrolling, file tree
- Starting a new project, prototype, or MVP
- Building a well-known capability — auth, payments, PDF parsing, OCR, scraping, charts
- "Is there a library for X", "how do people build X", "找个开源的", "有没有现成的"

**Three outcomes:**

- **Adapt** — the implementation fits; reimplement it, vendor the piece, or depend on the package
- **Reference** — nothing fits cleanly, but the technique is worth learning; write it yourself
- **Drop** — the code is worse than what you would write; build it directly

It stops to ask only when the choice is expensive — a new dependency, a large vendor, or a copyleft license.

## Installation

**Ask your agent to install it.** Paste this line into any agent that can run commands:

```
Install the agent skill at https://github.com/SS1XTEEN/find-github: clone it into your skills directory, or append SKILL.md to the instruction file you read at the start of every session, then confirm it is loaded.
```

**Or install it yourself.** Append `SKILL.md` to the file your agent reads at the start of every session (`AGENTS.md` or equivalent), or clone it into your agent's skills directory:

```bash
git clone https://github.com/SS1XTEEN/find-github <your-skills-dir>/find-github
```

## Usage

Skills are automatically available once installed. The agent uses this one when a relevant task is detected.

```
给网站加个粒子动画背景
```

```
Build me a small internal tool to OCR scanned invoices into a table.
```

## Skill Structure

- `SKILL.md` — instructions for the agent

## License

MIT
