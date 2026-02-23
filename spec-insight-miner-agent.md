# Skill Spec: UGC Comment Insights Miner

Version: 1.0 | Owner: [Your Name]
Domain: Content Writing / SEO / GEO
Status: Draft — Pending Supervisor Review

---

## 1. Problem Statement

Every time a Content Writer needs to add depth and real-world perspective to a blog post, they must manually hunt for community discussions, scroll through hundreds of comments, and cherry-pick the valuable ones.

Consequences:
- 1–2 hours wasted per article just to find and synthesize quality comments
- Most writers skip this step entirely because it's tedious → articles ship without UGC depth
- Blog posts read like every other AI-generated "slop content" — one-sided, generic, no real human voice
- Lower E-E-A-T signals → lower chance of being cited by AI search engines (Google SGE, Perplexity, ChatGPT Search)
- Reddit alone accounts for ~21% of citations in Google AI Overviews (early 2026) — content with real UGC gets 2–3x more visibility than theory-only articles

The bottleneck is NOT writing speed — experienced writers can outline and draft quickly. The bottleneck is **enriching content with authentic community insights** that make it stand out.

---

## 2. Skill Overview

Input: 1 or more URLs (X/Twitter thread, Reddit post, forum, blog with comments)
→ AI auto-fetches the page + all comments/replies
→ Filters noise, classifies valuable comments into 3 insight categories
→ Analyzes why top comments went viral
→ (Optional) Cross-references with user's blog draft to suggest insertion points
→ Output: 1 complete markdown block with ready-to-use content sections + viral pattern analysis.

---

## 3. Target Users

Content Writer / SEO Specialist / Content Team Lead.
Prerequisite: User has 1+ URLs containing community discussion relevant to their blog topic.

---

## 4. Input Requirements

### 4.1 URLs — Required

User pastes 1 or more links. The skill auto-fetches page content + comments.

| Supported Source | What Gets Extracted | Notes |
|---|---|---|
| X (Twitter) thread | Original post + all replies/quote tweets | Public threads only |
| Reddit post | Post body + all comments sorted by top/best | Works best with old.reddit.com format |
| Facebook public post | Post + public comments | Must be publicly accessible |
| YouTube video page | Video description + top comments | Sorted by relevance |
| Blog / forum with comments | Article body + comment section | HTML comment sections |

### 4.2 Optional Inputs

| Field | Description | Impact When Missing |
|---|---|---|
| topic_keyword | Blog topic or target keyword | AI infers topic from URL content — less precise matching |
| blog_draft | Full text or URL of user's existing blog post | Cannot run cross-reference analysis (Step 5) or suggest insertion points |
| language | Output language preference (EN / VI / auto) | Defaults to same language as source comments |
| max_comments | Maximum comments to process | Defaults to top 100 by engagement |

### 4.3 Clarifying Questions (Conditional)

The skill does NOT ask questions by default — it processes immediately upon receiving URLs. Questions are only asked when:

**Q1 — Ambiguous topic (only if no topic_keyword AND multiple unrelated URLs):**
> "These URLs seem to cover different topics. What is the main topic of your blog post?"

**Q2 — Blog draft available (only asked once, first time):**
> "Do you have a blog draft you'd like me to cross-reference? If yes, paste the content or URL. If not, I'll generate standalone insight sections."

→ After first interaction, the skill remembers the preference and does not ask again.

---

## 5. Processing Logic (Mandatory Sequence)

### Step 1: URL Fetch & Comment Extraction

For each URL provided:

1. Use `web_fetch` to retrieve the full page content
2. Parse and extract:
   - **Original post/article body** (the main content)
   - **All comments/replies** with available metadata:
     - Comment text
     - Engagement signals (likes, upvotes, retweets, reply count)
     - Author identifier (username or anonymized)
     - Timestamp
     - Nesting level (is it a reply to another comment?)
3. Store raw data for processing

**Fallback protocol:**
- If `web_fetch` cannot access the URL (login wall, rate limit, JavaScript-rendered comments):
  → Return message: "I couldn't access the comments on this URL. Please paste the comment text directly and I'll process it from there."
  → Continue processing with whatever content was retrieved (article body may still be available even if comments aren't)

### Step 2: Noise Filtering & Quality Scoring

Score each comment on a 0–10 quality scale:

| Signal | Weight | Scoring |
|---|---|---|
| Engagement (likes/upvotes) | 30% | Top 10% of thread = 10, top 25% = 7, top 50% = 5, bottom 50% = 2 |
| Content length | 20% | < 10 words = 1, 10–50 words = 5, 50–150 words = 8, 150+ words = 10 |
| Contains data/numbers | 15% | Yes = 10, No = 3 |
| Contains personal experience markers | 15% | "I tried...", "In my case...", "After using..." = 10, None = 3 |
| Substantive reply thread | 10% | Has 3+ quality replies = 10, 1–2 = 6, none = 3 |
| Sentiment strength | 10% | Strong opinion (positive or negative) = 8, Neutral = 4 |

**Filter actions:**
- Score ≥ 6: Include in analysis → label "High Value"
- Score 3–5: Include only if total comment pool is small (< 20 high-value comments)
- Score < 3: Discard → spam, memes, off-topic, too short

**Output after this step (internal):**
- Total comments fetched: [X]
- Passed quality filter: [Y] ([Z]%)
- Discarded: [X - Y]

### Step 3: Classify into 3 Insight Categories

Every comment that passed filtering gets classified into exactly 1 of 3 categories. If a comment contains elements of multiple categories, assign it to the **strongest** match.

#### Category 1: Counter-Arguments & Alternative Perspectives

**What qualifies:**
- Disagrees with or challenges a claim in the original post
- Presents a different interpretation of the same data
- Points out edge cases, exceptions, or overlooked downsides
- "Yeah but..." / "This ignores..." / "The problem with this take is..."

**Why this matters for GEO:**
Multi-perspective content signals to AI crawlers that the article is a comprehensive "hub" for the topic, not a one-sided promotional piece. Google's E-E-A-T guidelines explicitly reward balanced, nuanced content.

**Output format for this category:**
```
Section title: "Community Perspectives: Notable Counterpoints"
Format: 150–300 word prose paragraph summarizing the key counter-arguments.
Tone: Balanced, acknowledging validity of different viewpoints.
Structure: "[Main counterpoint]. According to discussions in the [platform] community,
[elaboration with specific examples from comments]. However, [nuance/context]."
```

#### Category 2: Real-World FAQs

**What qualifies:**
- Direct questions asked in comments (especially highly-upvoted ones)
- Repeated questions across multiple comments (signals content gap)
- "Can someone explain..." / "What about..." / "Does this apply to..."
- Questions that the original post did NOT answer

**Why this matters for GEO:**
FAQ sections with real user questions match conversational search queries and People Also Ask triggers. Real questions are phrased in natural language that mirrors how people actually search — AI engines prefer this over manufactured FAQ.

**Output format for this category:**
```
Section title: "FAQ: Real Questions from the Community"
Format: Q&A pairs (minimum 3, maximum 6)
Each Q: The actual question (paraphrased for clarity, keeping original intent)
Each A: AI-generated answer based on:
  1. Context from the original post/article
  2. Answers provided by other commenters
  3. AI's own knowledge for factual accuracy
Flag: If AI is uncertain about an answer → mark with
  ⚠️ "[Suggested answer — verify before publishing]"
```

#### Category 3: Personal Experiences & Mini Case Studies

**What qualifies:**
- First-person accounts: "I used X for 3 months and..."
- Specific results with numbers: "Saved 40% on fees after switching to..."
- Before/after narratives: "Before I tried this, I was... Now I..."
- Warnings from experience: "Be careful with X because when I did it..."

**Why this matters for GEO:**
This is the "Experience" in E-E-A-T — the one signal that AI-generated content fundamentally cannot fake. Articles containing real user experiences are the hardest to replicate and the most valuable to AI crawlers building knowledge graphs.

**Output format for this category:**
```
Section title: "Real User Experiences"
Format: 2–4 mini case studies, each structured as:

**[Descriptive title summarizing the experience]**
[Paraphrased experience — rewritten professionally while preserving
the original meaning, specific details, and emotional tone.
Never fabricate details not present in the original comment.]
— Source: [Platform] user | Engagement: [X likes/upvotes]

```

### Step 4: Viral Factor Analysis

For each high-value comment (score ≥ 7), analyze WHY it resonated:

**Viral trigger taxonomy:**

| Trigger | Description | Example Signal |
|---|---|---|
| Data-backed claim | Contains specific numbers, percentages, dollar amounts | "I saved $2,400/year by..." |
| Contrarian take | Directly opposes popular opinion with reasoning | "Everyone says X but actually..." |
| Personal vulnerability | Shares failure, loss, or honest struggle | "I lost everything when..." |
| Insider knowledge | Information that suggests expertise or access | "I work at [company] and..." |
| Simplification | Breaks down complex topic into clear explanation | "Think of it like..." |
| Prediction | Makes specific forward-looking claims | "In 6 months, this will..." |
| Humor + insight | Funny but also contains a real point | High engagement + substantive replies |

**Output for this step:**
```
VIRAL PATTERN ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Top viral triggers in this thread:
1. [Trigger type]: [X] comments — Example: "[brief quote or paraphrase]"
2. [Trigger type]: [X] comments — Example: "[brief quote or paraphrase]"
3. [Trigger type]: [X] comments — Example: "[brief quote or paraphrase]"

Actionable takeaway for your blog:
- [Specific writing tip derived from the dominant viral pattern]
- [Second tip if applicable]
```

### Step 5: Blog Cross-Reference (Conditional — only when blog_draft is provided)

Compare the user's blog draft against the extracted comments:

**Analysis performed:**
1. **Gap detection**: Topics/questions that appear frequently in comments but are NOT covered in the blog draft
2. **Insertion mapping**: For each output section (counter-arguments, FAQ, experiences), suggest the specific location in the blog where it should be added
3. **Conflict detection**: Flag any claims in the blog draft that are directly contradicted by high-engagement community feedback

**Output for this step:**
```
BLOG CROSS-REFERENCE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CONTENT GAPS FOUND:
- Your blog does not cover [topic X] — mentioned in [Y] comments with high engagement
- Question "[specific question]" appears [X] times — your blog has no answer for this

SUGGESTED INSERTIONS:
- After section "[H2 title from blog]" → insert Counter-Arguments section
- After section "[H2 title from blog]" → insert FAQ items #1 and #3
- Within section "[H2 title from blog]" → insert Case Study #2 (directly supports your claim)

POTENTIAL CONFLICTS:
- Your blog states "[claim]" — but [X] high-engagement comments challenge this with [reasoning]
  → Consider adding nuance or addressing this counterpoint
```

### Step 6: Output Assembly

Compile everything into 1 complete markdown output following the standard format below.

**Mandatory sections (always present):**
- Executive Summary
- Category 1: Counter-Arguments
- Category 2: Real-World FAQs
- Category 3: Personal Experiences
- Viral Pattern Analysis

**Conditional sections:**
- Blog Cross-Reference → only when blog_draft was provided
- Source quality warning → only when < 10 comments passed quality filter

---

## 6. Output: Standard Format

```markdown
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

Q1: [Real question from comments — paraphrased]
A1: [Answer synthesized from community + AI knowledge]
    Source: [X] users asked this | Top answer had [Y] upvotes

Q2: [Real question]
A2: [Answer]
    Source: [engagement data]

Q3: [Real question]
A3: [Answer]
    Source: [engagement data]

[... up to Q6 maximum]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECTION 3: REAL USER EXPERIENCES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CASE 1: [Descriptive title]
  [Paraphrased experience — professional tone, preserving
  original details and specifics. 50–100 words.]
  — Source: [Platform] user | [X] likes/upvotes
  — Viral trigger: [why this resonated]

CASE 2: [Descriptive title]
  [Paraphrased experience]
  — Source: [Platform] user | [X] likes/upvotes
  — Viral trigger: [why this resonated]

[... up to CASE 4 maximum]

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
SECTION 5: BLOG CROSS-REFERENCE (if blog draft provided)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CONTENT GAPS:
  → [Topic X] — mentioned [Y] times in comments, missing from your blog
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

## 7. Business Rules

### Always Do:
- Fetch URL content automatically — never ask user to paste comments manually as the first option
- Preserve the original meaning of comments when paraphrasing — never exaggerate or distort
- Include engagement metrics (likes, upvotes) for every cited comment to show credibility
- Flag uncertain AI-generated answers in FAQ with ⚠️ marker
- Provide fallback instructions when URL fetch fails
- Output in the same language as the source content unless user specifies otherwise

### Never Do:
- Never fabricate comments, experiences, or data points that don't exist in the source
- Never include usernames/real names in the output — use anonymized references ("a user on X", "a Reddit commenter")
- Never include comments that are purely memes, spam, or personal attacks — even if highly upvoted
- Never output more than 6 FAQ items or 4 case studies — quality over quantity
- Never skip the Viral Pattern Analysis — this is a key differentiator of the skill
- Never process URLs without attempting to extract engagement metrics — these inform quality scoring

### Edge Cases:
- **< 10 comments total**: Process all of them, but add a warning: "Limited comment data — insights may not be representative. Consider adding more source URLs."
- **All comments are low-quality (< 3 score)**: Return: "No high-value comments found in this thread. Try a different URL with more substantive discussion."
- **URL is an article without comments**: Extract article body only → Skip to Step 5 (cross-reference with blog draft if available) or return: "This URL has no comment section. Paste a URL with community discussion."
- **Mixed languages in comments**: Process all languages, output in user's preferred language
- **Duplicate insights across multiple URLs**: Deduplicate — combine into single stronger insight with multiple source citations

---

## 8. Success Criteria

Output is considered successful when:
- [ ] Content Writer can take any output section → paste directly into blog draft with minimal editing (< 5 min of touch-up per section)
- [ ] All 3 insight categories contain substantive content — not generic summaries but specific, data-rich, community-sourced insights
- [ ] FAQ questions are real questions from real people — not AI-manufactured "common questions"
- [ ] Personal experiences contain specific details (numbers, timeframes, outcomes) — not vague "one user said it was good"
- [ ] Viral Pattern Analysis provides at least 1 actionable writing tip the writer didn't already know
- [ ] Cross-reference (when used) correctly identifies at least 1 genuine content gap in the blog draft
- [ ] Full output generated in < 2 minutes from URL input

---

## 9. Demo Scenario

### Setup
Topic: "Comparing Perpetual DEX platforms in 2026"

**User input:**
```
URLs:
- [X thread comparing Hyperliquid vs dYdX vs GMX]
- [Reddit r/defi discussion on Perp DEX fees and UX]
- [X thread: trader's 3-month review of Hyperliquid]

Blog draft: [user's comparison article]
```

### Expected Demo Flow
1. User pastes 3 URLs → skill immediately begins fetching (no questions asked)
2. Within ~1 minute, output appears with:
   - Counter-arguments: "Several experienced traders push back on ranking Hyperliquid #1, citing centralization concerns around the small validator set and team token concentration..."
   - FAQ: "Q: How do funding rates actually compare between Hyperliquid and dYdX during high volatility? A: Based on community data shared during the March 2026 correction..."
   - Experience: "One trader documented losing $12K due to delayed liquidation processing on [DEX] during the March crash, noting that the same position on [other DEX] would have been liquidated $3K earlier..."
   - Viral analysis: "Data-backed claims dominated this thread — comments with specific PnL numbers received 5x more engagement than opinion-only comments"
   - Cross-reference: "Your blog draft does not address decentralization tradeoffs — this was the #1 most-discussed topic in the comment threads"
3. Writer copies sections into blog → publishes enriched article

### Before / After Comparison (for BGK)

| Metric | Before (Manual) | After (With Skill) |
|---|---|---|
| Time to gather UGC insights | 1–2 hours | 1–2 minutes |
| Comments actually reviewed | ~30 (skim) | 100+ (systematic) |
| Insight quality | Hit or miss, biased by fatigue | Scored and ranked by engagement |
| Viral analysis | None | Automated pattern detection |
| Blog cross-reference | Manual re-reading | Automated gap detection |
| Team scalability | Only experienced writers | Anyone on the team |

---

## 10. Roadmap

### Phase 1: Core Skill (Current — Cook a Skill Challenge)

**Scope:**
- Input: Paste URL(s) → AI auto-fetches content + comments
- Noise filtering with quality scoring
- 3-category classification (counter-arguments, FAQs, experiences)
- Viral factor analysis
- Blog cross-reference (optional)
- Output: Complete markdown report, ready for copy-paste

**Deliverable:** Working SKILL.md that runs live demo in < 2 minutes.

---

### Phase 2: Enhanced Blog Integration (Post-Challenge — Internal Team)

**Additions:**
- Multi-link batch processing (5–10 URLs at once, cross-source deduplication)
- Auto-detect which URL comments are most relevant to specific sections of blog draft
- Template variants for different blog types: review, comparison, how-to, analysis, listicle
- Tone/voice calibration to match company brand guidelines
- Output as structured DOCX or HTML with formatting preserved

---

### Phase 3: X Learning Companion — Public Product (Extension / App)

**Vision:** Transform the core insight-mining engine into a consumer product that helps anyone learn more effectively from social media.

#### 3.1 — Universal Comment Insight Miner
- Browser extension: click on any X/Reddit/YouTube page → instantly see filtered high-value comments + insights
- Not just for content writers — for anyone who wants to "read comments smarter"
- Same 3-category engine, adapted for personal learning instead of blog enrichment

#### 3.2 — Smart Bookmark Manager
- Problem: People bookmark/save dozens of posts but rarely revisit them
- Solution: App tracks bookmark status — read vs. unread
- Daily reminder: "You have [X] unread bookmarks. Here are today's 3 recommended reads."
- When user marks as "read" → app generates key takeaways summary
- Gamification: reading streaks, weekly stats ("You read 12/18 bookmarks this week")

#### 3.3 — Favorite Profile Tracker
- User adds list of favorite profiles (KOLs, researchers, builders)
- App monitors their activity — NOT every post, only when they:
  - Post a comment that goes viral (above engagement threshold)
  - Share an insight-rich reply (detected by content quality scoring)
  - Engage in a high-quality debate thread
- Notification: "[Profile] just dropped an insight on [topic] — [X] likes in 2 hours"
- Filters: minimum engagement threshold, keyword filters, content length minimum

#### 3.4 — Personalized Learning Feed (Long-term)
- Aggregates all sources: bookmarks + tracked profiles + manual URLs
- Generates "Daily Digest": "3 insights worth your time today"
- Spaced repetition: resurfaces key insights at intervals so users retain what they learn
- Personal knowledge graph: tracks topics user engages with most → smarter recommendations over time

**Product Evolution:**
```
Phase 1           Phase 2              Phase 3
AI Skill     →    Blog Tool      →     X Learning Companion
(internal)        (team)               (public product)

Fetch + Mine →    Cross-reference →    Browser extension
Filter       →    Batch process  →     Smart bookmarks
Classify     →    Template output →    Profile tracker
Viral analysis    Brand voice          Daily digest
                                       Spaced repetition
```

---

## 11. Technical Notes

### Phase 1 Stack
- **AI Engine**: Claude via SKILL.md prompt engineering
- **URL Fetching**: `web_fetch` tool (built into Claude's environment)
- **Input**: Raw URL(s) pasted by user
- **Output**: Markdown (.md) — easily convertible to HTML/Docs/CMS
- **No external dependencies**: No database, no API keys, no hosting required

### Known Technical Risks

| Risk | Severity | Mitigation |
|---|---|---|
| X/Twitter blocks scraping (login wall) | High | Test demo URLs in advance; prepare Reddit/forum backup links; fallback to user-pasted text |
| JavaScript-rendered comment sections | High | Choose platforms with HTML-accessible comments for demo; document which platforms work best |
| Rate limiting on web_fetch | Medium | Limit to 3–5 URLs per run; add delay between fetches if needed |
| Comment volume too large (>500) | Low | Cap at top 100 by engagement; process in batches if needed |
| Non-English comment parsing | Low | Skill handles multi-language; output defaults to source language |

### Pre-Demo Checklist
- [ ] Test 3–5 real URLs to confirm `web_fetch` extracts comments successfully
- [ ] Prepare 2 backup URLs (Reddit + forum) in case X links are blocked
- [ ] Have a pre-written blog draft ready for cross-reference demo
- [ ] Time the full run — must complete in < 2 minutes

---

## 12. MVP Boundary

### In Scope (Phase 1):
- Auto-fetch content + comments from 1–3 URLs via `web_fetch`
- Quality scoring and noise filtering for comments
- 3-category classification with formatted output sections
- Viral pattern analysis with actionable takeaways
- Blog cross-reference with gap detection and insertion suggestions
- Complete markdown output ready for copy-paste

### Out of Scope (for later):
- Automated URL discovery (suggesting where to find discussions)
- Scheduled/recurring monitoring of threads
- Direct CMS integration (WordPress, Webflow, etc.)
- Browser extension or standalone app
- User authentication or saved preferences
- Comment sentiment tracking over time
