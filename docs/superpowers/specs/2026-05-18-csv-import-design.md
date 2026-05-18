# CSV Import for Price Elasticity Tool — Design Spec

**Date:** 2026-05-18
**Feature:** Add CSV file import and template download to `Price-Elasticity-Tool.html` so category managers can load all data fields from a single file rather than entering them manually.

---

## Goal

Allow a category manager to prepare their data in a CSV file (or fill out a downloaded template) and load it into the tool in one click, pre-populating all three input sections: Item Baseline, Shopper Profile, and Competitive Context.

---

## Architecture

All changes are confined to `Price-Elasticity-Tool.html`. No external libraries. The browser's native `FileReader` API handles file reading. The template download uses a `Blob` + temporary anchor element — no server required.

A new import bar is inserted between the topbar and the tab nav. Two JS functions are added: `importCSV(file)` (parser + field populator) and `downloadTemplate()` (generates and triggers download of a sample CSV).

---

## CSV Format

Two sections separated by exactly one blank row.

### Section 1 — Scalar fields (key,value pairs)

```
field,value
item_name,Brand X 32oz
category,Beverages
retailer,Kroger
unit_cost,1.85
period_type,weekly
purchase_freq,6
basket_size,45.00
age_18_34,35
age_35_54,45
age_55_plus,20
income_under_50k,30
income_50_100k,45
income_100k_plus,25
hh_size_1_2,40
hh_size_3_4,40
hh_size_5_plus,20
comp_name,Brand Y 32oz
comp_price,3.79
your_current_price,3.99
```

### Section 2 — Price/volume table (after one blank row)

```
price,units,period
3.99,1200,Wk 1
3.79,1380,Wk 2
4.19,1050,Wk 3
3.59,1520,Wk 4
```

**Rules:**
- All scalar fields are optional except that at least 2 price/volume rows must be present for import to succeed (4 required to enable Calculate, per existing logic)
- `period_type` accepts `weekly` or `4week` (case-insensitive); defaults to `weekly` if unrecognized
- Unrecognized keys in section 1 are silently ignored
- Extra columns in section 2 beyond `price`, `units`, `period` are silently ignored
- Rows beyond 12 in section 2 are silently dropped (existing 12-row cap)
- The `field` header row in section 1 is optional — the parser skips any row whose value column is empty or whose key is `field`

---

## UI Changes

### Import bar

A new `<div class="import-bar">` inserted between the topbar and tab nav:

```
[ Import CSV ]   [ Download Template ]   [status message]
```

- **Import CSV** — `.btn-ghost` style; triggers a hidden `<input type="file" accept=".csv">` on click
- **Download Template** — `.btn-ghost` style; calls `downloadTemplate()` on click
- **Status message** — inline span, hidden by default; shows confirmation or error after import attempt

**CSS for import bar:**
```css
.import-bar {
  background: white;
  border-bottom: 1px solid var(--border);
  padding: 8px 24px;
  display: flex;
  align-items: center;
  gap: 10px;
}
.import-status {
  font-size: 12px;
  margin-left: 8px;
}
```

### Status messages

- **Success:** `✓ Loaded: [item_name] — [N] price/volume rows` (green, `color: var(--green-dark)`)
- **Fatal error:** `✗ [reason]` (red, `color: var(--red)`)
- **Warning:** `⚠ Loaded with warnings: [reason]` (amber, `color: var(--amber)`)

---

## Parser Logic

### `importCSV(file)`

```
1. FileReader.readAsText(file)
2. Split text into lines, trim whitespace, filter out Windows \r
3. Scan lines top-to-bottom:
   a. Key-value mode (default):
      - Skip the header row (key === 'field')
      - On blank line: switch to table mode
      - Otherwise: split on first comma → key, value → dispatch to field setter
   b. Table mode:
      - First non-blank line must contain 'price' and 'units' headers (case-insensitive); if not, abort with fatal error
      - Subsequent lines: parse price (float), units (float), period (string) → add row to data table
      - Stop after 12 data rows
4. Validate: fewer than 2 data rows → fatal error
5. Call onDataChange() to trigger existing button-enable/row-count logic
6. Auto-fill yourCurrentPrice from last price row IF not already set by the scalar section
7. Show status message
```

### Field setter mapping

| CSV key | Target element ID | Notes |
|---|---|---|
| `item_name` | `itemName` | text |
| `category` | `itemCategory` | text |
| `retailer` | `itemRetailer` | text |
| `unit_cost` | `unitCost` | number |
| `period_type` | toggle `toggleWeekly` / `toggle4wk` | calls `setPeriodType()` |
| `purchase_freq` | `purchaseFreq` | number; sets `state.hasLoyalty = true` |
| `basket_size` | `basketSize` | number; sets `state.hasLoyalty = true` |
| `age_18_34` | `age1834` | number; calls `onDemoInput('age')` after all age fields set |
| `age_35_54` | `age3554` | number |
| `age_55_plus` | `age55plus` | number |
| `income_under_50k` | `inc50` | number; calls `onDemoInput('income')` after all income fields set |
| `income_50_100k` | `inc100` | number |
| `income_100k_plus` | `inc100plus` | number |
| `hh_size_1_2` | `hh12` | number; calls `onDemoInput('hh')` after all HH fields set |
| `hh_size_3_4` | `hh34` | number |
| `hh_size_5_plus` | `hh5plus` | number |
| `comp_name` | `compName` | text |
| `comp_price` | `compPrice` | number |
| `your_current_price` | `yourCurrentPrice` | number; sets `dataset.manuallyEdited = 'true'` |

Demographic `onDemoInput()` calls are deferred until all fields in a group have been processed to avoid partial-sum false warnings.

### `downloadTemplate()`

Generates a CSV string with all scalar keys pre-filled with example values and 4 example price/volume rows. Creates a `Blob`, assigns it to a temporary `<a>` with `download="price-elasticity-template.csv"`, clicks it programmatically, then removes it.

---

## Error Handling

| Condition | Level | Behavior |
|---|---|---|
| File is not `.csv` / unreadable | Fatal | Red status; no fields populated |
| No blank-row separator found | Fatal | Red status; no fields populated |
| Table section header missing `price` or `units` | Fatal | Red status; no fields populated |
| Fewer than 2 data rows after parsing | Fatal | Red status; no fields populated |
| 2–3 data rows (valid import, Calculate disabled) | Warning | Amber status noting row count; fields populated; existing `calcNote` explains |
| Demographic values don't sum to 100% | Warning | Existing per-group amber warnings fire via `onDemoInput()` calls |
| Unrecognized scalar key | Silently ignored | |
| Extra columns in table section | Silently ignored | |
| Data rows beyond 12 | Silently dropped | |
| Non-numeric value in numeric field | Silently ignored (field left blank) | |

---

## Out of Scope

- Export / save current tool state to CSV
- Multi-file import
- Drag-and-drop file target
- Excel (.xlsx) format support
