# Blog publish dates — owner permission required

**Owner directive (2026-08-13):** Do not change blog post `published_at` dates, bylines,
JSON-LD `datePublished`, RSS `pubDate`, or sitemap `lastmod` values without explicit
permission from the site owner (Jennie / earlyed).

## Why

Historical posts were intentionally backdated across **2022–2025** so the journal reads
as a natural archive. New posts may use the real publish date when they go live.

## Allowed without asking

- Publishing a **new** post with today's date (normal `site_publisher` flow)
- Fixing a typo in title or body copy (no date change)

## Requires owner permission

- Changing the year, month, or day on any **existing** post
- Bulk remapping dates (e.g. `amend_blog_years.py`)
- Hiding or showing byline dates (`no_visible_date`) on posts that already went live

## Where dates live

Each `blog/*.html` file stores dates in:

1. `<!-- boonhouse:meta {"published_at": "..."} -->`
2. JSON-LD `"datePublished"`
3. Optional byline: `By BoonHouse Publishing · YYYY-MM-DD`

Regenerated together: `blog/index.html`, `rss.xml`, `sitemap.xml`.

If you need a date changed, ask the owner first.
