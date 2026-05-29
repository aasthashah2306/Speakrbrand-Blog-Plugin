---
name: drive-sync
description: >
  Pull new files from the active client's Google Drive folder and refresh their
  brand context, voice DNA, and content pillars. Use when the user says
  "sync drive", "refresh from drive", "pull latest from drive", "the client
  added new content to drive", "update voice from drive", or whenever the
  user mentions new keynotes/blogs/testimonials/materials are available in
  Drive. Runs incrementally — only pulls files modified since the last sync.
metadata:
  version: "0.1.0"
  author: "Speakr Brand"
---

# Drive Sync

Re-pulls new content from the active client's Google Drive folder and updates their local context files. The pipeline gets smarter about each client over time without re-running full onboarding.

## When to use this

- Before drafting a new blog post for a client — surfaces any new voice samples or content that should inform the post.
- When the colleague says the client added something to Drive — a new keynote transcript, a new blog draft, an updated brand book, fresh testimonials.
- Periodically (e.g. weekly) to keep voice DNA fresh as the client publishes more.
- When the user gives feedback like *"the voice was off — they have new blogs in Drive that show their evolved tone"*.

## Client Context Loading

1. Read `{cwd}/.speakr/config.json` → `clientsDir`
2. Read `{clientsDir}/.speakr/active-client` → active slug
3. Read `{clientsDir}/{slug}/BRAND-CONTEXT.md` → `Drive Sync` section
   - `driveFolderId`
   - `lastDriveSyncAt`
4. If `driveFolderId` is `# NOT_SYNCED` or missing, tell the colleague:
   > This client wasn't onboarded from Drive. Run `/client-onboarding` and pick "GDrive pull" to set up Drive sync, or add the Drive folder URL to BRAND-CONTEXT.md manually.

---

## Phase 1: Detect new and changed files

### Step 1.1: Verify connector

Confirm the Google Drive MCP is active. If not, prompt the colleague to connect it.

### Step 1.2: List files modified since last sync

Use `search_files` with a query like:
```
parentId = '<driveFolderId>' and modifiedTime > '<lastDriveSyncAt>'
```

Then recursively check each subfolder the canonical Speakr structure cares about:
- `Branding/` (brand updates)
- `Additional Resources/Blogs copy for speakrBrand/` (new blog drafts)
- `Additional Resources/Book/` (book updates)
- `Additional Resources/Testimonials*` (new testimonials)
- `Keynotes/` (new keynote links)

For each match, capture: `id`, `title`, `mimeType`, `modifiedTime`, parent folder name.

### Step 1.3: Categorize the changes

Bucket the changes into impact tiers:

**HIGH IMPACT** (re-run voice DNA after pulling):
- New files in `Blogs copy/`
- New file matching `*Brand Blueprint*`
- New file matching `*IP STACK*`
- New files in `Book/`
- New file matching `Draft New Web Copy*`

**MEDIUM IMPACT** (update specific section of BRAND-CONTEXT.md):
- New file matching `*Branding FINAL*` → re-extract colors, update `brand/`
- New file matching `*Competitive Analysis*` → update SEO-GUIDELINES.md
- New file matching `*Testimonials*` → update BRAND-CONTEXT.md Proof Points
- New file in `Logos/` folders → update logo

**LOW IMPACT** (just record, no rebuild):
- New file matching `*Dates/Milestones*`
- New keynote links

### Step 1.4: Show the colleague what's new

Report a summary and ask for approval:

```
📂 New since last sync (<lastDriveSyncAt>):

HIGH IMPACT (will refresh voice DNA):
  • 2 new blog drafts in Blogs copy/
  • IP STACK updated

MEDIUM IMPACT (will update specific brand fields):
  • Testimonials spreadsheet — 3 new entries
  • Competitive Analysis — modified

LOW IMPACT (record only):
  • 1 new keynote link

Pull all of these now?
  - Yes, pull all (Recommended)
  - Pull only HIGH IMPACT — skip the rest for now
  - Pick specifically — show me each file
  - Cancel
```

---

## Phase 2: Pull and apply

For each file the colleague approved:

### Step 2.1: Pull

- Google-native (Docs/Sheets): `read_file_content`
- Binaries (PDF/images): `download_file_content`
- Save to the canonical local path per the table in `client-onboarding/SKILL.md` → Step 4.0.2

### Step 2.2: Apply by impact tier

**HIGH IMPACT files:**
- After pulling, re-run the voice DNA extraction over the full `samples/` directory (including the new files).
- Regenerate `WRITING-GUIDELINES.md` — preserve the user's manual edits (look for `<!-- manual edit -->` comments and keep those sections intact).
- Update BRAND-CONTEXT.md → "Voice Quick Reference" if the new samples changed the most-characteristic quote.

**MEDIUM IMPACT files:**
- For each, update the specific section it feeds (e.g. testimonials → BRAND-CONTEXT.md Proof Points).
- Don't touch sections it doesn't feed.

**LOW IMPACT files:**
- Save the file. Note its presence in BRAND-CONTEXT.md but don't reparse anything else.

---

## Phase 3: Update sync state and report

### Step 3.1: Update BRAND-CONTEXT.md

- Set `lastDriveSyncAt` to current ISO timestamp.
- Add a Changelog entry:
  ```
  ## Changelog
  - 2026-05-21: drive-sync — pulled 2 blogs, updated IP STACK, 3 new testimonials. Voice DNA refreshed.
  ```

### Step 3.2: Report to the colleague

```
✅ Drive sync complete.

Changes applied:
  • Voice DNA refreshed from 2 new blog samples (was 5 samples, now 7)
  • IP STACK re-parsed — Content Pillars updated (now: Trust, Discomfort, Growth, Performance)
  • Proof Points expanded — N new testimonials added ({Company A}, {Company B}, {Company C})
  • Competitor list refreshed in SEO-GUIDELINES.md

Voice Quick Reference now reads:
  "{client}" SAYS: "<updated characteristic quote from new samples>"
  "{client}" NEVER SAYS: "<unchanged>"

→ Recommended: re-run blog-post-creation if you have a draft in flight — the refreshed voice may produce a better result.
```

---

## Phase 4: Conflict handling

If a HIGH IMPACT change conflicts with manual edits the colleague made to BRAND-CONTEXT.md or WRITING-GUIDELINES.md:

1. Show the diff: what Drive says vs. what's local.
2. AskUserQuestion:
   > **The Drive content conflicts with manual edits you made. What wins?**
   > - Keep my manual edits, ignore Drive change
   > - Take the Drive change, lose my edits
   > - Merge — show me each conflict and I'll decide

3. Apply the colleague's choice. Log to Changelog.

---

## Auto-sync mode (optional)

If `autoSyncEnabled: true` in BRAND-CONTEXT.md, this skill can be invoked automatically:

- Before every `blog-post-creation` run (so the post uses the latest voice)
- Once per week via the `/schedule` system (lightweight check — only HIGH IMPACT changes get applied without asking)

When auto-sync runs, **never** apply MEDIUM/LOW changes without prompting. Auto-mode is for keeping voice fresh, not for overwriting brand decisions.

To enable auto-sync, the colleague can say: *"enable auto-sync for {client}"*.

---

## Workflow position

```
client-onboarding (one-time per client)
        ↓
drive-sync (recurring — keeps context fresh)
        ↓
seo-content-brief → seo-research → content-research
        ↓
blog-post-creation (always uses the latest synced context)
```

---

## Common requests this skill handles

- "sync drive" / "refresh from drive" / "pull latest from drive"
- "update voice for {client}" → run drive-sync, then re-extract voice DNA
- "the client added new content to drive"
- "are there new blogs in drive?" → run Phase 1 only, report without pulling
- "fill gaps for {client}" → run drive-sync first to catch anything new from Drive before manually filling
