---
name: find-github
description: Find open-source code to build on instead of writing from scratch — either a repo to start from, or a single piece to study and adapt for one feature. Use when adding a specific component or behavior to an existing project (loading animation, drag-and-drop, resizable sidebar, command palette, virtual scrolling, file tree, rich text, chart, upload, auth, payments, PDF parsing, OCR, scraping); when starting a new project, prototype or MVP; or when the user says "is there a library for X", "how do people build X", "don't reinvent this", "找个开源的", "参考一下开源项目". Do NOT use for bugfixes, refactors, tests, config, copy changes, or when a library is already chosen.
license: MIT
---

# Find & Build On Open-Source Code

Most of the time this is not "here's a big idea, go find a whole repo." It's "this one small thing needs to exist" — a loading animation, a drag handle, a resizable sidebar — in a project that's already underway. The job is to find how people have solved it, take the technique, and land working code fast.

Two entry modes, and the first is by far the more common:

- **Component** — one feature or behavior inside an existing codebase.
- **Greenfield** — a whole app, prototype or MVP from scratch.

The deliverable is always working code in the user's project. Never a page of search results.

## 0. Decide whether to search

Fire when:

- A **new capability** is being added — not a change to something that exists. The tell: you're about to write a self-contained chunk (a component, an interaction, a parser, an animation) rather than edit a line or two.
- The problem is recognizable and probably solved before: drag-and-drop, virtual scrolling, file upload, auth, payments, OCR, PDF parsing, charts, editors, loading states, resizable panes.
- Nothing is already chosen — check the manifest and the conversation first.

Stay out when: bugfix, refactor, test, config, copy or styling tweak, rename, wiring up an existing dependency, pure conceptual question, "from scratch"/"no dependencies"/"clean room", or a library is already installed.

Also stay out when the chunk is trivial — 5–10 lines you already know how to write. Searching costs more than writing there.

If the user asked explicitly, always search. If you triggered on your own, go quiet (see **Present**) — looking is cheap and needs no permission.

## 1. Frame the need — a few lines, no tool calls

**Component:** the exact behavior in one sentence ("sidebar edge is draggable to resize; collapses to a 3-bar rail below a threshold"), the host stack and constraints (framework, version, styling approach, bundle budget, "no new deps"), and what done looks like.

**Greenfield:** what's being built, the stack, the hardest part, constraints.

Search for the **hard part**, not the whole product. If it's genuinely ambiguous, ask one question; otherwise infer and proceed.

## 2. Decide: study, adapt, or depend

This is the decision most agents get wrong. For a small self-contained piece, the cheapest good outcome is usually **read two or three implementations and write your own** — no dependency, fits the codebase, maybe 60 lines. That beats pulling in a package, and often beats writing it cold.

| The need is… | Do this | Cost |
|---|---|---|
| Small and self-contained, a few hundred lines at most | **Study & reimplement** | a few file reads |
| A technique you don't know, but a small surface | **Study & reimplement**, credit the source | a few file reads |
| Bigger than you want to own, actively maintained | **Vendor** the relevant part | copy + adapt |
| A standard, well-scoped problem (auth, payments, charts) | **Depend** on the library | install |
| Whole-app starting point | **Fork** | clone + strip |

Default to the cheapest row that works. A dependency for a loading spinner is a bad trade; hand-rolling a PDF parser is worse. If you can't tell which row applies yet, search first — the size and maturity of what exists makes it obvious.

## 3. Query ladder

Run 2–4 queries. Climb until the results look right. Rate limit is 10/min unauthenticated, 30 authenticated.

**Component / technique:**

| # | Pattern | Example |
|---|---|---|
| 1 | `<framework> <behavior>` | `react resizable sidebar` |
| 2 | Platform + effect | `css loading spinner animation` |
| 3 | The pattern's own name | `command palette`, `virtual scrolling` |
| 4 | `topic:` or curated lists | `topic:drag-and-drop`, `awesome css animations` |

**Greenfield:**

| # | Pattern | Example |
|---|---|---|
| 1 | `<product> alternative` / `open source <product>` / `<product> clone` | `open source notion alternative` |
| 2 | Capability phrase | `pdf table extraction` |
| 3 | Capability + `topic:` or `language:` | `scraping language:typescript` |
| 4 | `<stack> starter` / `boilerplate` / `template` | `nextjs saas starter` |

For a component, the target is a **file**, not a repo. A 300-file app containing the exact 80-line component you need is a hit — judge the file.

Always qualify with `stars:>100 pushed:>DATE archived:false in:name,description,readme` (DATE ≈ 12 months back — compute it). Drop `in:...` if results are thin; raise `stars:` if they're noise. For niche techniques `stars:>20` finds more.

## 4. Search

**Repos** — preferred form, if `gh` holds a token:

```bash
gh api -X GET search/repositories \
  -f q='<terms> stars:>100 pushed:>2025-06-01 archived:false' \
  -f sort=stars -F per_page=15 \
  --jq '.items[] | [.full_name, .stargazers_count, .pushed_at[:10], (.license.spdx_id // "none"), .description] | @tsv'
```

Unauthenticated fallback:

```bash
curl -sG https://api.github.com/search/repositories \
  --data-urlencode "q=<terms> stars:>100 pushed:>2025-06-01 archived:false" \
  -d sort=stars -d per_page=15
```

**Code** — when you want to see how a thing was built, not which repo it lives in (needs auth):

```bash
gh api -X GET search/code -f q='useResizable language:typescript' \
  -f per_page=10 --jq '.items[] | [.repository.full_name, .path] | @tsv'
```

Code search supports `repo:owner/name`, `path:`, `filename:`, and `language:` — but **not** `stars:`, so it can't filter by popularity on its own. To stay on well-known repos, pick a repo from the repository search first and scope with `repo:owner/name`, or look for a distinctive symbol name (`filename:useResizable.ts` beats describing the behavior in prose).

If rate-limited or empty, fall back to a web search for `site:github.com <terms>`, or fetch `https://github.com/topics/<topic>`. Use whatever search and fetch capability your agent has; the curl commands need no special tooling.

## 5. Shortlist

Gate — drop anything that fails:

- Archived, or no push in 12 months.
- No license at all (note it; see **Build on it**).
- Not code: `awesome-*` lists, docs-only repos, paper repos. Lists are lead sources — mine them for names.
- Wrong stack when the stack is fixed — unless it's language-agnostic.

Score survivors 0–2 on Fit / Alive / Adoptable / Footprint. For a component, **Fit and Footprint dominate**: a 400-star focused implementation beats a 30k-star app you'd have to excavate.

Then apply judgment:

- Stars are a lagging indicator. A repo created two months ago with 8k stars and no release is hype — check `created_at`.
- A 2k-star repo still committing beats a 30k-star repo abandoned in 2024.
- Building an app? Prefer starters over frameworks. Adding a capability? Prefer a library or a file over an app you'd have to strip.
- Recent *answered* issues mean the thing works and someone is home.

## 6. Read the technique, not the project

For each shortlisted candidate, read the one to three files that matter:

```bash
curl -sL https://raw.githubusercontent.com/OWNER/REPO/HEAD/PATH | head -150
```

Locate the file via code search, the repo's contents listing, or a path in its docs. You're after the **mechanism** — the CSS trick, the event math, the state shape — plus what it assumes (deps, framework version, browser quirks, known edge cases from issues).

Do not clone a repo to read one file. Do not read a repo end to end.

## 7. Present

**Mid-project (the common case)** — two or three lines, then act:

> Found three implementations. The cleanest is a `transform` + threshold check in `owner/repo` (~70 lines, no deps). Adapting that approach — no dependency needed.

Don't stop for approval to *look* or to reimplement. Stop only when the choice is expensive: adding a dependency, vendoring a large chunk, or a license question.

**Explicit ask or greenfield** — compact list, ≤12 lines, one recommendation, name the runner-up:

```
1. owner/repo  ★12.3k · pushed 2026-09 · MIT
   Fits: <one line — the hard part it covers>
   Catch: <one line — the real cost of adopting it>
```

State the exit condition out loud: "nothing here is a good fit — build it" is a valid and useful answer. Never pad the list to look thorough.

## 8. Build on it

| Mode | How |
|---|---|
| **Reimplement** | write it into the codebase in the host project's style; credit the source in a one-line comment when the technique is distinctive; never paste code you don't understand |
| **Vendor** | copy the part plus its license header, adapt |
| **Depend** | install, wire it up, copy no code |
| **Fork** | `git clone --depth 1`, `rm -rf .git`, strip, rename |

License rules — check before copying, and tell the user:

- MIT / Apache-2.0 / BSD / ISC: keep `LICENSE`, add attribution in the README. Fine.
- MPL-2.0: fine for vendored files if their headers stay intact.
- GPL / AGPL: the derivative must be open-sourced (AGPL reaches network services). Flag it; prefer reimplementing.
- No license: all rights reserved. Do not copy — reimplement from understanding, or ask the author.

Before trusting a repo, read its `package.json` scripts / `Makefile` / setup steps. Don't run install or build in a freshly cloned stranger repo without looking — install scripts execute code. `npm ci --ignore-scripts` to inspect first.

Then strip what isn't needed and write the delta as the user's own code.

## Failure modes

- Treating a one-component request as a whole-product request.
- Cloning a repo to answer a 60-line question.
- Adding a dependency for something three file reads would have covered.
- Reading a whole repo to extract one file.
- Copying code you don't understand.
- Returning a search page instead of working code.
- Ranking by stars alone.
- Adopting copyleft code without saying so.
- Firing on a typo because the word "extract" appeared in the prompt.
