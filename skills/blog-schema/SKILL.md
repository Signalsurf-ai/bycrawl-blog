---
name: blog-schema
description: >
  Generate complete JSON-LD schema markup for blog posts matching the byCrawl
  BlogPosting component. Uses flat structure with inline author/publisher objects.
  Validates against Google requirements and warns about deprecated types. Use when
  user says "schema", "blog schema", "json-ld", "structured data", "schema markup",
  "generate schema".
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
---

# Blog Schema -- JSON-LD Structured Data Generation

Generates validated JSON-LD schema markup for blog posts using a **flat BlogPosting
structure** with inline author and publisher objects. This matches the byCrawl
`<JsonLd>` component in `@/components/json-ld`.

## Output Structure

The schema is a single flat `BlogPosting` object -- NOT the @graph pattern. The
`<JsonLd>` React component renders it as a `<script type="application/ld+json">` tag.

```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "Post title",
  "description": "Post summary",
  "datePublished": "YYYY-MM-DD",
  "dateModified": "YYYY-MM-DD",
  "url": "https://bycrawl.com/blog/{slug}",
  "wordCount": 2400,
  "keywords": ["keyword1", "keyword2"],
  "articleSection": "Category",
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

## Workflow

### Step 1: Read Content

Read the blog post and extract schema-relevant data:
- **Title** (headline)
- **Summary** (description)
- **Dates** (datePublished, dateModified -- falls back to datePublished)
- **Slug** (from filename or frontmatter)
- **Content** (for word count calculation)
- **Keywords** (from frontmatter, optional)
- **Article section** (from frontmatter, optional)
- **Author** (from frontmatter, defaults to "Kyle Chung")

### Step 2: Generate BlogPosting Schema

Build the flat BlogPosting object with these exact fields:

| Property | Required | Source |
|----------|----------|--------|
| `@context` | Yes | Always `"https://schema.org"` |
| `@type` | Yes | Always `"BlogPosting"` |
| `headline` | Yes | `post.title` |
| `description` | Yes | `post.summary` |
| `datePublished` | Yes | `post.date` (ISO 8601) |
| `dateModified` | Yes | `post.dateModified \|\| post.date` |
| `url` | Yes | `https://bycrawl.com/blog/{slug}` |
| `wordCount` | Yes | Computed from `post.content` |
| `keywords` | Conditional | Only if `post.keywords` exists |
| `articleSection` | Conditional | Only if `post.articleSection` exists |
| `author` | Yes | Inline Person object |
| `publisher` | Yes | Inline Organization object |

### Step 3: Build Author Object

The author is an **inline Person object** (not an @id reference):

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

If the post has a custom `post.author`, use that name but keep the same structure.

### Step 4: Build Publisher Object

The publisher is an **inline Organization object**:

```json
{
  "@type": "Organization",
  "name": "byCrawl",
  "url": "https://bycrawl.com"
}
```

### Step 5: Compute Word Count

Strip HTML tags and markdown syntax, then count whitespace-delimited words:

```typescript
function countWords(content: string): number {
  const text = content.replace(/<[^>]*>/g, '').replace(/[#*_`~\[\](){}|>-]/g, '');
  return text.split(/\s+/).filter(Boolean).length;
}
```

### Step 6: Conditional Fields

Only include `keywords` and `articleSection` if they exist in frontmatter:

```typescript
...(post.keywords && { keywords: post.keywords }),
...(post.articleSection && { articleSection: post.articleSection }),
```

Do NOT include these fields with empty values.

### Step 7: Validate & Warn

**Validation checks:**
1. `dateModified` is equal to or after `datePublished`
2. `headline` does not exceed 110 characters
3. `description` is between 50-160 characters
4. `url` is absolute (starts with `https://`)
5. `wordCount` is a positive integer
6. `keywords` is an array of strings (if present)

**NEVER use these deprecated types:**
- **HowTo** -- Deprecated September 2023
- **SpecialAnnouncement** -- Deprecated July 2025
- **Q&A** -- Deprecated January 2026 (distinct from FAQPage)
- **Dataset**, **Sitelinks Search Box**, **Practice Problem**

### Step 8: Output

Write the schema as frontmatter fields or as a JSON block the user can add to
their blog post's markdown file. The `<JsonLd>` component in the Next.js app
reads these fields from the post object and renders them automatically.

**What the component does NOT include** (handle separately if needed):
- BreadcrumbList (rendered by a different component or not used)
- FAQPage (can be added as a separate `<JsonLd>` instance if the post has FAQs)
- ImageObject (cover image is in OG meta, not in JSON-LD)
- @graph pattern (the component uses a flat structure)
- @id references (all entities are inline)

If the user wants additional schema types (FAQ, Breadcrumb), generate them as
separate `<JsonLd>` component instances, each with their own flat structure
including `@context`.
