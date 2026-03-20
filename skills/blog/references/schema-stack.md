# Complete Blog Schema Reference

## Why Schema Matters

72% of first-page results use structured data markup. Pages using 3+ schema
types have approximately 13% higher likelihood of AI citation. Schema must
appear in HTML source -- not injected via JavaScript -- because most AI crawlers
do not execute JS.

---

## byCrawl BlogPosting Schema (Primary)

The primary schema for every blog post. Uses a **flat structure** with inline
author and publisher objects, rendered by the `<JsonLd>` component.

### Complete BlogPosting Example (matches code)

```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "Complete Guide to Technical SEO in 2026",
  "description": "Technical SEO has evolved beyond Core Web Vitals. 72% of top-ranking pages now use structured data.",
  "datePublished": "2026-01-15",
  "dateModified": "2026-02-10",
  "url": "https://bycrawl.com/blog/technical-seo-guide",
  "wordCount": 3200,
  "keywords": ["technical SEO", "structured data", "schema markup"],
  "articleSection": "SEO",
  "author": {
    "@type": "Person",
    "name": "Kyle Chung",
    "url": "https://bycrawl.com",
    "jobTitle": "Founder",
    "sameAs": [
      "https://x.com/kyelchung",
      "https://linkedin.com/in/kyelchung"
    ],
    "worksFor": {
      "@type": "Organization",
      "name": "byCrawl"
    }
  },
  "publisher": {
    "@type": "Organization",
    "name": "byCrawl",
    "url": "https://bycrawl.com"
  }
}
```

### Property Reference

| Property | Required | Type | Description |
|----------|----------|------|-------------|
| `@context` | Yes | URL | Always `"https://schema.org"` |
| `@type` | Yes | String | Always `"BlogPosting"` |
| `headline` | Yes | String | Post title, max 110 characters |
| `description` | Yes | String | Post summary |
| `datePublished` | Yes | ISO 8601 | Original publish date |
| `dateModified` | Yes | ISO 8601 | Falls back to datePublished |
| `url` | Yes | URL | `https://bycrawl.com/blog/{slug}` |
| `wordCount` | Yes | Integer | Computed from content |
| `keywords` | Conditional | Array | Only if post has keywords |
| `articleSection` | Conditional | String | Only if post has articleSection |
| `author` | Yes | Person | Inline Person object |
| `publisher` | Yes | Organization | Inline Organization object |

### Conditional Fields

`keywords` and `articleSection` are spread conditionally -- they are only present
in the output if they exist in the post frontmatter:

```typescript
...(post.keywords && { keywords: post.keywords }),
...(post.articleSection && { articleSection: post.articleSection }),
```

---

## Person Schema (Inline Author)

The author is embedded directly in the BlogPosting -- not referenced via @id.

### byCrawl Default Author

```json
{
  "@type": "Person",
  "name": "Kyle Chung",
  "url": "https://bycrawl.com",
  "jobTitle": "Founder",
  "sameAs": [
    "https://x.com/kyelchung",
    "https://linkedin.com/in/kyelchung"
  ],
  "worksFor": {
    "@type": "Organization",
    "name": "byCrawl"
  }
}
```

### Property Reference

| Property | Required | Type | Description |
|----------|----------|------|-------------|
| `@type` | Yes | String | Always `"Person"` |
| `name` | Yes | String | Author name (default: "Kyle Chung") |
| `url` | Yes | URL | Author URL (default: "https://bycrawl.com") |
| `jobTitle` | Yes | String | Role (default: "Founder") |
| `sameAs` | Yes | Array | Social profile URLs |
| `worksFor` | Yes | Organization | Inline Organization with name only |

If the post has a custom `post.author`, use that name. The fallback is
`post.author || "Kyle Chung"`.

---

## Organization Schema (Inline Publisher)

The publisher is embedded directly in the BlogPosting.

### byCrawl Publisher

```json
{
  "@type": "Organization",
  "name": "byCrawl",
  "url": "https://bycrawl.com"
}
```

This is a minimal Organization object. The code does not include `logo`,
`sameAs`, or `contactPoint` in the publisher schema.

---

## Word Count Computation

The `countWords` function strips HTML and markdown syntax before counting:

```typescript
function countWords(content: string): number {
  const text = content.replace(/<[^>]*>/g, '').replace(/[#*_`~\[\](){}|>-]/g, '');
  return text.split(/\s+/).filter(Boolean).length;
}
```

This runs on the raw markdown content, not the rendered HTML.

---

## Additional Schema Types (Separate Components)

These types are NOT part of the main BlogPosting JSON-LD but can be added as
separate `<JsonLd>` component instances on the page.

### BreadcrumbList

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://bycrawl.com"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Blog",
      "item": "https://bycrawl.com/blog"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Post Title",
      "item": "https://bycrawl.com/blog/{slug}"
    }
  ]
}
```

Rules:
- Always start with Home (position 1)
- Positions must be sequential integers starting at 1
- Final item is the current page

### FAQPage

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is the question?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "The complete answer text (40-60 words)."
      }
    }
  ]
}
```

Note: Since August 2023, FAQ rich results are restricted to government and
health sites. The markup still helps AI systems extract Q&A pairs for citation.

Guidelines:
- 3-5 FAQ items per page
- Answers should be 40-60 words
- Questions should match real user queries
- Each answer should be self-contained

---

## Deprecated Schema Types -- NEVER Use

| Type | Deprecated | Date | Notes |
|------|------------|------|-------|
| HowTo | Yes | September 2023 | Rich results removed entirely |
| SpecialAnnouncement | Yes | July 2025 | COVID-era, no longer processed |
| Practice Problem | Yes | -- | Educational, no longer generates rich results |
| Dataset | Yes | -- | For general search; still works in Google Dataset Search |
| Sitelinks Search Box | Yes | -- | Google generates these algorithmically now |
| Q&A | Yes | January 2026 | Replaced by community forum features |

### What to Use Instead

| Deprecated Type | Alternative |
|----------------|-------------|
| HowTo | Use standard BlogPosting with clear step headings (H2/H3) |
| Q&A | Use FAQPage for editorial Q&A |
| SpecialAnnouncement | Use standard Article or NewsArticle |

---

## Schema Validation Checklist

| Check | Pass | Fail |
|-------|------|------|
| JSON-LD in HTML source (not JS-injected) | In `<head>` or `<body>` tag | Loaded via JavaScript |
| Valid JSON syntax | Passes JSON.parse() | Syntax errors |
| @context is `https://schema.org` | Exact match | Missing or HTTP |
| dateModified >= datePublished | Same day or later | Earlier than publish |
| Author name is present | Non-empty string | Missing |
| All URLs are absolute | Start with `https://` | Relative paths |
| No deprecated types used | None from deprecated list | HowTo, Q&A, etc. |
| headline <= 110 characters | Within limit | Exceeds limit |
| Validates in Google Rich Results Test | No errors | Errors present |

### Validation Tools

- **Google Rich Results Test**: https://search.google.com/test/rich-results
- **Schema.org Validator**: https://validator.schema.org
- **JSON-LD Playground**: https://json-ld.org/playground/
