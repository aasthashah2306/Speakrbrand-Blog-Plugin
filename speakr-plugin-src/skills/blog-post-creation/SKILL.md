---
name: blog-post-creation
description: "Creates SEO-optimized blog posts for the active client's thought leadership brand. RECEIVES brief, SEO research, content research. MANDATORY: Run validation scripts after drafting and editing - formatting-validator.py, readability-scorer.py MUST PASS. Creates drafts with extensive bullets (8-12 sections), paragraph variety (1, 2, 3 sentences mixed), high readability (Flesch-Kincaid 60-70) for non-native English speakers. Applies 11-step editing framework. Scripts enforce variety and scannability programmatically."
---
## Client Context Loading

Before starting any work, load the active client's brand context:

1. Read `{cwd}/.speakr/config.json` to find `clientsDir` (the colleague's chosen client storage path).
2. Read `{clientsDir}/.speakr/active-client` to get the active client slug.
3. If the user specified a different client by name in the request, use that instead and update `.speakr/active-client`.
4. If `.speakr/config.json` does not exist, or no active client is set, instruct the user to run `/client-onboarding` first.
5. Read these files for the active client:
   - `{clientsDir}/{slug}/BRAND-CONTEXT.md` — master context (identity, voice quick reference, SEO context, gaps)
   - `{clientsDir}/{slug}/WRITING-GUIDELINES.md` — full voice DNA
   - `{clientsDir}/{slug}/SEO-GUIDELINES.md` — keyword clusters, business context
   - `{clientsDir}/{slug}/brand/colors.json` — for any color references in the draft
6. If any of these files contain `# TODO:` placeholders relevant to the post you're drafting, surface them to the user and offer to call `/client-onboarding` to fill the gap before proceeding.

All references in this skill to "brand voice", "keywords", "brand colors", "target audience", "internal link targets", and "content pillars" refer to the active client's context files. Do not use hardcoded examples.

---

## 🛑 PRECEDENCE RULE — VOICE ALWAYS WINS

**`WRITING-GUIDELINES.md` is the highest authority in this entire pipeline.** Every other rule in this skill — formatting requirements, paragraph variety, bullet counts, readability targets, validation scripts — is subordinate to it.

**The rule:**

> If the active client's `WRITING-GUIDELINES.md` (especially its "Forbidden patterns" or "Formatting habits" sections) conflicts with any formatting requirement in this skill, **the voice wins. Always. No exceptions.**

**Concrete examples of voice overrides:**

- Client's voice forbids bullets in personal essays → do not force 8-12 bullet sections. Write flowing prose. Skip `formatting-validator.py`'s bullet check.
- Client's voice uses long, literary sentences → do not enforce 15-20-word average. Skip `readability-scorer.py`'s sentence-length check.
- Client's voice prefers 4-5 sentence paragraphs for storytelling → do not auto-reject. Skip `formatting-validator.py`'s 4+ sentence check.
- Client's voice uses formal/academic register → do not force FK grade 8-10. Use whatever grade level the voice produces naturally.

**How to handle the conflict:**

1. **Detect the conflict** before drafting. Read WRITING-GUIDELINES.md → "Forbidden patterns" and "Formatting habits" sections in Phase 1.
2. **Note conflicts explicitly.** When you find one, log it: *"Voice forbids bullets → skipping formatting-validator's bullet section check."*
3. **Skip the conflicting validation.** Run the script with `--skip-bullets`, `--skip-readability`, etc. (or note in the script output why you accepted the failure).
4. **Document in the metadata file** (Phase 4):
   ```
   Voice Overrides Applied:
   - Skipped bullet validation (voice forbids bullets in personal essays)
   - Skipped readability check (voice uses long literary sentences)
   ```
5. **Never silently ignore.** The metadata file should make it clear which rules were overridden and why.

**Why this matters:** The whole point of multi-client support is that each client sounds like *themselves*. A generic post that hits every formatting metric but doesn't sound like the client is a failure, no matter what the scripts say. Voice is the product.

---

# Blog Post Creation Skill

## Purpose

Creates **publication-ready** blog posts for the active client's thought leadership brand based on provided research inputs.

**What This Skill Does:**
✅ Receives research inputs (brief, SEO research, content research)
✅ Creates drafts with enforced formatting (variety, bullets, visuals)
✅ Validates with scripts (formatting, readability, SEO - MANDATORY)
✅ Applies comprehensive editing (11 frameworks)
✅ Outputs publication-ready markdown

**What This Skill Does NOT Do:**
❌ SEO pre-research (use `blog-post-seo-research` skill)
❌ Content research (use `blog-post-content-research` skill)

---

## 🚨 CRITICAL: Formatting Requirements (ENFORCED BY SCRIPTS)

**The #1 Problem:** Previous outputs had fat paragraphs, few bullets, poor scannability.

**The Solution:** Validation scripts now **enforce** these requirements:

### Required Formatting (Validated Programmatically)

**Paragraph Variety (MANDATORY):**
- 10-15 single-sentence paragraphs (punch, emphasis)
- 20-25 two-sentence paragraphs (explanations)
- 5-10 three-sentence paragraphs (maximum depth)
- **ZERO** 4+ sentence paragraphs (auto-reject)

**Extensive Bullets (MANDATORY):**
- 8-12 bullet sections minimum
- 3-5 instances of sub-bullets (nested lists)

**Visual Elements (MANDATORY):**
- 2-4 visual placeholders: [Screenshot: ...], [Diagram: ...]
- 1-2 tables for data comparisons
- 8-12 bold headers for hierarchy

**Readability (MANDATORY - Non-Native English Speakers):**
- Flesch Reading Ease: 60-70 (Easy)
- Flesch-Kincaid Grade: 8-10 (Middle school)
- Average sentence length: 15-20 words
- Complex words: <15%
- Passive voice: <10%

> **📚 See:** `references/formatting-guide.md` for visual examples
> **📚 See:** `references/readability-guide.md` for simplification techniques
> **📚 See:** `references/paragraph-patterns.md` for variety patterns

---

## Required Inputs (ALL MANDATORY)

### Input #1: The Brief
**Contains:** Topic, target keyword, content structure, headings, word count, audience

**⚠️ BRIEF IS SACRED:** Follow structure exactly.

### Input #2: SEO Research File
**Format:** `seo-research-[keyword]-[date].md`
**Contains:** Primary keyword, search intent, SERP competitors, H2/H3 keywords, strategy

### Input #3: Content Research File
**Format:** `content-research-[keyword]-[date].md`
**Contains:** Statistics (with sources), architecture docs, workflow diagrams, ERD, competitor analysis

**⚠️ IF ANY INPUT MISSING:** Request all three before proceeding.

---

## Phase 0: Input Validation

**Confirm all inputs received:**
```
✅ Brief: [topic]
✅ SEO Research: [keyword] - [volume]
✅ Content Research: [X stats], architecture, workflow, ERD
✅ Ready to proceed
```

---

## Phase 1: Digestion & Planning (MANDATORY BEFORE DRAFTING)

**⚠️ CRITICAL:** Do NOT skip this phase. Review ALL inputs before writing.

### Step 1.1: Review Brief
- [ ] Topic, keyword, structure understood
- [ ] Word count target noted (or default 1,500-2,000)
- [ ] Required H2 sections documented

### Step 1.2: Review Brand Context
> **📚 Read:** `{clientsDir}/{active-slug}/BRAND-CONTEXT.md` and `WRITING-GUIDELINES.md`

- [ ] Target persona identified (from BRAND-CONTEXT.md → SEO Context → ICP)
- [ ] Pain point addressed (matches a Content Pillar)
- [ ] Client's unique angle clear (from BRAND-CONTEXT.md → Positioning)

### Step 1.3: Extract SEO Data
From SEO research file:
- Primary keyword: [keyword]
- H2 keywords (high-volume): [list for mapping]
- H3 keywords (medium-volume): [list]
- Target H2 count: 5-8
- Target H3 count: 10+

### Step 1.4: Extract Statistics & Sources
From content research file:
- Create statistics inventory
- Map stats to entities (use ERD to prevent mismatches)
- Track domain usage (each domain ONCE only)
- Max 3 external citations

### Step 1.5: Review Guidelines
> **📚 Read:** `{clientsDir}/{slug}/WRITING-GUIDELINES.md` (the active client's voice DNA)
> **📚 Read:** `{clientsDir}/{slug}/SEO-GUIDELINES.md` (the active client's keyword and business context)

Note requirements:
- Hook sentence pattern
- Citation format: `(Source: Domain)`
- NO third-party case studies
- Max 3 external links, min 2 internal links

### Step 1.6: Review Examples (reference patterns, not content)
> **📚 Open:** `examples/kevin-surace-reference/blog-post-creation/examples/` at the plugin root for fully-populated reference posts.

Note **structural** patterns only — do not copy phrasing into other clients' posts:
- Paragraph variety (1, 2, 3 sentence mix)
- Extensive bullet usage
- Visual placeholders
- Bold headers
- Tables

### Step 1.7: Create Drafting Plan
Document:
- H2 structure (from brief + SEO keywords)
- Statistics placement (which sections)
- Visual placeholders (where to insert)
- Link strategy (2+ internal, 3 max external)
- Word count distribution per section

**Digestion Complete Checkpoint:**
- [ ] All inputs reviewed
- [ ] Brand angle determined
- [ ] SEO data organized
- [ ] Statistics inventoried
- [ ] Drafting plan created

---

## Phase 2: Drafting

**⚠️ DRAFT AT TARGET WORD COUNT** - Do NOT overdraft and trim later.

### Step 2.1: Opening (MANDATORY STRUCTURE)

**Sentence 1: Hook with keyword**
- Contains primary keyword
- Complete standalone sentence
- States SPECIFIC value (NOT "transforms" vagueness)

**Sentences 2-3: Amplify with statistics**
- Use statistic from content research
- Add bridge phrase to connect
- Citation format: `(Source: Domain Name)`

**Example:**
```markdown
AI won't replace leaders — but it will expose the ones who stopped being curious.

McKinsey reports 70% of companies will adopt AI by 2030, yet fewer than 15% of leaders feel prepared (Source: McKinsey).

The gap isn't technical. It's about curiosity — and that's a skill you can build.
```

### Step 2.2: Body Structure

Follow brief structure exactly. For each section:

**Use Paragraph Variety:**
- Mix 1, 2, and 3 sentence paragraphs
- NEVER use 4+ sentences in one paragraph

**Use Extensive Bullets:**
- 8-12 bullet sections throughout article
- Use for: features, benefits, steps, comparisons, lists
- Add sub-bullets (3-5 instances of nested lists)

**Add Visual Hierarchy:**
- Bold headers (8-12 minimum): **Why this matters**, **How it works**
- Visual placeholders (2-4): [Screenshot: description], [Diagram: ...]
- Tables (1-2): For data comparisons

**Write for Non-Native Speakers:**
- Short sentences (15-20 words average)
- Simple vocabulary
- Active voice
- Clear structure

> **📚 See:** `references/paragraph-patterns.md` for specific patterns
> **📚 See:** `references/readability-guide.md` for simplification techniques

### Step 2.6: Article Ending Structure (MANDATORY)

**⚠️ CRITICAL:** Articles MUST end with a Conclusion section, NOT "Getting Started" or "Next Steps" as the final section.

**CORRECT Final Structure:**
1. Body sections (implementation, benefits, how-to)
2. **Final section: "Conclusion"** with:
   - Key takeaways (3-5 bullet points)
   - Actionable summary
   - Brief next steps (optional, as sub-section)
   - Closing statement

**Example Conclusion Structure:**
```markdown
## Conclusion

Your path to [topic]:

• [Key point 1]
  - [Sub-detail]
  - [Sub-detail]

• [Key point 2]
  - [Sub-detail]

• [Key point 3]
  - [Sub-detail]

Remember: [Closing wisdom statement]
```

**❌ WRONG (Do NOT end with):**
- "Getting Started" section
- "How to Get Started" section
- Implementation steps as final section

**✅ CORRECT (Always end with):**
- "Conclusion" section
- Key takeaways
- Summary of learnings

> **📚 See:** plugin root `examples/kevin-surace-reference/blog-post-creation/examples/` for two reference posts demonstrating conclusion patterns. These are reference-only — do not copy phrasing into other clients' posts.

### Step 2.3: Statistics Usage

For EVERY statistic:
1. Check entity mapping (use ERD from content research)
2. Add bridge phrase ("That reality shows...", "The data confirms...")
3. Track domain usage (each used ONCE only)

**NO Third-Party Case Studies:**
❌ "Company X saved 750 hours with their AI tool"
✅ "At [Client Company], we've found that [domain-specific result]"
✅ Use the client's own experiences, anonymized examples, or published research

### Step 2.4: Link Integration

**Internal Links (min 2):**
- Link to pages listed in BRAND-CONTEXT.md → "Internal Link Targets". These typically include the client's services page, about page, book/publication page (if applicable), other blog posts.
- Format: `[anchor text](https://{client-website}/page)` — use the website URL from BRAND-CONTEXT.md → Identity.

**External Citations (max 3):**
- Inline format: `(Source: Domain Name)`
- NO full URLs in text
- Tier 1/2 authoritative sources only

### Step 2.5: Natural Formatting (AI Detection Avoidance)

**Vary spacing** - NO uniform patterns:
- Some H2 sections start immediately after heading
- Other H2 sections have line break after heading
- Mix naturally, think "breathing room vs. immediate flow"

---

## 🚨 CHECKPOINT: Mandatory Validation After Drafting

**⚠️ CRITICAL:** Run these scripts BEFORE proceeding to editing.

### Validation 1: Formatting (MUST PASS)

```bash
python scripts/formatting-validator.py draft.md
```

**What it validates:**
- Paragraph variety (1, 2, 3 sentence distribution)
- NO fat paragraphs (4+ sentences = auto-reject)
- Bullet sections (need 8-12 minimum)
- Visual placeholders (need 2-4)
- Tables (recommend 1-2)
- Bold headers (need 8-12)
- Sub-bullets (need 3-5 instances)

**If FAIL:** Fix issues immediately. Re-run until PASS. DO NOT proceed to editing.

> **📚 See:** `references/validation-workflow.md` for interpretation guide

### Validation 2: Readability (MUST PASS)

```bash
python scripts/readability-scorer.py draft.md
```

**What it validates:**
- Flesch Reading Ease (target: 60-70)
- Grade Level (target: 8-10)
- Sentence length (target: 15-20 words avg)
- Complex words (target: <15%)
- Passive voice (target: <10%)

**If FAIL:** Simplify vocabulary, break long sentences, use active voice. Re-run until PASS.

### Validation 3: SEO (MUST PASS)

```bash
python scripts/seo-validation-helper.py draft.md "target keyword"
```

**What it validates:**
- H1 heading (one only, includes keyword)
- H2/H3 count and keyword usage
- Keyword placement (H1, first 100 words, conclusion)
- Keyword density (1.5-2.5%)
- Meta title/description

**If FAIL:** Fix SEO issues before editing. Re-run until PASS.

**✅ ALL THREE MUST PASS BEFORE EDITING.**

---

## Phase 3: Editing (ALL 11 FRAMEWORKS)

> **📚 See:** `editing-checklists/` directory for detailed checklists

**⚠️ SLOW DOWN:** 15-20 minutes minimum for 1,500 words. If you finish faster, you RUSHED.

### Framework 1: Comprehensive Edit Checklist (MOST IMPORTANT)
> **Apply:** `editing-checklists/01_Editing_-_Edit_Checklist.md`

- Opening redundancy (paragraphs 1-2 often repeat)
- Outcome-first headers
- Stats with timeframes and baselines
- Remove filler words
- Avoid UI-instruction tone
- Make CTAs specific
- Style consistency (en dashes, curly quotes, contractions)
- Tighten verbs

**Time:** 15-20 minutes minimum

### Frameworks 2-11: Apply in Order
> **See:** `editing-checklists/` for detailed instructions

2. Redundancy Deep Dive
3. Active Voice
4. Parallelism
5. Tenses (favor simple present)
6. Sentence Variety
7. Specificity (quantifiable numbers)
8. Arguments (Claim → Support → Takeaway)
9. Structure (flow and transitions)
10. Takeaways (clear conclusions)
11. Final Word-by-Word Polish

**For each framework:**
- Apply thoroughly
- Preserve formatting (variety, bullets, visuals)
- Maintain readability
- Don't break SEO

### Post-Editing SEO Verification

After all editing, verify SEO still intact:
- [ ] H1 still correct with keyword
- [ ] H2 count still 5-8
- [ ] Keyword still in first 100 words, conclusion
- [ ] Meta title/description unchanged
- [ ] Keyword density still 1.5-2.5%
- [ ] Links still present (2+ internal, 3 max external)

---

## Phase 3.5: Humanizer Pass (MANDATORY)

**⚠️ This step is not optional. Every blog post must pass through the humanizer before final validation.**

The editing phase enforces structure and SEO; the humanizer pass removes AI-writing tells that downstream readers (and detection tools) catch. Skipping this is the single biggest source of "this sounds AI-generated" feedback.

### Step 3.5.1: Invoke the humanizer skill

Call the `humanizer` skill with these inputs:

- **Text to humanize:** the current draft (post-editing)
- **Voice sample:** the most representative file from `{clientsDir}/{slug}/samples/` (pick the longest sample that matches this post's topic, or the first sample if none match). The humanizer uses this for voice calibration so it does not flatten the client's voice.
- **Forbidden patterns:** the "Forbidden patterns" section from `WRITING-GUIDELINES.md` — pass these explicitly so the humanizer enforces them.

If `samples/` is empty, run humanizer in default mode and flag this to the user: "No voice sample available — humanizer ran in generic mode. Add samples to improve next time."

### Step 3.5.2: Verify formatting did not regress

The humanizer rewrites sentences and may break paragraph variety, bullet structure, or sentence-length distribution. After humanizer returns:

```bash
python scripts/formatting-validator.py final-draft.md
python scripts/readability-scorer.py final-draft.md
```

If either fails, request another humanizer pass that preserves bullet structure and the 1-2-3 paragraph mix. The humanizer should not strip bullets, headers, or visual placeholders.

### Step 3.5.3: Confirm voice still matches

Spot-check 3 random paragraphs against the "{Client} SAYS / NEVER SAYS" quick reference in `BRAND-CONTEXT.md`. If anything sounds off (too generic, lost the client's verbal tics), iterate on humanizer with a stronger voice sample.

---

## 🚨 CHECKPOINT: Final Validation Before Delivery

**⚠️ CRITICAL:** This is the FINAL GATE. Must pass before delivery.

### Run Master Validator (ALL CHECKS)

```bash
python scripts/style-enforcer.py final-draft.md "target keyword"
```

**This runs ALL four validations:**
1. Formatting validation
2. Readability scoring
3. Paragraph analysis
4. SEO validation

**ALL MUST PASS.**

**If ANY FAIL:**
- Review detailed reports
- Fix issues
- Re-run until ALL PASS
- DO NOT deliver until 100% pass rate

> **📚 See:** `references/validation-workflow.md` for detailed workflow

---

## Phase 4: Final Deliverables

### Format Final Markdown

Ensure:
- H1 title at top
- SEO Title and Meta Description included
- Proper heading hierarchy
- Line breaks between paragraphs
- Natural spacing (varied, not uniform)
- Bold headers throughout
- Bullet lists formatted correctly
- Tables formatted correctly
- Visual placeholders noted

### Create Publication Package

**Output files:**
1. `blog-post-[keyword]-[date].md` - Complete blog post
2. `metadata-[keyword]-[date].txt` - SEO metadata

**Metadata format:**
```
SEO Title: [50-60 chars]
SEO Meta Description: [150-160 chars]
Primary Keyword: [keyword]
Word Count: [X,XXX]
Internal Links: [X] - [URLs]
External Citations: [X] - [domains]
Formatting Score: [X]/100
Readability Score: [Flesch-Kincaid]
Variety Score: [X]/100
Publication Date: [TBD]
```

### Final Quality Verification

**Before delivery, confirm:**

**Content Quality:**
- [ ] Brief structure followed exactly
- [ ] Brand positioning matches BRAND-CONTEXT.md → Content Pillars
- [ ] Voice matches WRITING-GUIDELINES.md (re-read "{Client} SAYS / NEVER SAYS")
- [ ] Content gaps filled

**Formatting (Script-Validated):**
- [ ] Paragraph variety: 1, 2, 3 sentence mix
- [ ] NO fat paragraphs (0 with 4+ sentences)
- [ ] Bullet sections: 8-12 minimum
- [ ] Visual placeholders: 2-4 minimum
- [ ] Tables: 1-2 for comparisons
- [ ] Bold headers: 8-12 minimum
- [ ] Sub-bullets: 3-5 instances

**Readability (Script-Validated):**
- [ ] Flesch Reading Ease: 60-70
- [ ] Grade Level: 8-10
- [ ] Average sentence: 15-20 words
- [ ] Complex words: <15%
- [ ] Passive voice: <10%

**SEO (Script-Validated):**
- [ ] Keyword in H1, opening, conclusion
- [ ] Keyword density: 1.5-2.5%
- [ ] H2 count: 5-8
- [ ] H3 count: 10+
- [ ] Meta title: 50-60 chars
- [ ] Meta description: 150-160 chars

**Scripts (All PASSED):**
- [ ] formatting-validator.py: PASS
- [ ] readability-scorer.py: PASS
- [ ] paragraph-analyzer.py: PASS
- [ ] seo-validation-helper.py: PASS
- [ ] style-enforcer.py: PASS (master check)

**✅ If all checked: READY FOR TEXT APPROVAL**

---

## Phase 5: Text Approval Gate (BLOCKING — DO NOT SKIP)

**This is a hard gate. Do not generate graphics, do not generate previews, do not invoke any downstream skill until the user has explicitly approved the text.**

### Step 5.1: Present text-only for review

Present the final text-only markdown to the user. Do NOT include:
- Graphics
- Visual previews
- Any other artifacts

Just the post. Say something like:

> Here's the draft. Read it carefully. Once you approve the text, I'll move to graphics, then visual preview, then publish. If anything needs to change, tell me now — it's much faster to revise text than to rework graphics afterward.

### Step 5.2: Wait for explicit approval

Acceptable approvals:
- "Approved"
- "Looks good, move on"
- "Yes, proceed to graphics"
- Any clear go-ahead from the user

NOT approvals:
- Silence
- "Hmm, interesting"
- "Let me think"
- Any ambiguous response

If unclear, ask: *"Should I treat this as approved and move to graphics, or do you want changes first?"*

### Step 5.3: Write the approval marker

Once approved, write a marker file:

```
{cwd}/.speakr/last-approved-text.json
```

With contents:
```json
{
  "client": "<active-slug>",
  "topic": "<short topic from brief>",
  "draftPath": "<path to the approved markdown>",
  "approvedAt": "<ISO timestamp>"
}
```

This marker is what `blog-graphics` checks before it agrees to run. Without it, graphics generation is blocked.

### Step 5.4: Hand off to graphics

Tell the user:

> ✅ Text approved. Moving to graphics next. Run **"create graphics for this post"** or I'll invoke `blog-graphics` directly.

---

## Success Criteria

A blog post is successful when:

✅ **All scripts pass** (formatting, readability, SEO)
✅ **Variety score ≥80/100** (programmatically validated)
✅ **Flesch Reading Ease 60-70** (non-native accessible)
✅ **Brief followed exactly** (structure, requirements met)
✅ **No fat paragraphs** (0 with 4+ sentences)
✅ **Extensive bullets** (8-12 sections minimum)
✅ **Visual hierarchy** (bold headers, placeholders, tables)
✅ **Publication-ready** (zero additional editing needed)

---

## Common Mistakes (Prevented by Scripts)

**Script NOW catches:**
1. ❌ Fat paragraphs (4+ sentences) → Auto-reject
2. ❌ Too few bullets (<8 sections) → Fail
3. ❌ Poor variety (all 2-sentence paragraphs) → Fail
4. ❌ Low readability (grade >10) → Fail
5. ❌ Missing visuals (<2 placeholders) → Fail
6. ❌ Complex language (>15% complex words) → Fail

**These issues can NO LONGER slip through.**

---

## Quick Reference

**Active client context (loaded at start of every run):**
- `{clientsDir}/{slug}/BRAND-CONTEXT.md` — master file (identity, voice quick reference, SEO context, content pillars)
- `{clientsDir}/{slug}/WRITING-GUIDELINES.md` — full 10-pattern voice DNA
- `{clientsDir}/{slug}/SEO-GUIDELINES.md` — keyword clusters, business context
- `{clientsDir}/{slug}/brand/colors.json` — brand colors for any color references

**Generic references (this skill's references/):**
- `references/formatting-guide.md` - Visual good vs bad examples
- `references/readability-guide.md` - Non-native speaker techniques
- `references/paragraph-patterns.md` - Variety pattern library
- `references/validation-workflow.md` - How to use scripts
- `references/STRUCTURE-QUICK-REF.md` - Structure requirements

**Editing Checklists:**
- `editing-checklists/01_Editing_-_Edit_Checklist.md` - Master checklist (APPLY FIRST)
- `editing-checklists/02-11_*.md` - Additional frameworks

**Reference examples** (plugin root — for inspiration only, do not copy phrasing):
- `examples/kevin-surace-reference/blog-post-creation/examples/` — two complete posts from a real client

**Scripts (MANDATORY):**
- `scripts/formatting-validator.py` - Validate variety, bullets, visuals
- `scripts/readability-scorer.py` - Validate accessibility
- `scripts/paragraph-analyzer.py` - Deep structure analysis
- `scripts/seo-validation-helper.py` - Validate SEO elements
- `scripts/style-enforcer.py` - Run ALL validations (FINAL GATE)

**Templates:**
- `brief-template.md` - Brief format reference

---

## Workflow Summary (Condensed)

```
Phase 0: Input Validation
  └─ Confirm brief + SEO research + content research received

Phase 1: Digestion (MANDATORY)
  └─ Review ALL inputs, create drafting plan
  └─ Time: 20-30 minutes

Phase 2: Drafting
  └─ Draft at target word count (1,500-2,000)
  └─ Paragraph variety (1, 2, 3 sentences)
  └─ Extensive bullets (8-12 sections)
  └─ Visual hierarchy (bold, placeholders, tables)
  └─ Write for non-native speakers (simple, clear)
  └─ Time: 60-90 minutes

CHECKPOINT 1: Run Validation Scripts (MANDATORY)
  ├─ formatting-validator.py → MUST PASS
  ├─ readability-scorer.py → MUST PASS
  └─ seo-validation-helper.py → MUST PASS
  └─ Fix issues, re-run until ALL PASS
  └─ Time: 15-30 minutes (including fixes)

Phase 3: Editing
  └─ Apply all 11 editing frameworks in order
  └─ Time: 45-60 minutes (15-20 min on framework 1 alone)

Phase 3.5: Humanizer Pass (MANDATORY)
  └─ Call humanizer skill with voice sample from samples/
  └─ Re-run formatting-validator and readability-scorer
  └─ Spot-check voice against "{Client} SAYS / NEVER SAYS"
  └─ Time: 15-30 minutes

CHECKPOINT 2: Final Validation (MANDATORY)
  └─ style-enforcer.py final-draft.md "keyword"
  └─ Runs ALL four validations
  └─ ALL MUST PASS before delivery
  └─ Time: 15-20 minutes (including fixes)

Phase 4: Delivery
  └─ Format markdown
  └─ Create metadata file (include voice overrides applied)
  └─ Final verification checklist
  └─ Time: 10-15 minutes

Phase 5: Text Approval Gate (BLOCKING)
  └─ Present text-only draft to user
  └─ Wait for explicit approval
  └─ Write .speakr/last-approved-text.json marker
  └─ Hand off to blog-graphics
  └─ Time: depends on review

THEN (and only then):
  Phase 6 (blog-graphics skill): Generate graphics
  Phase 7 (blog-post-creation again): Generate styled preview
  Phase 8 (cms-publisher): Auto-publish on preview approval

TOTAL TIME: 3-4 hours of active work per blog post (excluding user review)
```

---

**Key Insight:** Previous outputs failed because instructions alone don't work. Scripts **enforce** requirements programmatically. Use them religiously.
