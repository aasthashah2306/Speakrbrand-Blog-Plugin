---
name: cms-publisher
description: >
  Auto-publish an approved blog post to the client's CMS (Webflow or WordPress).
  Invoked automatically by blog-preview once the user approves the styled preview —
  no manual trigger required. Can also be invoked directly when the user says
  "publish this blog", "upload to Webflow", "push to WordPress", "publish the
  post". Handles content formatting, image upload, SEO metadata, backlinks, tags,
  and author bio. Reads client CMS config from BRAND-CONTEXT.md.
metadata:
  version: "0.2.0"
  author: "Speakr Brand"
---

# CMS Publisher

Publishes an approved blog post to the client's website. Supports Webflow and WordPress.

This skill is the **final step** of the pipeline. It runs automatically after `blog-preview` gets user approval — no manual confirmation needed at this point, because the user has already approved the text, graphics, and preview.

## Client Context Loading

Before publishing, load the active client's brand context:

1. Read `{cwd}/.speakr/config.json` to find `clientsDir`.
2. Read `{clientsDir}/.speakr/active-client` for the active client slug.
3. Read `{clientsDir}/{slug}/BRAND-CONTEXT.md` for CMS Platform, CMS Site ID, author info, internal link targets.

If `.speakr/config.json` is missing, instruct the user to run `/client-onboarding` first.

---

## 🛑 PRECONDITIONS

This skill requires **three** approval markers before it runs:

1. `{cwd}/.speakr/last-approved-text.json` (written by `blog-post-creation` Phase 5)
2. `{cwd}/.speakr/last-approved-graphics.json` (written by `blog-graphics` after approval)
3. `{cwd}/.speakr/last-approved-preview.json` (written by `blog-preview` after approval)

**If any of these markers is missing OR for a different client OR older than 24 hours:**

Refuse to publish. Tell the user which step needs to happen first:

> I can't publish yet — the {text|graphics|preview} hasn't been approved.
>
> The pipeline runs in strict order: text → graphics → preview → publish. Each approval locks the previous step.
>
> **Next:** Run `/{blog-post-creation|blog-graphics|blog-preview}` to complete that step, then come back.

This ordering protects the colleague from accidental publishes and ensures every artifact has been reviewed.

---

## Auto-Publish Flow (default — invoked by blog-preview)

When invoked by `blog-preview` after preview approval, **no further user confirmation is needed**. Run the publish flow end-to-end and report status.

When invoked manually by the user ("publish this blog"), still check the three markers first — refuse if any are missing.

---

## Pre-Publish Sanity Check (Automated)

These checks run automatically — don't ask the user about them, just verify:

1. **Blog post content** — approved markdown exists at the path in `last-approved-text.json`
2. **SEO metadata** — meta title, meta description, URL slug, primary keyword present in the metadata file
3. **Graphics** — all approved graphics exported as PNG (use a headless render of the .jsx artifacts if not already PNG)
4. **Backlinks** — internal and external links from BRAND-CONTEXT.md → "Internal Link Targets"
5. **Tags/Categories** — pulled from BRAND-CONTEXT.md → Content Pillars
6. **Author info** — author name, bio, avatar from BRAND-CONTEXT.md → Identity
7. **CTA** — end-of-post call-to-action — pulled from BRAND-CONTEXT.md if defined, otherwise skipped

If any check fails, report the specific gap and pause. Do not auto-fail the publish.

---

## Publishing Flow: Webflow

Use the Webflow MCP connector tools.

### Step 1: Identify the Collection

Use Webflow tools to list the site's CMS collections. Find the "Blog Posts" or equivalent collection (CMS Site ID from BRAND-CONTEXT.md helps narrow this). Note the collection ID and required fields.

### Step 2: Prepare the Content

Convert markdown to Webflow rich text:

- Convert markdown headings to Webflow rich text format
- Upload all images (graphics + featured image) via Webflow's asset API, get hosted URLs
- Replace local image paths with Webflow-hosted URLs
- Convert hyperlinks to anchor tags with `target="_blank"` for external links
- Preserve formatting: bold, italic, lists, blockquotes

### Step 3: Create the CMS Item

Create a new collection item with these fields (map to the client's actual field names):

| Field | Source |
|-------|--------|
| Title | Blog post H1 title |
| Slug | URL slug from SEO metadata |
| Post Body / Content | Rich text content |
| Meta Title | From SEO metadata |
| Meta Description | From SEO metadata |
| Featured Image | Hero/header graphic |
| Author | From BRAND-CONTEXT.md → Identity → Name |
| Category/Tags | From BRAND-CONTEXT.md → Content Pillars |
| Publish Date | Current date |
| Excerpt | First 2 sentences or meta description |

### Step 4: Publish

After creating the item, publish it immediately (live, not draft) — the user has already approved everything.

Return the live URL to the user.

### If Webflow MCP is not connected

Save a ready-to-upload package to `/mnt/user-data/outputs/<slug>-webflow-package/` containing:
- `content.html` — the rich text content
- `images/` — all graphics
- `metadata.json` — all SEO fields
- `README.md` — manual upload instructions

Tell the user: *"Webflow not connected. I've saved a ready-to-upload package at <path>. Connect Webflow and re-run, or upload manually."*

---

## Publishing Flow: WordPress

Use the WordPress MCP connector tools.

### Step 1: Authenticate and Identify

Use WordPress tools to verify connection to the client's site. Check available categories and tags. Confirm the author account exists.

### Step 2: Prepare the Content

Convert markdown to WordPress-compatible HTML:

- Convert markdown to clean HTML
- Upload all images via WordPress media library, get hosted URLs
- Replace local image paths with WordPress media URLs
- Apply proper CSS classes for custom styling
- Ensure links have correct attributes

### Step 3: Upload Images

For each graphic:
1. Upload to WordPress media library
2. Set alt text from the image's alt description
3. Get the media URL for embedding
4. Set the featured image (hero graphic)

### Step 4: Create the Post

Create a new WordPress post with:

| Field | Source |
|-------|--------|
| title | Blog post H1 title |
| slug | URL slug from SEO metadata |
| content | Full HTML content |
| excerpt | First 2 sentences or meta description |
| categories | From BRAND-CONTEXT.md → Content Pillars |
| tags | From blog tags |
| featured_media | Hero graphic media ID |
| status | `publish` (live immediately) |
| meta.yoast_wpseo_title | Meta title |
| meta.yoast_wpseo_metadesc | Meta description |
| meta.yoast_wpseo_focuskw | Primary keyword |

### Step 5: Verify

After publishing, fetch the live URL and verify the post renders correctly.

Return the live URL to the user.

### If WordPress MCP is not connected

Same fallback as Webflow — save a ready-to-upload package and instruct the user.

---

## Post-Publish Report

After publishing (on either platform), report back:

```
✅ Published to {Webflow|WordPress}

📍 Live URL: {url}
📊 SEO: Title ({char_count}/60) · Meta ({char_count}/160) · Keyword present
🖼️ Images: {N} uploaded ({N} graphics + 1 featured)
🔗 Internal links: {N} verified
🏷️ Tags: {list}
📅 Published: {timestamp}

Voice overrides applied (from metadata):
- {list any overrides documented during drafting}
```

---

## Error Handling

**If CMS connector is not connected:** save the upload package, tell the user, don't fail silently.

**If image upload fails:** log which images failed, continue with text content, flag for manual upload.

**If field mapping doesn't match:** list available CMS fields, ask the user to map manually once. Save the mapping to `{clientsDir}/{slug}/cms-field-map.json` for next time.

**If publish fails entirely:** save the post as a Webflow/WordPress draft (instead of live), report the error, give the user the draft URL.

---

## Workflow position

```
blog-post-creation → TEXT APPROVED
        ↓
blog-graphics → GRAPHICS APPROVED
        ↓
blog-preview → PREVIEW APPROVED
        ↓
cms-publisher (this skill, auto-invoked) → LIVE
```
