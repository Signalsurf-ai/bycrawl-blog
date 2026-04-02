# Blog-Write ByCrawl + Playwright Enhancement Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Enhance `skills/blog-write/SKILL.md` with bycrawl MCP social intelligence and Playwright-based SERP/Trends scraping to produce higher-ranking, data-enriched blog articles.

**Architecture:** The existing 7-phase workflow stays intact. We inject bycrawl + Playwright into existing phases as optional enrichment steps (user can opt out). New sub-phases (1.3, 2.5) are added as validation gates. A new reference file `references/social-serp-research.md` holds the detailed API call patterns so the SKILL.md stays under 500 lines.

**Tech Stack:** ByCrawl MCP (`mcp__bycrawl__*`), Playwright (via Bash), existing WebSearch/WebFetch tools.

---

## Task 1: Create the Social & SERP Research Reference File

This new reference file holds all bycrawl API call patterns and Playwright scraping scripts so the main SKILL.md can reference them without bloating.

**Files:**
- Create: `skills/blog/references/social-serp-research.md`

**Step 1: Write the reference file**

```markdown
# Social & SERP Research Guide

Reference for bycrawl MCP and Playwright-based research in blog-write workflow.

## ByCrawl MCP Tool Patterns

### Social Keyword Discovery (Phase 1.3 + Phase 2)

Search across platforms for the target keyword to find long-tail variations,
emerging synonyms, and question-format keywords.

#### Calls (~8 per topic):
- `reddit_search_posts(query="{keyword}", sort="new", count=30)`
- `reddit_search_posts(query="{keyword}", sort="top", count=30)`
- `x_search_posts(query="{keyword}", product="Latest", count=30)`
- `x_search_posts(query="{keyword}", product="Top", count=30)`
- `tiktok_search_videos(keyword="{keyword}", count=30)`
- `youtube_search_videos(q="{keyword}", count=15)`
- `threads_search_posts(query="{keyword}", search_type="top", count=20)`

#### Extract:
- **Emerging terms**: Keywords in "Latest/new" but not in "Top" = newly trending
- **Long-tail variations**: From Reddit titles and YouTube video titles
- **Question-format keywords**: High search intent signals
- **Volume signal**: If <10 results across 3+ platforms, topic may be too niche

### Content Gap Analysis (Phase 2)

Pull top-performing social content to identify saturated vs. underserved angles.

#### Calls (~8 per topic):
- `youtube_search_videos(q="{topic}", count=15)`
- `tiktok_search_videos(keyword="{topic}", count=20)`
- `reddit_search_posts(query="{topic}", sort="top", count=20)`
- `x_search_posts(query="{topic}", product="Top", count=20)`

#### For top 3-5 YouTube videos, pull transcripts:
- `youtube_get_video_transcription(videoId="{id}", language="en")`

#### Extract:
- What everyone already covers (saturated — differentiate or skip)
- Content gaps: audience demand but no good content
- Unique angles competitors miss

### Comment Mining for FAQs (Phase 2 + Phase 5l)

Pull comments from top content to extract real audience questions.

#### Calls (~6-10):
- `youtube_get_video_comments(videoId="{id}", count=50)` (top 3 videos)
- `tiktok_get_video_comments(videoId="{id}")` (top 3 videos)

#### Extract:
- Recurring questions → FAQ items
- Pain points → H2 section topics
- Exact phrases → natural language for article body

### Audience Language Mining (Phase 5)

Extract hooks, pain points, and desire language from top social content.

#### Calls (~10):
- `tiktok_get_video_subtitles(videoId="{id}", language="en")` (top 3)
- `youtube_get_video_transcription(videoId="{id}", language="en")` (top 3)
- Comment pulls from above

#### Emotional trigger searches:
- `x_search_posts(query="{topic} love OR hate OR wish OR need OR finally", product="Top", count=20)`
- `reddit_search_posts(query="{topic} frustrated OR amazing OR changed my life", sort="top", count=20)`

#### Apply to writing:
- **Intro**: Use proven hook patterns from top-performing social content
- **Body**: Weave in audience pain point phrases (their actual words)
- **Examples**: Reference real social discussions ("As one Reddit user noted...")
- **Tone**: Match what resonates on social (formal vs. casual by engagement data)

### Competitor Social Audit (Phase 2)

Check competitor content on social and audience reactions.

#### Calls (~6-9):
- `x_search_posts(query="{competitor} {topic}", product="Top", count=20)`
- `youtube_search_videos(q="{competitor} {topic}", count=10)`
- `reddit_search_posts(query="{competitor} {topic}", sort="top", count=15)`

#### Extract:
- What competitors say about this topic on social
- Audience sentiment (positive, skeptical, bored)
- Gaps in competitor messaging our article can fill

### Post-Write Keyword Validation (Phase 6)

After drafting, verify keyword is still actively discussed.

#### Calls (~4):
- `reddit_search_posts(query="{keyword}", sort="new", count=20)`
- `x_search_posts(query="{keyword}", product="Latest", count=20)`
- `tiktok_search_videos(keyword="{keyword}", count=15)`
- `threads_search_posts(query="{keyword}", search_type="latest", count=15)`

#### Interpret:
- High recent volume = keyword is alive, proceed
- Low volume with declining trend = consider pivoting or noting freshness risk

---

## Playwright SERP & Trends Scraping

### Google Trends Data (Phase 2)

Scrape interest-over-time, related queries, and rising breakout terms.

#### Script:
```bash
npx playwright-cli exec --browser chromium -- node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  await page.goto('https://trends.google.com/trends/explore?q={keyword}&date=today+12-m');
  await page.waitForTimeout(3000);
  // Extract interest over time data
  const data = await page.evaluate(() => {
    const points = document.querySelectorAll('.line-chart text');
    return Array.from(points).map(p => p.textContent);
  });
  // Extract related queries
  const related = await page.evaluate(() => {
    const items = document.querySelectorAll('.related-queries-content .label-text');
    return Array.from(items).map(i => i.textContent);
  });
  console.log(JSON.stringify({ interest: data, related }));
  await browser.close();
})();
"
```

**Fallback**: If Playwright is unavailable, use `WebFetch` on Google Trends embed URLs
or fall through to WebSearch: `[keyword] google trends 2026 interest`.

### SERP Feature Analysis (Phase 2.5)

Scrape Google search results to identify SERP features and competitor content.

#### Script:
```bash
npx playwright-cli exec --browser chromium -- node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  await page.goto('https://www.google.com/search?q={keyword}');
  await page.waitForTimeout(2000);
  const results = await page.evaluate(() => {
    // Extract People Also Ask
    const paa = Array.from(document.querySelectorAll('[data-q]'))
      .map(el => el.getAttribute('data-q'));
    // Extract organic titles
    const titles = Array.from(document.querySelectorAll('h3'))
      .map(h => h.textContent).slice(0, 10);
    // Extract related searches
    const related = Array.from(document.querySelectorAll('.k8XOCe'))
      .map(el => el.textContent);
    // Detect SERP features
    const features = {
      hasAIOverview: !!document.querySelector('[data-attrid=\"ai_overview\"]'),
      hasFeaturedSnippet: !!document.querySelector('.hgKElc'),
      hasPAA: paa.length > 0,
      hasKnowledgePanel: !!document.querySelector('.kp-wholepage'),
    };
    return { paa, titles, related, features };
  });
  console.log(JSON.stringify(results));
  await browser.close();
})();
"
```

**Fallback**: Use `WebSearch` for `"{keyword}" people also ask` and manually
inspect result snippets.

### People Also Ask Extraction (Phase 3)

PAA questions from SERP scraping feed directly into:
- H2 heading candidates (use as question-format headings)
- FAQ section items (high-intent questions Google already surfaces)

### Competitor Title Pattern Analysis (Phase 5a)

From the SERP scrape titles, analyze:
- Number usage frequency (e.g., "7 Ways...", "2026 Guide")
- Question vs. statement ratio
- Power word presence ("Ultimate", "Complete", "Essential")
- Year inclusion
- Average character length

Use these patterns to craft a title that fits the winning pattern but stands out.

---

## Social Proof Embedding Patterns

When writing (Phase 5), embed real social quotes as evidence:

```markdown
<!-- [SOCIAL DATA] -->
> As one Reddit user in r/webdev noted: "We switched to server components
> and saw a 40% reduction in bundle size" — a sentiment echoed across
> multiple threads with 2,000+ combined upvotes.
```

Attribution format:
- Reddit: "A discussion in r/{subreddit} with {upvotes} upvotes..."
- X/Twitter: "As @{handle} noted in a post with {likes} likes..."
- YouTube: "In a video with {views} views, {channel} demonstrated..."

---

## Estimated API Usage

| Phase | ByCrawl Calls | Playwright Ops |
|-------|---------------|----------------|
| Phase 1.3: Social Validation | ~8 | 0 |
| Phase 2: Research | ~20-30 | 1 (Trends) |
| Phase 2.5: SERP Intelligence | 0 | 1 (SERP) |
| Phase 3: Outline (PAA) | 0 | 0 (reuse 2.5) |
| Phase 5: Writing (language) | ~10 | 0 |
| Phase 6: Validation | ~4 | 0 |
| **Total** | **~42-52** | **~2** |
```

**Step 2: Verify the file was written**

Run: `wc -l skills/blog/references/social-serp-research.md`
Expected: ~180-200 lines

**Step 3: Commit**

```bash
git add skills/blog/references/social-serp-research.md
git commit -m "feat: add social & SERP research reference for bycrawl + Playwright integration"
```

---

## Task 2: Update SKILL.md Frontmatter — Add ByCrawl Tools

**Files:**
- Modify: `skills/blog-write/SKILL.md:1-22`

**Step 1: Update the frontmatter**

Replace the existing frontmatter with:

```yaml
---
name: blog-write
description: >
  Write new blog articles from scratch optimized for Google rankings and AI
  citations. Generates full articles with template selection, answer-first
  formatting, TL;DR box, information gain markers, citation capsules, sourced
  statistics, social intelligence from bycrawl MCP, SERP analysis via Playwright,
  Pixabay/Unsplash images, FAQ schema,
  internal linking zones, and proper heading hierarchy. Supports MDX, markdown,
  and HTML output.
  Use when user says "write blog", "new blog post", "create article",
  "write about", "draft blog", "generate blog post".
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - WebFetch
  - WebSearch
  - Task
  - mcp__bycrawl__*
---
```

**Step 2: Update key references block**

Add the new reference to the key references list after line 35:

```markdown
- `references/social-serp-research.md` — ByCrawl API patterns and Playwright SERP scraping
```

**Step 3: Verify**

Run: `head -25 skills/blog-write/SKILL.md`
Expected: `mcp__bycrawl__*` in allowed-tools list

**Step 4: Commit**

```bash
git add skills/blog-write/SKILL.md
git commit -m "feat: add bycrawl MCP tools to blog-write allowed-tools and references"
```

---

## Task 3: Add Phase 1.3 — Social Demand Validation Gate

Insert a new phase between Phase 1 (Topic Understanding) and Phase 1.5 (Template Selection).

**Files:**
- Modify: `skills/blog-write/SKILL.md` (insert after Phase 1, before Phase 1.5)

**Step 1: Insert Phase 1.3 after the Phase 1 block (after line 45)**

```markdown
### Phase 1.3: Social Demand Validation (Optional)

> "Would you like me to validate this topic with live social signals from bycrawl? (yes/no)"

If yes, run quick social volume check across platforms (see `references/social-serp-research.md`
§ Social Keyword Discovery):

1. **Cross-platform search** — Query Reddit, X, TikTok, YouTube, Threads for the keyword
2. **Signal assessment**:
   - **Strong** (50+ results across 3+ platforms): Proceed with confidence
   - **Moderate** (20-50 results): Proceed but note niche audience
   - **Weak** (<20 results total): Warn user — consider pivoting or niching down
3. **Extract early intelligence**:
   - Long-tail keyword variations from social titles
   - Question-format keywords (high intent)
   - Emerging terms in "Latest" but not "Top" (newly trending)
4. **Feed into Phase 1.5** — Social signals help select the right template
   (e.g., high question volume → `faq-knowledge`, viral debate → `thought-leadership`)

If no, skip to Phase 1.5.
```

**Step 2: Verify insertion**

Run: `grep -n "Phase 1.3" skills/blog-write/SKILL.md`
Expected: Line showing "Phase 1.3: Social Demand Validation"

**Step 3: Commit**

```bash
git add skills/blog-write/SKILL.md
git commit -m "feat: add Phase 1.3 social demand validation gate to blog-write"
```

---

## Task 4: Enhance Phase 2 — ByCrawl Social Research

Add bycrawl enrichment steps to the existing Phase 2 (Research).

**Files:**
- Modify: `skills/blog-write/SKILL.md` (Phase 2 section, after existing step 4)

**Step 1: Add bycrawl research steps after the existing step 4 (line ~100)**

Insert after `- Map data points to chart formats`:

```markdown

#### ByCrawl Social Research (Optional)

> "Would you like me to enrich research with live social data from bycrawl? (~20-30 API calls)"

If yes, run these enrichments (details in `references/social-serp-research.md`):

5. **Content gap analysis** — Search top-performing content across Reddit, X,
   TikTok, YouTube to identify what's saturated vs. underserved
6. **Comment mining** — Pull comments from top 3 YouTube and TikTok videos to
   extract real audience questions → feed into FAQ and H2 topics
7. **Real social statistics** — Use engagement data from viral posts as original
   data points (e.g., "A Reddit thread with 4.2K upvotes revealed...")
8. **Competitor social audit** — Check what competitors publish about this topic,
   how audiences react, and what gaps exist in their messaging
9. **Audience language** — Extract hooks, pain points, and desire language from
   transcripts and comments for use in Phase 5 writing

Record social findings in a `## Social Intelligence` section of research notes.
```

**Step 2: Verify**

Run: `grep -n "ByCrawl Social Research" skills/blog-write/SKILL.md`
Expected: Line showing the new section header

**Step 3: Commit**

```bash
git add skills/blog-write/SKILL.md
git commit -m "feat: add bycrawl social research enrichment to Phase 2"
```

---

## Task 5: Add Phase 2.5 — SERP Intelligence via Playwright

Insert a new phase between Phase 2 (Research) and Phase 3 (Outline Generation).

**Files:**
- Modify: `skills/blog-write/SKILL.md` (insert after Phase 2, before Phase 3)

**Step 1: Insert Phase 2.5 after Phase 2 block**

```markdown
### Phase 2.5: SERP Intelligence (Optional)

> "Would you like me to scrape Google for SERP features and competitor analysis? Uses Playwright."

If yes, run Playwright SERP scraping (scripts in `references/social-serp-research.md`):

1. **Google Trends data** — Scrape interest-over-time for the keyword (past 12 months)
   and related/rising queries. Feed trend data into chart generation (Phase 4).
2. **SERP feature detection** — Identify: AI Overview, Featured Snippet, PAA box,
   Knowledge Panel. Adapt content format to target available features.
3. **People Also Ask extraction** — Pull PAA questions to:
   - Use as H2 heading candidates (Phase 3)
   - Add to FAQ section (Phase 5l)
4. **Competitor title analysis** — Analyze top 10 organic titles for patterns:
   number usage, question format, year inclusion, power words, character length.
   Feed into title crafting (Phase 5a).
5. **Related searches** — Extract "Related searches" and "People also search for"
   for secondary keyword coverage.

**Fallback**: If Playwright is unavailable, use `WebSearch` for PAA and competitor
title data. Google Trends can be approximated via `WebSearch` queries.

Feed SERP intelligence into:
- Phase 3 outline (PAA → H2 headings, related searches → subtopics)
- Phase 4 charts (Google Trends data → trend line chart)
- Phase 5a title (competitor title patterns)
- Phase 5l FAQ (PAA questions)
```

**Step 2: Verify**

Run: `grep -n "Phase 2.5" skills/blog-write/SKILL.md`
Expected: Line showing "Phase 2.5: SERP Intelligence"

**Step 3: Commit**

```bash
git add skills/blog-write/SKILL.md
git commit -m "feat: add Phase 2.5 SERP intelligence via Playwright"
```

---

## Task 6: Enhance Phase 5 — Social Proof & Audience Language

Add social data integration to the content writing phase.

**Files:**
- Modify: `skills/blog-write/SKILL.md` (Phase 5 section)

**Step 1: Add new section 5d-bis for Social Data markers (after existing 5d, line ~250)**

Insert after the Information Gain Markers section (5d), updating the marker list:

```markdown
- `[SOCIAL DATA]` — Insights derived from social platform analysis via bycrawl.
  Real engagement metrics, audience sentiment patterns, cross-platform trends
  not available in traditional research
```

**Step 2: Add new section 5n — Social Proof Embedding (after 5m, before Phase 6)**

```markdown
#### 5n. Social Proof Embedding (if bycrawl data available)

When bycrawl research was run in Phase 2, embed real social evidence:

**Reddit quotes:**
```markdown
<!-- [SOCIAL DATA] -->
> A discussion in r/{subreddit} with {upvotes} upvotes highlighted a common
> frustration: "{exact quote}" — a pattern repeated across multiple threads.
```

**X/Twitter references:**
```markdown
According to @{handle}, whose post received {likes} likes: "{quote}."
```

**YouTube/TikTok data:**
```markdown
In a video with {views} views, {channel} demonstrated that {finding} —
confirmed by {N} comments echoing similar results.
```

Rules:
- Attribute with platform, engagement metric, and context
- Use social data to reinforce (not replace) tier 1-3 research sources
- Target 2-3 social proof embeddings per article
- Each embedding earns `[SOCIAL DATA]` information gain credit
```

**Step 3: Verify both insertions**

Run: `grep -n "SOCIAL DATA\|5n\. Social Proof" skills/blog-write/SKILL.md`
Expected: Lines showing both additions

**Step 4: Commit**

```bash
git add skills/blog-write/SKILL.md
git commit -m "feat: add social data markers and social proof embedding to Phase 5"
```

---

## Task 7: Enhance Phase 6 — Social Validation Gate + SERP Checks

Add new quality check items to Phase 6.

**Files:**
- Modify: `skills/blog-write/SKILL.md` (Phase 6 section)

**Step 1: Add social/SERP validation items after the existing check 18 (end of burstiness section)**

```markdown

#### Social & SERP Validation (if enrichment was used)
19. **Social content gap addressed** — Article covers at least 1 content gap identified from bycrawl social data
20. **Audience language incorporated** — At least 3 pain point phrases or desire language from social comments woven into body copy
21. **Social proof present** — At least 2 `[SOCIAL DATA]` markers with real engagement metrics
22. **PAA coverage** — If PAA was extracted, at least 2 PAA questions addressed in H2s or FAQ
23. **Keyword still active** — Run post-write keyword validation via bycrawl (see `references/social-serp-research.md` § Post-Write Keyword Validation). If social volume dropped significantly since Phase 1.3, flag for user review.
24. **Google Trends alignment** — If Trends data was scraped, verify article doesn't focus on a declining interest curve
```

**Step 2: Verify**

Run: `grep -n "Social & SERP Validation" skills/blog-write/SKILL.md`
Expected: Line showing the new subsection

**Step 3: Commit**

```bash
git add skills/blog-write/SKILL.md
git commit -m "feat: add social & SERP validation checks to Phase 6 quality gate"
```

---

## Task 8: Enhance Phase 7 — Social Metrics in Delivery Summary

**Files:**
- Modify: `skills/blog-write/SKILL.md` (Phase 7 delivery template)

**Step 1: Add social intelligence section to the delivery summary template (after Dual-Optimization Elements block)**

```markdown
### Social Intelligence (if bycrawl enrichment was used)
- Platforms queried: [list — Reddit, X, TikTok, YouTube, Threads]
- Content gaps identified: [N] ([brief descriptions])
- Audience phrases incorporated: [N]
- Social proof embeddings: [N] with [SOCIAL DATA] markers
- FAQ items sourced from comments: [N] of [total FAQ items]
- Keyword social volume: [strong/moderate/weak]

### SERP Intelligence (if Playwright was used)
- SERP features detected: [AI Overview, Featured Snippet, PAA, etc.]
- PAA questions extracted: [N] (used in: [H2s / FAQ / both])
- Google Trends direction: [rising / stable / declining]
- Competitor titles analyzed: [N]
```

**Step 2: Also add new Next Steps items**

Add after existing next steps:

```markdown
- If bycrawl was used: verify social quotes are still current
- If Trends data was used: schedule re-check in 3 months for freshness update
```

**Step 3: Verify**

Run: `grep -n "Social Intelligence\|SERP Intelligence" skills/blog-write/SKILL.md`
Expected: Lines showing both new delivery sections

**Step 4: Commit**

```bash
git add skills/blog-write/SKILL.md
git commit -m "feat: add social & SERP metrics to Phase 7 delivery summary"
```

---

## Task 9: Update Quality Scoring Reference (Optional)

Add `[SOCIAL DATA]` as a recognized information gain marker in the scoring rubric.

**Files:**
- Modify: `skills/blog/references/quality-scoring.md:11`

**Step 1: Update the Originality row**

Change:
```
| Originality/unique value markers | 5 | Original data, case studies, first-hand experience |
```

To:
```
| Originality/unique value markers | 5 | Original data, case studies, first-hand experience, social platform intelligence |
```

**Step 2: Commit**

```bash
git add skills/blog/references/quality-scoring.md
git commit -m "feat: add social platform intelligence to quality scoring originality criteria"
```

---

## Task 10: Final Line Count Verification

**Files:**
- Verify: `skills/blog-write/SKILL.md`

**Step 1: Check line count**

Run: `wc -l skills/blog-write/SKILL.md`
Expected: Under 550 lines (stretch target — detailed reference file absorbs most content)

If over 550, move verbose sections to `references/social-serp-research.md` and
replace with one-line pointers.

**Step 2: Check all references resolve**

Run: `grep 'references/' skills/blog-write/SKILL.md`
Expected: All referenced files exist

**Step 3: Final commit**

```bash
git add -A
git commit -m "chore: verify blog-write SKILL.md line count and reference integrity"
```

---

## Summary

| Task | What | Lines Added | API Cost |
|------|------|-------------|----------|
| 1 | Reference file for bycrawl + Playwright patterns | ~180 (new file) | 0 |
| 2 | Frontmatter update (allowed-tools + refs) | ~3 | 0 |
| 3 | Phase 1.3: Social demand validation | ~18 | ~8 calls |
| 4 | Phase 2: ByCrawl social research | ~15 | ~20-30 calls |
| 5 | Phase 2.5: SERP intelligence | ~22 | 2 Playwright ops |
| 6 | Phase 5: Social proof + audience language | ~30 | 0 (reuses Phase 2) |
| 7 | Phase 6: Social/SERP validation gate | ~8 | ~4 calls |
| 8 | Phase 7: Delivery summary additions | ~15 | 0 |
| 9 | Quality scoring update | ~1 | 0 |
| 10 | Line count verification | 0 | 0 |

**Total estimated bycrawl calls per enriched article: ~42-52**
**Total Playwright operations: ~2**
