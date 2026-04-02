# claude-blog

![Claude Blog — AI-Powered Blog Creation](assets/header.jpeg)

![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-blueviolet)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-blue)
![Sub-Skills](https://img.shields.io/badge/Sub--Skills-15-orange)
![ByCrawl MCP](https://img.shields.io/badge/ByCrawl-MCP_Powered-ff6b35)

**The most comprehensive blog creation skill for Claude Code — powered by [ByCrawl](https://bycrawl.com) social intelligence.**

Strategy, briefs, calendars, writing, optimization, schema, repurposing, and full-site audits — all from slash commands. Dual-optimized for Google rankings and AI citation platforms (ChatGPT, Perplexity, AI Overviews). ByCrawl MCP enriches every phase with live social signals from Reddit, X, TikTok, YouTube, Threads, LinkedIn, and Instagram.

### [Watch the Demo](https://www.youtube.com/watch?v=AeLC4iutG8w)

![Blog commands demo](assets/blog-command-demo.gif)
---

## Quick Start

### Install from Claude Code Plugin Marketplace (Recommended)

```bash
/plugin marketplace add signalsurf-ai/bycrawl-blog
/plugin install blog@bycrawl-blog
```

That's it. Claude Code automatically downloads the skill, registers all 15 sub-skills, and makes `/blog` commands available immediately.

### Update

```bash
/plugin marketplace update bycrawl-blog
/plugin install blog@bycrawl-blog
```

### Install via shell script

**One-command install (Unix/macOS):**

```bash
curl -fsSL https://raw.githubusercontent.com/Signalsurf-ai/bycrawl-blog/main/install.sh | bash
```

**Or clone and install manually:**

```bash
git clone https://github.com/Signalsurf-ai/bycrawl-blog.git
cd bycrawl-blog
chmod +x install.sh && ./install.sh
```

**Windows (PowerShell):**
```powershell
.\install.ps1
```

Restart Claude Code after installation to activate.

## Commands
![Blog write command demo](assets/blog-write-demo.gif)
| Command | Description |
|---------|-------------|
| `/blog write <topic>` | Write a new blog post from scratch |
| `/blog rewrite <file>` | Optimize an existing blog post |
| `/blog analyze <file>` | Quality audit with 0-100 score |
| `/blog brief <topic>` | Generate a detailed content brief |
| `/blog calendar` | Generate an editorial calendar |
| `/blog strategy <niche>` | Blog strategy and topic ideation |
| `/blog outline <topic>` | SERP-informed content outline |
| `/blog seo-check <file>` | Post-writing SEO validation |
| `/blog schema <file>` | Generate JSON-LD schema markup |
| `/blog repurpose <file>` | Repurpose for social, email, YouTube |
| `/blog geo <file>` | AI citation readiness audit |
| `/blog audit [directory]` | Full-site blog health assessment |
| `/blog content-research <topic>` | Content gap analysis and idea mining via bycrawl |


## Features

### ByCrawl Social Intelligence Integration

[ByCrawl MCP](https://bycrawl.com) enriches blog production with live cross-platform social data (~42-52 API calls per article). Social signals correlate **3x more with AI visibility than backlinks**.

#### Why ByCrawl + claude-blog?

Without ByCrawl, the blog skill relies on `WebSearch` — you get Google results but miss the demand signals, audience language, content gaps, and social proof that live on social platforms. ByCrawl turns the skill from a search-optimized writer into a **social-intelligence-powered content engine**.

| Without ByCrawl | With ByCrawl |
|----------------|-------------|
| Topics chosen by gut instinct or keyword tools | Topics validated by real cross-platform demand (Reddit, X, TikTok, YouTube, Threads) |
| Generic AI-sounding copy | Copy infused with real audience language — hooks, pain points, and phrasing mined from transcripts and comments |
| FAQ sections based on guesses | FAQ items sourced from actual YouTube/TikTok comments people are asking right now |
| No competitive social context | Competitor messaging gaps identified — what they say, how audiences react, what they miss |
| Publish and hope | Post-write freshness check confirms the keyword still has active social volume |
| Standard sourced statistics | Social proof with real engagement metrics: `"A Reddit thread with 4.2K upvotes revealed..."` |
| Content that ranks | Content that ranks **and** gets cited by AI systems — social signals are the #1 predictor of AI visibility |

#### ByCrawl Across All Skills

ByCrawl is integrated into **11 of 13 sub-skills** — the entire content lifecycle, not just writing:

| Skill | What ByCrawl Adds | API Calls |
|-------|-------------------|-----------|
| `/blog write` | Topic validation, content gaps, comment mining, audience language, social proof, post-write validation | ~42-52 |
| `/blog rewrite` | Content gap analysis, audience language for rewrites, social proof embedding | ~15-20 |
| `/blog brief` | Social demand validation, audience pain points, FAQ sourcing from comments | ~20-23 |
| `/blog outline` | Question-format H2 candidates from social, content gaps | ~8-12 |
| `/blog strategy` | Competitor social audit, audience demand mapping, language mining | ~20-30 |
| `/blog calendar` | Trending topic detection, social volume for scheduling priority | ~10-15 |
| `/blog analyze` | Social visibility check, brand presence assessment | ~4-6 |
| `/blog audit` | Per-post social demand validation (Strong/Moderate/Weak) | ~2-3/post |
| `/blog repurpose` | Platform-specific intelligence (Reddit, X, YouTube, LinkedIn) | ~8-12 |
| `/blog geo` | Social brand signals for AI citation audits | ~4-6 |

Skills without ByCrawl (structural — no social data needed): `blog-schema`, `blog-seo-check`, `blog-humanize`

#### How ByCrawl Enriches Each Phase

| Phase | What ByCrawl Adds | API Calls |
|-------|-------------------|-----------|
| **Topic Validation** | Cross-platform demand check across Reddit, X, TikTok, YouTube, Threads — catches low-demand topics before you write | ~8 |
| **Content Gap Analysis** | Finds what's saturated vs. underserved across platforms, mines competitor blind spots | ~8 |
| **Comment Mining** | Extracts real audience questions from YouTube/TikTok comments → FAQ items and H2 topics | ~6-10 |
| **Audience Language** | Pulls hooks, pain points, and desire language from transcripts and emotional-trigger posts | ~10 |
| **Competitor Social Audit** | Checks competitor messaging, audience sentiment, and positioning gaps | ~6-9 |
| **Post-Write Validation** | Confirms keyword still has active social volume before you publish | ~4 |

**Platforms covered**: Reddit, X/Twitter, TikTok, YouTube, Threads, LinkedIn, Instagram, Facebook, Dcard

**How it works in `/blog write`**:
```
Phase 1.3 → Social demand validation (strong/moderate/weak signal)
Phase 2   → Content gap analysis + comment mining + competitor audit
Phase 5   → Audience language mining → natural copy that resonates
Phase 5n  → Social proof embedding with real engagement metrics
Phase 6   → Post-write keyword freshness check
```

Social proof gets embedded inline with attribution:
- `"A discussion in r/{subreddit} with {upvotes} upvotes highlighted..."`
- `"In a video with {views} views, {channel} demonstrated..."`
- Each earns a `[SOCIAL DATA]` information gain marker

ByCrawl is optional — every skill gracefully falls back to `WebSearch`-only research if no API key is configured.

### 12 Content Templates
Auto-selected based on topic and intent: how-to guide, listicle, case study, comparison, pillar page, product review, thought leadership, roundup, tutorial, news analysis, data research, FAQ knowledge base.

### 5-Category Quality Scoring (100 Points)
| Category | Points | Focus |
|----------|--------|-------|
| Content Quality | 30 | Depth, readability, originality, engagement |
| SEO Optimization | 25 | Headings, title, keywords, links, meta |
| E-E-A-T Signals | 15 | Author, citations, trust, experience |
| Technical Elements | 15 | Schema, images, speed, mobile, OG tags |
| AI Citation Readiness | 15 | Citability, Q&A format, entity clarity |

Scoring bands: Exceptional (90-100), Strong (80-89), Acceptable (70-79), Below Standard (60-69), Rewrite (<60).

### AI Content Detection
Burstiness scoring, known AI phrase detection (17 phrases), vocabulary diversity analysis (TTR). Flags content that reads as AI-generated.

### Dual Optimization
Every article targets both Google rankings and AI citation platforms:
- **Google**: December 2025 Core Update compliance, E-E-A-T, schema markup, internal linking
- **AI Citations**: Answer-first formatting (+340% citations), citation capsules, passage-level citability, FAQ schema (+28% citations)

### Visual Media
- Pixabay/Unsplash/Pexels image sourcing with alt text
- Image density targets by content type
- Image URL verification (HTTP 200 check before embedding)

### Platform Support
Next.js/MDX, Astro, Hugo, Jekyll, WordPress, Ghost, 11ty, Gatsby, and static HTML.

## Architecture

```
claude-blog/
├── .claude-plugin/
│   └── plugin.json                     # Plugin metadata (name, description, author)
├── skills/
│   ├── blog/                           # Main orchestrator
│   │   ├── SKILL.md                    # Routes all 12 commands
│   │   ├── references/                 # 12 on-demand reference docs
│   │   └── templates/                  # 12 content type templates
│   ├── blog-write/SKILL.md            # Sub-skills (12 user-facing + 1 internal)
│   ├── blog-rewrite/SKILL.md
│   ├── blog-analyze/SKILL.md
│   ├── blog-brief/SKILL.md
│   ├── blog-calendar/SKILL.md
│   ├── blog-strategy/SKILL.md
│   ├── blog-outline/SKILL.md
│   ├── blog-seo-check/SKILL.md
│   ├── blog-schema/SKILL.md
│   ├── blog-repurpose/SKILL.md
│   ├── blog-geo/SKILL.md
│   ├── blog-audit/SKILL.md
│   └── blog-content-research/SKILL.md
├── agents/                             # 4 specialized agents
│   ├── blog-researcher.md
│   ├── blog-writer.md
│   ├── blog-seo.md
│   └── blog-reviewer.md
├── scripts/
│   └── analyze_blog.py                 # Python quality analysis (5-category scoring)
├── tests/                              # pytest test suite
│   ├── conftest.py
│   └── test_analyze_blog.py
├── docs/                               # 6 documentation files
├── .github/workflows/ci.yml           # CI pipeline
├── install.sh                          # Unix/macOS installer (fallback)
├── install.ps1                         # Windows PowerShell installer
├── pyproject.toml                      # Python project config
├── requirements.txt                    # Python dependencies
├── CONTRIBUTING.md
├── CHANGELOG.md
├── LICENSE
└── README.md
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed and configured
- Python 3.11+ (for `analyze_blog.py` quality scoring script)
- Optional: `pip install -r requirements.txt` for advanced analysis (readability scoring, schema detection)

## Uninstall

**If installed via marketplace:**
```bash
/plugin uninstall bycrawl-blog
```

**If installed via shell script:**

Unix/macOS:
```bash
chmod +x uninstall.sh && ./uninstall.sh
```

Windows (PowerShell):
```powershell
.\uninstall.ps1
```

## ByCrawl MCP Setup

Add ByCrawl to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "bycrawl": {
      "command": "npx",
      "args": ["-y", "@bycrawl/mcp"],
      "env": {
        "BYCRAWL_API_KEY": "your-api-key"
      }
    }
  }
}
```

Or add globally to `~/.claude/settings.json` to use across all projects.

Get your API key at [bycrawl.com](https://bycrawl.com).

### ByCrawl Tools Used by claude-blog

| Tool | Used In | Purpose |
|------|---------|---------|
| `reddit_search_posts` | Topic validation, gap analysis | Demand signals, content gaps, audience questions |
| `x_search_posts` | Topic validation, language mining | Trending angles, emotional triggers, competitor mentions |
| `tiktok_search_videos` | Topic validation, gap analysis | Visual content trends, emerging topics |
| `youtube_search_videos` | Gap analysis, competitor audit | Top-performing content, competitor positioning |
| `youtube_get_video_transcription` | Language mining | Audience vocabulary, proven hooks |
| `youtube_get_video_comments` | Comment mining | Real audience questions → FAQ items |
| `tiktok_get_video_comments` | Comment mining | Pain points → H2 topic candidates |
| `tiktok_get_video_subtitles` | Language mining | Platform tone, natural phrasing |
| `threads_search_posts` | Topic validation | Cross-platform volume confirmation |

## Integration

Chart generation is built-in — no external dependencies required for core functionality.

**Optional companion skills** (for deeper analysis of published pages):

| Skill | Integration |
|-------|-------------|
| `/seo` | Deep SEO analysis of published blog pages |
| `/seo-schema` | Schema markup validation and generation |
| `/seo-geo` | AI citation optimization audit (also uses ByCrawl for social brand signals) |

## Documentation

Detailed documentation is available in [docs/](docs/):

- [Installation Guide](docs/INSTALLATION.md) -- Unix, macOS, Windows, manual install
- [Command Reference](docs/COMMANDS.md) -- Full 12-command reference with examples
- [Architecture](docs/ARCHITECTURE.md) -- System design and component overview
- [Templates](docs/TEMPLATES.md) -- Template reference and customization
- [Troubleshooting](docs/TROUBLESHOOTING.md) -- Common issues and fixes
- [MCP Integration](docs/MCP-INTEGRATION.md) -- Optional MCP server setup

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT License. See [LICENSE](LICENSE) for details.

---

Built by [signalsurf.ai](https://github.com/Signalsurf-ai) with Claude Code. Social intelligence powered by [ByCrawl](https://bycrawl.com).
