---
name: authority-citations
description: >
  Discover, live-verify, and place 8-12 high-authority external backlinks (authority
  citations) into a blog draft as inline links. Use when adding backlinks/citations to a
  post, when the user says "add backlinks", "add authority citations", "add sources", or
  "redo the links", and as the automatic step blog-post-creation runs after the humanizer
  pass. Every link is fetched and confirmed live before placement — no dead links, no
  homepages, no paywalls, never padded. Runs inside the pipeline or standalone on any post,
  including old/existing ones.
metadata:
  version: "1.0.0"
  author: "Speakr Brand"
---

# Authority Citations

Places **8-12 authority citations** — external, high-authority, third-party links (HBR,
McKinsey, MIT Sloan, peer-reviewed research, major business publications) — inline in a blog
draft, as a credibility and AI-citation signal. Every link is verified **live** before it is
placed.

> **Terminology** ([CONTEXT.md](../../CONTEXT.md)): *authority citations* = external Tier-1
> links (this skill). *internal links* = the client's own pages (a separate min-2 rule that
> lives in `blog-post-creation` — this skill does not touch it).

## Client Context Loading

Before starting, load the active client's context (same pattern as the other skills):

1. Read `{cwd}/.speakr/config.json` to find `clientsDir`.
2. Read `{clientsDir}/.speakr/active-client` for the active client slug (or use the client the
   user named).
3. If `.speakr/config.json` is missing or no active client is set, instruct the user to run
   `/client-onboarding` first.
4. Read `{clientsDir}/{slug}/BRAND-CONTEXT.md` for:
   - **Identity → website URL** → the client's own domain (used to exclude internal links).
   - An optional **"Authority Sources"** section → per-client source override.

## When this runs

- **Automatically**, invoked by `blog-post-creation` right after the Phase 3.5 humanizer pass,
  before final validation. The verification report is shown together with the draft at the
  Text Approval Gate (one approval covers text + links).
- **Standalone / on demand**, when the user says "add backlinks to this post", "add authority
  citations", "add sources", or "redo the links" — including on an old/finished post with no
  research files.

## Inputs

| Input | Source |
|-------|--------|
| Draft path | from the caller, or `{cwd}/.speakr/last-approved-text.json`, or the file the user names |
| Topic / primary keyword | the brief / SEO metadata, or inferred from the draft's H1 + headings |
| Client domain | `BRAND-CONTEXT.md → Identity` (to exclude internal links) |
| Source-tier list | [references/source-tiers.md](references/source-tiers.md) + optional `BRAND-CONTEXT.md` override |
| Existing research (optional) | fact-check / content-research source lists, if present — a shortcut only |

## Process

### Step 1 — Resolve the effective source list

Build the effective authoritative-source list from [references/source-tiers.md](references/source-tiers.md):
global defaults, plus/minus the client's `BRAND-CONTEXT.md → Authority Sources` override. Always
apply the excluded categories.

### Step 2 — Discover candidates

Find candidate sources for the claims in the draft. **Always run a fresh search** — do not depend
on prior research files (reuse them only as a head start if they exist).

For each major claim, statistic, or section that would benefit from support, query Exa:

```
mcp__MCP_DOCKER__web_search_exa(query="<claim/topic> + <authoritative outlet or 'research'/'study'/'report'>")
```

Build queries from the post's topic, primary keyword, and the specific claim — e.g.
`"AI adoption enterprise productivity McKinsey report"`, `"remote work output study HBR"`. Aim to
collect **more candidates than needed** (15-20) so there's headroom for verification to drop some.
Filter out anything not on the effective list and anything in the excluded categories. Avoid
over-relying on one domain — spread citations across sources.

### Step 3 — Verify each candidate is LIVE (mandatory)

For every candidate URL, fetch it and confirm it is the real, live article:

```
WebFetch(url="<candidate>", prompt="Is this a live, specific article/study page (not a homepage,
search/category/tag page, paywall, or login wall)? What is its title and main claim? Does it
support: '<the claim we want to cite>'?")
```

Accept a candidate **only** if it is `live-article` — a specific, currently-live page whose content
matches the claim. **Reject** (and record the reason) when it is:

- `not-found` — 404 or fails to load
- `redirect-offsite` — redirects to a different page/site than intended
- `homepage-or-listing` — homepage, search results, category/tag/hub page
- `paywall-or-login` — paywall or login wall blocks the article
- `content-mismatch` — live, but does not actually support the claim
- `fetch-error` — could not be fetched

Cache results within the run (never re-fetch the same URL twice).

> **Never** place a link that has not passed this check. Never invent, guess, or approximate a
> URL. Prefer evergreen pages with stable URLs.

### Step 4 — Hard floor of 8 (never pad)

Keep looping Step 2 → Step 3 (new queries, new candidates) until **at least 8** citations verify
`live-article`. Target **8-12** total.

If, after genuinely exhausting reasonable searches, **fewer than 8** verify, **STOP** and present
the verified set to the user with options — do **not** pad with weak or unverified links:

> I could only verify **N** live authority sources for this post (target is 8-12). Here they are:
> [list]. How do you want to proceed?
> 1. Use these N as-is
> 2. Lower the target to N
> 3. Let me search more / broaden the source list

### Step 5 — Place the citations

Place the 8-12 verified citations inline as **Markdown links**, next to the claims they support:

```markdown
... according to [McKinsey's research on AI adoption](https://www.mckinsey.com/...).
```

Rules:
- Markdown links only (`[anchor phrase](url)`). Do **not** write raw `<a>` HTML — anchors are
  rendered at output by `blog-preview` / `cms-publisher`.
- Natural anchor phrases that read as part of the sentence (not "click here", not the bare URL).
- Never link to the client's own domain (that's an internal link).
- Place each citation near the statement it backs; spread them through the body, not clustered.

### Step 6 — Report

Show a verification report (see below). When running inside `blog-post-creation`, this report is
presented with the draft at the Text Approval Gate.

## Verification report (always shown)

```
AUTHORITY CITATIONS — <draft name>
Placed: <N> (target 8-12)

#  Anchor phrase                         Source (tier)            Status
1  McKinsey's research on AI adoption     mckinsey.com (T1)        ✓ live
2  a 2024 HBR analysis                    hbr.org (T1)             ✓ live
...
Rejected during verification:
- example.com/old-report — not-found (404)
- forbes.com/... — paywall-or-login
```

## "Redo the links" (re-run only this step)

When the user says "redo the links":

1. Capture the **current** authority citations already in the draft (anchor + URL).
2. Run Steps 1-5 again to produce a fresh set.
3. Show **OLD vs NEW side by side** before changing the draft:

   ```
   OLD → NEW citation comparison
   OLD (8):  [list]
   NEW (10): [list]
   Changes: removed 2 (reasons), added 4, kept 6
   ```
4. Only after showing the comparison, update the draft. Do **not** re-run drafting, editing, or
   the humanizer — this step touches links only.

## If the verification tool is unavailable

If `WebFetch` errors out for every candidate (the verification capability is down), do **not**
place any links. Surface it plainly and offer to continue without citations:

> I can't verify links right now (the fetch tool isn't responding), and I won't add links I
> haven't confirmed are live. Options:
> 1. Skip authority citations for now and continue to text approval — add them later with "redo the links"
> 2. Wait and retry verification

The rest of the draft is not blocked.

## Out of scope

- Internal links (client's own pages) — handled by `blog-post-creation`.
- Rendering anchors to HTML — handled by `blog-preview` / `cms-publisher`.
- Re-checking links after publication.
