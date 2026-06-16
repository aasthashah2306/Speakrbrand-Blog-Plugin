# Authority Source Tiers

Defines what counts as an "authority citation" (per [CONTEXT.md](../../../CONTEXT.md)): an
external, high-authority, third-party source — never the client's own site, never a blog/forum/
SEO-filler page.

## Effective list = global defaults (±) per-client override

1. Start from the **global default tiers** below.
2. If the active client's `BRAND-CONTEXT.md` has an **"Authority Sources"** section, apply it:
   - Sources listed there are **added** to the defaults.
   - If that section says `replace:` then use ONLY the client's list (for specialized fields).
3. Always apply the **excluded categories** — they override everything.

## Global default tiers (Tier-1)

**Tier 1 — strongest (prefer these):**
- Harvard Business Review (hbr.org)
- McKinsey & Company (mckinsey.com)
- Boston Consulting Group (bcg.com)
- Deloitte Insights (deloitte.com)
- MIT Sloan Management Review (sloanreview.mit.edu) / MIT (mit.edu)
- Stanford (stanford.edu) and other top-tier university research
- Peer-reviewed journals (nature.com, sciencedirect.com, *.edu papers, NBER)
- Pew Research Center (pewresearch.org)
- Gartner / Forrester (published reports, not gated landing pages)

**Tier 2 — acceptable when Tier 1 is thin:**
- Major business/news publications with editorial standards: The Economist, Financial Times,
  Wall Street Journal, Bloomberg, Reuters, The New York Times, Forbes (staff/contributor-vetted
  articles only), Harvard/Wharton/Kellogg faculty pieces.
- Established industry bodies and standards orgs relevant to the topic.

## Per-client override (read from BRAND-CONTEXT.md)

Specialized clients can define their own authoritative outlets. Example for a healthcare client:

```
## Authority Sources
- nejm.org
- jamanetwork.com
- thelancet.com
- who.int
```

Add `replace:` as the first line of that section to use ONLY the client list instead of merging.

## Excluded — never cite (always rejected)

- Personal blogs, Medium posts, Substack (unless the author IS the cited authority)
- Forums, Reddit, Quora, Stack-style Q&A
- SEO-content farms, listicle/affiliate sites, press-release wires
- Vendor marketing pages, product landing pages, pricing pages
- The client's own domain (those are **internal links**, a separate rule)
- Wikipedia (use it to find the primary source, then cite that)

## How "authoritative" is judged

A source qualifies if it is on the effective list OR is a clearly reputable primary source
(original research, official statistics, named expert publication) that a senior HBR/Forbes
editor would accept. When unsure, prefer a Tier-1 source over a borderline one. Editorial
judgment fills the gap the list cannot enumerate.

## Client-domain note (for separating internal links from citations)

The client's own website host comes from `BRAND-CONTEXT.md → Identity` (website URL). Any link to
that host is an **internal link** (existing min-2 rule in blog-post-creation), not an authority
citation. `aeo-validator.py` receives it via `--client-domain` to keep the counts separate.
