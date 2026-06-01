# Store Clustering App — Clustering & Planogram Assignment for CPG Category Managers

**Date:** 2026-06-01
**Status:** Design (pre-implementation)
**Owner:** Cheryl Overholser, Crisp

---

## 1. Summary

The Store Clustering App is a productized multi-tenant SaaS application that lets CPG category managers cluster a retailer's stores based on store-level data, then assign each cluster to a planogram (POG) variant. The output package — store-to-cluster, store-to-POG, a defensibility report, and a Blue Yonder Space Planning import file — is the deliverable a category manager hands to space planners and uses to present cluster logic to retailers.

The product targets the iterative, defensibility-driven workflow real category managers run today: try an algorithm, refine it manually, defend the result to a retailer. It is *not* a black-box ML system. The CPG brings the data either via file upload (v1) or — in a planned follow-on — by pointing at a Crisp direct-retailer data connection. The app provides the workflow, the math, and the defensible audit trail.

---

## 2. Goals and non-goals

### In scope for v1

- Multi-tenant web SaaS. Org admin invites users; each user belongs to one CPG org.
- Single-category clustering analyses (one category per project).
- Five required file uploads per project: store master, store-level sales, shopper/demographics, store attributes, POG variant definitions.
- Hybrid clustering: algorithm suggests, analyst refines. k-means and hierarchical (Ward) in v1.
- Project → multiple scenarios → side-by-side scenario comparison.
- Within a scenario: multiple refinement drafts on the same clustering run; one draft promoted at a time.
- Algorithm-vs-promoted-draft diff view for retailer defensibility.
- Four export artifacts packaged as a zip with a reproducibility manifest.
- Desktop browsers only.

### Out of scope for v1 (deliberately deferred)

- Direct connectors to Crisp's retailer data platform (planned for v2; see §14), Circana / NPD syndicated providers, retailer loyalty platforms.
- Multi-category roll-ups across projects.
- Predicted or automatic cluster-to-POG-variant assignment (analyst maps manually).
- Retailer-facing portals — clients export and present themselves.
- Mobile UI.
- Imputation of missing variable values (stores with missing values in selected variables are excluded from the run with explicit warning).
- Per-scenario store exclusions (use the master to control which stores exist).
- Additional clustering algorithms (DBSCAN, GMM, etc.).
- Real-time multi-analyst co-editing of a scenario.

---

## 3. Primary user and job-to-be-done

**Primary user:** Category Manager / Category Analyst at a CPG.

Day-to-day they live in Excel, syndicated data tools (Circana / Nielsen), and JDA / Blue Yonder Category Management. They own the category review process and must defend cluster logic to retailers (the buyers who ultimately approve the planogram).

**Job-to-be-done:** "Cluster this retailer's stores into a small number of groups, then tell space planning which planogram variant each store should receive — and let me defend every decision to the retailer."

Two non-negotiables emerge from this job:

1. **Defensibility** — the analyst must be able to show *why* clusters are what they are, and *what* they changed from the algorithm's suggestion. Black-box answers are unsellable.
2. **Iteration** — the analyst will try multiple approaches before committing. The tool must make iteration cheap (forking scenarios, drafting refinements) without losing prior versions.

---

## 4. Domain model

```
Organization (a CPG company)
  └── User (analyst, manager; role-based access)
  └── Project (one category review / reset)
        └── DataSources (5 file types, versioned per upload)
              ├── store_master
              ├── store_sales
              ├── store_demographics
              ├── store_attributes
              └── pog_variants
        └── Scenario (a saved variable + algorithm + refinement configuration)
              ├── ClusteringRun (algorithm execution + results — immutable)
              ├── RefinementDrafts (named layers of manual moves/merges/renames)
              │     ├── Draft (one at a time may be "promoted")
              │     └── …
              ├── ClusterToVariantAssignment (tied to the promoted draft)
              └── Notes (analyst rationale, for defensibility)
        └── Export (a frozen output package generated from the finalist scenario)
```

### Domain rules

- **Data sources live at the project level.** Uploaded once; all scenarios in the project share them. Re-uploading creates a new version; scenarios pin to a specific version of each file.
- **Scenarios are forkable.** Duplicating a scenario copies variables, algorithm settings, refinement drafts, and POG assignments.
- **ClusteringRun is immutable.** Refinements layer on top of it. This is the audit-trail backbone.
- **A scenario can hold multiple refinement drafts** under the same run. Exactly one is "promoted" at a time — the promoted draft feeds POG assignment and exports.
- **One scenario per project can be marked the "finalist."** Only the finalist may produce exports.

---

## 5. Architecture

Six components, each with one responsibility. Tech choices (specific frameworks, hosting, DB engine) are intentionally deferred to the implementation plan.

```
┌──────────────────────────────────────────────────────────────────┐
│  Browser (Category Manager)                                      │
│  - Project workspace UI (scenario sidebar + canvas)              │
│  - Comparison view                                               │
│  - File upload / column-mapping wizard                           │
└────────────────────────┬─────────────────────────────────────────┘
                         │ HTTPS (REST/JSON + signed upload URLs)
┌────────────────────────▼─────────────────────────────────────────┐
│  API service                                                     │
│  - Auth + org/role enforcement                                   │
│  - Project / scenario / data-source CRUD                         │
│  - Job orchestration; reads job status                           │
│  - Generates exports (CSV, XLSX, PDF, BY import) on demand       │
└──────┬──────────────────────┬────────────────────────┬───────────┘
       │                      │                        │
┌──────▼──────────┐  ┌────────▼────────────┐  ┌────────▼──────────┐
│ Relational DB   │  │ Object storage      │  │ Compute worker     │
│ - Orgs/users    │  │ - Raw uploads       │  │ - Clustering jobs  │
│ - Projects      │  │ - Generated exports │  │   (k-means, hier.) │
│ - Scenarios     │  │   (PDF, CSV, BY)    │  │ - Profile stats    │
│ - Runs + drafts │  │                     │  │ - Async, queued    │
└─────────────────┘  └─────────────────────┘  └────────────────────┘
```

### Multi-tenancy

- Every persisted row carries an `org_id`.
- Every API request resolves an `org_id` from the auth token. Cross-org reads or writes must be impossible.
- Code-review rule: queries without `org_id` filtering fail review. Tests enforce isolation on every endpoint.

### Concurrency

- Soft lock per scenario in v1. Second analyst sees a read-only banner identifying the current editor. Lock auto-releases after 15 minutes of editor idle time.
- No real-time co-editing in v1.

### Capacity assumptions

- Per project: up to ~50,000 stores × ~50 variables. Fits comfortably in a single Postgres-class DB.
- Per org: hundreds of projects per year.
- Object storage holds raw uploads and generated exports indefinitely (retention policy is a future concern).

---

## 6. Workflow and UX

### 6.1 Project setup (once per analysis)

1. Create project — name + category.
2. Upload the five data files. For each file:
   - App auto-detects the file type (or analyst picks).
   - Preview of the first 100 rows.
   - Column-mapping wizard maps the analyst's headers to the canonical schema (see §7). Smart defaults via header-name matching; saved per org + file type.
3. Validation runs immediately (see §7.3).
4. If validation has blocking errors, analyst resolves them before any scenario is usable.

### 6.2 Scenario canvas (the working surface)

A vertical stack of collapsible blocks. Each block shows current state and a re-run/edit affordance. Changes to a block invalidate downstream blocks (visual indicator + re-run prompt).

| Block | Purpose | Analyst controls |
|---|---|---|
| Data | Joined dataset summary (rows, missing %, file versions) | Pick file versions |
| Variables | Available variables across the four numeric files | Toggle inclusion, set weights, standardize/normalize options |
| Cluster | Runs the algorithm | Algorithm (k-means or hierarchical), k, random seed, distance metric |
| Profile | Defensibility table from the run | (Read-only output) |
| Refine | Manual overrides as named drafts | Create drafts, move/merge/split/rename, promote a draft, view algorithm-vs-draft diff |
| Assign POG | Map each cluster → POG variant | Pick a variant per cluster |
| Notes | Free-text rationale | Plain text |

### 6.3 Refinement: drafts and defensibility diff

The Refine block exposes:

- **Draft tabs.** Each draft is a named layer of moves/merges/renames over the immutable clustering run. Tabs show a small diff badge: `N stores moved · M ops · profile shift`.
- **Promote action.** "Promote this draft" makes it the basis for POG assignment and exports. Only one draft is promoted at a time.
- **Algorithm-vs-promoted-draft diff view.** A side-by-side panel showing:
  - The algorithm's original cluster assignment for each store.
  - The promoted draft's assignment.
  - Every refinement action, with timestamp, user, action type (move / merge / split / rename), and optional rationale note.
  - Summary stats: total stores moved, profile centroid drift, cluster name changes.
- This diff also renders as a section of the cluster profile PDF export (§8.3).

### 6.4 Scenario forking and comparison

- "Duplicate scenario" copies variables, algorithm, refinement drafts, and POG assignments. The analyst then tweaks one thing (k, variables, weights, refinements).
- The project's **Compare Scenarios** view shows 2–3 selected scenarios side-by-side: cluster size distribution bars, silhouette score, POG variant coverage, and a diff line ("312 stores move between Scenario 1 and Scenario 2; 89 stores change POG variant").
- One scenario can be marked the project's **finalist**. Only the finalist can produce exports.

---

## 7. Data ingestion

### 7.1 Canonical schema (target after column mapping)

| File | Required canonical fields | Optional |
|---|---|---|
| `store_master` | `store_id`, `banner`, `region`, `format`, `status` | `address`, `lat`, `long`, `open_date` |
| `store_sales` | `store_id`, `category_id`, `sales_dollars_52w`, `units_52w` | Weekly time series, sub-category sales |
| `store_demographics` | `store_id`, `hh_income_median`, `hh_count`, plus ≥ 3 demographic % fields | Any additional demographic % fields |
| `store_attributes` | `store_id`, `sqft_total`, `category_linear_ft` | `freezer_doors`, `cooler_doors`, `parking`, custom flags |
| `pog_variants` | `variant_id`, `name`, `linear_ft`, `sku_count` | `description`, `image_url`, BY variant code |

A "project" is scoped to one `category_id`. The `category_id` column in `store_sales` lets the analyst point at a multi-category sales file; the app filters to the project's category at ingestion. Multi-category analysis (one project spanning multiple categories) is out of scope for v1.

### 7.2 Column mapping

- Mapping UI lists canonical fields on the left, source columns on the right.
- Header-name matching provides smart defaults (e.g. "Total $ L52W" → `sales_dollars_52w`).
- Mappings are saved per `org_id` + file type and auto-applied on re-upload.
- Mappings can be edited and re-applied retroactively.

### 7.3 Validation

Validation runs whenever any of the five files is uploaded or replaced.

**Blocking errors (clustering cannot proceed until resolved):**

- A required file is missing.
- Required canonical fields are absent or untyped after mapping.
- **Orphan store IDs** — any store referenced in `store_sales`, `store_demographics`, `store_attributes`, or `pog_variants` that does not exist in `store_master`. The validation summary lists every orphan ID and the file(s) that reference it, in a top-of-page banner that cannot be missed. The analyst resolves this in one of two ways:
  - Download orphan list as a starter CSV (`store_id` pre-populated), fill in required fields offline, re-upload `store_master` as a new version.
  - Add to master in-app via a quick form for each orphan store. The edit creates a new master version with the additions, preserving audit trail.
  - There is no "ignore orphans" escape hatch in v1.
- POG variant referenced by an in-app assignment but missing from the uploaded `pog_variants` file.

**Warnings (allow clustering but surface in profile report):**

- Stores in master with no row in `store_sales`, `store_demographics`, or `store_attributes`. These stores are eligible for clustering but excluded from any variable they lack. The cluster profile report flags this.
- Required canonical fields under the configured null threshold (default 95% populated).

### 7.4 Versioning

- Every upload of a file type creates a new version of that file.
- Scenarios pin to specific versions of each file. Changing data does not silently shift past results.
- Re-running a scenario against newer file versions is a deliberate analyst action.

### 7.5 Data-source provider abstraction (forward-looking)

Although v1 supports only file upload, the ingestion layer is designed around a provider abstraction so that alternative data sources — most importantly Crisp's direct-retailer data connection — can be added in v2 without rearchitecting downstream code.

- Each of the five data types (`store_master`, `store_sales`, `store_demographics`, `store_attributes`, `pog_variants`) is fronted by a provider interface (e.g. `StoreSalesProvider`) with a uniform contract: produce a typed, validated, versioned dataset for a given project.
- The v1 implementation of every interface is a `FileUploadProvider` backed by the uploaded CSV/XLSX file and the saved column mapping.
- Future implementations — `CrispConnectorProvider`, syndicated-data providers — plug into the same interface. Scenarios, clustering, refinement, exports, and the audit trail remain unchanged.
- A scenario's pinned "version" of a data source becomes a more general concept: for file-upload providers it's the uploaded file version; for connector providers it is a frozen snapshot taken at scenario-creation time (to preserve the reproducibility guarantee in §11).

---

## 8. Clustering engine

### 8.1 Algorithms

- **k-means** (default). Analyst sets `k` (2–12).
- **Hierarchical (Ward linkage).** Analyst sets target `k` or distance threshold; dendrogram available.
- No other algorithms in v1.

### 8.2 Variable handling

- All numeric variables standardized (z-score) before clustering.
- Categorical variables (region, banner, format) one-hot encoded if the analyst includes them.
- Analyst can weight variables (default 1.0 each); useful for "sales matters 3× more than demographics for this category."
- **Missing values:** stores with missing values in any selected variable are excluded from the run with a clear named warning. No imputation in v1.

### 8.3 Run output (immutable)

Each ClusteringRun persists:

- Cluster assignment per included store.
- Cluster centroids on every variable in the model.
- Quality metrics: silhouette score, inertia (k-means), cluster size distribution.
- Run configuration snapshot: selected variables, weights, algorithm, k, random seed, file versions used.

### 8.4 Profile generation

The Profile block reads a ClusteringRun and produces a defensibility view:

- Per-cluster mean and median for every selected variable, plus important *unselected* variables (confirmatory evidence).
- Top 3 variables that most distinguish each cluster (greatest standardized centroid distance from grand mean).
- Auto-generated plain-English narrative per cluster ("Cluster 1: 2.3× sales velocity, 18% higher freezer linear ft, premium-shopper skew").

### 8.5 Refinement actions

Layered on top of the immutable run, captured in a refinement draft:

- Move stores between clusters (individual or bulk via filter, e.g. "all cluster-3 stores with sqft > 25k → cluster 1").
- Merge two clusters into one.
- Split a cluster (re-runs sub-clustering on just that cluster's stores).
- Rename clusters to business-meaningful labels ("Premium Urban", "Value Rural").

Every action logs `user_id`, `timestamp`, `action_type`, `before`, `after`, and an optional rationale note.

### 8.6 Performance budget

- 50k-store × 50-variable dataset clusters in under 30 seconds on a single worker.
- Longer runs go async via a job queue with progress feedback in the UI; the analyst does not watch a spinner.
- A browser refresh mid-run does not lose the job — the UI re-attaches by job ID.

---

## 9. Outputs and exports

Exports are generated from the project's **finalist scenario + promoted draft**.

### 9.1 Export 1 — Store → Cluster (CSV + XLSX)

```
store_id, banner, region, cluster_id, cluster_name, was_moved_manually
```

- One row per included store.
- `was_moved_manually` flag exposes refinement decisions.
- Excluded stores listed on a second sheet with the reason.

### 9.2 Export 2 — Store → POG Variant (CSV + XLSX)

```
store_id, banner, region, cluster_name, pog_variant_id, pog_variant_name, pog_linear_ft
```

- Inner join of Export 1 with the cluster→variant assignment.
- This file is what space planners most often consume directly.

### 9.3 Export 3 — Cluster Profile Report (PDF)

- Cover page: project name, finalist scenario, promoted draft, generation date, run config hash.
- Per-cluster page: name, store count, top distinguishing variables, mean/median table, geographic distribution by region and banner.
- **Algorithm-vs-promoted-draft diff section** — full table of refinement actions with timestamps, users, and rationale notes.
- POG variant assignment summary with the analyst's rationale notes.
- Run quality appendix: silhouette score, full variable list, file version stamps.
- Clean type, page numbers, optional CPG logo on the cover.

### 9.4 Export 4 — Blue Yonder Space Planning import file

- Format: BY Space Planning's store-cluster import schema (CSV with the exact column names BY expects).
- Maps cluster names → BY cluster codes and POG variants → BY planogram references using `variant_id` from the uploaded `pog_variants` file.
- A two-tab Excel companion documents each field for the receiving space planner.
- **Dependency:** BY's import schema is not publicly documented. A real customer's BY environment or vendor documentation is required to pin the schema before development. This is the single external dependency in v1 and the most likely source of late surprises.

### 9.5 Export packaging and reproducibility

- All four artifacts are bundled into a zip with a `manifest.json` containing:
  - The full run config snapshot (variables, weights, algorithm, k, seed).
  - File versions used.
  - Promoted draft ID and ordered list of refinement actions.
  - SHA-256 hashes of each artifact.
- Past exports are listed on the project page (date, scenario, promoted draft, generating user) and re-downloadable.
- Re-exporting from the same scenario + draft + file versions produces **byte-identical** artifacts.

---

## 10. Error handling and edge cases

| Situation | Behavior |
|---|---|
| Orphan store IDs (any non-master file references a store missing from master) | **Block** clustering. Banner lists orphans; fix in-app via re-upload or add-to-master form. |
| Stores in master with no row in another file | Warning. Stores remain eligible; profile report flags missing-data exposure. |
| Required canonical fields under 95% populated | Warning. Surfaced in validation summary. |
| `k` requested exceeds distinct possible groupings | Clear UI error; analyst lowers `k`. |
| Cluster ends up with 1–3 stores | Warning surfaced in profile; analyst may merge in refinement. Doesn't block export. |
| POG variant referenced in assignment but missing from `pog_variants` | Block; assignment cannot save. |
| Cluster has no POG variant assigned when exporting | Block export. Every cluster in the promoted draft must have a variant. |
| File re-uploaded mid-analysis | New version created; existing scenarios stay pinned to prior version unless analyst opts in. |
| Internet drops during large upload | Resumable signed-URL upload; partial uploads garbage-collected after 24h. |
| Two analysts open the same scenario | Soft lock: second analyst sees read-only with editor's name; 15-min idle releases lock. |
| Worker dies mid-clustering | Run is all-or-nothing. Scenario stays in last valid state. UI shows "job failed, retry." No partial persisted outputs. |
| Browser refresh mid-clustering | Job continues server-side; UI re-attaches by job ID. |

---

## 11. Audit and defensibility

The audit trail is a first-class feature, not an afterthought.

- Every refinement action logged: `user_id`, `timestamp`, `action_type`, `before`, `after`, optional `rationale_note`.
- Every export pins file versions, run configuration, promoted draft ID, and the full action log.
- An export downloaded six months later can be regenerated from its manifest and produce byte-identical artifacts.
- The cluster profile PDF includes the audit trail in human-readable form for retailer presentations.

---

## 12. Testing strategy

| Layer | Coverage |
|---|---|
| Unit | Clustering math wrappers, column-mapping logic, validation rules, export formatters |
| Integration | Upload → mapping → validation → cluster → refine → export end-to-end against three canned fixture datasets (small / normal / edge cases) |
| Multi-tenancy | Every API endpoint has an isolation test (cross-org read and write paths both fail) |
| Export determinism | Same finalist scenario + promoted draft + file versions + seed produce byte-identical exports across re-runs |
| Performance | 50k × 50 dataset clusters in under 30s; UI stays responsive during async jobs |
| User-acceptance | A Crisp category-management consultant runs an end-to-end exercise on a real anonymized CPG dataset before launch |

---

## 13. Open questions and risks

1. **Blue Yonder Space Planning import schema.** Format is not publicly documented. Need a customer's BY environment or vendor docs before §9.4 can be implemented. Possible mitigation: ship v1 with Exports 1–3 and a "BY export — coming soon" placeholder; add Export 4 once schema is pinned with a beta customer.
2. **Auth / SSO posture for enterprise CPGs.** Password + MFA likely sufficient for v1 launch, but enterprise CPG procurement will demand SAML SSO. Should be scoped into the plan as an early follow-on.
3. **Pricing model and limits.** Not a design concern, but capacity assumptions (50k stores, 50 variables, hundreds of projects/org/year) inform infrastructure cost; will need a business-model conversation before scale-out.
4. **PDF generation quality.** Retailer-facing presentation quality is critical. Plan should evaluate at least two PDF rendering approaches and pick on output quality, not just dev speed.

---

## 14. Planned v2 — Crisp direct-retailer data connector

v1 ships with file upload as the only data source. v2 adds an alternative: instead of (or alongside) uploading files, a CPG user can point a project at a Crisp direct-retailer data connection and pull the equivalent of `store_master`, `store_sales`, `store_demographics`, and `store_attributes` directly. (`pog_variants` continues to come from upload — Crisp's feeds do not provide planogram-variant definitions.)

This section captures the v2 intent so v1 architecture choices stay forward-compatible. It is **not** a v1 commitment — none of the work below is in scope for v1 development.

### 14.1 Why v2, not v1

Three reasons the connector is deliberately deferred:

- **Different auth and permissioning model.** v1's permission model is org-internal: a CPG user has access to their org's projects. A Crisp connector requires verifying that the CPG has an active Crisp subscription, that the requested retailer's feed is licensed to that CPG, and that the user has access to that feed. This is a new dimension of access control that needs Crisp data-team alignment.
- **Snapshot semantics need design.** Audit and defensibility (§11) require byte-identical re-export months later. A live data feed contradicts that unless the system explicitly snapshots the data at scenario-creation time and pins the scenario to that snapshot — equivalent to a file version in v1.
- **It's a strategic moat.** Worth designing carefully with Crisp's data-platform team, not bolting on under v1 pressure.

### 14.2 What v1 does to stay forward-compatible

- §7.5 provider abstraction: ingestion is interface-driven, not file-coupled.
- Data-source versioning is conceptual ("pin a version") rather than file-coupled ("pin a file upload row").
- The validation layer (§7.3) operates on the typed canonical schema, not on raw uploaded files — so the same validation rules apply whether the data comes from a file or a connector.

### 14.3 Open questions for v2 design

These are explicitly *not* answered in this spec; they are the agenda for v2 design:

- How does a CPG user authorize the app to read their Crisp connection? OAuth-style handshake, service account, or something else?
- How does the user select *which* retailer's feed to use for a given project?
- Snapshot at scenario-creation only, or also support periodic re-snapshots with diff alerts ("9 stores added since your last refresh")?
- What happens when the connector data shape differs from the file-upload canonical schema (extra fields, different units)? Provider does normalization, or the connector defines its own canonical mapping?
- Pricing and packaging — is the connector a free add-on for Crisp subscribers, a paid upgrade, or bundled with a higher SKU?

---

## 15. Next steps

The next document is the **implementation plan** — produced by the `writing-plans` skill from this design. The plan will:

- Pick the actual tech stack (frontend framework, API runtime, DB engine, job queue, hosting).
- Sequence the work into shippable increments (likely: foundation + auth → upload + validation → clustering + scenario canvas → refinement drafts → comparison view → exports → BY format).
- Honor the provider abstraction (§7.5) so v2's Crisp connector slots in cleanly.
- Identify per-increment acceptance criteria and tests.
