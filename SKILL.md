---
name: find-github
description: >-
  Helps users find open-source GitHub code to build on — a repository to start from,
  or a technique to study and adapt for one feature. Use when the user asks
  "is there a library for X", "how do people build X", "find an open source project for X",
  "找个开源的", "参考一下开源项目", when starting a new project or MVP, or when adding a
  common component or capability (loading animation, drag-and-drop, resizable sidebar,
  command palette, virtual scrolling, auth, payments, PDF parsing, OCR, scraping).
---

# Find GitHub

This skill helps you find existing open-source code instead of writing everything from scratch, and then build on it.

Most of the time you are not looking for a whole repository. You are looking for one thing that already needs to exist inside a project that is already underway — a loading animation, a drag handle, a file tree. The job is to find how people solved it, take the technique, and land working code fast.

## When to Use This Skill

Use this skill when:

- The user asks "is there a library for X", "how do people build X", "找点开源的", "有没有现成的"
- The user wants to add a recognizable component or interaction to an existing project — a loading state, a resizable sidebar, drag-and-drop, a command palette, a file tree, virtual scrolling
- The user starts a new project, prototype, or MVP
- The user says "don't reinvent this" or "we shouldn't build this ourselves"
- You are about to write a self-contained chunk that probably exists somewhere, not edit a line or two

Do **not** use this skill for:

- Bugfixes, refactors, tests, config changes, renames, styling tweaks
- Wiring up a dependency that is already installed
- Pure conceptual questions
- "From scratch", "no dependencies", or "clean room" requests
- A 5–10 line chunk you already know how to write — searching costs more than writing

If the user asked explicitly, always search. If you triggered on your own, look quietly and report briefly — searching is cheap and needs no permission.

## Two Modes

**Component** — one feature inside an existing codebase. This is the common case. The target is a *file*, not a repository: a 300-file app containing the exact 80-line component you need is a hit.

**Greenfield** — a whole app, prototype, or MVP from scratch. The target is a repository you can start from.

## Step 1: Frame the Need

Before searching, identify:

1. **The exact behavior** — "sidebar edge is draggable to resize, collapses to a 3-bar rail below a threshold"
2. **The host stack** — framework and version, styling approach, constraints ("no new deps", bundle budget)
3. **The hard part** — the specific piece that would take longest to build

Search for the hard part, not the whole product. If the need is genuinely ambiguous, ask one question; otherwise infer and proceed.

## Step 2: Search GitHub

### Repository search

Authenticated (preferred — `gh` usually already holds a token):

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

### Code search

When you want to see how a thing was built rather than which repository it lives in (requires auth):

```bash
gh api -X GET search/code -f q='useResizable language:typescript' \
  -f per_page=10 --jq '.items[] | [.repository.full_name, .path] | @tsv'
```

Code search supports `repo:owner/name`, `path:`, `filename:`, and `language:` — but **not** `stars:`. To stay on well-known repositories, pick one from repository search first and scope with `repo:owner/name`. A distinctive symbol name (`filename:useResizable.ts`) beats describing the behavior in prose.

### Query patterns

Run 2–4 queries, not more (10 req/min unauthenticated, 30 authenticated). Climb until the results look right.

For a component:

| Query | Example |
|-------|---------|
| `<framework> <behavior>` | `react resizable sidebar` |
| Platform + effect | `css loading spinner animation` |
| The pattern's own name | `command palette`, `virtual scrolling` |
| `topic:` or curated lists | `topic:drag-and-drop` |

For greenfield:

| Query | Example |
|-------|---------|
| `<product> alternative` | `open source notion alternative` |
| Capability phrase | `pdf table extraction` |
| `<stack> starter` | `nextjs saas starter` |

Always qualify with `stars:>100 pushed:>DATE archived:false`, where DATE is about 12 months back — compute it, don't copy it blind. Drop the qualifiers if results are thin; raise `stars:` if they are noise. For niche techniques, `stars:>20` finds more.

If the API is rate-limited or empty, fall back to a web search for `site:github.com <terms>`, or fetch `https://github.com/topics/<topic>`.

## Step 3: Verify Quality Before Recommending

Do not recommend something based solely on search results. Drop anything that:

- Is archived, or has had no push in 12 months
- Has no license at all (note it — see Step 5)
- Is not code: `awesome-*` lists, docs-only repos, paper repos. Lists are lead sources — mine them for names.
- Uses the wrong stack when the stack is fixed, unless it is language-agnostic

Then check, in order:

1. **Fit** — does it actually do the hard part, or something adjacent?
2. **Activity** — commits and merged PRs in the last 3 months?
3. **Adoptability** — permissive license, docs, a clear entrypoint, real seams to hook into?
4. **Footprint** — small enough to read and adapt? Heavy dependency chain?

Be careful with stars. They are a lagging indicator: a repository created two months ago with 8k stars and no release is hype — check `created_at`. A 2k-star repository still committing beats a 30k-star one abandoned in 2024. Recent *answered* issues mean the thing works and someone is home.

For a component, **fit and footprint dominate**: a 400-star focused implementation beats a 30k-star app you would have to excavate.

## Step 4: Read the Technique, Not the Project

For each shortlisted candidate, read the one to three files that matter:

```bash
curl -sL https://raw.githubusercontent.com/OWNER/REPO/HEAD/PATH | head -150
```

Locate the file through code search, the repository's contents listing, or a path in its docs. You are after the **mechanism** — the CSS trick, the event math, the state shape — plus what it assumes: dependencies, framework version, browser quirks, and known edge cases from issues.

Do not clone a repository to read one file. Do not read a repository end to end.

## Step 5: Choose How to Build On It

This is the decision agents get wrong most often. For a small self-contained piece, the cheapest good outcome is usually **read two or three implementations and write your own** — no dependency, fits the codebase, maybe 60 lines.

| The need is… | Do this |
|--------------|---------|
| Small and self-contained, a few hundred lines at most | **Reimplement** the technique |
| A technique you don't know, but a small surface | **Reimplement**, credit the source |
| Bigger than you want to own, actively maintained | **Vendor** the relevant part |
| A standard, well-scoped problem (auth, payments, charts) | **Depend** on the library |
| A whole-app starting point | **Fork** |

Default to the cheapest row that works. A dependency for a loading spinner is a bad trade; hand-rolling a PDF parser is worse.

### License rules

Check before copying, and tell the user:

- **MIT / Apache-2.0 / BSD / ISC** — keep `LICENSE`, add attribution in the README. Fine.
- **MPL-2.0** — fine for vendored files if their headers stay intact.
- **GPL / AGPL** — the derivative must be open-sourced, and AGPL reaches network services. Flag it; prefer reimplementing.
- **No license** — all rights reserved. Do not copy. Reimplement from understanding, or ask the author.

### Safety

Before trusting a repository, read its `package.json` scripts, `Makefile`, or setup steps. Do not run install or build in a freshly cloned stranger repository without looking — install scripts execute code. Use `npm ci --ignore-scripts` to inspect first.

## Step 6: Present

**Mid-project** — two or three lines, then act:

```
Found three implementations. The cleanest is a transform + threshold check in
owner/repo (~70 lines, no deps). Adapting that approach — no dependency needed.
```

Do not stop for approval to look or to reimplement. Stop only when the choice is expensive: adding a dependency, vendoring a large chunk, or a license question.

**Explicit ask or greenfield** — a compact list, one recommendation, name the runner-up:

```
1. owner/repo  ★12.3k · pushed 2026-09 · MIT
   Fits: <one line — the hard part it covers>
   Catch: <one line — the real cost of adopting it>
```

Never pad the list to look thorough.

## Common Categories

| Category | Example Queries |
|----------|-----------------|
| UI components | loading, spinner, modal, command palette, toast |
| Interaction | drag-and-drop, resize, virtual scrolling, keyboard shortcuts |
| Data display | charts, tables, timeline, file tree |
| Documents | pdf parsing, ocr, markdown, export |
| Auth & payments | oauth, jwt, stripe, subscription |
| Infrastructure | scraping, queue, cache, cron |

## Tips for Effective Searches

1. **Search the pattern's name**, not the description — "command palette" beats "a search box that opens with a keyboard shortcut"
2. **Try alternative terms** — if "resizable" fails, try "split pane", "drag handle", "resize observer"
3. **Search for a symbol, not a concept** — `filename:useResizable.ts` finds implementations directly
4. **Read the highest-starred implementation first** to learn the shape of the problem, then find the smallest one that fits

## When Nothing Fits

Say so plainly — it is a useful answer, not a failure:

```
Nothing here is a good fit — the closest is owner/repo, but it drags in a
dependency chain you don't want. I'll build it directly.
```

Then build it yourself. Note in one line what you checked, so the user knows the search happened.
