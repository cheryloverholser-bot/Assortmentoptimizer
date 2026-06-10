# Shelf Intelligence Solution Design Document — Template Design Spec

**Date:** 2026-06-10
**Author:** Cheryl Overholser (ProServices)
**Status:** Approved for implementation planning

## 1. Purpose

Design a **reusable Solution Design Document (SDD) template** for Crisp ProServices to use whenever a Shelf Intelligence client requests modifications to the standardized dashboards.

The template will be filled in by the ProServices lead at the start of a dashboard-modification engagement (Phase 2 Discovery → Phase 3 Build & Validation in the Shelf Intelligence Implementation playbook) and used as the canonical, client-facing artifact for sign-off on the modifications scope.

## 2. Context

### 2.1 Why this template is needed

Shelf Intelligence comes with a set of standardized dashboards for retail account teams. The Implementation & Training playbook (Notion) lists "Solution Design Document (if required)" as a Phase 2 deliverable, but no standard template exists today. ProServices leads currently improvise per engagement, which slows discovery and creates inconsistent client-facing artifacts.

### 2.2 What this template is for

- Engagements where one or more retail account teams within a client request modifications to the standard SI dashboards
- Captures discovery findings, per-team modifications, and sign-off criteria in a single client-facing document

### 2.3 What this template is NOT for

- Full Shelf Intelligence implementation SDD (broader scope — out of scope here)
- Project plan, timeline, or resource assignment (separate artifacts)
- Business Requirements Document (BRD) — separate artifact
- Replacement for the SOW (the SOW remains the canonical scope document; this SDD references it)
- Internal Crisp engineering specs (custom dashboards still need product/engineering specs in addition to this SDD)

## 3. Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Deliverable type | Reusable template | ProServices needs a standard artifact; per-client copies generated from it |
| Scope of template | Dashboard modifications only | Narrower scope = faster to fill, easier to maintain. Aligns with the discrete "modifications" request pattern |
| Output formats | Markdown source + Crisp-branded .docx export | Markdown is version-controllable and AI-editable; .docx is client-deliverable |
| Per-mod structure | Section per mod (PGSM Feature pattern adapted) | Depth needed for client sign-off; matches established Cantactix/Crisp documentation style |
| Discovery findings | Captured inline (not just referenced) | Single source of truth for client sign-off; reader can pick up the SDD cold |
| Body organization | Per-retail-team blocks | Matches how the work actually happens (Phase 2 & 3 are explicitly "Per Retail Team" in the Notion); supports multi-team clients (Walmart team + Ahold team + etc.) |
| Considerations scope | Both client-level AND per-team | Some risks/dependencies are cross-team (platform-wide); others are team-specific |
| Project Scope section | Short pointer to SOW | Avoids restating canonical scope; SOW is the source of truth |
| Assumptions section | Removed | Replaced by per-team structure; project-wide assumptions belong in SOW |
| Modification ID convention | `[TEAM]-[NN]` (e.g., `WMT-01`, `AHD-03`) | Referenceable in JIRA, Slack, appendices |
| Per-team summary table | Yes, inside each team block | Small enough scope to benefit from a skim table; supports sign-off |
| Cross-team mod master table | No | Each team's modifications are independent; a cross-team master would just duplicate the per-team tables |
| Rollout & Go-Live section | Yes, lightweight | Mostly references the SI playbook; included for completeness at sign-off |
| Data Dictionary appendix | Yes, placeholder | Clients often request at sign-off; included as optional placeholder |

## 4. Document Structure (Full Table of Contents)

### Frontmatter
1. Document Reviewers / Contributors *(table: Name · Role · Reviewer Y/N · Contributor Y/N)*
2. Final Approvers *(table: Name · Version · Signature · Date)*
3. Document Revision History *(table: Version · Date · Author(s) · Revision Notes)*

### Project Framing
4. Introduction
5. Purpose of the Document
   - 5.1 Project Scope *(short pointer to SOW)*
   - 5.2 Not in Project Scope
   - 5.3 Related Documents *(table: Document Type · Description)*
   - 5.4 Requirements Gathering Methodology

### Client Overview
6. Client Overview
   - Client name & business context
   - Sponsor & COE lead
   - Master Notion page link
   - Master JIRA epic
   - In-scope retail teams *(small table — becomes the index for per-team blocks)*

### Client-Level Considerations *(cross-team only)*
7. Client-Level Considerations
   - 7.1 Constraints
   - 7.2 Risks *(table: Risk · Impact · Likelihood · Mitigation · Owner)*
   - 7.3 System Dependencies
   - 7.4 Success Criteria

### Per Retail Team Blocks *(repeat the entire structure below for each retail team in scope)*
8. **[Retail Team Name]**
   - 8.1 Team Overview *(scope, stakeholders, kickoff date)*
   - 8.2 Discovery Findings
     - Category management process & buyer-team workflow
     - Personas & intended dashboard users
     - Key decision drivers
     - Current tools (replaced or augmented)
     - Standard offering vs. identified gaps
     - Working style & recommendation horizon
   - 8.3 Constraints *(team-specific)*
   - 8.4 Risks *(team-specific table)*
   - 8.5 System Dependencies *(team-specific data feeds, retailer data quirks)*
   - 8.6 Success Criteria *(team-specific)*
   - 8.7 Dashboard Modifications
     - 8.7.1 Modifications Summary *(table: Mod ID · Mod Name · Classification · JIRA · Status · Target Delivery)*
     - 8.7.2 Detailed Modifications *(one Heading-3 section per mod, structure in §5)*

### Closing Chapters *(cross-team)*

Each closing chapter is a short reference section that points to the canonical Shelf Intelligence document(s) for that topic. The chapter captures only client-specific notes that supplement those standards.

9. Data & Integration Considerations
   - See `Shelf Intelligence/Shelf Intelligence Data Quality Verification.docx` for the standard data quality framework
   - See `Shelf Intelligence/Shelf Intelligence TPA.docx` for Third-Party Agreement scope and signed positions
   - Capture only client-specific data quality or TPA notes that supplement these references
10. UAT & Acceptance Criteria
    - See `Shelf Intelligence/Shelf Intelligence UAT.docx` for the standard UAT framework, test scripts, and sign-off process
    - Capture only client-specific UAT scope, environments, or acceptance criteria that supplement the standard framework
11. Training & Enablement Tie-In
    - See `Shelf Intelligence/Shelf Intelligence Training Paths.docx` for the standard per-persona training paths
    - See `Shelf Intelligence/Shelf Intelligence Facilitator Guide.docx` for the facilitator-led training approach
    - Capture only client-specific training adaptations, scheduling, or office-hours cadence
12. Rollout & Go-Live
    - See `Shelf Intelligence/Go Live Documentation & Client Signoff.docx` for the standard go-live documentation and client sign-off process
    - Capture only client-specific rollout sequencing, readiness checkpoints, or post go-live cadence

### Appendices
13. Appendices
    - 13.1 Glossary *(Shelf Intelligence + retail/CPG terms)*
    - 13.2 Mockup gallery / screenshots
    - 13.3 JIRA ticket index *(link to live JIRA filter + snapshot table)*
    - 13.4 Data dictionary *(placeholder)*
    - 13.5 Open questions / parking lot

## 5. Per-Modification Field Schema

Every detailed modification (under §8.7.2) uses this structure:

**[MOD-ID] [Modification Name]** *(Heading 3)*

| Field | Description |
|---|---|
| Modification Summary | 1-paragraph narrative description |
| Classification | One of: Product Enhancement / Retailer Specific / Personalization — with brief rationale. *(Source: Notion Phase 3 framing)* |
| Business Need | Why the client requested this; explicit link back to a Discovery decision driver (§8.2) |
| Current State | What the standard dashboard does today |
| Future State / Solution Design | What the modified version will do, with mockup or screenshot reference |
| Data Dependencies | New fields, calculations, filters, joins, source-system impacts |
| Acceptance Criteria | Bulleted, testable |
| References | JIRA ticket · Mockup link · Related discovery notes · Gong timestamp |
| Status & Delivery | Owner · Target Date · Current Status |

## 6. Template Authoring Conventions

- **Heading levels:** Frontmatter and major numbered sections use Heading 1. Per-team blocks use Heading 1 for the team name. Sub-sections within a team use Heading 2 (e.g., `8.2 Discovery Findings`). Individual modifications use Heading 3.
- **Placeholder text:** All fill-in fields use `[bracketed italic placeholder]` style so they're visually obvious and easy to find/replace.
- **Instructional callouts:** Sections that repeat (per-team blocks, per-mod sections) include an instructional callout at the top, e.g.,
  > *Duplicate this block for each retail team in scope. If only one team is in scope, retain a single instance and remove this note.*
- **Tables:** Used for any tabular data (revision history, risks, mods summary). Markdown tables convert cleanly to .docx tables via the export workflow.
- **Crisp branding:** The exported .docx must use Crisp brand colors, fonts (Montserrat per the Crisp Brand Book 2025), and styling. Markdown source is brand-agnostic; styling is applied at export.

## 7. Output Artifacts

Naming follows the existing convention in the `Shelf Intelligence/` folder (`Shelf Intelligence [Topic].docx`).

| Artifact | Path | Purpose |
|---|---|---|
| Markdown template | `Shelf Intelligence/Shelf Intelligence SDD Template.md` | Source of truth, version-controlled, AI-editable |
| Crisp-branded .docx | `Shelf Intelligence/Shelf Intelligence SDD Template.docx` | Client-deliverable; generated from the markdown |
| Usage README | `Shelf Intelligence/Shelf Intelligence SDD README.md` | Short guide: when to use the template, how to fill it in, how to regenerate the .docx |

The .docx export workflow will use the existing Crisp brand skill (`crisp-marketing:crisp-brand`) to ensure brand compliance. Exact tooling (pandoc, python-docx, or skill-driven generation) to be determined in implementation planning.

## 8. Out of Scope for This Spec

- Filling in the template for a specific client engagement — that's a separate downstream task
- Building tooling for automated SDD generation from JIRA data — possible future enhancement, not in this spec
- Modifying the Notion Shelf Intelligence Implementation playbook — out of scope; this spec consumes that playbook as a reference
- Building a Crisp-branded .docx template from scratch — leverage existing Crisp brand assets and the `crisp-brand` skill

## 9. Next Steps

This spec is the input to the writing-plans skill, which will produce a sequenced implementation plan covering:

1. Draft the Markdown template with all sections, tables, and placeholder text
2. Author the usage README
3. Establish the Markdown → .docx export workflow
4. Generate the initial Crisp-branded .docx
5. Validate the template by walking it through a representative dashboard-modification scenario
