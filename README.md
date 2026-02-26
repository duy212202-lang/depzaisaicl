# UGC Comment Insights Miner

> **AI-Powered Content Enhancement Skill** — Transform messy comment threads into structured, ready-to-use blog sections.

## The Problem

Content writers spend 1-2 hours per article hunting for community insights. Most skip this step entirely — articles end up generic, with no real human voice. Reddit accounts for ~21% of citations in Google AI Overviews. Content enriched with real UGC gets significantly more visibility than theory-only articles.

## What This Skill Does

Paste URLs or comment text → skill auto-processes → outputs a complete insights report in under 2 minutes.

**5 output sections:**
1. Counter-Arguments & Alternative Perspectives
2. Real-World FAQs (from actual community questions)
3. Personal Experiences & Mini Case Studies
4. Viral Pattern Analysis (why top comments resonated + writing tips)
5. Blog Cross-Reference (optional — gap detection + insertion mapping)

## Quick Start

### Option A: Paste URL(s)
```
Mine the comments from these threads for my blog:
- https://reddit.com/r/programming/comments/xxx/...
- https://old.reddit.com/r/webdev/comments/yyy/...
```

### Option B: Paste Comment Text (when platform blocks fetching)
```
Here are comments from an X thread about [topic]. Mine these for insights:

@user1 (245 likes): "Comment text here..."
@user2 (1.2K likes): "Another comment..."
[... paste all comments]
```

Both options produce the same quality output.

## Before vs. After

| Metric | Before (Manual) | After (With Skill) |
|---|---|---|
| Time to gather insights | 1-2 hours | 1-2 minutes |
| Comments reviewed | ~30 (skimmed) | 100+ (systematic scoring) |
| Insight quality | Hit or miss | Scored 0-10, ranked by engagement |
| Viral analysis | None | Automated pattern detection |
| Blog cross-reference | Manual re-reading | Automated gap detection |

## Repo Structure

```
depzaisaicl/
├── SKILL.md          — Main skill instruction file (Claude Project Instructions)
├── spec.md           — Full specification document
├── skill-card.md     — One-page Skill Card reference
└── README.md         — This file
```

## How to Use

1. Create a new Claude Project
2. Copy the contents of `SKILL.md` into the Project Instructions
3. Start a conversation — paste URLs or comment text
4. Receive your insights report, copy-paste into your blog

## Limitations

- X/Twitter and Reddit often block direct URL fetching → use paste workaround
- JavaScript-rendered comment sections may not load via web_fetch
- Maximum 100 comments processed per URL
- No scheduled monitoring or CMS integration

---

*An open-source AI skill for content enrichment and community insights mining.*
