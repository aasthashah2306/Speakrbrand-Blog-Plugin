---
name: blog-preview
description: >
  Generate a styled HTML preview of an approved blog post — looks like a real
  blog page using the active client's brand colors, typography, logo, and the
  approved graphics inline. Use when the user says "preview the blog", "show
  me the blog page", "render the preview", or after blog-graphics finishes.
  Runs after text approval AND graphics approval, before cms-publisher.
metadata:
  version: "0.1.0"
  author: "Speakr Brand"
---

# Blog Preview

Generates a fully-styled HTML preview of the approved blog post — what it will look like when published. The colleague clicks a link, sees a branded page in their browser. Approving this preview triggers `cms-publisher` to auto-publish.

## Client Context Loading

Before generating any preview, load the active client's brand context:

1. Read `{cwd}/.speakr/config.json` to find `clientsDir`.
2. Read `{clientsDir}/.speakr/active-client` for the active client slug.
3. Load brand assets:
   - `{clientsDir}/{slug}/BRAND-CONTEXT.md` — for client name, website, CMS, author info
   - `{clientsDir}/{slug}/brand/colors.json` — primary, secondary, accent, background, text
   - `{clientsDir}/{slug}/brand/typography.json` — headingFont, bodyFont
   - `{clientsDir}/{slug}/brand/logo.png` — for the page header

If any are missing, surface the gap and offer to call `/client-onboarding`. Never hardcode brand values.

---

## 🛑 PRECONDITIONS

This skill requires **two** approval markers to exist before it runs:

### Marker 1: Text approval
`{cwd}/.speakr/last-approved-text.json` (written by `blog-post-creation` Phase 5).

### Marker 2: Graphics approval
`{cwd}/.speakr/last-approved-graphics.json` (written by `blog-graphics` after the user approves the .jsx artifacts).

If either marker is missing, refuse to run and direct the user back to the missing step.

---

## Phase 1: Read the approved post + graphics

1. From `last-approved-text.json`, read `draftPath` → load the approved markdown.
2. From `last-approved-graphics.json`, read the list of approved .jsx graphic paths.
3. For each graphic, locate the `[Graphic: ...]` placeholder in the markdown and prepare to inline-replace it.

---

## Phase 2: Render to styled HTML

Use the generic blog template at `references/blog-template.html` as the base. Substitute the active client's brand values into the CSS variables at the top of the template:

```css
:root {
  --color-primary: {colors.json → primary};
  --color-secondary: {colors.json → secondary};
  --color-accent: {colors.json → accent};
  --color-background: {colors.json → background};
  --color-text: {colors.json → text};
  --font-heading: {typography.json → headingFont};
  --font-body: {typography.json → bodyFont};
}
```

Inject the post into the template:

- **Header bar:** Client logo (from `brand/logo.png`) + client name (from BRAND-CONTEXT.md → Identity → Name) linked to website
- **Article hero:** Post H1 + meta description from the post's metadata file
- **Article body:** Render the approved markdown to HTML. Use standard markdown-to-HTML conversion. For each `[Graphic: N]` placeholder, inline the corresponding .jsx graphic rendered as HTML (or as an `<iframe>` to the .jsx artifact if rendering inline is complex).
- **Author block:** Pull from BRAND-CONTEXT.md → Identity (Name, Title, Company, LinkedIn).
- **Footer:** Generic, branded with the client's primary color.

### Style notes

- The template should look like a clean, modern blog page — think the kind of layout Medium, Substack, or a thought-leadership site uses.
- Respect the client's voice: if their site is minimal, the preview should feel minimal. If their site is bold, the preview should feel bold. The colors.json + typography.json should carry most of this.
- Make the preview **responsive** — render correctly on desktop and mobile breakpoints.
- Set the page title to the post's SEO title.
- Include OpenGraph + Twitter card meta tags using the post's metadata.

### Write the output

Save to:

```
/mnt/user-data/outputs/preview-{slug}-{date}.html
```

Provide the user with a clickable link:

> 🖼️ **Preview ready:** [View blog preview](computer:///mnt/user-data/outputs/preview-{slug}-{date}.html)

---

## Phase 3: Approval gate + auto-publish

After showing the preview link, AskUserQuestion:

> **Looks good to publish?**
> - Yes, publish to {Webflow/WordPress} now (Recommended) — `cms-publisher` runs automatically
> - Yes but I want to tweak something first — tell me what
> - No, send back to graphics — preview revealed a layout issue
> - No, send back to text — I see a writing change I want

### If "Yes, publish":

1. Write a third marker: `{cwd}/.speakr/last-approved-preview.json` with the preview path + timestamp + client slug.
2. **Immediately invoke `cms-publisher`** with the approved markdown path and the client's CMS config from BRAND-CONTEXT.md. Do not wait for the user to ask — auto-publish is the agreed flow.
3. Report the publish status back to the user when done.

### If "tweak something":

Make the tweak directly to the HTML template (for layout fixes) or send back to the upstream skill (for content fixes). Re-render and ask again.

### If "send back to graphics":

Delete `last-approved-graphics.json` so graphics is forced to re-run. Hand off.

### If "send back to text":

Delete `last-approved-text.json` AND `last-approved-graphics.json` (since graphics depended on the text). Hand off to `blog-post-creation`.

---

## Workflow

```
blog-post-creation (Phase 5) → TEXT APPROVED marker
        ↓
blog-graphics → GRAPHICS APPROVED marker
        ↓
blog-preview (this skill) → render styled HTML
        ↓
User approves preview → PREVIEW APPROVED marker
        ↓
cms-publisher (auto-invoked) → published
```

---

## References

- `references/blog-template.html` — the generic styled blog template
- `references/markdown-to-html.md` — markdown rendering rules for this skill
