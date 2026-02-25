---
name: ugc-comment-insights-miner
description: >
  Mine high-value insights from community comments (Reddit, X, YouTube, forums).
  Trigger when user pastes URLs or comment text and wants to extract community insights,
  find counter-arguments, real FAQs, personal experiences, or analyze viral patterns.
  Also trigger on: "mine comments", "what are people saying", "UGC insights",
  "comment analysis", "enrich my blog with real opinions".
---

# UGC Comment Insights Miner

Turn messy comment threads into structured, ready-to-use blog sections in under 2 minutes.

## Quick Start

**Fastest workflow:**
1. Paste 1+ URLs (or paste comment text directly if the platform blocks fetching)
2. Optionally provide your blog draft for cross-referencing
3. Receive a complete insights report — copy-paste into your blog

**Example prompt:**
```
Here's a Reddit thread about AI coding tools, mine the comments for my blog:
https://reddit.com/r/programming/comments/xxx/...
```

Or if URL fetch is blocked:
```
Here are comments from an X thread about [topic]. Mine these for insights:
[paste comment text here]
```

---

## Input

### Option A: URLs (preferred)
Paste **1+ URLs** with community discussion. Skill auto-fetches and processes.

| Source | What Gets Extracted |
|---|---|
| Reddit post | Post body + all comments (top/best) |
| X (Twitter) thread | Original post + replies/quote tweets |
| YouTube video | Description + top comments |
| Blog / forum | Article body + comment section |

### Option B: Pasted Content (fallback)
If a platform blocks fetching (X and Reddit often do), user pastes comment text directly. Skill processes it the same way — extract engagement data from text if available (e.g., "245 likes", "1.2K retweets").

### Optional Fields
| Field | Default |
|---|---|
| `topic_keyword` | Inferred from content |
| `blog_draft` | Skip cross-reference |
| `language` | Same as source |
| `max_comments` | Top 100 by engagement |

### When to Ask Questions
Process immediately. Only ask when:
1. Multiple unrelated URLs + no topic keyword
2. First interaction only: "Do you have a blog draft to cross-reference?"

---

## Processing Steps

### Step 1: Fetch & Extract

For each URL: use `web_fetch` to retrieve content + comments with metadata (text, engagement, timestamps, nesting).

**If fetch fails:** Tell user: "I couldn't access comments on [URL]. Please paste the comment text directly and I'll process it." Continue with whatever was retrieved.

### Step 2: Score & Filter (0-10)

| Signal | Weight | Scoring |
|---|---|---|
| Engagement (likes/upvotes) | 30% | Top 10%=10, top 25%=7, top 50%=5, bottom=2 |
| Content length | 20% | <10w=1, 10-50=5, 50-150=8, 150+=10 |
| Contains data/numbers | 15% | Yes=10, No=3 |
| Personal experience markers | 15% | "I tried...", "In my case..."=10, None=3 |
| Substantive replies | 10% | 3+ quality=10, 1-2=6, none=3 |
| Sentiment strength | 10% | Strong=8, Neutral=4 |

**Filter:** Score 6+ = include. Score 3-5 = include only if <20 high-value. Score <3 = discard.

### Step 3: Classify into 3 Categories

Each filtered comment goes into exactly 1 category (strongest match):

**Category 1: Counter-Arguments & Alternative Perspectives**
Disagreements, challenges, different interpretations, edge cases. "Yeah but...", "This ignores..."
Output: 150-300 word prose paragraph, balanced tone. Ready to copy-paste.

**Category 2: Real-World FAQs**
Highly-upvoted questions, repeated questions, unanswered questions. "Can someone explain...", "What about..."
Output: 3-6 Q&A pairs. Answers from community + AI knowledge. Uncertain answers flagged with warning.

**Category 3: Personal Experiences & Mini Case Studies**
First-person accounts with specific results. "I used X for 3 months...", "Saved 40% after switching..."
Output: 2-4 mini cases. Each: title, paraphrased experience (50-100 words), source + engagement + viral trigger.

### Step 4: Viral Pattern Analysis

For comments scoring 7+, identify why they resonated:

| Trigger | Example |
|---|---|
| Data-backed claim | "I saved $2,400/year by..." |
| Contrarian take | "Everyone says X but actually..." |
| Personal vulnerability | "I lost everything when..." |
| Insider knowledge | "I work at [company] and..." |
| Simplification | "Think of it like..." |
| Prediction | "In 6 months, this will..." |
| Humor + insight | Funny but makes a real point |

Output: Top 3 triggers with counts, examples, and 1-2 actionable writing tips.

### Step 5: Blog Cross-Reference (only when blog_draft provided)

Compare user's blog draft against extracted comments. Produce 3 analyses:

1. **Gap Detection:** Topics/questions frequent in comments but missing from blog. List each gap with comment count.
2. **Insertion Mapping:** For each output section, suggest exact placement in blog. Reference specific H2/H3 headings. Example: "After your 'Platform Comparison' section, insert the Counter-Arguments paragraph."
3. **Conflict Detection:** Blog claims contradicted by high-engagement comments. For each conflict, quote the blog claim, summarize the community pushback, and suggest how to address it.

If blog_draft is a URL, fetch it first. If text, process directly.

### Step 6: Assemble Output

Use the exact template below.

---

## Output Template

```
UGC COMMENT INSIGHTS REPORT

EXECUTIVE SUMMARY
  Sources analyzed:    [X] URL(s) — [platforms]
  Total comments:      [X]
  High-value comments: [Y] ([Z]%)
  Categories:          [X] counter-arguments | [Y] FAQs | [Z] experiences
  Top viral trigger:   [trigger type]

SECTION 1: COMMUNITY PERSPECTIVES — NOTABLE COUNTERPOINTS

[150-300 word prose paragraph — copy-paste ready]

Key debate points:
  1. [Point] — [X] commenters | Top: [Y] likes
  2. [Point] — [X] commenters | Top: [Y] likes
  3. [Point] — [X] commenters | Top: [Y] likes

SECTION 2: FAQ — REAL QUESTIONS FROM THE COMMUNITY

Q1: [Question]
A1: [Answer]
    Source: [X] users asked | Top answer: [Y] upvotes

[... up to Q6]

SECTION 3: REAL USER EXPERIENCES

CASE 1: [Title]
  [50-100 word paraphrased experience. Never fabricate.]
  — Source: [Platform] user | [X] likes/upvotes
  — Viral trigger: [why this resonated]

[... up to CASE 4]

SECTION 4: VIRAL PATTERN ANALYSIS

Top viral triggers:
  1. [Trigger]: [X] comments — "[example]"
  2. [Trigger]: [X] comments — "[example]"
  3. [Trigger]: [X] comments — "[example]"

Actionable takeaways for your blog:
  > [Writing tip #1]
  > [Writing tip #2]

SECTION 5: BLOG CROSS-REFERENCE [only if blog draft provided]

CONTENT GAPS:
  > [Topic] — mentioned [Y] times, missing from your blog
SUGGESTED INSERTIONS:
  > After "[heading]" > add [section]
POTENTIAL CONFLICTS:
  > Your claim "[X]" challenged by [Y] comments > [suggestion]
```

---

## Business Rules

**Always:**
- Fetch URLs automatically — never ask user to paste as first option
- Preserve original meaning — never exaggerate or distort
- Include engagement metrics for every cited comment
- Flag uncertain FAQ answers with warning marker
- Anonymize all users — "a Reddit commenter", "an X user"

**Never:**
- Fabricate comments, experiences, or data not in the source
- Include memes, spam, or personal attacks — even if highly upvoted
- Output more than 6 FAQs or 4 case studies
- Skip Viral Pattern Analysis
- Process URLs without extracting engagement metrics

**Edge Cases:**
- <10 comments: Process all, add warning about limited data
- All low-quality: Return "No high-value comments found. Try a different URL."
- No comment section: Extract article only, proceed to cross-reference if blog_draft available
- Mixed languages: Process all, output in user's preferred language
- Duplicates across URLs: Deduplicate into single stronger insight

## What This Skill Does NOT Do

- Does not generate fake/AI-fabricated comments or experiences
- Does not monitor threads over time or send alerts
- Does not publish directly to any CMS or blog platform
- Does not access private/login-walled content
- Does not provide SEO keyword research or content strategy advice
