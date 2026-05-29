# {Client Name} — Brand Context

> This is the master file. Every pipeline skill reads from here. Keep it accurate.

## Identity
- **Name:** {full name}
- **Slug:** {client-slug}
- **Title:** {professional title}
- **Company:** {company and role}
- **Credentials:** {key credentials, awards, patents, publications, or `# TODO`}
- **Website:** {URL or `# TODO`}
- **LinkedIn:** {URL or `# TODO`}
- **CMS Platform:** {webflow | wordpress | none}
- **CMS Site ID:** {site/collection ID or `# TODO`}

## Brand Assets
- **Logo:** `brand/logo.png` ({present | `# TODO`})
- **Logo (dark):** `brand/logo-dark.png` ({present | `# TODO`})
- **Colors:** see `brand/colors.json`
- **Typography:** see `brand/typography.json`
- **Brand book:** `brand/brand-guidelines.pdf` ({present | `# TODO`})

## Drive Sync
<!-- Populated by client-onboarding when pulled from Drive. Used by drive-sync to watch for new files. -->

- **driveFolderId:** {Drive folder ID or `# NOT_SYNCED` if local-only client}
- **driveFolderUrl:** {URL}
- **lastDriveSyncAt:** {ISO timestamp}
- **autoSyncEnabled:** {true | false} <!-- if true, drive-sync may run on schedule; if false, only on-demand -->

## Proof Points
<!-- Pulled from Testimonials spreadsheet during onboarding. Used by blog-post-creation as social proof. -->

- {Name}, {Title}, {Company} — "{short quote}" → [video]({url})
- ...

## Voice
- **Samples on file:** {N samples in `samples/` | `# TODO: needs samples`}
- **Voice DNA:** see `WRITING-GUIDELINES.md`
- **{Client} SAYS:** "{characteristic quote}"
- **{Client} NEVER SAYS:** "{anti-quote}"

## SEO Context
- **Industry:** {specific industry — e.g. "B2B SaaS for HR teams"}
- **ICP:** {ideal customer profile / reader}
- **Geo:** {global | US | EU | specific region}
- **Products/Services:** {what they sell or offer}
- **Competitors:**
  1. {competitor 1 URL}
  2. {competitor 2 URL}
  3. {competitor 3 URL}
- **Seed keywords:** {comma-separated, or `# TODO`}
- **See:** `SEO-GUIDELINES.md` for full keyword strategy

## Content Pillars
1. {Pillar 1} — {description}
2. {Pillar 2} — {description}
3. {Pillar 3} — {description}

## Internal Link Targets
- {Page name} → {URL}
- {Page name} → {URL}

## Gaps to fill
<!-- Auto-maintained. Lists every `# TODO` in this file. -->

- {gap 1}
- {gap 2}

## Changelog
<!-- Auto-maintained. Notes voice/brand/SEO adjustments from user feedback. -->

- {YYYY-MM-DD}: {change description}
