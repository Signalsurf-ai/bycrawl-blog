---
name: blog-content-research
description: >
  Content gap analysis, idea mining, and trend research across social platforms using bycrawl MCP.
  Use when user says "content ideas", "content research", "what should I post about",
  "content gap", "trending topics", "video ideas", "blog ideas from social",
  "what questions are people asking about [topic]".
allowed-tools: ["mcp__bycrawl__*", "WebFetch", "WebSearch"]
user-invocable: true
argument-hint: "<topic> [format] e.g. AI coding tools, video"
---

# Content Research

## ICP (Ideal Customer Profile)
- **Content marketers** planning editorial calendars
- **YouTube/TikTok creators** looking for video ideas with proven demand
- **SEO writers** mining social platforms for content gaps
- **Founders** who need to create thought leadership content

## What This Skill Does
Mines social platforms for trending content, audience questions, and gaps in existing coverage for a given topic. Extracts ideas from comments, transcripts, and discussions — then synthesizes into an actionable content plan.

## Required Input
- **Topic or niche** (required): What to research
- **Content format** (optional): "video", "blog", "social posts", or "all"
- **Platform focus** (optional): Specific platforms to prioritize


## Prerequisites

- **ByCrawl MCP server** must be installed and configured. If MCP tools are not available, install with:
  ```
  npx @anthropic-ai/claude-code mcp add bycrawl -- npx @bycrawl/mcp
  ```
- **ByCrawl API key** set as environment variable: `export BYCRAWL_API_KEY=sk_byc_...`
- Get your API key at [bycrawl.com](https://bycrawl.com)

## Workflow

### Step 1: Surface Existing Content
Run in parallel:
```
youtube_search_videos(q="{topic}", count=15)
tiktok_search_videos(keyword="{topic}", count=20)
reddit_search_posts(query="{topic}", sort="top", count=20)
x_search_posts(query="{topic}", product="Top", count=20)
```

### Step 2: Extract Transcripts from Top Videos
For the top 3-5 YouTube videos by views:
```
youtube_get_video_transcription(videoId="{id}", language="en")
```

For top 3-5 TikTok videos with subtitles:
```
tiktok_get_video_subtitles(videoId="{id}", language="en")
```

Analyze transcripts to identify:
- Key points every video covers (saturated topics)
- Points NONE of them cover (content gaps)
- Unique angles that got high engagement

### Step 3: Comment Mining for Questions
Pull comments from top-performing content:
```
youtube_get_video_comments(videoId="{id}", count=50)
tiktok_get_video_comments(videoId="{id}")
```

Extract:
- Recurring questions (= content demand)
- Complaints about existing content (= improvement opportunities)
- "I wish someone would explain..." patterns

### Step 4: Reddit Deep Dive
```
reddit_search_posts(query="{topic}", sort="comments", count=20)
```

Reddit posts sorted by comments reveal the most-discussed sub-topics. Scan for:
- Unanswered questions
- Debates (= controversy content performs well)
- "How do I..." posts (= tutorial demand)

### Step 5: Trend Detection
```
tiktok_get_category_videos(category="{relevant_category}", count=20)
threads_get_public_feed(count=20)
```

Identify emerging formats and angles before they saturate.

## Output Format

```markdown
## Content Research Report — {topic} — {date}

### Content Landscape
- Total videos found on YouTube: {n} (top video: {views} views)
- Total TikTok videos: {n} (top video: {views} views)
- Reddit discussions: {n} (most commented: {n} comments)

### What Everyone Already Covers
1. {saturated_topic_1}
2. {saturated_topic_2}
3. {saturated_topic_3}

### Content Gaps (Your Opportunity)
1. **{gap_1}** — No one covers this despite {n} people asking in comments
2. **{gap_2}** — Only 1 video exists, and it has {views} views (demand > supply)
3. **{gap_3}** — Reddit thread with {n} comments, zero video coverage

### Top Questions from Audiences
1. "{question_1}" — asked {n} times across platforms
2. "{question_2}"
3. "{question_3}"

### Content Ideas (Ready to Execute)

| # | Title Idea | Format | Platform | Why It'll Work |
|---|-----------|--------|----------|----------------|
| 1 | {title} | Video | YouTube | {reason} |
| 2 | {title} | Thread | X | {reason} |
| 3 | {title} | Blog | SEO | {reason} |
| 4 | {title} | Short | TikTok | {reason} |
| 5 | {title} | Post | Reddit | {reason} |

### Trending Formats in This Niche
- {format_1}: {description}
- {format_2}: {description}
```

## Error Handling

- If a platform returns **empty results**, skip it and note "No data found on {platform}" in the report — do not fail the entire workflow.
- If the **API key is missing or invalid**, stop and instruct the user: set `BYCRAWL_API_KEY` environment variable with a valid key from [bycrawl.com](https://bycrawl.com).
- If a **rate limit** is hit, reduce `count` parameters by half and retry once.
- If a specific **MCP tool is unavailable**, check that the ByCrawl MCP server is installed (`npx @bycrawl/mcp`).
- **Always deliver partial results** rather than failing entirely — a report covering 5 of 7 platforms is still valuable.

## Estimated API Usage

- **Basic run**: ~15-20 API calls
- **Full run**: ~30-40 with transcript and comment pulls API calls
- Each API call consumes 1 ByCrawl credit. Reduce `count` parameters for cost-sensitive usage.
