# Connectors

## How tool references work

Plugin skills reference external tools by category. During client onboarding, the specific tool is configured per client in their BRAND-CONTEXT.md file.

## Connectors for this plugin

| Category | Placeholder | Options | Required? |
|----------|------------|---------|-----------|
| CMS | ~~cms | Webflow, WordPress | Yes (one per client) |
| SEO Research | ~~seo-research | DataForSEO | Recommended |
| Web Search | ~~web-search | Exa | Recommended |
| Deep Research | ~~deep-research | Perplexity | Optional |
| File Sharing | ~~file-sharing | Google Drive, Notion | Optional |

## Setup Instructions

### Required: CMS Connector (one per client)

**Webflow:**
1. Go to Claude settings > Connectors
2. Search for "Webflow" and connect
3. Authorize with your Webflow account
4. The cms-publisher skill will use this to publish blog posts

**WordPress:**
1. Go to Claude settings > Connectors
2. Search for "WordPress" and connect
3. Provide your WordPress site URL and API credentials
4. The cms-publisher skill will use this to publish blog posts

### Recommended: Research Connectors

**DataForSEO:**
1. Go to Claude settings > Connectors
2. Search for "DataForSEO" and connect
3. Provide your DataForSEO API credentials
4. Used by: seo-content-brief, seo-research skills

**Exa:**
1. Go to Claude settings > Connectors
2. Search for "Exa" and connect
3. Provide your Exa API key
4. Used by: content-research, fact-checker skills

### Optional

**Perplexity** — for deeper topic research during content-research phase
**Google Drive / Notion** — for team collaboration on drafts and approvals
