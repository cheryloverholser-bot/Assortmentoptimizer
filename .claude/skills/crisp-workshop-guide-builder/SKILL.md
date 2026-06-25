---
name: crisp-workshop-guide-builder
description: Generate a tailored Phase 2 Discovery Workshop Guide for a CM Assessment engagement. Pulls questions from the SP Discovery and FP Discovery banks, prioritizes gap areas identified in the returned CMS Questionnaire, and outputs a markdown outline by default with an optional populated PPTX. Use after the questionnaire has been returned and before the discovery workshops.
---

# Crisp Workshop Guide Builder

Use when the client has returned the CMS Questionnaire and you need to prepare the Discovery Workshop materials. The skill takes the completed questionnaire (which surfaces the client's current-state strengths and gaps), pulls questions from the standard SP and FP Discovery question banks, and assembles a tailored workshop guide.

## When to use

- The client has returned the CMS Questionnaire (Phase 1 deliverable)
- You're preparing for Phase 2 — Discovery workshops
- You need a Workshop Guide tailored to this engagement (not the generic template)

If the questionnaire hasn't been returned, point the user to `crisp-questionnaire-outreach`.
If you need the discovery interview invites/logistics email, point the user to `crisp-discovery-outreach`.

## Inputs — prompt the user

Ask one item at a time. Don't dump a form on them.

1. **Returned CMS Questionnaire** — REQUIRED. Ask the user to paste the completed questionnaire text OR provide a file path (.docx is fine — read it). The skill cannot proceed without this.
2. **Client company name** (e.g., "Acme CPG")
3. **Engagement scope** — Space Planning only / Floor Planning only / Both
4. **Workshop date(s)** — to anchor the agenda
5. **Workshop format** — On-site / Remote (Zoom) / Hybrid
6. **Number of workshop days** — typically 1–3
7. **Stakeholder list with roles** — to map questions to personas. If the user already provided this for `crisp-discovery-outreach`, they can paste the same list
8. **Output format** — Markdown only (default) OR Markdown + PPTX file

## Question banks (built-in)

Reads from `Assessments Templates/CX.ASSESSMENT.ONSITE.QUESTIONS.xlsx`:
- **SP Discovery Questions** — Space Planning, organized by Planogram Process Phase → Topic → Question
- **FP Discovery Questions** — Floor Planning, organized by Floorplan Process Phase → Topic → Primary Exploratory Question + Secondary Questions

If scope is "SP only" → use SP bank. "FP only" → use FP bank. "Both" → use both, separated by domain.

## How to select & prioritize questions (hybrid approach)

1. **Include the full standard set** for the in-scope domain(s). Coverage still matters even if the client scored high.
2. **Mark gap-area questions [HIGH PRIORITY]** — read the questionnaire results:
   - Identify dimensions where the client scored low or flagged a gap
   - Map those dimensions to phases/topics in the question bank (e.g., Data Quality → Identify Phase → Categories/Schedules/Data Needs)
   - Tag those questions `[HIGH PRIORITY]` in the output
3. **Inject custom follow-ups** — if the questionnaire surfaced something specific (named systems, regional differences, named challenges), add a `[CUSTOM]` follow-up question for that topic
4. **Map questions to stakeholders** — if you have the stakeholder list with roles, tag each question with `(Persona: Role)`

## Workshop structure (matches existing Workshop Guide template)

Organize output by Planogram Process Phase if SP is in scope:

1. **Cover & Setup** — Client name, dates, attendees, format
2. **Questionnaire Summary** — high-scoring areas, gap areas, client-flagged topics
3. **The Planogram Team** — Team Structure, Roles, Vision, Gaps, Regional variations
4. **Identify Phase** — Categories, Schedules, Data Needs
5. **Prepare Phase** — Foundation & Performance, Data Collation
6. **Build Phase** — Construction, Assortment, Fixtures
7. **Review Phase** — Quality Control & Exception Reporting
8. **Publish Phase** — Ordering, Execution, Compliance
9. **Analyze Phase** — Performance, Reporting
10. **Data Integrations** — Inbound/Outbound, Security, Archiving
11. **Reporting & Analytics** — Stakeholder requirements, Future State Vision
12. **Future State Goals** — End-state for all teams

If FP is in scope, add a parallel **Floor Planning** section after the SP phases (or replace SP with FP if FP-only).

For each section produce:
- Section title
- Standard questions from the bank
- `[HIGH PRIORITY]` tags on gap-area questions
- `(Persona: Role)` annotations where stakeholder info is available
- `[CUSTOM — from questionnaire]` block when applicable

## Output format

### Default — Markdown outline

Single markdown document. Top-matter block, then phase-by-phase sections. Example top-matter:

~~~
# Workshop Guide — Acme CPG
**Engagement:** Category Management Assessment
**Workshop dates:** June 3–5, 2026
**Format:** On-site, Bentonville office
**Attendees:** [list with roles]
**Crisp lead:** Cheryl Overholser, Director, Business Solutions

---

## Questionnaire Summary
**High-scoring areas:** Strategy, Collaboration
**Gap areas (high priority for workshops):**
  - Data Quality (2/5) — "POS data inconsistencies across banners"
  - Technology stack — "Legacy Excel processes, no central source of truth"
**Client-flagged topics to explore:**
  - Regional differences across US/CA banners (Sarah's comment, Q7)
  - New ProSpace rollout starting Q3

---

## Identify Phase — Categories, Schedules, Data Needs
- [HIGH PRIORITY] What are the assortment change triggers? (line reviews, innovation, seasons) (Persona: Category Manager)
- [HIGH PRIORITY] [CUSTOM — from questionnaire] How do you currently reconcile POS data across US and CA banners?
- What is the reset type frequency? Major vs Minor reset definition? (Persona: Space Planner)
...
~~~

### Optional — PPTX file (only if user said yes in step 8)

Use the `anthropic-skills:pptx` skill to populate the existing `Assessments Templates/Assessment Workshop Guide.pptx` template:
- Replace placeholder text with engagement-specific content
- Slide 1 title includes client name
- Insert high-priority questions into the appropriate phase slides
- Preserve brand styling (use `anthropic-skills:crisp-brand` if available)

Always produce the markdown first. The user reviews it before you spend tokens generating the PPTX.

## Tone & style

- Operational, not consultative — this is a working document for the workshop facilitator, not a client deliverable
- Use the question bank wording verbatim where possible (well-tested phrasing)
- Inline `[bracketed]` notes for facilitator guidance
- Reference `crisp-cm-consulting` for any CM-specific tone considerations

## Anti-patterns

- Don't fabricate questions not in the bank — pull verbatim or tag `[CUSTOM — from questionnaire]`
- Don't drop standard questions because the client scored high — coverage still matters
- Don't generate the PPTX unless the user explicitly asked for it in step 8
- Don't ask the user to upload the question banks — they live in the assessment templates folder, the skill reads them directly
- Don't fabricate stakeholder personas — leave as `(Persona: TBD)` if not provided
- Don't proceed without the questionnaire — without it, the skill can't prioritize gap areas; stop and ask
