---
name: linkedin-post-creation
description: "Creates LinkedIn posts and carousels for the active client's personal brand. This is NOT the blog pipeline — LinkedIn is short, conversational, and human, with no SEO, no H2 structure, and no readability scripts. Enforces: one strong scroll-stopping hook with NO formulaic rehook; the client's real voice pulled from their own post exemplars (capitalized unless the client says otherwise); an AI-tell removal pass via the humanizer skill; a prior-work check so used stories/angles are never repeated; truth-tier tagging so no invented sentiment is put in the client's mouth; per-post media direction; and carousels authored slide-shaped (every slide a takeaway, never the caption re-chopped). Use when the user asks to draft LinkedIn posts, carousels, hooks, or social copy for a speaker/thought-leader client, or to fill a client's LinkedIn/Asana content calendar."
---

## Client Context Loading

Before writing anything, load the active client's context:

1. Read `{cwd}/.speakr/config.json` to find `clientsDir`.
2. Read `{clientsDir}/.speakr/active-client` for the active client slug. If the user named a client, use that instead.
3. If `.speakr/config.json` is missing or no active client is set, tell the user to run `/client-onboarding` first.
4. Read for the active client:
   - `{clientsDir}/{slug}/BRAND-CONTEXT.md` — identity, positioning, audience
   - `{clientsDir}/{slug}/WRITING-GUIDELINES.md` — full voice DNA
   - `{clientsDir}/{slug}/CONTENT-LEARNINGS.md` — **if present: the per-client voice target, used/burned angles, source constraints, sensitive items.** This file overrides stale brand DNA.
   - `{clientsDir}/{slug}/samples/` — the client's real post exemplars (the single most important voice input)

---

## 🛑 Two things always win: the client's voice and the truth

Every rule below is subordinate to (a) the client's actual voice and (b) factual accuracy. A post that hits every structural rule but doesn't sound like the client, or that puts words/feelings in their mouth they never expressed, is a failure.

---

## What this skill does / does NOT do

✅ Drafts LinkedIn text posts, carousels, quote/stat cards, and short video scripts
✅ Pulls voice from the client's real exemplars and locks it before scaling
✅ Runs a prior-work check and a truth pass
✅ Hands off to the `humanizer` skill and gives per-post media direction

❌ No SEO, keywords, H2/H3 structure, or readability scripts (that's the blog pipeline)
❌ No long-form essays (use `blog-post-creation`)

---

## The default voice (when no stronger client signal exists)

Conversational and thinking-out-loud, like a smart person talking, not a keynote. **Capitalized normally unless the client's profile says lowercase.**

- Go **deep on ONE idea**. Do not list three names/points to seem thorough.
- Hedge like a human ("I think", "maybe I'm biased", "I don't anymore").
- Short lines. White space. One thought per line.
- End soft and genuine (a quiet line or a real question), never a rally cry.
- Self-deprecation beats self-flattery. Own "what I got wrong" instead of lecturing "what others get wrong."

Always prefer the cadence in the client's own exemplars over this default.

---

## Phase 0 — Prior-work check (MANDATORY, do this first)

Before pitching or drafting, make sure the angle is fresh:

1. Scan `{clientsDir}/{slug}/drafts/`, any prior LinkedIn series, `samples/`, and the `CONTENT-LEARNINGS.md` "used/burned angles" list.
2. If the client has an Asana content board, scan recent and scheduled tasks for the same story.
3. If the angle (or its core anecdote) has been used, pick a different angle or a genuinely new lens on it. Log what you avoided and why.

Never recycle a story the client has already published just because it's strong.

---

## Phase 1 — Voice calibration

1. Read 2–3 of the client's real exemplars. Note line length, how they open, their asides, how they end, their verbal tics.
2. **If this is the first batch for a client, or the voice is uncertain: draft 1–2 sample posts and get explicit approval BEFORE writing the full set.** Do not batch in an unconfirmed voice.

---

## Phase 2 — Hook and structure

- **Hook (line 1):** a genuine scroll-stopper that lands before the "…see more" fold. A bold claim, a surprising specific, or a real confession.
- **No formulaic rehook.** Do NOT bolt a templated second line onto the hook ("I used to think X. I don't anymore." / "Most people read it as Y. I read it as Z." / "And it's never what you'd expect."). The line after the hook is just the next honest sentence, not an engineered twist.
- Short lines, one idea per line, generous white space.
- Close soft: a quiet landing or one real question that invites a reply.

---

## Phase 3 — The value test

Every post must hand the reader something: a reframe, a usable idea, a story that teaches, a decision aid. A clever observation alone is not value — "what do I do with this?" is.

For carousels, see Phase 4.

---

## Phase 4 — Carousel authoring

- **Author slide-shaped from scratch. Never slice a finished post into slides.**
- Each slide hands the reader ONE takeaway: who wins/loses, a signal to watch, the order things happen, the white space, a role-by-role move, a checklist step.
- Shape: hook slide (one bold line) → 4–6 point slides (heading + ≤2 support lines) → CTA slide (one question/invite).
- Caption is short (~400 chars): one hook line + one line of context. Hashtags live in the caption (0–3).

---

## Phase 5 — Truth pass (MANDATORY)

Tag every claim as one of:
- **(a) documented** third-party fact,
- **(b) the client's own first-person claim** (true they said it; self-reported), or
- **(c) AI-dramatized** framing.

Rules:
- Never invent feelings, rankings, or scenes and attribute them to the client (e.g. "the invention I'm proudest of" with no source).
- Flag any (c) lines for client sign-off.
- Honor source constraints in `CONTENT-LEARNINGS.md` (e.g. unreleased books) and withhold sensitive negatives unless the client opts in.
- Get dates and history right.

---

## Phase 6 — Humanizer pass (MANDATORY)

Call the `humanizer` skill with:
- the draft,
- the client's most representative `samples/` file (voice calibration),
- the "Forbidden patterns" from `WRITING-GUIDELINES.md`,
- and an instruction to enforce the **LinkedIn ban list** (formulaic rehooks, zinger-stacking, keynote cadence — see humanizer patterns 30–31).

---

## Phase 7 — Mechanics

- Put any link (site, booking, AI twin) in the **first comment**, never the body. Links in the body suppress reach.
- Hashtags: 0–3, tasteful. LinkedIn deprioritizes them.
- Capitalization per the client's setting (default: normal sentence case, not lowercase).
- Confirm the hook reads strong in the first 1–3 lines before the fold.

---

## Phase 8 — Deliver

1. Present text-only for approval (one post or the batch). Do not generate graphics or media assets unless asked.
2. Give per-post **media direction**: a specific format + concept.
   - Native talking-head **video** for nuanced/sensitive takes and signature stories.
   - **Real, authentic photos** beat polished/stock graphics.
   - **Quote card** for one strong line; **stat card** for one number; **carousel** for a framework/list.
3. If the client has an Asana board and the user asks, write the approved copy into the relevant dated task.

---

## Forbidden patterns (quick reference)

- Formulaic rehooks (the templated "twist" second line)
- Em dashes; stacked "It's not X, it's Y"; decorative rule-of-three; tidy parallel-zinger or double-rhetorical-question closes
- Motivational-keynote / "starting today" energy when the client's voice is conversational
- Lowercase (unless the client explicitly wants it)
- Links in the post body; more than ~3 hashtags
- A carousel that is just the caption re-chopped into slides
- Inventing sentiment/feelings and attributing them to the client

---

## Success criteria

✅ Sounds like the client's own exemplars, not a generic LinkedIn voice
✅ Strong line-1 hook; no manufactured rehook
✅ Every post/slide gives the reader a real takeaway
✅ No repeated/used angle; passed the truth pass and the humanizer
✅ Mechanics correct (links in comment, ≤3 hashtags, right capitalization)
✅ Each post shipped with a specific media recommendation
