# Cantactix / Crisp Assessment Templates — Project Context

## Plugin Repository
**Assessmentskills plugin:** https://github.com/cheryloverholser-bot/Assessmentskills

Install command:
```
claude plugin install https://github.com/cheryloverholser-bot/Assessmentskills.git
```

## Key Files
| File | Purpose |
|---|---|
| `Crisp-Assessment-AI-Process-Plan.html` | Main deliverable — single self-contained HTML reference for Crisp's AI-enhanced Assessment process |
| `docs/superpowers/specs/2026-05-06-assessment-process-plan-design.md` | Design spec (v2.0) |
| `docs/superpowers/plans/2026-05-06-assessment-process-plan.md` | Implementation plan with 9 tasks and all 10 prompt definitions |

## Skills in the Plugin
| Skill | Status | Purpose |
|---|---|---|
| `crisp-prompt-builder` | ✅ Production-ready | Add new AI prompts to the HTML file |
| `crisp-questionnaire-outreach` | ✅ Production-ready | Draft the Phase 1 (Onboarding) client email introducing the CMS Questionnaire |
| `crisp-discovery-outreach` | ✅ Production-ready | Draft the Phase 2 (Discovery) workshop logistics + individual stakeholder interview invites from the client's reply |
| `crisp-engagement-setup` | 🔲 Scaffold | Configure a new client engagement (localStorage project setup) |
| `crisp-phase-generator` | 🔲 Scaffold | Generate or update a phase section in the HTML |
| `crisp-raci-sync` | 🔲 Scaffold | Sync RACI matrix content into the HTML |
| `crisp-template-manager` | 🔲 Scaffold | Manage template links per phase |
| `crisp-search-indexer` | 🔲 Scaffold | Build/refresh the client-side search index |
| `crisp-progress-reporter` | 🔲 Scaffold | Report task completion state from localStorage |
| `crisp-engagement-generator` | 🔲 Scaffold | Generate a new engagement-specific copy of the HTML |

## Brand Tokens
```
--green:       #00C49A
--green-dark:  #009E7C
--teal-dark:   #0D3D35
--green-light: #e6f9f4
```

## Prompt HTML Pattern
Prompt IDs are sequential: card = `pc{N}`, text = `pt{N}`.
Badge classes: `badge-current` (NOW, dark teal) · `badge-new` (NEW, amber `#d97706`)
