# Athena — GitHub Repository Discovery Skill for Claude

A Claude skill for **searching, deep-scanning, and comparing GitHub repositories** — inspired by the [Athena](https://huggingface.co/spaces/aeras/athena) HuggingFace Space by aeras.

## What it does

Athena helps you find the right open-source tool for any job. It goes beyond a simple GitHub search by:

- **Searching GitHub** with multiple query variations to surface the best candidates
- **Fetching and reading READMEs** to verify that repos actually do what they claim
- **Scoring each repo** on stars, activity, docs quality, relevance, and ease of use
- **Giving an opinionated recommendation** — not just a list, but a clear "use this one" verdict with reasoning

## Installation

1. Download [`SKILL.md`](./SKILL.md)
2. In Claude.ai, go to **Settings → Skills** and upload the file
3. Claude will now use this skill automatically when you ask it to find or compare repos

## Triggers

Claude will use this skill when you say things like:

- "Find me a GitHub repo for X"
- "What's the best Python library for X?"
- "Compare open source tools for X"
- "Is there a good Rust crate that does X?"
- "What do people use for X in JavaScript?"
- "GitHub alternatives to [paid tool]"
- "Help me choose between [repo A] and [repo B]"

## How it works

The original Athena app used the GitHub REST API + README keyword matching. This skill adapts that approach for Claude:

| Original Athena | This Skill |
|----------------|------------|
| GitHub API search | `web_search` with targeted GitHub queries |
| README keyword check | Full README fetch + semantic understanding |
| Match status labels | Scored quality assessment across 6 signals |
| Results table | Comparison table + opinionated recommendation |

## Example output

```
## 🔍 GitHub Repository Search: async web scraping Python

Searched for: python async web scraping | Candidates evaluated: 6 | READMEs read: 5

### 🏆 Top Picks

#### 1. microsoft/playwright-python ⭐ 11.2k
What it does: Reliable end-to-end testing and scraping for modern web apps
Best for: JavaScript-heavy sites, SPAs, sites requiring interaction
Install: pip install playwright
Strengths:
- Handles JS rendering natively
- Auto-wait and browser context management
- Actively maintained by Microsoft

#### 2. scrapy/scrapy ⭐ 51k
...

### 💡 Recommendation
For async scraping of JS-heavy sites, go with playwright-python — it handles
modern SPAs that tools like BeautifulSoup can't touch. If you're scraping
static HTML at scale, scrapy is the battle-tested choice with better
built-in pipeline support.
```

## Notes

- Works best for programming tools, libraries, frameworks, and CLIs
- Results depend on web search availability
- For very niche topics, Claude may find fewer candidates — it will say so rather than hallucinating results
