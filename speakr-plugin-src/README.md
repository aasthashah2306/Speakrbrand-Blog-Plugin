# Speakr Blog Pipeline

End-to-end blog content pipeline for thought leadership brands. Turns a topic idea into a published blog post — written in the client's voice, SEO-optimized, fact-checked, humanized, previewed in the client's brand, and auto-published to their CMS.

**v0.4.0** — adds:
- 🧠 **Smart GDrive onboarding.** Knows the canonical Speakr client folder structure (`Branding/Brand Blueprint.xlsx`, `Branding/IP STACK.pdf`, `Branding/Branding FINAL.pdf`, `Additional Resources/Blogs copy/`, `Additional Resources/Testimonials`, `Keynotes/`, etc.). Pulls each file to the right local destination and parses the high-impact ones (Brand Blueprint, IP STACK) into BRAND-CONTEXT.md and WRITING-GUIDELINES.md.
- 🔄 **`drive-sync` skill.** When the client adds new blogs / testimonials / brand updates to their Drive, run `/drive-sync` to pull only what's changed since last sync, refresh voice DNA, and update the right context fields. The pipeline gets smarter about each client over time.

**v0.3.0** — earlier:
- 🗂️ **GDrive-first onboarding.** Pulls samples / logo / brand book from Drive instead of asking for local uploads.
- 🎙️ **Voice always wins.** When `WRITING-GUIDELINES.md` conflicts with formatting rules, voice wins.
- 🔒 **Strict ordering: text → graphics → preview → publish.** Approval markers between each step.
- 🖼️ **Styled HTML preview** via the `blog-preview` skill.
- 🚀 **Auto-publish after preview approval.**

**v0.2.0** — guided multi-client onboarding, brand-aware graphics, mandatory humanizer pass, per-colleague client isolation.

## What It Does

This plugin runs the full content pipeline. It supports multiple clients per colleague, with each client's brand voice, keyword strategy, brand colors, logo, and CMS config kept in an isolated `clients/<slug>/` folder.

## Skills (12)

| # | Skill | What it does |
|---|-------|-------------|
| 1 | **client-onboarding** | 8-phase guided setup. Knows Speakr's canonical Drive folder structure — pulls Brand Blueprint, IP STACK, Branding book, Competitive Analysis, blogs, testimonials, logos automatically. |
| 2 | **drive-sync** | Pulls only what's new in the client's Drive since last sync. Refreshes voice DNA from new blogs, updates testimonials/competitors/brand. Run periodically or before drafting. |
| 3 | **seo-content-brief** | SEO brief with keywords, structure, word targets |
| 4 | **seo-research** | Keyword validation + SERP analysis via DataForSEO |
| 5 | **content-research** | Deep topic research with source discovery |
| 6 | **blog-post-creation** | Draft post in client's voice. **Voice always wins** over formatting rules. Mandatory humanizer pass. Ends with TEXT APPROVAL gate. |
| 7 | **fact-checker** | Verify claims, statistics, sources |
| 8 | **editing-checklist** | 11-step editing framework |
| 9 | **humanizer** | Auto-invoked from blog-post-creation with client's voice sample |
| 10 | **blog-graphics** | Branded JSX graphics. Refuses to run until text is approved. |
| 11 | **blog-preview** | Renders approved post + graphics as styled HTML using client's brand. Approval triggers auto-publish. |
| 12 | **cms-publisher** | Auto-publishes to Webflow / WordPress once preview is approved. |

## Quick Start

1. **Install the plugin** in Cowork.
2. Say: **"onboard a new client"**.
   - The skill checks which MCP connectors are active (Webflow / WordPress / DataForSEO / Exa / Perplexity / Notion / Drive) and tells you exactly how to connect missing ones.
   - It then walks you step-by-step through: choosing where to keep your client files, client identity, brand materials (writing samples + logo + colors + typography), voice DNA, SEO business context.
   - At the end you get a `clients/<slug>/` folder with everything downstream skills need, and a `Gaps to fill` section listing anything still missing.
3. Say: **"write a blog for <client> about <topic>"** — the pipeline runs end-to-end.
4. Say: **"publish this blog"** — uploads to the client's CMS.

## Required Connectors

- **Webflow** or **WordPress** — at least one, depending on the client's CMS. Pipeline still drafts without these; only publishing is blocked.

## Strongly Recommended Connectors

- **DataForSEO** — keyword research and SERP analysis (skills 2-3). Without it, SEO research falls back to manual keyword lists.
- **Exa** — web search for content research and fact-checking (skills 4, 6). Without it, content research and fact-checker run shallow.

## Optional Connectors

- **Perplexity** — deeper topic research during content-research phase
- **Notion** — collaboration on drafts and approvals
- **Google Drive** — collaboration on drafts and approvals

The `client-onboarding` skill checks for these automatically and tells you what's missing on first run. See `CONNECTORS.md` for full setup instructions.

## Multi-Client Architecture

Each colleague maintains their own clients in their own workspace — **clients are never shared or mixed across colleagues**.

### How it works

On first run, `client-onboarding` writes `{cwd}/.speakr/config.json`:
```json
{
  "clientsDir": "/Users/<colleague>/work/clients",
  "version": "0.2.0"
}
```

This `clientsDir` is where the colleague's clients live. Default is `./clients/` in the working directory, but it can be set to anywhere (including `~/speakr-clients/` for a stable per-user location).

Every client folder under `clientsDir` has the same structure:

```
<slug>/
├── BRAND-CONTEXT.md          # master context (loaded by every skill)
├── WRITING-GUIDELINES.md     # voice DNA, 10 patterns
├── SEO-GUIDELINES.md         # keyword clusters + business context
├── brand/
│   ├── logo.png
│   ├── logo-dark.png         # optional
│   ├── colors.json           # primary/secondary/accent/background/text
│   └── typography.json       # heading/body/mono fonts
├── samples/                  # writing samples for voice calibration
└── references/               # competitors, target audience, etc.
```

The active client is tracked in `<clientsDir>/.speakr/active-client` — just the slug. Switch by saying *"work on <client>"*.

## Pipeline Flow (v0.3.0)

```
Onboard Client (once per client)
   └─ Connector check → MCP gaps surfaced
   └─ Brand materials: GDrive pull first, local-drop fallback
   └─ Voice DNA extracted from samples
   └─ SEO business context captured (industry, ICP, competitors)
        ↓
SEO Brief → SEO Research → Content Research
        ↓
Blog Post Creation
   └─ Loads active client context
   └─ Drafts in client's voice — VOICE WINS OVER FORMATTING RULES
   └─ Fact Checker → Editing Checklist → Humanizer (mandatory)
   └─ 🛑 TEXT APPROVAL GATE — user reviews text-only draft
        ↓
Blog Graphics  (refuses to run without TEXT APPROVED marker)
   └─ Loads brand colors / typography / logo from client's brand/ folder
   └─ Generates .jsx artifacts with BRAND.* constants
   └─ 🛑 GRAPHICS APPROVAL GATE
        ↓
Blog Preview  (refuses to run without TEXT + GRAPHICS approved)
   └─ Renders styled HTML using client's colors + fonts + logo
   └─ Inlines approved graphics
   └─ 🛑 PREVIEW APPROVAL GATE
        ↓
CMS Publisher  (auto-invoked once preview is approved)
   └─ Webflow / WordPress
   └─ Live URL returned
```

The three approval markers (`text`, `graphics`, `preview`) live in `{cwd}/.speakr/`. Each downstream skill checks the relevant markers before running. This prevents accidentally rewriting graphics after a text change, or publishing before the visual is reviewed.

## Continuous Improvement

The pipeline is designed to get smarter about each client over time:

- **Gap detection.** Skills that hit a `# TODO:` placeholder in BRAND-CONTEXT.md surface it inline and offer to top it up via `/client-onboarding`.
- **Feedback loop.** Tell the pipeline *"the voice was too formal in that last post"* and it updates WRITING-GUIDELINES.md and logs the change to BRAND-CONTEXT.md's Changelog.
- **Sample-driven voice refinement.** Add more samples to `samples/` and run *"update voice for <client>"* to refresh the voice DNA.

## Reference Example

For a fully-populated example client (real materials from Kevin Surace's AI thought leadership brand), see `examples/kevin-surace-reference/` at the plugin root. **Do not copy these files into your own clients/ folder** — they contain a specific client's voice. Use them as a reference for what good looks like.

## Team Usage

Any team member with Cowork can install this plugin and run the full pipeline independently. Each colleague's `clientsDir` is private to them. The plugin itself ships with only the skill instructions, templates, and the kevin-surace-reference example — no actual client data.

## Built by

Speakr Brand — aastha@speakrbrand.com
