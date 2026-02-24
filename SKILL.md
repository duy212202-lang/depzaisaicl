---
name: ugc-comment-insights-miner
description: >
  Mine high-value insights from community comments on any URL (Reddit, X/Twitter, YouTube, forums, blogs).
  Use this skill whenever the user pastes 1 or more URLs and wants to extract community insights, 
  find real user opinions, enrich blog content with UGC, analyze viral comments, build FAQ sections 
  from real questions, or find counter-arguments and personal experiences from comment threads.
  Also trigger when user mentions "mine comments", "extract insights from thread", "what are people saying about",
  "find community reactions", "UGC insights", "comment analysis", "enrich my blog with real opinions",
  "viral comment analysis", or anything related to extracting value from online discussions.
  This skill fetches URLs automatically, scores comment quality, classifies insights into 3 categories 
  (counter-arguments, FAQs, real experiences), analyzes viral patterns, and optionally cross-references 
  with the user's blog draft to find content gaps.
---

# UGC Comment Insights Miner

Turn messy comment threads into structured, ready-to-use content sections for blogs and articles.

## Why This Skill Exists

Content writers waste 1–2 hours per article hunting for community insights. Most skip this step entirely, producing generic AI-slop content with no real human voice. This skill does the heavy lifting: fetch comments, filter noise, classify insights, analyze viral patterns — all in under 2 minutes. The output is ready to copy-paste into a blog draft.

Reddit alone accounts for ~21% of citations in Google AI Overviews. Content enriched with real UGC gets significantly more visibility than theory-only articles. This skill helps writers tap into that advantage systematically.

---

## Input

### Required
User pastes **1 or more URLs** containing community discussion. Process immediately — do not ask unnecessary questions.

| Supported Source | What to Extract | Notes |
|---|---|---|
| Reddit post | Post body + all comments (sorted by top/best) | old.reddit.com format works best for extraction |
| X (Twitter) thread | Original post + all replies/quote tweets | Public threads only |
| YouTube video page | Video description + top comments | Sorted by relevance |
| Blog / forum with comments | Article body + comment section | HTML comment sections |
| Facebook public post | Post + public comments | Must be publicly accessible |

### Optional (user may provide)
| Field | Description | Default |
|---|---|---|
| `topic_keyword` | Blog topic or target keyword | Infer from URL content |
| `blog_draft` | User's existing blog post (text or URL) | Skip cross-reference step |
| `language` | Output language (EN / VI / auto) | Same as source comments |
| `max_comments` | Max comments to process | Top 100 by engagement |

### When to Ask Questions (rarely)
This skill processes immediately upon receiving URLs. Only ask questions when:
1. **Multiple unrelated URLs + no topic_keyword** → "These URLs cover different topics. What's the main topic of your blog?"
2. **First interaction only** → "Do you have a blog draft to cross-reference? Paste content/URL, or I'll generate standalone sections."

After the first interaction, remember preference — do not ask again.

---

## Processing Sequence

Follow steps 1–6 in order. Do not skip any step.

### Step 1: Fetch & Extract

For each URL:
1. Use `web_fetch` to retrieve full page content
2. Extract:
   - **Original post/article body** (main content)
   - **All comments/replies** with metadata: text, engagement (likes/upvotes/retweets), author (anonymized), timestamp, nesting level (is it a reply?)
3. Store raw data for processing

**If fetch fails** (login wall, JS-rendered comments, rate limit):
→ Tell user: "I couldn't access comments on [URL]. Please paste the comment text directly and I'll process it."
→ Continue with whatever content was retrieved (article body may still be available)

### Step 2: Score & Filter Comments

Score each comment 0–10:

| Signal | Weight | Scoring |
|---|---|---|
| Engagement (likes/upvotes) | 30% | Top 10% = 10, top 25% = 7, top 50% = 5, bottom 50% = 2 |
| Content length | 20% | <10 words = 1, 10–50 = 5, 50–150 = 8, 150+ = 10 |
| Contains data/numbers | 15% | Yes = 10, No = 3 |
| Personal experience markers | 15% | "I tried...", "In my case...", "After using..." = 10, None = 3 |
| Substantive reply thread | 10% | 3+ quality replies = 10, 1–2 = 6, none = 3 |
| Sentiment strength | 10% | Strong opinion = 8, Neutral = 4 |

**Filter actions:**
- Score ≥ 6 → **High Value** — include in analysis
- Score 3–5 → Include only if fewer than 20 high-value comments exist
- Score < 3 → **Discard** (spam, memes, off-topic, too short)

Track internally: total fetched, passed filter, discarded.

### Step 3: Classify into 3 Categories

Assign each filtered comment to exactly 1 category. If a comment spans multiple categories, assign to the strongest match.

#### Category 1: Counter-Arguments & Alternative Perspectives

Comments that disagree, challenge, present different interpretations, point out edge cases or overlooked downsides. Phrases like "Yeah but...", "This ignores...", "The problem with this take is..."

**Why it matters for SEO/GEO:** Multi-perspective content signals comprehensive coverage. E-E-A-T rewards balanced, nuanced articles.

**Output format:** 150–300 word prose paragraph summarizing key counter-arguments. Tone: balanced, acknowledging different viewpoints. Structure: "[Main counterpoint]. According to discussions in the [platform] community, [elaboration with specific examples]. However, [nuance/context]."

#### Category 2: Real-World FAQs

Highly-upvoted questions, repeated questions across comments, questions the original post didn't answer. "Can someone explain...", "What about...", "Does this apply to..."

**Why it matters for SEO/GEO:** Real user questions match conversational search queries and People Also Ask triggers. Natural language phrasing that AI engines prefer over manufactured FAQ.

**Output format:** Q&A pairs (min 3, max 6). Each Q paraphrased for clarity but keeping original intent. Each A synthesized from community answers + AI knowledge. If uncertain → flag with ⚠️ "[Suggested answer — verify before publishing]"

#### Category 3: Personal Experiences & Mini Case Studies

First-person accounts with specific results. "I used X for 3 months and...", "Saved 40% on fees after switching...", before/after narratives, warnings from experience.

**Why it matters for SEO/GEO:** "Experience" in E-E-A-T — the signal AI content cannot fake. Most valuable to AI crawlers building knowledge graphs.

**Output format:** 2–4 mini case studies. Each: descriptive title, paraphrased experience (50–100 words, professional tone, preserving original details — never fabricate), source platform + engagement count + viral trigger tag.

### Step 4: Viral Pattern Analysis

For each comment scoring ≥ 7, analyze why it resonated using this taxonomy:

| Trigger | Description | Signal |
|---|---|---|
| Data-backed claim | Specific numbers, percentages, dollar amounts | "I saved $2,400/year by..." |
| Contrarian take | Opposes popular opinion with reasoning | "Everyone says X but actually..." |
| Personal vulnerability | Shares failure, loss, honest struggle | "I lost everything when..." |
| Insider knowledge | Suggests expertise or access | "I work at [company] and..." |
| Simplification | Breaks complex topic into clear explanation | "Think of it like..." |
| Prediction | Specific forward-looking claims | "In 6 months, this will..." |
| Humor + insight | Funny but contains real point | High engagement + substantive replies |

Output the top 3 viral triggers with comment counts, examples, and 1–2 actionable writing tips for the user's blog.

### Step 5: Blog Cross-Reference (only when blog_draft provided)

Compare user's blog draft against extracted comments:

1. **Gap detection** — Topics/questions frequent in comments but missing from blog
2. **Insertion mapping** — Where each output section should go in the blog (reference specific H2 titles)
3. **Conflict detection** — Blog claims contradicted by high-engagement community feedback

### Step 6: Assemble Output

Compile everything into the standard output format below.

---

## Output Format

Always use this exact template structure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
UGC COMMENT INSIGHTS REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

EXECUTIVE SUMMARY
  Sources analyzed:    [X] URL(s) — [list platforms]
  Total comments:      [X]
  High-value comments: [Y] ([Z]%)
  Categories:          [X] counter-arguments | [Y] FAQs | [Z] experiences
  Top viral trigger:   [trigger type]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 1: COMMUNITY PERSPECTIVES — NOTABLE COUNTERPOINTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[150–300 word prose paragraph — ready to copy-paste into blog]

Key debate points:
  1. [Point] — raised by [X] commenters | Top engagement: [Y] likes
  2. [Point] — raised by [X] commenters | Top engagement: [Y] likes
  3. [Point] — raised by [X] commenters | Top engagement: [Y] likes

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 2: FAQ — REAL QUESTIONS FROM THE COMMUNITY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Q1: [Real question from comments — paraphrased for clarity]
A1: [Answer synthesized from community + AI knowledge]
    Source: [X] users asked this | Top answer had [Y] upvotes

Q2: [Real question]
A2: [Answer]
    Source: [engagement data]

Q3: [Real question]
A3: [Answer]
    Source: [engagement data]

[... up to Q6 max]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 3: REAL USER EXPERIENCES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CASE 1: [Descriptive title]
  [Paraphrased experience — professional tone, preserving original
  details and specifics. 50–100 words. Never fabricate.]
  — Source: [Platform] user | [X] likes/upvotes
  — Viral trigger: [why this resonated]

CASE 2: [Descriptive title]
  [Paraphrased experience]
  — Source: [Platform] user | [X] likes/upvotes
  — Viral trigger: [why this resonated]

[... up to CASE 4 max]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 4: VIRAL PATTERN ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Top viral triggers in this thread:
  1. [Trigger]: [X] comments — "[brief example]"
  2. [Trigger]: [X] comments — "[brief example]"
  3. [Trigger]: [X] comments — "[brief example]"

Actionable takeaways for your blog:
  → [Specific writing tip #1 based on patterns]
  → [Specific writing tip #2 based on patterns]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 5: BLOG CROSS-REFERENCE [only if blog draft provided]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CONTENT GAPS:
  → [Topic X] — mentioned [Y] times in comments, missing from blog
  → [Topic Z] — [details]

SUGGESTED INSERTIONS:
  → After "[section title]" → add Section 1 (Counter-Arguments)
  → After "[section title]" → add FAQ items #1, #3
  → Within "[section title]" → add Case Study #2

POTENTIAL CONFLICTS:
  → Your claim "[X]" challenged by [Y] high-engagement comments
    → Suggestion: [how to address]
```

---

## Business Rules

### Always Do
- Fetch URL content automatically — never ask user to paste comments as the first option
- Preserve original meaning when paraphrasing — never exaggerate or distort
- Include engagement metrics (likes, upvotes) for every cited comment
- Flag uncertain AI-generated FAQ answers with ⚠️
- Provide fallback instructions when URL fetch fails
- Output in source content language unless user specifies otherwise
- Anonymize all users — "a Reddit commenter", "an X user" — never real names/usernames

### Never Do
- Never fabricate comments, experiences, or data points not in the source
- Never include purely meme/spam/personal attack comments — even if highly upvoted
- Never output more than 6 FAQ items or 4 case studies — quality over quantity
- Never skip Viral Pattern Analysis — it's a key differentiator
- Never process URLs without attempting to extract engagement metrics

### Edge Cases
- **< 10 comments total**: Process all, add warning: "Limited comment data — insights may not be representative. Consider adding more source URLs."
- **All comments score < 3**: Return: "No high-value comments found. Try a different URL with more substantive discussion."
- **URL is article without comments**: Extract article body only → proceed to cross-reference if blog_draft available, otherwise return: "This URL has no comment section. Paste a URL with community discussion."
- **Mixed languages**: Process all, output in user's preferred language
- **Duplicate insights across URLs**: Deduplicate — combine into single stronger insight with multiple source citations

---

## Example

**User input:**
```
Here are some Reddit threads about Perp DEX platforms, mine the comments for my blog:
- https://reddit.com/r/defi/comments/xxx/hyperliquid_vs_dydx_comparison
- https://reddit.com/r/cryptocurrency/comments/yyy/best_perp_dex_2026

My blog draft is about comparing perpetual DEX platforms.
```

**Expected behavior:**
1. Immediately fetch both URLs (no questions)
2. Extract all comments + engagement data
3. Score and filter → keep high-value comments
4. Classify into 3 categories
5. Analyze viral patterns
6. Since user mentioned blog draft → ask once for the draft text/URL if not provided, then cross-reference
7. Output complete report in standard format within ~1–2 minutes

**Sample output excerpt:**
```
SECTION 1: COMMUNITY PERSPECTIVES — NOTABLE COUNTERPOINTS

Several experienced DeFi traders push back on ranking Hyperliquid as the top
perpetual DEX. The most common concern centers on centralization — specifically 
the small validator set and significant team token concentration. According to 
discussions in the r/defi community, multiple users pointed out that while 
Hyperliquid excels in execution speed and UX, its security model relies on 
assumptions that may not hold during black swan events. One highly-upvoted 
counterpoint noted that dYdX's move to its own Cosmos chain, while slower, 
offers a more battle-tested decentralization approach. However, proponents 
counter that Hyperliquid's approach is a pragmatic tradeoff — optimizing for 
performance now while progressively decentralizing over time.
```
