---
name: client-onboarding
description: >
  Onboard a new client into the Speakr blog pipeline. Use when the user says
  "onboard a new client", "set up a new client", "add a client", "create brand
  profile", "configure client for blog pipeline", or "switch clients". Walks
  through connector checks, workspace setup, brand material collection, voice
  DNA extraction, SEO business context, and generates the client brand context
  files used by every other pipeline skill. Lenient — never blocks; produces a
  working profile from whatever materials are available and marks gaps for
  later top-up.
metadata:
  version: "0.2.0"
  author: "Speakr Brand"
---

# Client Onboarding

This skill is the entry point of the Speakr blog pipeline. It produces a `clients/<slug>/` directory that every other skill reads from. Each colleague maintains their own clients in their own workspace — clients are never shared or mixed across colleagues.

## Operating Principles

1. **Never block.** If the user can't provide something, write a `# TODO:` placeholder and proceed. Downstream skills will prompt to fill the gap when they need it.
2. **Always confirm before writing.** Show the user what you're about to write to BRAND-CONTEXT.md and ask for approval.
3. **Use AskUserQuestion for structured input.** Don't ask freeform questions when a multiple-choice or list format works better.
4. **Update, don't replace.** If a `clients/<slug>/` directory already exists, treat this run as an update — merge new info, preserve existing.

---

## Phase 1: Connector Check

Run this BEFORE asking for any client materials. Colleagues need to know what's missing up front.

### Step 1.1: List currently connected MCPs

Call `mcp__mcp-registry__list_connectors` to see what's connected in this Claude environment.

### Step 1.2: Print connector status

Present a status table covering the connectors this pipeline uses:

| Connector | Status | Powers | Required? |
|-----------|--------|--------|-----------|
| Webflow | ✅ / ❌ | cms-publisher (Webflow clients) | One of Webflow/WordPress required for publishing |
| WordPress | ✅ / ❌ | cms-publisher (WordPress clients) | One of Webflow/WordPress required for publishing |
| DataForSEO | ✅ / ❌ | seo-research, seo-content-brief | Strongly recommended |
| Exa | ✅ / ❌ | content-research, fact-checker | Strongly recommended |
| Perplexity | ✅ / ❌ | content-research (deep mode) | Optional |
| Notion | ✅ / ❌ | client collaboration | Optional |
| Google Drive | ✅ / ❌ | client collaboration | Optional |

### Step 1.3: Guide on missing connectors

For each missing connector, give a one-line setup instruction:

- **Webflow** — Settings → Connectors → search "Webflow" → connect with your client's Webflow API token
- **WordPress** — Settings → Connectors → search "WordPress" → connect with site URL + application password
- **DataForSEO** — Settings → Connectors → search "DataForSEO" → connect with API credentials from dataforseo.com
- **Exa** — Settings → Connectors → search "Exa" → connect with API key from exa.ai
- **Perplexity** — Settings → Connectors → search "Perplexity" → connect with API key from perplexity.ai
- **Notion** — Settings → Connectors → search "Notion" → authorize workspace
- **Google Drive** — Settings → Connectors → search "Google Drive" → authorize Google account

### Step 1.4: Ask the user to proceed

Use AskUserQuestion:

> **Connectors look like this — want to connect missing ones now, or proceed with what you have?**
> - Proceed now (Recommended) — you can connect more later, pipeline runs with what's available
> - Pause while I connect missing ones — come back and run "onboard a new client" again
> - Skip publishing for this client — they don't need CMS publishing

Record the user's CMS choice (Webflow / WordPress / neither) for use in Phase 3.

---

## Phase 2: Workspace Setup

### Step 2.1: Find or create `.speakr/config.json`

Check for `{cwd}/.speakr/config.json`. This file records where this colleague keeps their clients.

If it exists, read `clientsDir` from it.

If it does not exist, AskUserQuestion:

> **Where do you want to keep your client files?**
> - `./clients/` in this directory (Recommended) — clients live next to your current project
> - A custom path — you'll provide one
> - `~/speakr-clients/` — a stable per-user location outside any project

Create `.speakr/config.json` with:
```json
{
  "clientsDir": "<chosen path>",
  "version": "0.2.0"
}
```

Add `.speakr/` to `.gitignore` if a `.gitignore` exists, so client data isn't accidentally committed to shared repos.

### Step 2.2: Confirm the clients/ directory exists

If `<clientsDir>/` doesn't exist yet, create it with a `.gitkeep` file inside.

### Step 2.3: List existing clients

If there are already client folders inside, show them. The user might be onboarding a new client OR updating an existing one. AskUserQuestion:

> **What are we doing?**
> - Onboard a new client
> - Update an existing client — pick from the list
> - Set an existing client as active (no changes) — pick from the list

If updating: load the existing BRAND-CONTEXT.md and treat the rest of this skill as a merge.

---

## Phase 3: Client Identity

This phase collects the standard onboarding inputs. **Website, LinkedIn, and blog-post examples are first-class fields — always ask for them here, up front.** Do not bury them in later phases and do not offer a "just give me the name" shortcut that skips them. Every colleague should see these as the standard questions.

If the colleague genuinely doesn't have one, that's fine — write a `# TODO:` placeholder for that field and proceed (never block). But the ask must be explicit, not optional-by-omission.

### Step 3.1: Core identity

Use AskUserQuestion in a single call with multiple questions:

> **Client basics:**
> 1. Client's full name? (free text)
> 2. Professional title? (free text)
> 3. Company and role? (free text)

### Step 3.2: Web presence (always ask — do not skip)

Use AskUserQuestion with two questions in a single call. Each must include an explicit "I don't have it yet" choice that maps to a `# TODO:`:

> **Website URL?**
> - I'll paste it (free text via "Other")
> - I don't have it yet — mark as `# TODO`

> **LinkedIn URL?**
> - I'll paste it (free text via "Other")
> - I don't have it yet — mark as `# TODO`

Record whatever the colleague provides into BRAND-CONTEXT.md → Identity. For any "I don't have it yet" answer, write the field as `# TODO: confirm <field>` and add it to "Gaps to fill".

### Step 3.3: Blog-post examples (always ask — do not skip)

Voice DNA quality depends on real writing samples. Ask for them here, up front, so the colleague knows they matter — even before the Drive pull. Use AskUserQuestion:

> **Do you have examples of blog posts this client has written?**
> - Paste 3–5 published blog URLs — I'll fetch them (Recommended)
> - They're in the client's Drive folder — I'll pull them in Phase 4
> - I'll drop local files into `samples/` later
> - None available yet — mark as `# TODO`

- If the colleague picks **"Paste blog URLs"**: collect the URLs (free text, one per line) and **defer fetching to Step 4.1.5** (the dedicated blog-URL ingest step). Record the URLs so Step 4.1.5 can act on them.
- If **"in Drive"**: the Phase 4.0 Drive pull will collect `Blogs copy for speakrBrand/` as usual.
- If **"local files later"**: proceed; Step 4.1 covers the local drop.
- If **"None yet"**: write `# TODO: needs blog samples` and continue. WRITING-GUIDELINES.md will carry placeholders until samples arrive.

### Step 3.4: CMS + content type

Then AskUserQuestion for structured fields:

> **CMS platform for publishing?**
> - Webflow
> - WordPress
> - None yet (skip publishing for now)

> **What kind of content does this client publish?**
> - Thought leadership (executive POV, opinion pieces)
> - Educational / how-to (tutorials, frameworks)
> - Case studies / customer stories
> - Mix of the above

Derive a `client-slug` from the name (lowercase, hyphens). Confirm it with the user before creating the directory.

Create `<clientsDir>/<client-slug>/` with subdirectories:
```
<client-slug>/
├── brand/
├── samples/
└── references/
```

---

## Phase 4: Brand Materials Collection

Speakr clients almost always have a Google Drive folder with their materials, organized to a **consistent convention** that this skill knows how to read. Always check Drive first — it's faster, more complete, and the colleague rarely has to upload anything manually.

### Step 4.0: GDrive pull (the canonical Speakr flow)

AskUserQuestion:

> **Does this client already have a Google Drive folder (the standard Speakr client folder)?**
> - Yes — pull from Drive (Recommended — this is the standard Speakr flow)
> - No — I'll drop files locally instead
> - Some in Drive, some I'll add locally

#### If Drive:

##### Step 4.0.1: Verify connector and get the folder

1. Verify the **Google Drive MCP connector** is active. If not, prompt:
   > Settings → Connectors → search "Google Drive" → authorize the speakrbrand.com Google account. Then say "continue".
2. Ask the colleague for the Drive folder:
   > **Paste the Drive folder URL, or just type the client name** and I'll find it (Speakr folders are usually named `{Client} x SpeakrBrand` or just `{Client}`).
3. Use `search_files` to locate the folder. Confirm with the colleague before proceeding: *"Found `<folder name>` (modified <date>) — pull from this one?"*

##### Step 4.0.2: Map the canonical Speakr folder structure

Speakr client Drive folders follow a consistent structure. Look for and pull these specific files/folders **by name pattern**:

| Drive location | Purpose | Pull to | Priority |
|---|---|---|---|
| `Branding/{Client} Brand Blueprint*.xlsx` | The brand questionnaire the client filled out — 3 adjectives, dream brand ambassador, competitors, inspirations, favorite books/movies + WHY, PCT (Problem/Credibility/Transformation) | `references/brand-blueprint.xlsx` + parse into BRAND-CONTEXT.md and WRITING-GUIDELINES.md | **CRITICAL** — this is the voice/positioning foundation |
| `Branding/{Client} *IP STACK*.pdf` | Master positioning doc: Brand Attributes, Audience, Essence, Brand Enemies, Position, Vision, Mission, Values, Core Beliefs, Reasons to Believe, Manifesto | `references/ip-stack.pdf` + parse into BRAND-CONTEXT.md (Positioning, Content Pillars, Voice Quick Reference) | **CRITICAL** |
| `Branding/{Client} Branding FINAL*.pdf` | Visual brand book — logos, colors, typography, design samples | `brand/brand-guidelines.pdf` + extract colors + extract typography | **CRITICAL** for brand colors/fonts |
| `Branding/{Client} Competitive Analysis*.xlsx` | Full competitive landscape, 4-6 named competitors with positioning notes | `references/competitive-analysis.xlsx` + parse competitor URLs into SEO-GUIDELINES.md | **HIGH** |
| `Branding/Social Branding + Style Checklist*` | Visual style rules (fonts, colors, post examples) | `references/social-style-checklist.md` | MEDIUM |
| `Branding/Logos + Assets/` or `Branding/Logos and Branding/` | Logo files | `brand/logo.png`, `brand/logo-dark.png` | **CRITICAL** |
| `Branding/Fonts/` | Font files | `brand/fonts/` | MEDIUM |
| `Additional Resources/Book/` | Client's published book | `samples/book/` | **CRITICAL** — primary voice source |
| `Additional Resources/Draft New Web Copy*` | Full website copy (positioning, taglines, sections) | `samples/web-copy.md` | **HIGH** — voice sample |
| `Additional Resources/Blogs copy for speakrBrand/` | Existing blog posts | `samples/blogs/` (each as separate file) | **CRITICAL** — primary voice samples |
| `Additional Resources/Testimonials*` | Spreadsheet of client testimonials with companies, quotes, video URLs | `references/testimonials.xlsx` + parse into BRAND-CONTEXT.md (Proof Points) | **HIGH** |
| `Additional Resources/Client Logos/` | Logos of brands the client has worked with | `references/client-logos/` | LOW (proof/credibility) |
| `Additional Resources/News logos/` | Media logos (Forbes, NYT, Inc, etc.) | `references/news-logos/` | LOW |
| `Keynotes/Full Keynote Links*` | List of keynote video URLs (Vimeo, YouTube, Dropbox) | `references/keynote-links.md` | MEDIUM (record for later transcription) |
| `{Client} Dates/Milestones*` (top level) | Speaking events, book launches, important dates | `references/dates-milestones.xlsx` | LOW — useful for timely content ideas |
| `{Client} Marketing Roadmap*.xlsx` (top level) | Project management tracker | **DO NOT PULL** — not blog-relevant |
| `Outreach/`, `Social Media/`, `Photos/`, `Videos/` | Various — usually too much to pull blindly | Ask colleague case by case | LOW |

##### Step 4.0.3: Execute the pull

For each file matched above:

1. Use `get_file_metadata` to confirm it exists and check modified time.
2. For Google Docs and plain text: use `read_file_content` to get text representation.
3. For binaries (PDF, images, fonts): use `download_file_content` and save the base64 content as the target file.
4. For folders: recursively `search_files` with `parentId = '<folder-id>'` and pull contents.

**Special handling for spreadsheets (.xlsx, Google Sheets, Brand Blueprint / Competitive Analysis / Testimonials):**

Spreadsheet content from `read_file_content` comes back as a flattened text snippet — rows mashed together, column structure usually lost. Brand Blueprint and Competitive Analysis specifically need clean row/column data to parse correctly. **Three-tier fallback:**

1. **First try:** Use `download_file_content` with `exportMimeType: 'text/csv'`. This returns the spreadsheet as proper CSV. Parse with a CSV reader. Works for both Google Sheets and uploaded .xlsx files.
2. **If that fails (file is multi-sheet .xlsx that doesn't export cleanly):** Use `download_file_content` with `exportMimeType: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'` to grab the raw .xlsx and parse with a library like `openpyxl` (via a Bash + Python step).
3. **If both fail (Drive MCP error, file too large, complex formulas):** Tell the colleague:

   > **Heads up — I couldn't cleanly parse `{filename}` from Drive.** This usually happens with multi-sheet Excel files or files with complex formulas.
   >
   > Two ways to fix:
   > 1. **Quickest:** Open the file in Google Sheets → File → Download → Comma-separated values (.csv) → drop the CSV into `<clientsDir>/<client-slug>/references/{slug}.csv` → say "continue".
   > 2. **Or:** Open the file → File → Download → Tab-separated values → save as `.tsv` → drop into the same folder.
   >
   > I'll pick up the file once it's there.

   Poll the `references/` folder every few seconds for the file. Once it appears, parse it locally and proceed.

Always log which fallback tier succeeded in the BRAND-CONTEXT.md Changelog so future runs know what worked.

##### Step 4.0.4: Parse the high-priority files

After pulling the files, extract structured data:

- **Brand Blueprint** → answers to: 3 adjectives, dream ambassador, community/tribe, direct competitors, personal brand inspirations, favorite movies + WHY (essence!), favorite books + WHY, brand strengths, biggest challenges, PCT. Feed these into BRAND-CONTEXT.md (Identity, Positioning) and WRITING-GUIDELINES.md (Voice Quick Reference, Forbidden Patterns from the "competitors I avoid" answer).
- **IP STACK** → extract Position statement, Vision, Mission, Values, Core Beliefs, Brand Enemies (these become the "Forbidden patterns" in WRITING-GUIDELINES.md), Reasons to Believe (proof points), Messaging Hierarchy. Feed into BRAND-CONTEXT.md → Positioning, Content Pillars.
- **Branding FINAL PDF** → extract hex colors via image analysis + identify typography. Feed into `brand/colors.json` + `brand/typography.json`.
- **Competitive Analysis** → extract competitor names and URLs. Feed into SEO-GUIDELINES.md → Competitors.
- **Testimonials** → extract Name, Company, Title, Quote, Video URL. Feed into BRAND-CONTEXT.md → Proof Points.
- **Blog samples** → each becomes a separate file in `samples/blogs/{slug}.md`. Used by humanizer for voice calibration.

##### Step 4.0.5: Record the Drive folder ID

Write to BRAND-CONTEXT.md → "Drive Sync":
```yaml
driveFolderId: <Drive folder ID>
driveFolderUrl: <Drive folder URL>
lastDriveSyncAt: <ISO timestamp>
```

This lets the `drive-sync` skill watch this folder for new files later (see "Continuous Improvement" section).

##### Step 4.0.6: Report

Tell the colleague:

> ✅ **Pulled from Drive:**
> - Brand Blueprint → parsed into voice DNA
> - IP STACK → parsed into positioning + content pillars
> - Branding book → colors + typography extracted
> - Competitive Analysis → 6 competitors → SEO-GUIDELINES.md
> - {N} blog samples → voice calibration ready
> - {N} testimonials → proof points
> - Logo + brand assets
>
> ⏭️ **Skipped (not blog-relevant):** Marketing Roadmap, Outreach, Photos, Videos
>
> ❓ **Want me to also pull from:** Keynotes (link list — manual transcription needed?), Social Media, Photos?

If the colleague picked "Some in Drive, some I'll add locally", continue to Step 4.1 for any gaps.

#### If local-drop only: continue to Step 4.1.

### Step 4.1: Writing samples (local drop — fallback path)

Use this only when the client doesn't have materials in Drive, or for samples the colleague has locally that weren't in Drive.

Tell the user:

> **Drop 2-5 writing samples for this client into `<clientsDir>/<client-slug>/samples/`.**
> These can be:
> - Past blog posts
> - LinkedIn articles
> - Newsletter issues
> - Book chapters or excerpts
> - Speech transcripts
>
> Format: `.md`, `.txt`, `.pdf`, or `.docx` — any of these work. Name them anything.
>
> The more diverse the samples (different lengths, different topics), the better the voice DNA. Aim for 3+ if possible.
>
> Say **"continue"** when you've added them, or **"skip"** if you have no samples yet (we'll work without and prompt you later).

Poll the samples/ directory before proceeding. If empty after "continue", warn but proceed.

### Step 4.1.5: Blog URL ingest (fetch published posts)

Run this whenever the colleague provided blog URLs in **Step 3.3** (or offers them now). This path works even when the client has no Drive folder and no local files — it's the most universal way to collect voice samples.

If no URLs were provided in Step 3.3, you may still prompt once here:

> **Have any published blog posts at a URL?** Paste 3–5 links (one per line) and I'll fetch them as voice samples. Or say **"skip"**.

For each URL the colleague provides:

1. **Fetch** the page with `WebFetch`, asking it to return the article's title, author, publish date, and full body text (strip nav, ads, related-posts, comments, and CTAs — keep only the article prose).
2. **Derive a slug** from the article title (lowercase, hyphens, no stop-words bloat). If the title is unavailable, derive from the URL path.
3. **Save** to `<clientsDir>/<client-slug>/samples/blogs/{slug}.md` with this front matter, then the cleaned body:
   ```markdown
   ---
   source_url: <original URL>
   title: <article title>
   fetched_at: <ISO timestamp>
   ---

   # <article title>

   <cleaned article body>
   ```
4. **De-dupe:** if a file with that slug already exists (e.g. also pulled from Drive), skip and note it rather than overwriting.
5. **On fetch failure** (paywall, JS-only render, 403, timeout): don't fail the run. Tell the colleague:
   > Couldn't fetch `<url>` (likely a paywall or script-rendered page). Paste the article text directly, or save it as `samples/blogs/{slug}.md` yourself and say "continue".
   Then move on to the next URL.

After processing all URLs, report a summary:

> ✅ **Fetched {N} blog posts as voice samples:**
> - {title 1} → `samples/blogs/{slug-1}.md`
> - {title 2} → `samples/blogs/{slug-2}.md`
> ⚠️ **Couldn't fetch:** {any failures, with the reason}

These saved files feed Phase 5 (Voice DNA Extraction) exactly like Drive-pulled or locally-dropped samples. Log the count and source URLs in the BRAND-CONTEXT.md Changelog.

### Step 4.2: Brand visuals

First check what's already in `<clientsDir>/<client-slug>/brand/` from the GDrive pull (Step 4.0). Report what was found, e.g. *"Logo and brand book already pulled from Drive. Anything else to add locally?"*

If gaps remain (no logo, no brand book), tell the user:

> **Drop missing brand assets into `<clientsDir>/<client-slug>/brand/`:**
> - `logo.png` — primary logo (required for graphics)
> - `logo-dark.png` — dark-mode variant (optional)
> - `brand-guidelines.pdf` — full brand book if they have one (optional)
>
> Say **"continue"** when done, or **"skip"** if no assets are available.

### Step 4.3: Brand colors

AskUserQuestion (allow free text for hex values):

> **What are this client's brand colors?**
> - I'll provide hex codes — primary, secondary, accent, background, text
> - Extract them from the logo I uploaded (auto)
> - Pull from their website (you'll provide URL, I'll inspect)
> - Use defaults for now — neutral dark/light palette

If user picks "I'll provide": collect the 5 hex codes via follow-up AskUserQuestion.

If user picks "extract from logo": use image analysis on `brand/logo.png` to suggest a palette, then confirm.

If user picks "from website": fetch the website CSS and extract dominant brand colors, then confirm.

Write `<clientsDir>/<client-slug>/brand/colors.json`:
```json
{
  "primary": "#hex",
  "secondary": "#hex",
  "accent": "#hex",
  "background": "#hex",
  "text": "#hex"
}
```

### Step 4.4: Typography

AskUserQuestion:

> **What fonts does this client use?**
> - I'll provide font names — heading, body, mono
> - Use defaults — Inter for headings, Inter for body, JetBrains Mono for code
> - Pull from their website CSS — you'll provide URL

Write `<clientsDir>/<client-slug>/brand/typography.json`:
```json
{
  "headingFont": "Font Name",
  "bodyFont": "Font Name",
  "monoFont": "Font Name"
}
```

---

## Phase 5: Voice DNA Extraction

Read every file in `<clientsDir>/<client-slug>/samples/`. If there are no samples, skip to Step 5.3.

### Step 5.1: Analyze the 10 voice patterns

For each pattern, extract specific examples from the samples:

1. **Sentence rhythm** — short/long mix, paragraph length patterns. Quote 2-3 examples.
2. **Opening moves** — story / question / bold claim / data. Quote 2-3 examples.
3. **Transition style** — rhetorical questions / direct pivots / bridges. Quote 2-3 examples.
4. **Proof patterns** — personal stories / data / analogies / case studies. Note which dominate.
5. **Emotional register** — warm / direct / academic / conversational / provocative. Pick the strongest 1-2.
6. **Signature phrases** — recurring expressions, verbal tics. List 5+ if found.
7. **Formatting habits** — lists / arrows / bold / headers / parentheses. Quantify.
8. **Authority signals** — credentials / experience / name-drops. List concrete examples.
9. **Close style** — CTA / reflection / challenge / callback. Note patterns.
10. **Forbidden patterns** — language the client clearly avoids. Infer from absence.

### Step 5.2: Voice Quick Reference

Build a concise reference block:

```
**{Client} SAYS:** "{characteristic quote from samples}"
**{Client} NEVER SAYS:** "{example of corporate/generic language they avoid}"
```

### Step 5.3: If no samples available

Write `WRITING-GUIDELINES.md` with `# TODO: needs samples` placeholders for each of the 10 patterns. Tell the user:

> Voice DNA needs samples. Drop some into `<clientsDir>/<client-slug>/samples/` and run **"update voice for <client>"** anytime to refresh.

### Step 5.4: Write WRITING-GUIDELINES.md

Use the template at `references/writing-guidelines-template.md`.

---

## Phase 6: SEO Business Context

This phase determines whether SEO keyword research will be client-specific (good) or generic (bad). Don't skip.

### Step 6.1: Business basics

AskUserQuestion:

> **What's this client's industry?** (free text — be specific, e.g. "B2B SaaS for HR teams" not just "SaaS")

> **Who is the ideal reader?**
> - C-suite / executives
> - Mid-level managers / directors
> - Practitioners / individual contributors
> - Mixed audience — describe in free text

> **Geographic focus?**
> - Global / English-speaking
> - US-focused
> - EU-focused
> - Specific region (free text)

### Step 6.2: Products and services

AskUserQuestion:

> **What does this client sell or offer?** (free text — list products, services, or thought leadership topics they want associated with their brand)

### Step 6.3: Competitors

AskUserQuestion:

> **Top 3 competitor URLs?** (free text — these will drive SERP competitive analysis)

If DataForSEO is connected, use these URLs to seed competitor keyword analysis. If not, store them for manual reference.

### Step 6.4: Existing keyword wishlist

AskUserQuestion:

> **Are there keywords this client already wants to rank for?**
> - Yes, I have a list (free text)
> - No, build from scratch
> - I'll check their Search Console / SEMrush later

### Step 6.5: Write SEO-GUIDELINES.md

Use the template at `references/seo-guidelines-template.md`. Include:
- Business context (industry, ICP, geo, products)
- Competitor URLs
- Seed keywords
- `# TODO: keyword clusters` placeholder — will be filled by first run of seo-research

---

## Phase 7: Generate BRAND-CONTEXT.md

This is the master file every other skill reads. Use the template at `references/brand-context-template.md`.

Required structure (do not deviate — downstream skills parse this format):

```markdown
# {Client Name} — Brand Context

## Identity
- **Name:** {full name}
- **Slug:** {client-slug}
- **Title:** {professional title}
- **Company:** {company}
- **Website:** {URL or `# TODO`}
- **LinkedIn:** {URL or `# TODO`}
- **CMS Platform:** {webflow|wordpress|none}
- **CMS Site ID:** {ID or `# TODO`}

## Brand Assets
- **Logo:** `brand/logo.png` ({present|`# TODO`})
- **Logo (dark):** `brand/logo-dark.png` ({present|`# TODO`})
- **Colors:** see `brand/colors.json`
- **Typography:** see `brand/typography.json`

## Voice
- **Samples on file:** {N samples in samples/ or `# TODO: needs samples`}
- **Voice DNA:** see `WRITING-GUIDELINES.md`
- **{Client} SAYS:** "{quote}"
- **{Client} NEVER SAYS:** "{anti-quote}"

## SEO Context
- **Industry:** {industry}
- **ICP:** {ideal reader}
- **Geo:** {geo focus}
- **Products/Services:** {summary}
- **Competitors:** {3 URLs}
- **See:** `SEO-GUIDELINES.md`

## Content Pillars
{3-5 pillars derived from voice DNA + business context, or `# TODO`}

## Gaps to fill
{Bulleted list of every `# TODO` in this file — surfaces what's missing at a glance}
```

The "Gaps to fill" section at the bottom is critical — colleagues should be able to glance at this file and see what's incomplete.

---

## Phase 8: Confirm and Activate

### Step 8.1: Show summary

Display the BRAND-CONTEXT.md to the user and AskUserQuestion:

> **Looks good?**
> - Yes, set as active client (Recommended) — pipeline is ready
> - Yes, but don't set as active — just save it
> - Adjust something — tell me what

### Step 8.2: Set active client

Write the slug to `.speakr/active-client`:
```
client-slug
```

Downstream skills read this file to know which client they're working on.

### Step 8.3: Final message

Tell the user:

> ✅ **{Client Name}** is onboarded and active.
>
> **Next steps:**
> - Write a blog post: say **"write a blog for {Client} about <topic>"**
> - Update voice: say **"update voice for {Client}"** after adding more samples
> - Switch clients: say **"work on <other-client>"**
> - Fill gaps: say **"fill gaps for {Client}"** to top up missing data
>
> {If gaps exist, list them here.}

---

## Continuous Improvement

Other skills will detect gaps and prompt the user to fill them. When they do:

1. They use AskUserQuestion to collect the missing field.
2. They call back into this skill via the `fill-gap` operation:
   - Update BRAND-CONTEXT.md (preserve everything else)
   - Update the relevant supporting file (colors.json, WRITING-GUIDELINES.md, etc.)
   - Remove the gap from the "Gaps to fill" section

When the user gives feedback like *"the voice was too formal in that last post"* or *"the colors don't match their site anymore"*:

1. Read the relevant file (WRITING-GUIDELINES.md, colors.json, etc.).
2. Show what's there and ask what to change.
3. Update the file. Note the change in a `Changelog` section at the bottom of BRAND-CONTEXT.md:
   ```
   ## Changelog
   - 2026-05-21: Loosened voice — was too formal per user feedback
   - 2026-05-15: Updated primary color from #171420 to #1A1A1A
   ```

---

## Multi-Client Management

- Active client lives in `<clientsDir>/.speakr/active-client` (just the slug, no extension).
- To switch: "work on {client}" → updates `.speakr/active-client`.
- To list: scan `<clientsDir>/*/BRAND-CONTEXT.md` and show name + last-modified.
- Each colleague's clients are isolated by their own `clientsDir` config.

---

## References

- `references/brand-context-template.md` — master file template
- `references/writing-guidelines-template.md` — voice DNA format
- `references/seo-guidelines-template.md` — keyword and business context format
- `examples/sample-client/` — a fully-populated example to learn from
