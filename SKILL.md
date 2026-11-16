---
name: find-github
description: Find an existing open-source GitHub repo to build on instead of writing from scratch. Use when starting a new project, prototype or MVP; building a common capability (auth, payments, parsing, OCR, scraping, vector search, editor, dashboard, uploads, AI pipeline, "an X-like app"); or when the user says "is there a library for X", "don't reinvent this", "找个开源的", "参考一下开源项目". Do NOT use for bugfixes, refactors, tests, config, small edits, conceptual questions, or when asked to build from scratch or a library is already chosen.
license: MIT
---

# Find & Adopt Open-Source Code

Goal: ship faster by standing on something that already exists. The deliverable is a repo the user can actually build on — not a page of search results.

## 0. Decide whether to search

Search when all three hold:

- A new capability, app or prototype is being built — not an edit to existing code.
- The capability is a recognizable category or a common hard sub-problem (file upload, PDF parsing, auth, payments, vector search, an editor, charting, an "X clone" app...).
- Nothing is already chosen. Check the manifest (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`) and the conversation first.

Skip, and say nothing, when: bugfix / refactor / test / config / small feature in an existing codebase; the user said "from scratch", "no dependencies", "clean room"; a library is already installed or picked; the question is conceptual.

If the user asked explicitly, always search. If you triggered on your own, stay quiet and brief (§6 Quiet mode).

## 1. Build brief — 30 seconds, no tool calls

Write down, in one short block, before searching:

- What is being built, in the user's words.
- The stack in play (manifest, repo layout, or the conversation).
- **The hard part** — the specific thing that would take the longest to build.
- Constraints: must self-host? license-sensitive? small footprint? runs in the browser?

Search for the hard part, not for the whole product — unless the user wants a whole app.

If the hard part is genuinely ambiguous, ask one question. Otherwise infer and proceed.

## 2. Query ladder

Run 2–4 queries, not more (GitHub search: 10 req/min unauthenticated, 30 authenticated). Climb until the results look right:

| # | Pattern | Example |
|---|---------|---------|
| 1 | `<product> alternative` / `open source <product>` / `<product> clone` | `open source notion alternative` |
| 2 | Capability phrase | `pdf table extraction` |
| 3 | Capability + `topic:` or `language:` | `scraping language:typescript` |
| 4 | `<stack> starter` / `boilerplate` / `template` | `nextjs saas starter` |

For MVPs, rungs 1 and 4 pay best: a working clone or starter beats a pile of libraries.

Always qualify with `stars:>100 pushed:>DATE archived:false in:name,description,readme`, where DATE is ~12 months back — compute it, don't copy it blind. Drop `in:...` if results are thin; raise `stars:` if they're noise.

## 3. Search

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

Pipe through `jq -r '.items[] | [.full_name, .stargazers_count, .pushed_at[:10], .description] | @tsv'` when jq is available.

If the API is rate-limited or empty, fall back to WebSearch `site:github.com <terms>`, or WebFetch `https://github.com/topics/<topic>`.

## 4. Shortlist

Gate — drop anything that fails:

- Archived, or no push in 12 months.
- Actually no license (note it; see §7).
- Not code: `awesome-*` lists, docs-only repos, paper repos. Lists are a lead source — mine them for names, never recommend them as a base.
- Wrong stack when the stack is fixed — unless it's language-agnostic (a Go binary is fine for a TS app).

Score the survivors 0–2 on each:

| Axis | 2 | 0 |
|---|---|---|
| Fit | implements the hard part nearly as-is | adjacent or partial |
| Alive | commits and merged PRs in the last 3 months | stale but functional |
| Adoptable | permissive license, docs, clear entrypoint, real seams | no docs, monolith, nothing to hook into |
| Footprint | small enough to read and adapt | huge, heavy deps |

Then apply judgment:

- Stars are a lagging indicator. A repo created two months ago with 8k stars and no release is hype — check `created_at`.
- A 2k-star repo still committing beats a 30k-star repo abandoned in 2024.
- Building an app? Prefer starters/templates over frameworks.
- Adding a capability? Prefer a library over an app you'd have to strip.
- Recent *answered* issues mean the thing works and someone is home.

## 5. Deep-dive the top 3 only

Cheap reads, no cloning yet:

```bash
curl -sL https://raw.githubusercontent.com/OWNER/REPO/HEAD/README.md | head -120
curl -s https://api.github.com/repos/OWNER/REPO/contents | jq -r '.[].name'
curl -s https://api.github.com/repos/OWNER/REPO --jq '{license: .license.spdx_id, size_kb: .size, issues: .open_issues_count, pushed: .pushed_at}'
```

Answer for each: does it really do the hard part? Is there a seam to adapt (plugin API, config, a single module)? What does it drag in? What does the license force?

## 6. Present

**Wide mode** (the user asked explicitly) — compact, ≤12 lines:

```
1. owner/repo  ★12.3k · pushed 2026-09 · MIT
   Fits: <one line — the hard part it covers>
   Catch: <one line — the real cost of adopting it>

2. ...
```

Then one recommendation and one question. Name the runner-up. State the exit condition out loud: "nothing here is a good fit — build it from scratch" is a valid and useful answer. Never pad the list to look thorough.

**Quiet mode** (auto-triggered mid-build) — two lines per candidate, at most two candidates, recommendation folded into the next action. Do not block on approval for a *search*; block before §7.

## 7. Adopt

Only after the user picks. Take the cheapest mode that works:

| Situation | Mode | How |
|---|---|---|
| A library covers the need | **Depend** | install it, wire it up, copy no code |
| An app where one subsystem matters | **Vendor** | copy that directory + LICENSE in, keep the header |
| A starter matching the target | **Fork** | `git clone --depth 1`, `rm -rf .git`, keep LICENSE, rename |
| GPL/AGPL, or too entangled | **Reference** | read it, reimplement the part needed, cite the source |

License rules — check before copying, and tell the user:

- MIT / Apache-2.0 / BSD / ISC: keep `LICENSE`, add attribution in the README. Fine.
- MPL-2.0: fine for vendored files if their headers stay intact.
- GPL / AGPL: the derivative must be open-sourced (AGPL reaches network services). Flag it; prefer Reference mode.
- No license: all rights reserved. Do not copy. Recommend asking the author, or Reference mode.

Before trusting a repo, read its `package.json` scripts / `Makefile` / setup steps. Do not run install or build in a freshly cloned stranger repo without looking — install scripts execute code. Use `npm ci --ignore-scripts` to inspect first.

Then: strip what isn't needed, keep the license files, and write the delta as the user's own code.

## Failure modes

- Returning a search page. The deliverable is a decision.
- Recommending a repo whose README you haven't read.
- Ranking by stars alone.
- Cloning before the user picked.
- Adopting copyleft code without saying so.
- Firing on a bugfix because the word "extract" showed up in the prompt.
