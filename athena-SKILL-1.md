---
name: athena
description: >
  GitHub repository discovery and deep analysis skill. Use this skill whenever
  the user wants to find, explore, compare, or evaluate GitHub repositories.
  Triggers on: "find me a GitHub repo", "search GitHub for", "what's a good
  library for", "recommend a package for", "find open source tools for",
  "what repos exist for", "GitHub alternatives to X", "find a Python/JS/Rust
  library that does X", "compare GitHub repos for X", "best starred repos for",
  "is there an open source X", or any request to discover or evaluate code
  libraries, frameworks, tools, or projects. Even casual phrasing like "know of
  any good repos for X?" or "what do people use for X in Python?" should trigger
  this skill. Use it proactively — if the user is asking about tooling choices
  or what library to use, this skill applies.
---

# Athena — GitHub Repository Discovery Skill

Athena searches GitHub for repositories matching the user's intent, reads
README content to deeply verify relevance, and produces an opinionated
recommendation to help the user pick the right tool.

---

## Workflow

### Step 1 — Understand intent

Before searching, extract:
- **Core need**: what problem does the repo need to solve?
- **Language/ecosystem**: Python, JS, Rust, etc. (infer from context if not stated)
- **Constraints**: license requirements, async support, no heavy deps, etc.
- **Depth wanted**: quick recommendation, or full comparison?

Default to **deep dive** when the user seems to be making a real tooling decision.

---

### Step 2 — Search via web_search

Craft 2–3 varied queries to surface good candidates:

```
github <topic> <language> stars
github best <topic> library <language>
<topic> python library comparison OR benchmark site:github.com
```

Collect 6–12 unique candidate repos from results. Prefer results that show
star counts or explicit repo URLs (`github.com/owner/repo`).

**Examples:**
- "scraping library Python" → `github python web scraping library stars`, `playwright scrapy beautifulsoup python comparison`
- "fast REST framework" → `github python REST API framework benchmark`, `fastapi starlette flask comparison stars`

---

### Step 3 — Read READMEs for top candidates

For **4–6 top candidates**, fetch the GitHub repo page using `web_fetch`:

```
https://github.com/{owner}/{repo}
```

**Do NOT** attempt `raw.githubusercontent.com` directly — those URLs are often
blocked unless they appeared in a prior search result. Fetch the GitHub repo
page instead; it contains the rendered README, star count, language, license,
and last-updated info all in one request.

Use `text_content_token_limit: 1000` per fetch to keep things fast.

From each page extract:
- Tagline / first README paragraph (what it actually does)
- Install command
- Key features or "why use this" section
- License type (MIT, Apache, GPL, AGPL — matters for commercial use)
- Whether it's a fork or original repo
- Star count and last commit/release date
- Any benchmarks or comparisons mentioned

**Quick scan mode** (user wants a fast answer): fetch at least 2 READMEs —
the top result and one alternative. Never skip entirely; README verification
is the core value of this skill.

---

### Step 4 — Score each repo

| Signal | What to look for |
|--------|-----------------|
| ⭐ Stars | Raw popularity — note the count |
| 🔄 Activity | Last commit/release date; are issues being responded to? |
| 📖 Docs | Clear README with examples? Dedicated docs site? |
| 🎯 Relevance | Does it actually solve what the user needs, per the README? |
| 🧩 Ease of use | Simple API? Good defaults? Low dependency count? |
| ⚖️ License | MIT/Apache = permissive; GPL/AGPL = copyleft (may affect commercial use) |
| 🍴 Fork status | Is this a fork? If so, is it active or just a dead mirror? |
| ⚠️ Red flags | No commits in 2+ years, no license, open security issues |

---

### Step 5 — Produce the report

```
## 🔍 GitHub Repository Search: [topic]

**Searches run:** N | **Candidates evaluated:** N | **READMEs read:** N

---

### 🏆 Top Picks

#### 1. [owner/repo](https://github.com/owner/repo) ⭐ [Xk stars] · [License] · [Language]
**What it does:** [1-sentence summary from README]
**Best for:** [specific use case]
**Install:** `pip install X`
**Strengths:**
- [concrete point from README]
- [concrete point]
**Watch out for:** [caveat — be specific, e.g. "AGPL license", "no Windows support", "last release 2021"]

#### 2. [owner/repo](...) ⭐ ...
...

---

### 📊 Comparison

| Repo | Stars | License | Last Active | Async | Best For |
|------|-------|---------|-------------|-------|---------|
| owner/repo | 12k | MIT | 2025 | ✅ | General use |
| owner/repo | 4k | Apache | 2024 | ❌ | Lightweight |

---

### 💡 Recommendation

[Be direct. Pick one. Explain the tradeoff in 2–3 sentences.]

For [user's use case], use **owner/repo** because [specific reason from README/signals].
If you need [specific constraint instead], **other/repo** is the better fit because [reason].
```

---

## Important notes

- **Be opinionated**: end with a clear verdict. Don't just list — recommend.
- **Stars ≠ best**: a 600-star niche library that fits perfectly beats a 30k-star
  general one. Say so.
- **Flag dead projects prominently**: no commits in 2+ years = lead with that.
- **License matters**: if the user is building a commercial product, flag GPL/AGPL
  upfront — it's a blocker many developers miss.
- **Fork awareness**: a repo with 500 stars that's a fork of a 50k-star project
  with no meaningful changes is not a real recommendation.
- **Language/ecosystem respect**: don't recommend a Go library to a Python dev
  unless there is genuinely no Python equivalent worth using.
- **Honest about gaps**: if search returns few quality candidates for a niche topic,
  say so rather than padding with irrelevant results.
