# Shelf Intelligence SDD Template Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce a reusable Shelf Intelligence SDD template (Markdown source + Crisp-branded .docx + usage README) per the design spec.

**Architecture:** Build the Markdown template incrementally section-by-section with frequent commits. Use Claude Code skills (`anthropic-skills:docx` + `crisp-marketing:crisp-brand`) for the .docx export workflow. Document the workflow in the README so it is repeatable.

**Tech Stack:** Markdown (template source), Claude Code skills for .docx generation, Crisp Brand Book 2025 (styling reference).

**Spec:** [`docs/superpowers/specs/2026-06-10-shelf-intelligence-sdd-design.md`](../specs/2026-06-10-shelf-intelligence-sdd-design.md)

---

## File Structure

| File | Purpose |
|------|---------|
| `Shelf Intelligence/Shelf Intelligence SDD Template.md` | Markdown source of truth |
| `Shelf Intelligence/Shelf Intelligence SDD Template.docx` | Crisp-branded export (regenerated from the .md, never hand-edited) |
| `Shelf Intelligence/Shelf Intelligence SDD README.md` | Usage guide for ProServices leads |

The Markdown template is built up section-by-section across Tasks 1–8 (matching spec §4's TOC). Task 9 self-reviews it. Task 10 adds the README. Task 11 produces the .docx. Task 12 validates with a fabricated scenario.

---

### Task 1: Create the template skeleton

Create the markdown template file with just the H1 title, a top-of-file usage note, and document identity fields. Establishes the file location for the incremental build that follows.

**Files:**
- Create: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Create the skeleton file**

Write the following content to `Shelf Intelligence/Shelf Intelligence SDD Template.md`:

```markdown
# Shelf Intelligence Solution Design Document

> **Template for ProServices leads.** Replace all `[bracketed placeholders]` with client-specific content. Duplicate any block marked "Repeat for each retail team in scope" as needed. See `Shelf Intelligence SDD README.md` for the full usage guide.

> **Client:** [Client Name]
> **Engagement:** [Engagement Name / SOW Reference]
> **Document version:** [v0.1 — Draft]
> **Last updated:** [YYYY-MM-DD]

---
```

- [ ] **Step 2: Verify file exists**

Run: `ls "Shelf Intelligence/Shelf Intelligence SDD Template.md"`
Expected: file path printed, no error.

- [ ] **Step 3: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SI SDD template skeleton"
```

---

### Task 2: Frontmatter section (§1–§3)

Build the three frontmatter tables (Reviewers, Approvers, Revision History) per spec §4 sections 1–3.

**Files:**
- Modify: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Append the frontmatter section**

Append the following to the end of the file:

```markdown
## 1. Document Reviewers / Contributors

| Name | Role | Reviewer Y/N | Contributor Y/N |
|------|------|--------------|-----------------|
| [Name] | [Role] | [Y/N] | [Y/N] |
| [Name] | [Role] | [Y/N] | [Y/N] |

## 2. Final Approvers

| Name | Version | Signature | Date |
|------|---------|-----------|------|
| [Approver Name] | [v1.0] | [Signature] | [YYYY-MM-DD] |

## 3. Document Revision History

| Version | Date | Author(s) | Revision Notes |
|---------|------|-----------|----------------|
| [v0.1] | [YYYY-MM-DD] | [Author] | [Initial draft] |

---
```

- [ ] **Step 2: Verify section structure**

Read the file and confirm three numbered H2 sections (`## 1.`, `## 2.`, `## 3.`) each followed by a table.

- [ ] **Step 3: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SDD frontmatter section"
```

---

### Task 3: Project Framing section (§4–§5)

Build spec §4 sections 4–5: Introduction, Purpose of the Document, Project Scope (as a pointer to the SOW), Not in Project Scope, Related Documents (referencing the canonical SI documents in this repo), Requirements Gathering Methodology.

**Files:**
- Modify: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Append the Project Framing content**

Append to the end of the file:

```markdown
## 4. Introduction

[1–2 paragraphs describing the engagement context. What dashboard modifications were requested, by whom, and at a high level why. This SDD captures the scope and design of those modifications. It is not the full Shelf Intelligence implementation plan and it does not replace the SOW.]

## 5. Purpose of the Document

[State the purpose: to document the agreed-upon dashboard modifications for sign-off by the client and to serve as the build reference for the implementation team.]

### 5.1 Project Scope

> The canonical scope is the SOW (see Related Documents below). The bullets here are an at-a-glance summary only.

- [Bullet — high-level summary of in-scope work]
- [Bullet]
- [Bullet]

### 5.2 Not in Project Scope

- [Bullet — explicit out-of-scope items, captured to prevent misalignment at sign-off]
- [Bullet]
- [Bullet]

### 5.3 Related Documents

| Document Type | Description |
|---------------|-------------|
| SOW | [Path or link to the signed SOW] |
| Discovery Questionnaire | `Shelf Intelligence/Shelf Intelligence Questionnaire.docx` (filled-in copy: [link]) |
| Notion Project Page | [Link] |
| JIRA Epic | [Link to master JIRA epic] |
| Training Paths | `Shelf Intelligence/Shelf Intelligence Training Paths.docx` |
| Facilitator Guide | `Shelf Intelligence/Shelf Intelligence Facilitator Guide.docx` |
| UAT Reference | `Shelf Intelligence/Shelf Intelligence UAT.docx` |
| Go-Live & Client Signoff | `Shelf Intelligence/Go Live Documentation & Client Signoff.docx` |
| Data Quality Verification | `Shelf Intelligence/Shelf Intelligence Data Quality Verification.docx` |
| TPA | `Shelf Intelligence/Shelf Intelligence TPA.docx` |

### 5.4 Requirements Gathering Methodology

[Short paragraph: how requirements were gathered for this engagement — Discovery questionnaire, discovery sessions per retail team, Gong recordings, stakeholder workshops, etc.]

---
```

- [ ] **Step 2: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SDD Project Framing section"
```

---

### Task 4: Client Overview section (§6)

Build spec §4 section 6: Client Overview, including the In-Scope Retail Teams index table that points readers to the per-team blocks below.

**Files:**
- Modify: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Append the Client Overview content**

Append to the end of the file:

```markdown
## 6. Client Overview

**Client name:** [Client Name]
**Business context:** [1 paragraph — client's industry, scale, why they purchased Shelf Intelligence, business outcomes they're trying to drive.]
**Sponsor:** [Executive sponsor name & title]
**Center of Excellence lead:** [Name & title]
**Master Notion page:** [Link]
**Master JIRA epic:** [Link]

### In-Scope Retail Teams

This table indexes the per-team blocks that follow. Each row corresponds to one Section 8 block below.

| Retail Team | Section | COE Rep | Category Leader | Kickoff Date |
|-------------|---------|---------|-----------------|--------------|
| [e.g., Walmart] | 8 | [Name] | [Name] | [YYYY-MM-DD] |
| [e.g., Ahold] | 9 | [Name] | [Name] | [YYYY-MM-DD] |

---
```

- [ ] **Step 2: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SDD Client Overview section"
```

---

### Task 5: Client-Level Considerations section (§7)

Build spec §4 section 7: cross-team considerations only (Constraints, Risks, System Dependencies, Success Criteria). Per-team variants of these live inside each Section 8 block.

**Files:**
- Modify: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Append the Client-Level Considerations content**

Append to the end of the file:

```markdown
## 7. Client-Level Considerations

Cross-team considerations that apply across all in-scope retail teams. Team-specific items belong in each team's block (Section 8 onward).

### 7.1 Constraints

- [Bullet — e.g., "Client cannot move to v2 platform until Q4 2026"]
- [Bullet]

### 7.2 Risks

| Risk | Impact | Likelihood | Mitigation | Owner |
|------|--------|-----------|------------|-------|
| [Risk description] | [High/Med/Low] | [High/Med/Low] | [Mitigation plan] | [Name] |
| [Risk description] | [High/Med/Low] | [High/Med/Low] | [Mitigation plan] | [Name] |

### 7.3 System Dependencies

- [Bullet — e.g., "Data warehouse cutover scheduled for [date]"]
- [Bullet]

### 7.4 Success Criteria

- [Bullet — quantifiable, e.g., "All in-scope retail teams report adoption >70% within 90 days of go-live"]
- [Bullet]

---
```

- [ ] **Step 2: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SDD Client-Level Considerations section"
```

---

### Task 6: Per Retail Team block with embedded per-mod sub-template (§8)

Build spec §4 section 8 (the heart of the template): the duplicatable per-retail-team block, including Discovery Findings, team-specific Considerations, the Modifications Summary table, and the detailed per-mod sub-template (all 9 fields from spec §5).

**Files:**
- Modify: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Append the per-retail-team block**

Append to the end of the file:

```markdown
## 8. [Retail Team Name]

> **Duplicate this entire Section 8 block for each retail team in scope** (renumbering as Section 9, 10, etc.). If only one team is in scope, retain a single instance and remove this note.

### 8.1 Team Overview

**Retailer scope:** [Which retailer(s) this team covers]
**Stakeholders:** [Names & roles — COE rep, Category leader, intended users]
**Kickoff date:** [YYYY-MM-DD]
**Brief context:** [1–2 sentences on the team's business context within the client organization]

### 8.2 Discovery Findings

#### Category management process & buyer-team workflow
[Describe how this team interacts with category buyers and the broader merchandising process. Source: discovery questionnaire + sessions.]

#### Personas & intended dashboard users

| Persona | Role | Primary Goal |
|---------|------|--------------|
| [Persona name] | [Role] | [What they want the dashboard to do for them] |

#### Key decision drivers
- [Driver — what each persona needs the dashboard to answer]
- [Driver]

#### Current tools
[Tools this team uses today that the SI dashboards augment or replace]

#### Standard offering vs. identified gaps
- **Standard SI dashboards cover:** [What out-of-the-box SI handles for this team]
- **Identified gaps:** [What's missing for this team — these gaps drive the modifications below]

#### Working style & recommendation horizon
[How far ahead this team wants recommendations; their preferred working cadence]

### 8.3 Constraints (team-specific)

- [Bullet]

### 8.4 Risks (team-specific)

| Risk | Impact | Likelihood | Mitigation | Owner |
|------|--------|-----------|------------|-------|
| [Risk] | [H/M/L] | [H/M/L] | [Mitigation] | [Owner] |

### 8.5 System Dependencies (team-specific)

- [Bullet — e.g., "Retailer X provides POS data weekly via [feed]"]

### 8.6 Success Criteria (team-specific)

- [Bullet — how this team measures success]

### 8.7 Dashboard Modifications

#### 8.7.1 Modifications Summary

At-a-glance index of all modifications for this team. Detailed write-ups follow in 8.7.2.

| Mod ID | Mod Name | Classification | JIRA | Status | Target Delivery |
|--------|----------|----------------|------|--------|-----------------|
| [TEAM]-01 | [Mod Name] | [Product Enhancement / Retailer Specific / Personalization] | [JIRA-1234] | [Not Started / In Progress / In QA / Delivered] | [YYYY-MM-DD] |
| [TEAM]-02 | [Mod Name] | [Classification] | [JIRA] | [Status] | [Date] |

> **Mod ID convention:** `[TEAM]-[NN]` where TEAM is the 2–4 letter team code (e.g., WMT for Walmart, AHD for Ahold) and NN is a zero-padded sequence number.

#### 8.7.2 Detailed Modifications

> **Duplicate the sub-section below for each modification listed in 8.7.1.**

##### [TEAM]-01 [Modification Name]

**Modification Summary:** [1-paragraph narrative description of what is changing and at what scope.]

**Classification:** [Product Enhancement / Retailer Specific / Personalization] — [brief rationale tied to the Notion Phase 3 framing: Product Enhancement goes into the standard product for everyone; Retailer Specific is custom for this retailer only; Personalization is handled in the personalization layer and is not a custom build.]

**Business Need:** [Why the client requested this. Explicit link back to a decision driver from §8.2.]

**Current State:** [What the standard dashboard does today for this scenario.]

**Future State / Solution Design:** [What the modified version will do.]

- Mockup: [Link or attachment reference]

**Data Dependencies:**
- [New fields, calculations, filters, joins, or source-system impacts]
- [Bullet]

**Acceptance Criteria:**
- [ ] [Testable criterion]
- [ ] [Testable criterion]

**References:**
- JIRA ticket: [JIRA-XXXX]
- Mockup link: [URL or path]
- Related discovery notes: [Section reference or link]
- Gong timestamp: [Recording + timestamp]

**Status & Delivery:**
- Owner: [Name]
- Target Date: [YYYY-MM-DD]
- Current Status: [Not Started / In Progress / In QA / Delivered]

---
```

- [ ] **Step 2: Verify all 8.x sub-sections are present**

Read the file and confirm the following sub-headings exist under Section 8: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.7.1, 8.7.2. Confirm the per-mod sub-template contains all 9 fields from spec §5 (Modification Summary, Classification, Business Need, Current State, Future State / Solution Design, Data Dependencies, Acceptance Criteria, References, Status & Delivery).

- [ ] **Step 3: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SDD per-retail-team block with per-mod sub-template"
```

---

### Task 7: Closing Chapters as reference pointers (§9–§12)

Build spec §4 sections 9–12. Per the revised spec, each chapter is a short pointer to canonical SI documents with a slot for client-specific supplementary notes.

**Files:**
- Modify: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Append the closing chapters**

Append to the end of the file:

```markdown
## 9. Data & Integration Considerations

> Each closing chapter is a short reference. Standards live in the canonical Shelf Intelligence documents linked below. Capture only client-specific notes in this section.

**Canonical references:**
- `Shelf Intelligence/Shelf Intelligence Data Quality Verification.docx` — standard data quality framework
- `Shelf Intelligence/Shelf Intelligence TPA.docx` — Third-Party Agreement scope and signed positions

**Client-specific notes:**
- [Any data quality or TPA notes that supplement the standard for this client. Leave empty if none.]

---

## 10. UAT & Acceptance Criteria

**Canonical reference:**
- `Shelf Intelligence/Shelf Intelligence UAT.docx` — standard UAT framework, test scripts, sign-off process

**Client-specific notes:**
- [UAT environment specifics, scope adjustments, additional acceptance criteria beyond the standard. Leave empty if none.]

---

## 11. Training & Enablement Tie-In

**Canonical references:**
- `Shelf Intelligence/Shelf Intelligence Training Paths.docx` — standard per-persona training paths
- `Shelf Intelligence/Shelf Intelligence Facilitator Guide.docx` — facilitator-led training approach

**Client-specific notes:**
- [Training adaptations, scheduling, office-hours cadence. Leave empty if none.]

---

## 12. Rollout & Go-Live

**Canonical reference:**
- `Shelf Intelligence/Go Live Documentation & Client Signoff.docx` — standard go-live documentation and client sign-off process

**Client-specific notes:**
- [Rollout sequencing, readiness checkpoints, post go-live cadence. Leave empty if none.]

---
```

- [ ] **Step 2: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SDD closing chapters as reference pointers"
```

---

### Task 8: Appendices section (§13)

Build spec §4 section 13: Glossary, Mockup Gallery, JIRA Ticket Index, Data Dictionary (placeholder), Open Questions / Parking Lot.

**Files:**
- Modify: `Shelf Intelligence/Shelf Intelligence SDD Template.md`

- [ ] **Step 1: Append the appendices**

Append to the end of the file:

```markdown
## 13. Appendices

### 13.1 Glossary

| Term | Definition |
|------|------------|
| Center of Excellence (COE) | [Definition] |
| Product Enhancement | Modification that goes into the standard SI product for all clients |
| Retailer Specific | Custom dashboard or modification for one retailer only |
| Personalization | Handled in the SI personalization layer; not a custom build |
| [Add additional terms as needed] | |

### 13.2 Mockup Gallery / Screenshots

[Embed or link to each mockup referenced in Section 8 modification blocks. Organize by Mod ID.]

| Mod ID | Mockup |
|--------|--------|
| [TEAM]-01 | [Embedded image or link] |

### 13.3 JIRA Ticket Index

**Live JIRA filter:** [Link to JIRA filter showing all tickets for this engagement]

**Snapshot table:**

| Mod ID | JIRA | Title | Status | Assignee |
|--------|------|-------|--------|----------|
| [TEAM]-01 | [JIRA-XXXX] | [Title] | [Status] | [Name] |

### 13.4 Data Dictionary

> Placeholder. Populate with fields, calculations, and data definitions referenced in any of the modifications above. Often requested by clients at sign-off.

| Field Name | Source | Definition | Used in Mod(s) |
|------------|--------|------------|----------------|
| [Field] | [Source system / table] | [Plain-English definition] | [TEAM]-01 |

### 13.5 Open Questions / Parking Lot

| Question | Raised By | Date | Status | Resolution |
|----------|-----------|------|--------|------------|
| [Question] | [Name] | [YYYY-MM-DD] | [Open/Closed] | [Resolution] |
```

- [ ] **Step 2: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: add SDD Appendices section"
```

---

### Task 9: Self-review the complete markdown template

Read the full markdown template end-to-end and verify it matches every section the spec requires. Fix any gaps inline before continuing to the .docx export.

**Files:**
- Read: `Shelf Intelligence/Shelf Intelligence SDD Template.md`
- Read: `docs/superpowers/specs/2026-06-10-shelf-intelligence-sdd-design.md`

- [ ] **Step 1: Read both files in full.**

- [ ] **Step 2: Check spec coverage with this checklist:**

| Spec § | Template § | Confirmed? |
|--------|-----------|------------|
| 1 Doc Reviewers / Contributors | §1 | [ ] |
| 2 Final Approvers | §2 | [ ] |
| 3 Doc Revision History | §3 | [ ] |
| 4 Introduction | §4 | [ ] |
| 5 Purpose of Doc | §5 | [ ] |
| 5.1 Project Scope (pointer to SOW) | §5.1 | [ ] |
| 5.2 Not in Project Scope | §5.2 | [ ] |
| 5.3 Related Documents | §5.3 | [ ] |
| 5.4 Requirements Gathering Methodology | §5.4 | [ ] |
| 6 Client Overview + In-Scope Retail Teams table | §6 | [ ] |
| 7.1–7.4 Client-Level Considerations (all 4) | §7 | [ ] |
| 8.1–8.6 Per-team Discovery + Considerations | §8 | [ ] |
| 8.7.1 Modifications Summary table | §8.7.1 | [ ] |
| 8.7.2 Detailed mod sub-template (all 9 fields per spec §5) | §8.7.2 | [ ] |
| 9 Data & Integration → DQV + TPA refs | §9 | [ ] |
| 10 UAT → UAT.docx ref | §10 | [ ] |
| 11 Training → Training Paths + Facilitator Guide refs | §11 | [ ] |
| 12 Rollout → Go Live & Client Signoff ref | §12 | [ ] |
| 13.1–13.5 Appendices (all 5) | §13 | [ ] |

- [ ] **Step 3: Fix any gaps found and commit**

If a row above is unchecked, add the missing content directly to the template, then commit:

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md"
git commit -m "docs: fix SDD template gaps found in self-review"
```

Skip the commit if no gaps were found.

---

### Task 10: Write the usage README

Author the usage guide for ProServices leads. Covers when to use the template, how to fill it in, MOD-ID convention, and how to regenerate the .docx.

**Files:**
- Create: `Shelf Intelligence/Shelf Intelligence SDD README.md`

- [ ] **Step 1: Create the README**

Write the following to `Shelf Intelligence/Shelf Intelligence SDD README.md`:

````markdown
# Shelf Intelligence SDD — Usage Guide

This folder contains the **Shelf Intelligence Solution Design Document template** for ProServices use when a client requests modifications to the standard SI dashboards.

## When to use this template

Use this template when:
- A client engagement includes dashboard modifications for one or more retail account teams
- You need a client-facing document for sign-off on the modifications scope
- Phase 2 discovery has identified modifications that fall outside the standard SI offering

Do **not** use this template for:
- Full SI implementations without dashboard modifications (no SDD required)
- Project plans, timelines, or resource assignments (use the project plan templates)
- Business Requirements Documents (BRD has a separate template)

## Files in this template

| File | Purpose |
|------|---------|
| `Shelf Intelligence SDD Template.md` | **Source of truth.** Markdown template — edit this. |
| `Shelf Intelligence SDD Template.docx` | Crisp-branded export, regenerated from the .md. **Do not hand-edit this file.** |
| `Shelf Intelligence SDD README.md` | This guide. |

## How to fill in the template

1. **Copy the template** into your engagement's working folder. Rename it `[Client Name] SDD v0.1.md`.
2. **Fill in the frontmatter** (Sections 1–3): reviewers, approvers, revision history.
3. **Complete Project Framing** (Sections 4–5): introduction, purpose, scope. The Project Scope section is a short pointer to the SOW — do not restate the full SOW.
4. **Complete Client Overview** (Section 6) and the **In-Scope Retail Teams** table. The table indexes the per-team blocks you will create next.
5. **Fill in Client-Level Considerations** (Section 7) — only items that apply across all teams.
6. **For each retail team in scope, duplicate Section 8** (renumber to 9, 10, etc. for subsequent teams). Fill in:
   - Team Overview, Discovery Findings, team-specific Constraints/Risks/Dependencies/Success Criteria
   - Modifications Summary table
   - One detailed sub-section per modification (duplicate the `[TEAM]-01 [Modification Name]` block as needed)
7. **Closing Chapters (Sections 9–12)** — each is a short reference to the canonical SI documents. Add only client-specific supplementary notes.
8. **Appendices (Section 13)** — populate as needed; the Data Dictionary in particular is often requested at sign-off.

## Modification ID convention

`[TEAM]-[NN]` where:
- `TEAM` = 2–4 letter team code (e.g., `WMT` for Walmart, `AHD` for Ahold)
- `NN` = zero-padded sequence number per team

Examples: `WMT-01`, `WMT-02`, `AHD-01`.

Use these IDs consistently in JIRA, Slack, and the appendices so cross-references stay clean.

## How to regenerate the .docx

The Crisp-branded .docx is generated from the markdown source. Two ways to regenerate:

### Option A — via Claude Code (recommended)

From this repository's root, open Claude Code and prompt:

> Convert `Shelf Intelligence/Shelf Intelligence SDD Template.md` to a Crisp-branded `.docx` at `Shelf Intelligence/Shelf Intelligence SDD Template.docx`. Use the `anthropic-skills:docx` skill to generate the document and the `crisp-marketing:crisp-brand` skill to apply Crisp brand styling (Montserrat fonts, Crisp colors per the Crisp Brand Book 2025).

Claude will invoke both skills, produce the .docx, and report back. Eyeball the output and commit if it looks correct.

### Option B — via pandoc (alternative)

If pandoc is installed and a Crisp brand reference template (`crisp-brand-reference.docx`) is available:

```bash
pandoc "Shelf Intelligence/Shelf Intelligence SDD Template.md" \
  --reference-doc=crisp-brand-reference.docx \
  -o "Shelf Intelligence/Shelf Intelligence SDD Template.docx"
```

No reference doc exists in this repo today; Option A is the supported path.

## Updating the template itself

The template is versioned in git. To propose changes:
1. Edit `Shelf Intelligence SDD Template.md`
2. Regenerate the `.docx` (Option A above)
3. Commit both files together with a clear message
4. Update the spec at `docs/superpowers/specs/2026-06-10-shelf-intelligence-sdd-design.md` if the structural changes are non-trivial
````

- [ ] **Step 2: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD README.md"
git commit -m "docs: add SI SDD template usage README"
```

---

### Task 11: Generate the Crisp-branded .docx

Produce the first `.docx` export. Doing this once during implementation proves the workflow that the README describes as Option A.

**Files:**
- Create: `Shelf Intelligence/Shelf Intelligence SDD Template.docx`

- [ ] **Step 1: Generate the .docx via Claude Code skills**

Invoke the `anthropic-skills:docx` skill to convert the markdown source into a Word document at the target path:

> Read `Shelf Intelligence/Shelf Intelligence SDD Template.md` and produce a Word document at `Shelf Intelligence/Shelf Intelligence SDD Template.docx`, preserving all headings, tables, blockquotes, and bracketed placeholders.

Once the .docx exists, invoke the `crisp-marketing:crisp-brand` skill to apply Crisp brand styling:

> Apply Crisp brand styling to `Shelf Intelligence/Shelf Intelligence SDD Template.docx`: Montserrat fonts, Crisp brand colors and accents per the Crisp Brand Book 2025, on-brand table styles, and page setup.

- [ ] **Step 2: Visual validation**

Open the .docx in Word and confirm:
- [ ] H1/H2/H3 heading hierarchy renders correctly
- [ ] All tables are styled and readable
- [ ] Fonts are Montserrat (or the Crisp brand body font)
- [ ] Crisp brand colors appear in headings / accents
- [ ] All `[bracketed placeholders]` are visible and clearly distinguishable as fill-in fields
- [ ] No broken markdown syntax leaked into the output (e.g., raw `|` table separators, raw `**` bold markers, raw `>` blockquote markers)

- [ ] **Step 3: Fix any rendering issues**

If headings are flat, styling is missed, or markdown syntax leaked through:
- Re-invoke the brand skill, or
- Adjust the markdown source (e.g., promote/demote headings, escape stray characters)
- Iterate until visual validation passes.

- [ ] **Step 4: Commit**

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.docx"
git commit -m "docs: generate initial Crisp-branded SDD .docx export"
```

---

### Task 12: Validate with a representative scenario

Verify the template captures everything it needs to by walking through a fabricated scenario. The goal is to surface gaps before a real ProServices lead encounters them mid-engagement.

**Files:**
- Read: `Shelf Intelligence/Shelf Intelligence SDD Template.md`
- (Mental walkthrough only — no commit of a scenario file)

- [ ] **Step 1: Define the scenario**

Scenario: **"ACME Foods" engagement with two retail teams — Walmart (3 mods) and Ahold (1 mod).**

Walmart mods:
- `WMT-01` Add a velocity-decile column to the Top SKUs dashboard — Retailer Specific
- `WMT-02` Replace default 13-week trend chart with a 52-week trend — Personalization
- `WMT-03` Add a planogram-compliance scorecard panel — Product Enhancement candidate

Ahold mods:
- `AHD-01` Add a private-label vs. national-brand share view — Retailer Specific

- [ ] **Step 2: Walk through the template**

Open `Shelf Intelligence SDD Template.md` and, for each section, mentally fill in what you would write for the ACME Foods scenario. For each field flag whether:
- You don't know what to put (template is too vague)
- You would want a field that isn't there (template is missing something)
- A field doesn't apply and there's no clean way to omit it (template is over-prescriptive)

- [ ] **Step 3: Record findings and decide on fixes**

For each gap surfaced in Step 2, decide:
- **Fix in template** → edit `Shelf Intelligence SDD Template.md`, regenerate the .docx (re-run Task 11 Steps 1–4), commit
- **Acceptable variation** → no template change; consider documenting under a "Common adaptations" subsection in the README

If template changes were made, commit:

```bash
git add "Shelf Intelligence/Shelf Intelligence SDD Template.md" "Shelf Intelligence/Shelf Intelligence SDD Template.docx"
git commit -m "docs: address SDD template gaps from scenario validation"
```

- [ ] **Step 4: Update the spec if structural changes were made**

If Step 3 produced structural changes (new section, removed section, materially different field schema), update the spec so it stays accurate:

```bash
# Edit docs/superpowers/specs/2026-06-10-shelf-intelligence-sdd-design.md to reflect the changes
git add docs/superpowers/specs/2026-06-10-shelf-intelligence-sdd-design.md
git commit -m "docs: update SI SDD spec to reflect template validation findings"
```

Skip if no structural changes were made.

- [ ] **Step 5: Final summary report to user**

Report:
- Files produced (paths to .md, .docx, README)
- Validation result (Task 12 walkthrough outcome — clean or what was fixed)
- Any deferred follow-ups (e.g., "no Crisp brand reference template exists; pandoc Option B in the README is aspirational until one is built")
