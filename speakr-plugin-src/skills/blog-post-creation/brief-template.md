# Blog Brief Template

> Fill in this template for every new blog post. The brief is the contract for everything downstream — drafting, editing, fact-checking, graphics. Be specific.
>
> For a fully-completed example, see `examples/kevin-surace-reference/blog-post-creation/examples/` at the plugin root.

---

1. ## Target Audience(s)

*Use this to guide your tone, examples, and level of detail. Pull from BRAND-CONTEXT.md → SEO Context → ICP if unsure.*

{e.g. CTOs, founders, mid-level managers, practitioners — be specific}

---

2. ## Readability & Formatting Requirements

*These requirements ensure content is accessible to non-native English speakers and highly scannable. Defaults shown — override only if the client's audience requires it.*

**Target Readability Metrics:**
- Flesch Reading Ease: 60-70 (Easy - 9th-10th grade)
- Flesch-Kincaid Grade Level: 8-10 (Middle school)
- Average Sentence Length: 15-20 words
- Complex Words: <15%
- Passive Voice: <10%

**Formatting Requirements:**
- [ ] Paragraph variety: Mix of 1, 2, 3 sentence paragraphs (NEVER 4+)
- [ ] Bullet sections: 8-12 minimum throughout article
- [ ] Visual placeholders: 2-4 minimum ([Screenshot], [Diagram], etc.)
- [ ] Tables: 1-2 for data comparisons
- [ ] Bold headers: 8-12 for hierarchy
- [ ] Sub-bullets: 3-5 instances of nested lists

**Non-Native Speaker Focus:** {YES | NO} — adjust based on client's audience

---

3. ## Sample Summary

*High-level idea of what this content should be about. Think of it as a mini elevator pitch for the article. One sentence is ideal.*

{e.g. "Why curiosity beats expertise for leaders navigating AI" — short, sharp, hooks the angle}

---

4. ## Suggested Content Structure

*Use this as a starting point to organize your content. Pull H2/H3 keywords from SEO Research. Total 5-8 H2 sections.*

1. ##### **{Section 1 Title — usually the hook / problem framing}**
   * {Hook angle}
   * {Client's credibility on this topic}
   * {Thesis statement}

2. ##### **{Section 2 Title — usually "why this matters" or evidence}**
   * {Supporting point}
   * {Example from client's experience or research}
   * {Pattern or principle}

3. ##### **{Section 3 Title — usually the core argument or framework}**
   * {Main framework / principle}
   * {Concrete illustration}
   * {Client's signature angle}

4. ##### **{Section 4 Title — usually deeper application or implications}**
   * {Application}
   * {Connection to client's content pillars from BRAND-CONTEXT.md}
   * {Counter-intuitive insight}

5. ##### **{Section 5 Title — usually action / conclusion}**
   * {Practical takeaway}
   * {Concrete next step for the reader}
   * {Closing statement that lands the thesis}

---

5. ## Primary Search Intent

*Understand what the searcher actually wants so you can match your content to their needs.*

**{Informational | Commercial | Transactional | Navigational}** — {one sentence on what the reader is looking for and how the post will satisfy it}

---

6. ## Keywords to Focus On

*Use these keywords in your content to help it rank in search engines. Pull from SEO Research output.*

| Content Topic Keyword (For your H1 Heading) | Relevance | Volume | Difficulty |
| :---- | ----: | ----: | ----: |
| {primary keyword} | 100% | {volume} | {0-100} |
| **Suggested Supporting Keywords (For your H2s and H3s)** | **Relevance** | **Volume** | **Difficulty** |
| {keyword 2} | {%} | {vol} | {0-100} |
| {keyword 3} | {%} | {vol} | {0-100} |
| {keyword 4} | {%} | {vol} | {0-100} |
| {keyword 5} | {%} | {vol} | {0-100} |

---

7. ## What's Already Ranking

*Examples of successful content on this topic. Use them to inspire structure, tone, or what to cover. Pull from SEO Research → SERP competitors.*

| Content Title / URL |
| :---- |
| {Competitor 1 title and URL} |
| {Competitor 2 title and URL} |
| {Competitor 3 title and URL} |

---

8. ## Existing Content (Cannibalization Check)

*Avoid writing a post that competes with the client's existing content.*

Check `{client-website}/blog` for existing posts on this topic. If something close exists, decide: refresh existing post, or angle this post differently.

---

9. ## Key Questions to Consider

*Additional useful or interesting questions that help your content stand out from competitors.*

* {Question 1 — something the SERP competitors are not answering well}
* {Question 2 — connects topic to client's unique angle}
* {Question 3 — practical / actionable}
* {Question 4 — counter-intuitive angle}
* {Question 5 — connects to client's other content pillars}

---

10. ## Interesting Facts to Consider

*Fun and interesting starter facts to use as inspiration for your research. Pull from content-research output. Each must have a source.*

* {Fact 1 — with source}
* {Fact 2 — with source}
* {Fact 3 — with source}
* {Fact 4 — with source}
* {Fact 5 — with source}

---

## Pre-flight checklist before passing to blog-post-creation

- [ ] All sections above are filled in (no `{placeholder}` text left)
- [ ] Brief matches active client in `{clientsDir}/.speakr/active-client`
- [ ] SEO research file is ready: `seo-research-{keyword}-{date}.md`
- [ ] Content research file is ready: `content-research-{keyword}-{date}.md`
