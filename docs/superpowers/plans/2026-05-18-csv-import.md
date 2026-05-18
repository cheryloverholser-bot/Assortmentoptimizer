# CSV Import — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add CSV file import and "Download Template" to `Price-Elasticity-Tool.html` so category managers can pre-populate all three input sections from a single file.

**Architecture:** Single file change only — `Price-Elasticity-Tool.html`. New import bar HTML between topbar and tab nav, two CSS rules, two new JS functions (`importCSV` and `downloadTemplate`). No external libraries. FileReader API for reading; Blob + anchor for download.

**Tech Stack:** Vanilla HTML/CSS/JS, browser FileReader API, browser Blob API.

---

## File Structure

- Modify: `Price-Elasticity-Tool.html`
  - Add CSS at line 187 (just before `</style>`)
  - Add HTML at line 198 (between `</div>` topbar close and `<div class="tab-nav">`)
  - Add JS at line 1084 (just before `</script>`)

---

## Task 1: Import bar CSS + HTML scaffold

**Files:**
- Modify: `Price-Elasticity-Tool.html:187` (CSS insertion)
- Modify: `Price-Elasticity-Tool.html:198` (HTML insertion)

No JS yet — this task produces the visible import bar with non-functional buttons.

- [ ] **Step 1: Add CSS for import bar**

In `Price-Elasticity-Tool.html`, insert these two rules **immediately before** the closing `</style>` tag (currently line 188):

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

The `</style>` tag stays — insert before it.

- [ ] **Step 2: Add import bar HTML**

Insert the following block **between** the topbar closing `</div>` (currently line 197) and the opening `<div class="tab-nav">` (currently line 199):

```html
<div class="import-bar">
  <button class="btn-ghost" onclick="document.getElementById('csvFileInput').click()">Import CSV</button>
  <input type="file" id="csvFileInput" accept=".csv" style="display:none" onchange="importCSV(this.files[0])">
  <button class="btn-ghost" onclick="downloadTemplate()">Download Template</button>
  <span id="importStatus" class="import-status" style="display:none"></span>
</div>
```

- [ ] **Step 3: Verify visually in browser**

Open `Price-Elasticity-Tool.html` in a browser. Between the teal topbar and the tab nav, a white bar should appear with two ghost-style buttons: "Import CSV" and "Download Template". Clicking "Import CSV" should open the OS file picker (filtered to .csv). No JS errors in the console.

- [ ] **Step 4: Commit**

```bash
git add Price-Elasticity-Tool.html
git commit -m "feat: add import bar HTML and CSS scaffold"
```

---

## Task 2: `downloadTemplate()` function

**Files:**
- Modify: `Price-Elasticity-Tool.html` (JS section, just before `</script>`)

- [ ] **Step 1: Add `downloadTemplate` function**

Insert the following immediately before the closing `</script>` tag (currently line 1084):

```javascript
// ── CSV Import ────────────────────────────────────────────────────────────────
function showImportStatus(msg, level) {
  const el = document.getElementById('importStatus');
  el.style.display = 'inline';
  el.textContent = msg;
  el.style.color = level === 'error' ? 'var(--red)' : level === 'warn' ? 'var(--amber)' : 'var(--green-dark)';
}

function downloadTemplate() {
  const csv = [
    'field,value',
    'item_name,Brand X 32oz',
    'category,Beverages',
    'retailer,Kroger',
    'unit_cost,1.85',
    'period_type,weekly',
    'purchase_freq,6',
    'basket_size,45.00',
    'age_18_34,35',
    'age_35_54,45',
    'age_55_plus,20',
    'income_under_50k,30',
    'income_50_100k,45',
    'income_100k_plus,25',
    'hh_size_1_2,40',
    'hh_size_3_4,40',
    'hh_size_5_plus,20',
    'comp_name,Brand Y 32oz',
    'comp_price,3.79',
    'your_current_price,3.99',
    '',
    'price,units,period',
    '3.99,1200,Wk 1',
    '3.79,1380,Wk 2',
    '4.19,1050,Wk 3',
    '3.59,1520,Wk 4',
  ].join('\n');

  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'price-elasticity-template.csv';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}
```

- [ ] **Step 2: Verify in browser**

Open the file. Click "Download Template". A file named `price-elasticity-template.csv` should download. Open it in a text editor and confirm it contains two sections separated by a blank line: 20 `field,value` rows (including the header) then 5 rows (`price,units,period` header + 4 data rows). No console errors.

- [ ] **Step 3: Commit**

```bash
git add Price-Elasticity-Tool.html
git commit -m "feat: add downloadTemplate function"
```

---

## Task 3: `importCSV()` — parser core

**Files:**
- Modify: `Price-Elasticity-Tool.html` (JS section, appended to CSV Import block)

This task implements the FileReader + section detection + key-value extraction. It does **not** yet populate fields — it just parses and logs to console. Field population and table population come in Tasks 4–5.

- [ ] **Step 1: Add `importCSV` function (parser core only)**

Append the following function immediately after the `downloadTemplate` function added in Task 2, still before `</script>`:

```javascript
function importCSV(file) {
  if (!file) return;
  if (!file.name.toLowerCase().endsWith('.csv')) {
    showImportStatus('✗ File must be a .csv', 'error');
    return;
  }

  const reader = new FileReader();
  reader.onload = function(e) {
    const lines = e.target.result
      .split('\n')
      .map(l => l.replace(/\r/g, '').trim());

    // Split into scalar section and table section at first blank line
    const blankIdx = lines.findIndex(l => l === '');
    if (blankIdx === -1) {
      showImportStatus('✗ File must contain a blank row separating scalar fields from the price/volume table', 'error');
      return;
    }

    const scalarLines = lines.slice(0, blankIdx);
    const tableLines  = lines.slice(blankIdx + 1).filter(l => l !== '');

    // Parse scalar key-value pairs
    const scalars = {};
    for (const line of scalarLines) {
      const commaIdx = line.indexOf(',');
      if (commaIdx === -1) continue;
      const key = line.slice(0, commaIdx).trim().toLowerCase();
      const val = line.slice(commaIdx + 1).trim();
      if (!key || key === 'field') continue; // skip header row
      scalars[key] = val;
    }

    // Validate table header
    if (tableLines.length === 0) {
      showImportStatus('✗ No price/volume table found after blank row', 'error');
      return;
    }
    const headerCols = tableLines[0].toLowerCase().split(',').map(c => c.trim());
    if (!headerCols.includes('price') || !headerCols.includes('units')) {
      showImportStatus('✗ Price/volume table must have "price" and "units" columns', 'error');
      return;
    }
    const priceCol = headerCols.indexOf('price');
    const unitsCol = headerCols.indexOf('units');
    const periodCol = headerCols.indexOf('period');

    // Parse data rows (max 12)
    const dataRows = [];
    for (const line of tableLines.slice(1)) {
      if (dataRows.length >= 12) break;
      const cols = line.split(',');
      const price  = parseFloat(cols[priceCol]);
      const units  = parseFloat(cols[unitsCol]);
      const period = periodCol !== -1 ? (cols[periodCol] || '').trim() : '';
      if (!isNaN(price) && !isNaN(units)) {
        dataRows.push({ price, units, period });
      }
    }

    if (dataRows.length < 2) {
      showImportStatus('✗ File must contain at least 2 price/volume data rows', 'error');
      return;
    }

    // TODO: populate fields (Tasks 4–5)
    // For now, log to confirm parsing works:
    console.log('Scalars:', scalars);
    console.log('Data rows:', dataRows);
    showImportStatus('✓ Parsed ' + dataRows.length + ' rows (fields not yet populated)', 'ok');
  };

  reader.onerror = function() {
    showImportStatus('✗ Could not read file', 'error');
  };

  reader.readAsText(file);
}
```

- [ ] **Step 2: Verify parser in browser**

Open the file. Click "Download Template" to get the template. Then click "Import CSV" and select that template file. The status bar should show green text: "✓ Parsed 4 rows (fields not yet populated)". In DevTools console, confirm `Scalars` object has 19 keys and `Data rows` array has 4 entries. No console errors.

Test error cases:
- Upload a `.txt` file → status shows red "✗ File must be a .csv"
- Upload a CSV with no blank line → red "✗ File must contain a blank row..."
- Upload a CSV where the table header has no "price" column → red "✗ Price/volume table must have..."

- [ ] **Step 3: Commit**

```bash
git add Price-Elasticity-Tool.html
git commit -m "feat: add importCSV parser core"
```

---

## Task 4: `importCSV()` — scalar field population

**Files:**
- Modify: `Price-Elasticity-Tool.html` (replace the `// TODO: populate fields` placeholder with real field population)

- [ ] **Step 1: Replace TODO comment with scalar field population**

Find this comment block inside `importCSV`:

```javascript
    // TODO: populate fields (Tasks 4–5)
    // For now, log to confirm parsing works:
    console.log('Scalars:', scalars);
    console.log('Data rows:', dataRows);
    showImportStatus('✓ Parsed ' + dataRows.length + ' rows (fields not yet populated)', 'ok');
```

Replace it with:

```javascript
    // ── Populate scalar fields ────────────────────────────────────────────────
    const setVal = (id, val) => {
      const el = document.getElementById(id);
      if (el && val !== '') el.value = val;
    };
    const setNum = (id, val) => {
      const n = parseFloat(val);
      if (!isNaN(n)) { const el = document.getElementById(id); if (el) el.value = n; }
    };

    setVal('itemName',     scalars['item_name']     || '');
    setVal('itemCategory', scalars['category']       || '');
    setVal('itemRetailer', scalars['retailer']       || '');
    setNum('unitCost',     scalars['unit_cost']      || '');

    if (scalars['period_type']) {
      const pt = scalars['period_type'].toLowerCase();
      setPeriodType(pt === '4week' ? '4week' : 'weekly');
    }

    setNum('purchaseFreq', scalars['purchase_freq']  || '');
    setNum('basketSize',   scalars['basket_size']    || '');
    if (scalars['purchase_freq'] || scalars['basket_size']) state.hasLoyalty = true;

    // Demographic fields — set all in group before calling onDemoInput to avoid partial-sum warnings
    setNum('age1834',   scalars['age_18_34']    || '');
    setNum('age3554',   scalars['age_35_54']    || '');
    setNum('age55plus', scalars['age_55_plus']  || '');
    if (scalars['age_18_34'] || scalars['age_35_54'] || scalars['age_55_plus']) onDemoInput('age');

    setNum('inc50',     scalars['income_under_50k'] || '');
    setNum('inc100',    scalars['income_50_100k']   || '');
    setNum('inc100plus',scalars['income_100k_plus'] || '');
    if (scalars['income_under_50k'] || scalars['income_50_100k'] || scalars['income_100k_plus']) onDemoInput('income');

    setNum('hh12',   scalars['hh_size_1_2']   || '');
    setNum('hh34',   scalars['hh_size_3_4']   || '');
    setNum('hh5plus',scalars['hh_size_5_plus'] || '');
    if (scalars['hh_size_1_2'] || scalars['hh_size_3_4'] || scalars['hh_size_5_plus']) onDemoInput('hh');

    setVal('compName',  scalars['comp_name']  || '');
    setNum('compPrice', scalars['comp_price'] || '');

    if (scalars['your_current_price']) {
      setNum('yourCurrentPrice', scalars['your_current_price']);
      document.getElementById('yourCurrentPrice').dataset.manuallyEdited = 'true';
    }

    // TODO: populate table rows (Task 5)
    showImportStatus('✓ Scalars loaded (' + dataRows.length + ' price rows not yet populated)', 'ok');
```

- [ ] **Step 2: Verify scalar population in browser**

Open the file. Click "Import CSV" and select the downloaded template. Verify:
- Item Baseline card: Item Name = "Brand X 32oz", Category = "Beverages", Retailer = "Kroger", Unit Cost = "1.85"
- Period toggle shows "Weekly" active
- Shopper Profile card: Purchase Freq = "6", Basket Size = "45"
- Age group fields: 35 / 45 / 20 (no amber warning — sums to 100)
- Income fields: 30 / 45 / 25 (no amber warning — sums to 100)
- HH size fields: 40 / 40 / 20 (no amber warning — sums to 100)
- Competitive Context: Competitor name = "Brand Y 32oz", comp price = "3.79", your current price = "3.99"
- Status bar shows amber "✓ Scalars loaded (4 price rows not yet populated)"
- No console errors

- [ ] **Step 3: Commit**

```bash
git add Price-Elasticity-Tool.html
git commit -m "feat: importCSV scalar field population"
```

---

## Task 5: `importCSV()` — table row population + status + auto-fill

**Files:**
- Modify: `Price-Elasticity-Tool.html` (replace the Task 5 TODO comment)

- [ ] **Step 1: Replace TODO comment with table population**

Find this comment block inside `importCSV`:

```javascript
    // TODO: populate table rows (Task 5)
    showImportStatus('✓ Scalars loaded (' + dataRows.length + ' price rows not yet populated)', 'ok');
```

Replace it with:

```javascript
    // ── Populate price/volume table ───────────────────────────────────────────
    const tbody = document.getElementById('dataRows');
    // Clear existing rows
    while (tbody.rows.length > 0) tbody.deleteRow(0);

    for (const { price, units, period } of dataRows) {
      const rowCount = tbody.rows.length;
      const row = tbody.insertRow();
      row.innerHTML = `
        <td style="color:var(--muted);font-size:12px;padding:6px 8px">${rowCount + 1}</td>
        <td><input type="number" placeholder="0.00" min="0" step="0.01" oninput="onDataChange()" value="${price}"></td>
        <td><input type="number" placeholder="0" min="0" step="1" oninput="onDataChange()" value="${units}"></td>
        <td><input type="text" placeholder="Wk ${rowCount + 1}" value="${period}"></td>
      `;
    }

    onDataChange(); // triggers button-enable logic and auto-fills yourCurrentPrice if not manually set

    // Auto-fill yourCurrentPrice from last row only if scalar section didn't set it
    if (!scalars['your_current_price']) {
      const lastPrice = dataRows[dataRows.length - 1].price;
      const ycpField = document.getElementById('yourCurrentPrice');
      if (!ycpField.dataset.manuallyEdited) {
        ycpField.value = lastPrice.toFixed(2);
      }
    }

    // ── Final status message ──────────────────────────────────────────────────
    const itemLabel = scalars['item_name'] || 'item';
    if (dataRows.length < 4) {
      showImportStatus(`⚠ Loaded with warnings: ${itemLabel} — ${dataRows.length} price rows (4 required to calculate)`, 'warn');
    } else {
      showImportStatus(`✓ Loaded: ${itemLabel} — ${dataRows.length} price/volume rows`, 'ok');
    }
```

- [ ] **Step 2: Verify full import in browser**

Open the file. Click "Import CSV" and select the template. Verify:
- All scalar fields populated (same as Task 4 checks)
- Data table has exactly 4 rows with prices 3.99 / 3.79 / 4.19 / 3.59, units 1200 / 1380 / 1050 / 1520, periods "Wk 1"–"Wk 4"
- "Calculate Elasticity" button is **enabled** (4 rows ≥ threshold)
- Status shows green: "✓ Loaded: Brand X 32oz — 4 price/volume rows"
- Your Current Price field shows "3.99" (from scalar section)
- No console errors

Test with a 2-row CSV (create a minimal version manually):
```
field,value
item_name,Test Item

price,units,period
4.00,100,Wk 1
3.50,120,Wk 2
```
- 2 rows import successfully
- Status shows amber: "⚠ Loaded with warnings: Test Item — 2 price rows (4 required to calculate)"
- Calculate button remains disabled

Test with a CSV where `your_current_price` is absent — the "Your Current Price" field should auto-fill from the last price row.

- [ ] **Step 3: Commit**

```bash
git add Price-Elasticity-Tool.html
git commit -m "feat: importCSV table population and status messages"
```

---

## Task 6: End-to-end verification + final commit

**Files:** No code changes — verification and final commit only.

- [ ] **Step 1: Full happy-path test**

Download the template, fill it with real-looking data, import it. Then click "Calculate Elasticity". Verify:
- Elasticity coefficient appears
- Demand curve SVG renders
- Tab 2 and Tab 3 buttons become enabled
- Switching to Tab 2 shows pre-filled baseline price and units from the import
- Switching to Tab 3 shows pre-filled competitor price from the import
- No console errors throughout

- [ ] **Step 2: Edge case — re-import with different file**

Import the template (4 rows). Then import again with a different file that has 6 rows and different values. Verify the table is fully replaced (no leftover rows from the first import) and the status message reflects the new file.

- [ ] **Step 3: Edge case — non-CSV file**

Click "Import CSV" and select a `.txt` or `.json` file. Verify red status message appears and no fields are cleared or changed.

- [ ] **Step 4: Edge case — CSV with missing blank line**

Create a CSV with no blank separator and upload it. Verify red fatal error status and no field changes.

- [ ] **Step 5: Commit**

```bash
git add Price-Elasticity-Tool.html
git commit -m "feat: csv import complete — all tasks verified"
```

---

## Self-Review

**Spec coverage:**
- ✅ Import bar HTML between topbar and tab nav
- ✅ `.btn-ghost` style for both buttons
- ✅ Hidden file input with `.csv` accept filter
- ✅ `importCSV(file)` — FileReader, section detection, scalar parsing, table parsing
- ✅ `downloadTemplate()` — all 19 scalar example rows + 4 price/volume rows
- ✅ All 19 field setter mappings (CSV key → element ID)
- ✅ `setPeriodType()` called for `period_type` key
- ✅ Deferred `onDemoInput()` — called after all fields in a group are set
- ✅ `dataset.manuallyEdited = 'true'` on `yourCurrentPrice` when set from CSV
- ✅ Auto-fill `yourCurrentPrice` from last row if scalar section doesn't set it
- ✅ `onDataChange()` called after table population
- ✅ 12-row cap on table section
- ✅ Fatal error: non-.csv file
- ✅ Fatal error: no blank line separator
- ✅ Fatal error: table header missing `price` or `units`
- ✅ Fatal error: fewer than 2 data rows
- ✅ Warning: 2–3 data rows (imports, amber status)
- ✅ Silent: unrecognized scalar keys (loop just skips them)
- ✅ Silent: extra columns in table section (parser only reads indexed columns)
- ✅ Silent: rows beyond 12 (loop breaks at 12)
- ✅ Silent: non-numeric value in numeric field (`setNum` guards with `isNaN`)
- ✅ Status message format: "✓ Loaded: [item_name] — [N] price/volume rows"
- ✅ Warning status format: "⚠ Loaded with warnings: [item_name] — [N] price rows..."
- ✅ Error status format: "✗ [reason]"
- ✅ `showImportStatus` uses `var(--green-dark)`, `var(--amber)`, `var(--red)` per spec

**Nothing out of scope was added.** No drag-and-drop, no Excel support, no export.
