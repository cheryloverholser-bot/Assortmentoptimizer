---
name: crisp-discovery-outreach
description: Draft stakeholder interview invitations and a workshop logistics confirmation email for the Phase 2 (Discovery) kickoff, based on the client's reply that listed stakeholder names and proposed discovery dates/times. Use after the client has returned the CMS Questionnaire and identified their interview participants.
---

# Crisp Discovery Outreach & Logistics Email Drafter

Use this skill when the client has replied to the Phase 1 (Onboarding) questionnaire outreach with their stakeholder list and proposed/confirmed discovery workshop dates. You'll produce a logistics confirmation email back to the primary client contact AND individual interview invites for each named stakeholder.

## When to use

- The client has already returned the CMS Questionnaire
- Their reply includes named stakeholders (typically 3–8 people) for Discovery interviews
- Their reply includes proposed or confirmed dates/times for the Discovery workshops
- You need draft emails to confirm logistics and schedule interviews — ready for the engagement lead to review, personalize, and send

If the questionnaire hasn't been returned yet, this isn't the right skill — point the user to `crisp-questionnaire-outreach`.

## Inputs — prompt the user

Ask one item at a time. Don't dump a form on them.

1. **Paste the client's reply email** (full text) — the skill will parse stakeholder names and workshop dates from it
2. **Crisp engagement lead** — full name and title (default: Director, Business Solutions)
3. **Client company name** (e.g., "Acme CPG") — confirm even if it can be inferred from the reply
4. **Primary client contact** — name and email (the person you're replying to)
5. **Workshop format** — On-site / Remote (Zoom) / Hybrid
6. **Workshop location** — only if on-site; ask for office address
7. **Interview format** — typically 45–60 min, remote (Zoom) by default; confirm
8. **Calendar handling** — will the engagement lead send calendar invites separately after sending these emails? (default: yes — the emails should reference "calendar invite to follow")
9. **Any additional context** — sensitive topics to flag, pre-reads to attach, hierarchy notes (e.g., interview the VP last)

## Parsing the reply email

From the pasted reply, extract:
- **Stakeholder names** — typically in a bulleted list, table, or signature block. Look for "interviewees", "stakeholders", "participants", role titles
- **Stakeholder roles/titles** — if mentioned; if missing, ask the user to confirm
- **Workshop date(s)** — typically a single date range ("June 3–5") or a few specific days
- **Workshop times** — start/end times if mentioned; otherwise use a sensible default (9:00am–4:00pm local)
- **Any constraints** — black-out days, time zones, language preferences

If the reply is missing critical info (no dates, no stakeholders), stop and ask the user to clarify before drafting.

## What the skill produces

Two outputs in this order:

### 1. Workshop Logistics Confirmation Email
Reply to the primary client contact. Confirms the workshop dates/times and outlines what happens next.

Contents:
- Acknowledgement of receipt (questionnaire + stakeholder list)
- Confirmed workshop dates, times, format (on-site / remote / hybrid), and location if on-site
- Brief framing of what the workshops cover (current-state walk-through, pain points)
- Statement that individual interview invites are going out to each stakeholder (mention names if 3 or fewer; otherwise reference the list)
- Pre-work requested from each stakeholder (15 min review of role + current process if applicable)
- Confirmation that calendar invites will follow
- Sign-off from engagement lead

### 2. Individual Stakeholder Interview Invitations
One email per stakeholder named in the reply. Each is a standalone email the engagement lead can send.

Contents (per stakeholder):
- Subject line referencing their name and the engagement
- Warm greeting using first name
- One-sentence context: Crisp is running a CM assessment for [Client]; their leadership asked us to talk with [name] about [role/topic]
- The ask: a 45–60 min Discovery interview during [workshop dates]
- 2–3 sample topics the interview will cover, tailored to their role if known
- Confirmation that a Zoom (or on-site time slot) calendar invite is coming separately
- Offer to reschedule if conflict
- Sign-off from engagement lead

## Tone & style

- Follow Crisp voice: confident, plain-spoken, no consulting jargon ("synergies", "leverage", "deep dive" — avoid)
- Reference `crisp-cm-consulting` skill for voice if available
- Short paragraphs. Bullets only for "what we'll cover" or "what we need"
- Each stakeholder email feels personally addressed — not a copy-paste mail-merge with names swapped
- Acknowledge their time. Don't pad.
- No emoji. No marketing fluff.

## Output format

Return all emails in a single response, clearly separated:

~~~
========================================
1. LOGISTICS CONFIRMATION — TO: [primary contact name]
========================================

Subject: <subject>

<body>

<sign-off>

========================================
2. INTERVIEW INVITE — TO: [stakeholder 1 name]
========================================

Subject: <subject>

<body>

<sign-off>

========================================
3. INTERVIEW INVITE — TO: [stakeholder 2 name]
========================================

...
~~~

No framing commentary above or below. The engagement lead wants to copy each block into Outlook/Gmail and edit, not read a debrief.

## Example structures (don't copy verbatim — write fresh each time)

### Logistics confirmation

~~~
Subject: Acme Assessment — Discovery workshops confirmed (June 3–5)

Hi Sarah,

Thanks for the completed questionnaire and the stakeholder list. We're set to
run the Discovery workshops on Tuesday June 3 through Thursday June 5, 9:00am–
4:00pm Central, on-site at your Bentonville office.

Here's what to expect:
  • Day 1 — current-state walk-through with your team
  • Day 2 — targeted interviews with the stakeholders you named
  • Day 3 — pain point synthesis and gap discussion

I'm sending individual interview invites to each of the six stakeholders this
afternoon — calendar invites will follow within 24 hours.

If anything in the schedule needs adjusting, let me know by Friday and I'll
rework.

Best,
Cheryl Overholser
Director, Business Solutions | Crisp
cheryl.overholser@gocrisp.com
~~~

### Individual interview invite

~~~
Subject: Acme CM Assessment — Discovery interview with you (~45 min)

Hi Marcus,

Sarah Lin shared your name as one of the stakeholders we'll be talking with
during the upcoming category management assessment Crisp is running for Acme.

I'd like to schedule a 45-minute Discovery interview during the workshop week
(June 3–5). The conversation focuses on:
  • How your team uses current category data and tools today
  • Where current process slows you down or breaks
  • What an improved workflow would unlock for you

I'll send a Zoom calendar invite separately with a time slot — if it doesn't
work, just let me know what does and I'll move it.

Best,
Cheryl Overholser
Director, Business Solutions | Crisp
cheryl.overholser@gocrisp.com
~~~

## Anti-patterns

- Don't send identical interview emails — tailor 2–3 topics to each stakeholder's role if it can be inferred from their title
- Don't include the full 4-phase process in every email — one sentence of context is enough at this stage
- Don't ask stakeholders to do pre-work in the invite (other than show up) — that comes in the calendar invite
- Don't include internal Crisp jargon (RACI, gate criteria, AI Steps) — that's for the consultant
- Don't fabricate stakeholder roles if the reply didn't include titles — leave the topic list generic or ask the user
- Don't propose dates the client's reply didn't mention — if dates are missing, stop and ask
- Don't promise specific interview times in the invite email — the calendar invite handles scheduling
