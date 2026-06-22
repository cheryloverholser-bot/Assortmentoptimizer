---
name: crisp-questionnaire-outreach
description: Draft a professional client email introducing the Crisp CMS Assessment Questionnaire, requesting completion and return by a target date. Use when starting a new assessment engagement and needing to send the Onboarding (Phase 1) outreach to the client contact.
---

# Crisp Questionnaire Outreach Email

Use this skill when you need to draft an email to a new client to send them the CMS Assessment Questionnaire (the Phase 1 — Onboarding deliverable in the Crisp Assessment Process).

## When to use

- Starting a new Category Management assessment engagement
- The SOW is signed and you need to kick off the Onboarding phase
- You have the primary client contact's name and role
- You need a polished, on-brand draft email that the engagement lead can review, personalize, and send

If the user is mid-engagement or asking about a different artifact, this isn't the right skill — point them to `crisp-prompt-builder` or the assessment HTML.

## Inputs — prompt the user

Ask one question at a time. Don't dump a form on them.

1. **Client company name** (e.g., "Acme CPG")
2. **Primary client contact** — full name and role/title (e.g., "Sarah Lin, VP Category Management")
3. **Crisp engagement lead** — full name and title (default: Director, Business Solutions)
4. **Target return date for the questionnaire** (e.g., "by Friday, May 23")
5. **Tentative kickoff or workshop date(s)** — optional, include if known
6. **Specific context** — optional. Any recent conversation, named systems in scope, sponsor introductions worth referencing
7. **Tone** — default warm-professional. Ask only if the user has signaled a preference

## What the email must do

The draft is the Phase 1 — Onboarding outreach. It frames the questionnaire as the first deliverable in a structured 4-phase assessment, ending in a prioritized solution roadmap. The phases (for reference, not all to include in the email):

- Phase 1 — Onboarding: client completes pre-workshop questionnaire; stakeholders & sample files identified; logistics confirmed → **Gate 1: Kickoff**
- Phase 2 — Discovery workshops & interviews
- Phase 3 — Analysis
- Phase 4 — Roadmap & readout

The email itself should:
- Open with a warm greeting using the contact's first name
- One-sentence reminder of why we're engaging (assume sponsor introduced us)
- Briefly frame the assessment: 4-phase, structured, ends with a prioritized roadmap they can act on
- The ask: **complete the attached CMS Questionnaire by [target date]**
- What they'll also need to provide: stakeholder list for Discovery interviews, sample data files
- What happens next: kickoff + Discovery workshops on the tentative dates
- Sign-off from the engagement lead with phone/email placeholders
- Note that `CMS_Questionnaire.docx` is attached

## Tone & style

- Follow Crisp voice: confident, plain-spoken, no consulting jargon ("synergies", "leverage", "best-in-class" — avoid)
- Specific over generic. If the user provided context in step 6, weave it in naturally
- Short paragraphs. Bullets only for "what we need" or "what's next"
- No emoji. No marketing fluff
- Reference the related Crisp skills if they apply: `crisp-cm-consulting` for voice, `crisp-brand` for any formatting if the user wants HTML

## Output format

Return the draft as a clean copy-paste-ready block:

~~~
Subject: <subject line>

<email body>

<sign-off>
~~~

Do not include framing commentary above or below the email. The user wants to paste it into Outlook/Gmail and edit, not a debrief.

## Example structure (don't copy verbatim — write fresh each time)

~~~
Subject: Kicking off the Acme Category Management Assessment — Questionnaire attached

Hi Sarah,

Following up on our intro from [sponsor name] — we're excited to start the
category management assessment for Acme. To kick off, the first step is for
your team to complete the attached questionnaire.

The assessment runs across four phases — Onboarding (where we are now), Discovery
workshops & interviews, Analysis, and a final Roadmap & readout with prioritized
recommendations for your leadership team.

To stay on schedule, we'd like the completed questionnaire back by Friday, May 23.
Alongside that, please send:
  • A short list of 3–8 stakeholders we should interview during Discovery
  • Sample data files we can review ahead of workshops (e.g., POS extracts,
    planogram exports, recent category reviews)

We're targeting Discovery workshops the week of June 3 — I'll send a draft
agenda once the questionnaire is back.

Reply here with any questions and I'll handle quickly.

Best,
Cheryl Overholser
Director, Business Solutions | Crisp
cheryl.overholser@gocrisp.com
~~~

## Anti-patterns

- Don't summarize the whole 4-phase process in the email — one sentence is enough; depth confuses the contact at this stage
- Don't ask the client to do more than: complete questionnaire, send stakeholder list, send sample files
- Don't include internal Crisp jargon (RACI, gate criteria, AI contribution steps) — that's for the consultant, not the client
- Don't fabricate a sponsor name or context — if the user didn't provide it in step 6, use a neutral phrasing ("Thanks for the time to kick off …")
- Don't promise specific dates the user didn't give you
