# Price Elasticity Tool — Design Spec

**Date:** 2026-05-18
**Feature:** Interactive HTML tool for category managers to calculate price elasticity and model pricing decisions.

---

## Goal

Give client-side category managers at CPG manufacturers a self-contained tool to calculate price elasticity from their own data and apply it to three core pricing decisions: setting or defending an everyday price, evaluating a promotion, and responding to a competitive price move.

---

## Architecture

Single self-contained HTML file — same pattern as `Crisp-Assessment-AI-Process-Plan.html`. Three tabs share a common JavaScript data state object. The elasticity coefficient and item context calculated in Tab 1 flow forward automatically into Tabs 2 and 3. No backend, no save state, no external dependencies.

Visual design follows the existing Crisp system: `--teal-dark` (`#0D3D35`) headers, `--green` (`#00C49A`) accents, white cards, `--bg` (`#eef3f2`) page background, same CSS variable set, topbar, and card patterns.

**Data flow:**
```
Tab 1 inputs → elasticity coefficient + baseline price/volume + competitor price
    ↓
Tab 2: Promo Modeler (uses coefficient + baseline)
Tab 3: Competitive Response (uses coefficient + baseline + competitor price)
```

Minimum viable data for Tab 1: 4 price/volume data pairs. Loyalty and panel data are optional throughout — the tool flags when they're absent and notes reduced segmentation fidelity.

---

## Tab 1 — Price Elasticity Calculator

### Inputs

Three collapsible input sections:

**Item Baseline (required)**
- Item name, category, retailer (text fields — context only, not used in calculations)
- Data entry table: up to 12 rows of Price ($) / Units Sold / Time Period label
- Toggle: Weekly data or 4-week period data (affects axis labels only)
- Optional: Unit cost ($) — used in Tab 3 margin calculations if provided
- Minimum 4 rows required before calculation runs

**Shopper Profile (optional — loyalty or panel data)**
- Avg purchase frequency (purchases per year)
- Avg basket size ($)
- Demographic breakdown inputs (% of shoppers):
  - Age: 18–34 / 35–54 / 55+
  - Income: Under $50K / $50–100K / $100K+
  - Household size: 1–2 / 3–4 / 5+
- When empty: tool displays a flag — "Shopper profile data not entered. Demographic sensitivity analysis will not be available."

**Competitive Context (optional — seeds Tab 3)**
- Competitor item name
- Competitor current price ($)
- Your current price (pre-filled from last row of data table if available, editable)

### Calculation

Arc elasticity formula applied across all data pairs:

```
E = ((Q2 - Q1) / ((Q2 + Q1) / 2)) / ((P2 - P1) / ((P2 + P1) / 2))
```

For more than two data pairs, calculate arc elasticity for each consecutive pair and average. Display the resulting coefficient to two decimal places.

### Outputs

- **Elasticity coefficient** — displayed prominently with plain-language interpretation:
  - |E| > 1.5 → "Highly elastic — demand drops sharply with price increases"
  - 1.0 < |E| ≤ 1.5 → "Elastic — shoppers are price-sensitive"
  - 0.5 < |E| ≤ 1.0 → "Moderately inelastic — some price sensitivity"
  - |E| ≤ 0.5 → "Inelastic — demand is relatively price-stable"
- **Demand curve chart** — price on X axis, projected units on Y axis, plotted from the entered data pairs with a fitted curve
- **Shopper sensitivity table** — shown only when demographic data is entered; applies research-based default sensitivity index values to the overall coefficient to estimate relative segment response. Defaults (editable): Income — under $50K: 1.6×, $50–100K: 1.0×, $100K+: 0.7×. Age — 18–34: 1.3×, 35–54: 1.0×, 55+: 0.8×. Household size — 1–2: 1.1×, 3–4: 1.0×, 5+: 1.2×. Displayed as a ranked table: segment name, index, estimated segment elasticity coefficient.

---

## Tab 2 — Promo Modeler

Pulls elasticity coefficient and baseline price/volume from Tab 1. Requires Tab 1 to have a valid coefficient before activating.

### Inputs

- Promo price — toggle between absolute $ entry and % off entry
- Promo duration (weeks)
- Merchandising support selector: None / Feature only / Display only / Feature + Display
  - Default lift multipliers: 1.0 / 1.15 / 1.20 / 1.35 (editable)
- Baseline weekly units (pre-filled from Tab 1 average, editable)
- Baseline price (pre-filled from Tab 1, editable)

### Calculation

```
Price change % = (promo price - baseline price) / baseline price
Unit lift % = Price change % × elasticity coefficient × merchandising multiplier
Incremental units per week = baseline units × unit lift %
Total incremental units = incremental units per week × promo duration
Promo revenue = (baseline units + incremental units per week) × promo price × promo duration
Baseline revenue = baseline units × baseline price × promo duration
Breakeven incremental units = (baseline price - promo price) × baseline units × promo duration / promo price
```

If loyalty data entered:
- Frequency vs. pantry load split: if promo depth > 20%, flag pantry loading risk ("Promotions deeper than 20% typically drive pantry loading rather than new trip occasions. Incremental units may not represent new buyers.")

### Outputs

- Projected weekly unit lift (units + %)
- Total incremental units over promo period
- Promo revenue vs. baseline revenue — side-by-side comparison cards
- Breakeven indicator — clear pass/fail: "Needs X incremental units to break even. Projected lift is Y units. [On track / At risk]"
- If loyalty data entered: frequency impact vs. basket size impact breakdown; demographic segment lift table

---

## Tab 3 — Competitive Response

Pulls elasticity coefficient, your current price, and competitor current price from Tab 1. Requires Tab 1 to have a valid coefficient before activating.

### Inputs

- Competitor's new price (pre-filled from Tab 1, editable)
- Cross-price sensitivity: Low / Medium / High
  - Low = 0.3× own-price elasticity coefficient
  - Medium = 0.6× own-price elasticity coefficient
  - High = 0.9× own-price elasticity coefficient
  - Tooltip on each option explaining when it applies (e.g., "High — your item and the competitor's are frequently swapped by shoppers; common in commodity or private-label-heavy categories")
- Your response price (optional — for modeling a specific counter-price beyond the three scenario presets)

### Calculation

```
Competitor price change % = (competitor new price - competitor current price) / competitor current price
Cross-elasticity demand shift % = competitor price change % × cross-price sensitivity coefficient
Demand shift (units) = baseline units × cross-elasticity demand shift %

Match scenario: your price = competitor new price; apply own-price elasticity to calculate volume recovery
Hold scenario: your price unchanged; demand shift applied directly
Counter-promote: apply Promo Modeler logic at a user-specified or auto-suggested depth to recover shifted demand
```

If unit cost entered in Tab 1: calculate margin impact for Hold and Match scenarios.

### Outputs

Three response scenario cards displayed side-by-side:

**Hold Price**
- Projected demand shift away from your item (units + %)
- Revenue impact vs. current baseline
- Price gap vs. competitor (before and after their move) — flag if gap approaches switching threshold. Threshold defined as: (1 / |elasticity coefficient|) expressed as a price gap %; warning fires at 70% of that threshold (e.g., coefficient of −2.0 → indifference point at 50% gap; warning at 35% gap)

**Match Price**
- Volume recovery from holding vs. matching
- Revenue tradeoff (match typically recovers units but sacrifices margin)
- Margin impact (if cost data available)

**Counter-Promote**
- Promo depth needed to recover projected demand shift
- Estimated volume recovery
- Revenue and margin cost of the promo response

**Price Gap Meter** — visual bar showing your price vs. competitor price, with a flagged danger zone derived from the elasticity coefficient (when gap exceeds a threshold associated with significant switching).

If loyalty data entered: shopper segment switching risk — flags which demographic groups are most at risk based on their sensitivity profile from Tab 1.

---

## Error States & Validation

- Tab 1: fewer than 4 data pairs → disable Calculate button, show inline note
- Tabs 2 and 3: Tab 1 coefficient not yet calculated → tabs show a "Complete Tab 1 first" placeholder
- Demographic inputs that don't sum to 100% → inline warning, calculation still runs using entered values
- Promo price higher than baseline price → flag as unusual, allow it
- All negative elasticity coefficients are expected (price up → demand down); positive coefficients flag as: "Positive elasticity detected — check your data for entry errors"

---

## Out of Scope

- Save/export functionality (no file download, no localStorage)
- Multi-item or category-level elasticity (single item only)
- Time-series trending or seasonality adjustment
- Integration with any external data source or API
