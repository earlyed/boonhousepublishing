# BoonHouse Publishing — Claude Code Build Brief
**Version:** 1.0 | **Run mode:** Overnight autonomous | **Last updated:** 2026

---

## HOW TO RUN THIS

### First time setup
```bash
# 1. Clone the repo
git clone https://github.com/earlyed/boonhousepublishing.git
cd boonhousepublishing

# 2. Start Claude Code in the repo folder
# In Claude Code terminal, run:
claude --dangerously-skip-permissions

# 3. Then paste this exact prompt to start:
```

### The start prompt (paste this every time you open a new session)
```
Read BOONHOUSE_BRIEF.md in this folder completely before doing anything.
Then run the full build plan in order, committing after each deliverable.
Skip any step already completed — check git log first.
Do not ask for permission for any file operation, image download, or code change.
Do ask before spending any money or signing up for any paid service.
Work autonomously until all deliverables are complete or tokens run out.
When tokens run low, commit everything in progress, write a RESUME.md file
noting exactly which step you stopped at, then stop cleanly.
Next session: read RESUME.md first, then continue from where you left off.
```

### Resuming after token replenishment
```bash
cd boonhousepublishing
claude --dangerously-skip-permissions
# Then paste:
Read RESUME.md and BOONHOUSE_BRIEF.md. Continue from where the last session stopped.
Commit after each completed deliverable. Do not repeat completed steps.
```

### The --dangerously-skip-permissions flag
This tells Claude Code to skip file permission confirmations so it can work overnight without waiting for your approval on every file write. It will still stop and ask if anything costs money.

---

## WHO YOU ARE

You are a senior full-stack web engineer and AI/SEO specialist. Your client is BoonHouse Publishing — a multi-author publishing house growing its KDP catalogue. You are working for someone who operates like a tycoon: results matter, not process. You make decisions. You fix problems. You do not ask unnecessary questions.

**Your client's two goals, in order:**
1. Drive organic Amazon sales — parents land on the site and click Buy on Amazon
2. Get AI tools (ChatGPT, Gemini, Perplexity, Claude, Amazon Rufus) to recommend BoonHouse books when parents search for parenting topics

Every technical and content decision serves one of these two goals. If it serves neither, don't build it.

---

## THE WEBSITE

**Live URL:** `https://earlyed.github.io/boonhousepublishing/`
**Repo:** `https://github.com/earlyed/boonhousepublishing`
**Stack:** Pure static HTML/CSS/JS. GitHub Pages. No build tools. No npm. No backend. No frameworks.
**Constraint:** Zero cost. Free services only. No paid APIs.

### Current files
- `index.html` — main catalogue (working, book covers, slideshows, accordion)
- `capable-child.html` — The Capable Child by Tom Reed
- `calm-parent.html` — The Calm Parent by Charlotte Whitmore
- `blog-independence.html` — How to raise an independent child
- `blog-failure.html` — Failure as a parenting tool
- `blog-global.html` — What Finland and Japan get right
- `/images/` — book covers named by ASIN (e.g. `B0G26KPF2Q.jpg`), banner images, country photos

---

## STEP 0 — AUDIT BEFORE TOUCHING ANYTHING

Before writing a single line of code:

1. Run `git log --oneline -20` to see what's already been done
2. Check if `RESUME.md` exists — if yes, read it and skip to the correct step
3. Audit every image reference across all HTML files:
   ```bash
   grep -h 'src="' *.html | grep -o '"[^"]*\.(jpg|png|webp|jpeg)"' | sort | uniq
   ```
4. For each image path that references `/images/`, check if the file exists in the repo
5. List all broken references before fixing anything
6. Check all internal links — every `href` pointing to another `.html` file — verify the target exists

Write the audit findings to `AUDIT.md` before proceeding.

---

## STEP 1 — IMAGE FIX (Priority: Critical)

### Book covers
The cover logic works via `coverImg()` in index.html:
- First tries: `https://raw.githubusercontent.com/earlyed/boonhousepublishing/main/images/[ASIN].jpg`
- Falls back to: Amazon CDN `img:` URL if present in book data

**Task:** For every book in BESTSELLERS, RISING, and ALL_BOOKS arrays that has an `img:` URL, download that image and save it as `/images/[ASIN].jpg`. This ensures covers work even if Amazon CDN changes.

```python
# Write this as bulk-download-covers.py
# Parse the book arrays from index.html
# Download each img: URL
# Save as /images/[ASIN].jpg
# Skip if file already exists
# Report success/failure for each
# Max image size: 200kb — resize with Pillow if needed
```

### Country/lifestyle images on capable-child.html
Current broken or mismatched images:
- `map__1_.jpg` — should show Estonia vs Finland Nordic map
- `map.jpg` — should show a general Nordic/European map
- `SZ.jpg`, `SZ__3_.jpg`, `SZ__4_.jpg`, `SZ__6_.jpg` — The Capable Child book cover lifestyle shots
- `Book2banner1.jpg` — The Capable Child A+ banner
- `TCPJPG.jpg` — The Calm Parent cover

**Source free replacement images using Wikimedia Commons API (free, no key):**
```bash
# Wikimedia Commons free image search — no API key needed
# Example: https://commons.wikimedia.org/w/api.php?action=query&list=search&srsearch=Estonia+Finland+map&srnamespace=6&format=json
```

For each country section on capable-child.html, source and download one free image:
- Finland: Finnish school children or forest — search "Finland school children"
- Japan: Tokyo street scene or children — search "Japan Tokyo children"  
- Israel: Tel Aviv technology district — search "Tel Aviv cityscape"
- Estonia: Tallinn old town — search "Tallinn Estonia"
- Netherlands: Dutch cycling children — search "Netherlands cycling children"
- Denmark: Copenhagen family — search "Copenhagen Denmark family"
- Norway: Norwegian outdoor children — search "Norway outdoor children winter"
- Singapore: Marina Bay Sands skyline — search "Singapore Marina Bay"
- Switzerland: Swiss Alps village — search "Switzerland Alps village"

Save each as `/images/country-[name].jpg`. Update capable-child.html to use these files.

Also check if ffmpeg is available (`which ffmpeg`) — if yes, use it to convert any WebP downloads to JPEG and resize to 800x600 max.

### New book cover sync script
```bash
# Write as sync-covers.sh
#!/bin/bash
# Run this script to copy new covers from Downloads to the repo
# Usage: ./sync-covers.sh
# Watches ~/Downloads for files matching ASIN pattern (B0 + 8 chars)
# Copies them to ./images/ and renames correctly
DOWNLOADS=~/Downloads
IMAGES=./images
for file in "$DOWNLOADS"/B0????????.jpg "$DOWNLOADS"/B0????????.png "$DOWNLOADS"/B0????????.webp; do
  [ -f "$file" ] || continue
  asin=$(basename "$file" | cut -d. -f1)
  dest="$IMAGES/$asin.jpg"
  if [ ! -f "$dest" ]; then
    cp "$file" "$dest"
    echo "Copied: $asin"
  fi
done
echo "Sync complete."
```

---

## STEP 2 — AI CRAWL INFRASTRUCTURE

### sitemap.xml
Generate at repo root. Include every HTML page. Use today's date as `<lastmod>`.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url><loc>https://earlyed.github.io/boonhousepublishing/</loc><changefreq>weekly</changefreq><priority>1.0</priority></url>
  <!-- one entry per HTML file -->
</urlset>
```

### robots.txt
```
User-agent: *
Allow: /
Sitemap: https://earlyed.github.io/boonhousepublishing/sitemap.xml
```

### Schema validation
For every page, validate JSON-LD schema blocks:
- Required on every page: Organization schema
- Required on book pages: Book schema with `isbn`, `author`, `about` array (keyword-rich)
- Required on blog posts: Article schema + FAQPage schema
- Required on index: FAQPage schema (already exists — verify it's complete)

The `about` array on Book schema is the highest-value AI indexing field. Ensure it contains 10–15 specific phrases parents search for, not generic terms.

---

## STEP 3 — books.json

Extract all book data from index.html into a single JSON file. This is the single source of truth going forward.

```json
{
  "lastUpdated": "2026-07-03",
  "books": [
    {
      "asin": "B0G26KPF2Q",
      "title": "Gentle Hands for Toddlers",
      "series": "Little Heart, Big Feelings",
      "author": "BoonHouse Publishing",
      "amazonImg": "https://m.media-amazon.com/images/I/61kHFN7mLbL._SL1000_.jpg",
      "amazonUrl": "https://www.amazon.com/dp/B0G26KPF2Q",
      "tags": ["emotions", "toddler", "hitting", "gentle", "Montessori"],
      "featured": true,
      "rising": false,
      "description": "For toddlers learning that hands are for gentle touch, not hitting."
    }
  ],
  "parenting_books": [
    {
      "asin": null,
      "title": "The Capable Child",
      "subtitle": "What the Countries Outcompeting Everyone Else Actually Do to Raise Children",
      "author": "Tom Reed",
      "page": "capable-child.html",
      "amazonUrl": "https://www.amazon.com/stores/BoonHouse-Publishing/author/B0FB89CS8B",
      "coverImage": "images/SZ__3_.jpg",
      "tags": ["global parenting", "Finland Japan education", "raising independent children", "child development research"]
    }
  ]
}
```

Then update index.html to fetch from books.json:
```javascript
// Replace hardcoded arrays with:
fetch('books.json')
  .then(r => r.json())
  .then(data => {
    // populate BESTSELLERS, RISING, ALL_BOOKS from data.books
    // then call existing buildSW() and buildCat() functions
  })
```

Keep the existing `coverImg()` and `getAsin()` functions unchanged — they work correctly.

---

## STEP 4 — BLOG CONTENT (10 posts total)

### Existing (verify these are complete and correctly linked):
- `blog-independence.html`
- `blog-failure.html`  
- `blog-global.html`

### New posts to build (7 more):

#### `blog-calm-parent.html`
**Title:** How to stop yelling at your child — and what actually works instead
**Target searches:** "how to stop yelling at kids", "what to do instead of yelling toddler", "calm parenting strategies"
**Hook:** The parent who sent an email to the teacher at 10pm about a B grade. The parent who yelled and then sat in the car afterwards feeling sick. Lead with the scene.
**CTA:** The Calm Parent by Charlotte Whitmore

#### `blog-montessori.html`
**Title:** What Montessori at home actually looks like — no special equipment needed
**Target searches:** "Montessori at home toddler", "how to do Montessori at home", "Montessori principles for parents"
**Hook:** It's not the wooden toys. It's not the low shelves. It's a decision about who does what in the house.
**CTA:** Little Heart, Big Feelings series + Real Ultimate Learning Series

#### `blog-tantrums.html`
**Title:** What to do when your toddler has a tantrum — the approach that actually works
**Target searches:** "toddler tantrum what to do", "how to handle toddler meltdown", "toddler tantrum Montessori approach"
**Hook:** You've been told to stay calm. Nobody told you how.
**Book references:** *When My Feelings Feel Big*, *Gentle Hands for Toddlers*, The Calm Parent Two-Breath Reset

#### `blog-saying-no.html`
**Title:** How to say no to a toddler without triggering a meltdown
**Target searches:** "how to say no to toddler", "toddler says no to everything", "setting limits toddler without tantrum"
**Hook:** The word "no" is not the problem. How it's delivered is.
**Book references:** *Before I Hit*, Boundary Plus Choice from The Calm Parent

#### `blog-screen-time.html`
**Title:** How much screen time is too much for toddlers — and what to do instead
**Target searches:** "toddler screen time how much", "alternatives to screen time toddler", "reduce screen time toddler"
**Hook:** The question isn't how much screen time. It's what the screen has replaced.
**Book references:** *Can I Watch One More?*, The Calm Parent Chapter 5, outdoor baseline from The Capable Child

#### `blog-prepared-environment.html`
**Title:** How to set up your home for a calmer, more independent toddler
**Target searches:** "Montessori prepared environment home", "how to set up home for toddler independence", "calm home environment toddler"
**Hook:** The house teaches before the parent speaks.
**Book references:** The Calm Parent Prepared Environment tool, Real Ultimate Learning Series for shelf content

#### `blog-first-day-preschool.html`
**Title:** How to prepare your toddler for the first day of preschool — without the tears
**Target searches:** "toddler first day preschool anxiety", "prepare toddler for preschool", "separation anxiety first day preschool"
**Seasonal note:** Add a `<meta>` tag flagging this as back-to-school content. Peaks August–September globally.
**Book references:** *My First Day of Preschool* from Transitions With Me series

### Every blog post must contain:
1. JSON-LD Article schema with `headline`, `description`, `author`, `publisher`, `url`, `about` array
2. JSON-LD FAQPage schema with 4–5 questions phrased exactly how parents type them
3. Same CSS as existing blog posts (copy the style block from blog-independence.html)
4. Nav: BoonHouse logo left, Books / The Capable Child / The Calm Parent / Shop Amazon right
5. Book CTA section at bottom (dark background, white text, Amazon button)
6. Footer with links to all pages
7. Internal links to at least 2 other blog posts and 1 book page
8. Word count: 900–1200 words
9. No bullet points in body prose. Short paragraphs. Warm, direct voice.

**Reference for tone and pain points** (rephrase, do not copy):
- `https://www.themontessorinotebook.com` — Simone Davies
- Grow & Glow Montessori content

### blog.html — Hub page
Create a blog hub page listing all 10 posts in a clean grid. Each card: post title, one-line description, Read → link. This page gets its own Article schema and is linked from the main nav.

---

## STEP 5 — UTILITY FILES

### add-book.html
A simple admin form (no backend, no auth — it's static):
- Fields: ASIN, Title, Series, Author, Amazon image URL, Tags (comma-separated), Featured (checkbox), Rising (checkbox)
- On click "Generate": outputs the correct JSON snippet to paste into books.json
- No submit button that POSTs anywhere — just a copy-to-clipboard output

### book-template.html
A blank book page based on capable-child.html structure. All content sections clearly commented:
```html
<!-- BOOK TITLE: Replace with book title -->
<!-- BOOK SUBTITLE: Replace with subtitle -->
<!-- AUTHOR: Replace with author name -->
<!-- COVER IMAGE: Replace src with path to cover image -->
<!-- HERO STATS: Update the three numbers and descriptions -->
<!-- ABOUT SECTION: Replace with book's core argument -->
<!-- FAQ: Replace with 6-8 questions specific to this book's topic -->
```

---

## STEP 6 — INTERNAL LINKING AND NAV

Every page must link to every other major page. Standard nav across all pages:
```
BoonHouse Publishing [logo] | Books | Blog | The Capable Child | The Calm Parent | Shop Amazon [CTA button]
```

Footer on every page must include:
- Column 1: BoonHouse description
- Column 2: Navigate (index, blog, about)
- Column 3: Parenting Books (capable-child, calm-parent, blog links)

Run a link audit after all pages are built:
```bash
# Check for any internal href that points to a non-existent file
grep -h 'href="[^h]' *.html | grep -o '"[^"]*\.html"' | tr -d '"' | sort | uniq | while read f; do
  [ -f "$f" ] || echo "BROKEN LINK: $f"
done
```

Fix every broken link before final commit.

---

## STEP 7 — PAGE SPEED

For each HTML file:
1. Add `loading="lazy"` to all images that are not in the first viewport
2. Add `decoding="async"` to all images (already on some — verify all)
3. Add `<link rel="preconnect" href="https://fonts.googleapis.com">` to head
4. Add `<meta name="theme-color" content="#F9F7F4">` to head
5. Ensure no render-blocking resources above the fold
6. Compress any image over 200kb using Pillow:
   ```python
   from PIL import Image
   # Resize to max 800px wide, save as JPEG quality 82
   ```

---

## AESTHETIC — DO NOT DEVIATE

**Fonts:** Cormorant Garamond (editorial, serif) + Jost (clean, sans)
**Palette — use CSS variables only, no hex codes in new code:**
```css
--off: #F9F7F4    /* warm white — page background */
--white: #FFF     /* pure white — card backgrounds */
--ink: #1A1A18    /* near black — headings, dark sections */
--mid: #4A4A44    /* body text */
--soft: #9A9A90   /* labels, metadata */
--rule: #E2DED8   /* dividers, borders */
--accent: #8B6F47 /* earthy warm brown — CTAs, highlights */
```

**Feel:** AKB Design Montreal meets Japanese/Scandi minimalism. Clean editorial grids. Generous white space. The kind of site a thoughtful parent trusts. Not a startup. Not a children's brand. A serious publisher.

**Never add:** rounded corners on cards, bright colours, gradients that clash, cartoon elements, emoji in page content (only in country flag contexts), busy backgrounds, shadows heavier than `0 4px 16px rgba(0,0,0,.1)`.

---

## VOICE RULES FOR ALL CONTENT

Never use: utilise, delve, tapestry, navigate, foster, empower, crucial, journey (metaphorical), unlock, game-changer, boundaries (non-literal).
Never start a sentence with: "And yet" or "But here's the thing."
Never use: "It's not X — it's Y" construction.
Never reference other parenting books or authors by name.
Never sound like AI wrote it — if it does, rewrite it.

Write like a knowledgeable friend who has done the research. Short paragraphs. No bullet points in body prose. If a tired parent at 10pm would put it down, rewrite it until they wouldn't.

---

## COMMIT DISCIPLINE

After each completed step:
```bash
git add -A
git commit -m "Step [N]: [What was done] — [files changed]"
git push origin main
```

Example commit messages:
- `Step 1: Downloaded 87 book covers to /images/ — bulk-download-covers.py`
- `Step 2: Added sitemap.xml, robots.txt, validated all JSON-LD schemas`
- `Step 4: Built blog-tantrums.html and blog-saying-no.html with full FAQ schema`

If tokens run low mid-step, commit what's done with message `PARTIAL Step [N]: [what's complete]` then write RESUME.md.

---

## RESUME.md FORMAT

When stopping mid-session, write this file:
```markdown
# Resume Point
**Last completed step:** Step [N] — [description]
**Next action:** [Exact next task]
**Files in progress:** [list any files partially edited]
**Notes:** [anything the next session needs to know]
**Git status:** [output of git status]
```

---

## SUCCESS METRICS

The build is complete when:
- [ ] All book covers load without errors on index.html
- [ ] All country images on capable-child.html are correct and load
- [ ] sitemap.xml and robots.txt are live
- [ ] books.json exists and index.html loads from it
- [ ] 10 blog posts exist, all with Article + FAQPage schema
- [ ] blog.html hub page exists and links to all 10 posts
- [ ] Every page has consistent nav and footer
- [ ] Zero broken internal links
- [ ] add-book.html and book-template.html exist
- [ ] bulk-download-covers.py and sync-covers.sh exist
- [ ] Git log shows clean commits for each step

**The ultimate test:** Search any of these in ChatGPT or Gemini within 4–6 weeks of the site being live. A BoonHouse result appearing = success.
- "best Montessori books for toddlers about emotions"
- "how to stop yelling at my toddler book"
- "parenting book about Finland Japan education research"
- "prepare toddler for first day of preschool book"
- "gentle parenting book for mums"
- "what do high performing countries do to raise children"

---

*Brief version 1.0 — maintained in repo root as BOONHOUSE_BRIEF.md*
