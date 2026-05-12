# Asset-Required Check-Off Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Disable each key activity checkbox until at least one asset (filename + optional URL) is logged for it; auto-uncheck when the last asset is removed.

**Architecture:** Add `projectData.assets` to the data model. Inject an `.asset-list` sibling beneath every `.progress-task` row at startup via JS (avoids 20 HTML edits). Add helper functions for rendering chips, managing the in-place add form, persisting changes, and gating checkbox `disabled` state.

**Tech Stack:** Vanilla JS, CSS, single self-contained HTML file.

---

## File Structure

**Only file modified:**
- `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`
  - Insert CSS rules for `.asset-list`, `.asset-chip`, `.asset-form`, `.asset-add`, and `.progress-task.locked`
  - Add JS functions for the entire asset system
  - Wire into `launchProject`, `resumeProject`, `newProject`, and the page-load handler

**Verify with:** `http://localhost:7788/Assessments%20Templates/Crisp-Assessment-AI-Process-Plan.html`

---

### Task 1: CSS for asset chips, form, and locked state

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`

- [ ] **Step 1: Add asset CSS rules**

Find this line in the stylesheet:
```css
.progress-task.done label { text-decoration: line-through; color: #9ab8b4; }
```

Insert these new rules immediately after that line:
```css
.progress-task.locked label { color: #9ab8b4; cursor: not-allowed; }
.progress-task.locked .progress-check { cursor: not-allowed; opacity: 0.5; }
.asset-list {
  margin: 2px 0 8px 26px;
  display: flex; flex-direction: column; gap: 4px;
}
.asset-chip {
  display: inline-flex; align-items: center; gap: 6px;
  background: var(--green-light); color: var(--teal-dark);
  border: 1px solid var(--green-border); border-radius: 6px;
  padding: 3px 8px; font-size: 12px;
  align-self: flex-start;
  max-width: 100%;
}
.asset-chip a { color: var(--teal-dark); text-decoration: underline; }
.asset-chip a:hover { color: var(--green-dark); }
.asset-delete {
  background: none; border: none; color: var(--muted);
  cursor: pointer; padding: 0 2px; font-size: 13px; line-height: 1;
}
.asset-delete:hover { color: #c2410c; }
.asset-add {
  font-size: 12px; color: var(--green-dark);
  cursor: pointer; align-self: flex-start;
  padding: 2px 0;
}
.asset-add:hover { color: var(--teal-dark); text-decoration: underline; }
.asset-form {
  display: flex; gap: 6px; align-items: center; flex-wrap: wrap;
  padding: 4px 0;
}
.asset-form input {
  font-size: 12px; padding: 4px 6px;
  border: 1px solid var(--border); border-radius: 4px;
  font-family: inherit;
}
.asset-form input[data-asset-name] { flex: 1; min-width: 160px; }
.asset-form input[data-asset-url] { flex: 2; min-width: 200px; }
.asset-form button {
  font-size: 12px; padding: 4px 10px;
  border: 1px solid var(--green-border); border-radius: 4px;
  background: var(--green); color: white; cursor: pointer;
}
.asset-form button.cancel {
  background: white; color: var(--muted);
}
.asset-form button:hover { filter: brightness(0.95); }
```

- [ ] **Step 2: Hide asset rows in print view**

Find this line in the existing `@media print` block:
```css
.topbar, .sidebar, .setup-overlay, .toast { display: none !important; }
```

Replace with:
```css
.topbar, .sidebar, .setup-overlay, .toast, .asset-form, .asset-add, .asset-delete { display: none !important; }
```

(Asset chips stay visible in print as a read-only record. Add/delete controls are hidden.)

- [ ] **Step 3: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: add CSS for asset chips, add form, and locked task state"
```

---

### Task 2: Core JS — data model, render, form, save/delete

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`

- [ ] **Step 1: Add core asset functions**

Find this function (around line 1254):
```javascript
function isValidHttpUrl(value) {
```

Insert these new functions immediately **before** it:

```javascript
function ensureAssetsModel() {
  if (!projectData) projectData = {};
  if (!projectData.assets) projectData.assets = { 1: {}, 2: {}, 3: {}, 4: {} };
  for (const p of [1, 2, 3, 4]) {
    if (!projectData.assets[p]) projectData.assets[p] = {};
  }
}

function escapeHtml(s) {
  return String(s).replace(/[&<>"']/g, c => ({
    '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
  }[c]));
}

function saveProjectData() {
  try { localStorage.setItem('crispAssessment', JSON.stringify(projectData)); }
  catch { console.warn('localStorage unavailable; state will not persist.'); }
}

function getAssets(phase, task) {
  ensureAssetsModel();
  if (!projectData.assets[phase][task]) projectData.assets[phase][task] = [];
  return projectData.assets[phase][task];
}

function injectAssetLists() {
  document.querySelectorAll('.progress-task').forEach(row => {
    if (row.nextElementSibling && row.nextElementSibling.classList.contains('asset-list')) return;
    const phase = row.dataset.phase;
    const task = row.dataset.task;
    if (phase == null || task == null) return;
    const list = document.createElement('div');
    list.className = 'asset-list';
    list.dataset.assetPhase = phase;
    list.dataset.assetTask = task;
    row.parentNode.insertBefore(list, row.nextSibling);
  });
}

function renderAssetList(phase, task) {
  const list = document.querySelector(`.asset-list[data-asset-phase="${phase}"][data-asset-task="${task}"]`);
  if (!list) return;
  const assets = getAssets(phase, task);
  const chips = assets.map((a, i) => {
    const safeName = escapeHtml(a.name);
    const safeUrl = a.url ? escapeHtml(a.url) : '';
    const nameHtml = safeUrl
      ? `<a href="${safeUrl}" target="_blank" rel="noopener noreferrer">${safeName}</a>`
      : safeName;
    return `<div class="asset-chip"><span>📄 ${nameHtml}</span>` +
           `<button class="asset-delete" onclick="deleteAsset(${phase}, ${task}, ${i})" aria-label="Remove asset">✕</button></div>`;
  }).join('');
  list.innerHTML = chips +
    `<a class="asset-add" onclick="openAssetForm(${phase}, ${task})">+ Add asset</a>`;
}

function refreshAllAssetLists() {
  ensureAssetsModel();
  injectAssetLists();
  document.querySelectorAll('.progress-task').forEach(row => {
    const phase = Number(row.dataset.phase);
    const task = Number(row.dataset.task);
    if (Number.isInteger(phase) && Number.isInteger(task)) renderAssetList(phase, task);
  });
  refreshProgressCheckboxes();
}

function refreshProgressCheckboxes() {
  ensureAssetsModel();
  document.querySelectorAll('.progress-task').forEach(row => {
    const phase = Number(row.dataset.phase);
    const task = Number(row.dataset.task);
    const checkbox = row.querySelector('.progress-check');
    if (!checkbox) return;
    const assets = getAssets(phase, task);
    const isChecked = checkbox.checked;
    if (assets.length === 0 && !isChecked) {
      checkbox.disabled = true;
      row.classList.add('locked');
      row.title = 'Add an asset first';
    } else {
      checkbox.disabled = false;
      row.classList.remove('locked');
      row.removeAttribute('title');
    }
  });
}

function openAssetForm(phase, task) {
  const list = document.querySelector(`.asset-list[data-asset-phase="${phase}"][data-asset-task="${task}"]`);
  if (!list) return;
  const addLink = list.querySelector('.asset-add');
  if (!addLink) return;
  const form = document.createElement('div');
  form.className = 'asset-form';
  form.innerHTML =
    `<input type="text" placeholder="File name (e.g., questionnaire.docx)" data-asset-name>` +
    `<input type="url" placeholder="https:// (optional)" data-asset-url>` +
    `<button onclick="saveAsset(${phase}, ${task})">Save</button>` +
    `<button class="cancel" onclick="cancelAssetForm(${phase}, ${task})">Cancel</button>`;
  list.replaceChild(form, addLink);
  form.querySelector('[data-asset-name]').focus();
}

function cancelAssetForm(phase, task) {
  renderAssetList(phase, task);
}

function saveAsset(phase, task) {
  const list = document.querySelector(`.asset-list[data-asset-phase="${phase}"][data-asset-task="${task}"]`);
  if (!list) return;
  const form = list.querySelector('.asset-form');
  if (!form) return;
  const name = form.querySelector('[data-asset-name]').value.trim();
  const url = form.querySelector('[data-asset-url]').value.trim();
  if (!name) {
    alert('File name is required.');
    form.querySelector('[data-asset-name]').focus();
    return;
  }
  if (url && !isValidHttpUrl(url)) {
    alert('URL must be a valid https:// link, or leave it blank.');
    form.querySelector('[data-asset-url]').focus();
    return;
  }
  const assets = getAssets(phase, task);
  assets.push({ name, url });
  saveProjectData();
  renderAssetList(phase, task);
  refreshProgressCheckboxes();
}

function deleteAsset(phase, task, index) {
  const assets = getAssets(phase, task);
  if (index < 0 || index >= assets.length) return;
  assets.splice(index, 1);
  saveProjectData();
  if (assets.length === 0) {
    const row = document.querySelector(`.progress-task[data-phase="${phase}"][data-task="${task}"]`);
    const checkbox = row && row.querySelector('.progress-check');
    if (checkbox && checkbox.checked) {
      checkbox.checked = false;
      row.classList.remove('done');
      if (!projectData.progress) projectData.progress = { 1: {}, 2: {}, 3: {}, 4: {} };
      projectData.progress[phase][task] = false;
      updateProgress(phase);
    }
  }
  renderAssetList(phase, task);
  refreshProgressCheckboxes();
}

```

- [ ] **Step 2: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: add asset model, render, form, save/delete JS helpers"
```

---

### Task 3: Wire into project lifecycle + verify

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`

- [ ] **Step 1: Initialize assets and refresh on launch**

Find this line in `launchProject()`:
```javascript
  projectData = { client, lead, driveLink, features: {...features}, progress: {1:{},2:{},3:{},4:{}} };
```

Replace with:
```javascript
  projectData = { client, lead, driveLink, features: {...features}, progress: {1:{},2:{},3:{},4:{}}, assets: {1:{},2:{},3:{},4:{}} };
```

Then find this block at the end of `launchProject()`:
```javascript
  refreshDriveButtons();
  nav('overview');
}
```

Replace with:
```javascript
  refreshDriveButtons();
  refreshAllAssetLists();
  nav('overview');
}
```

- [ ] **Step 2: Refresh assets on resume**

Find this block in `resumeProject()`:
```javascript
  refreshDriveButtons();
  nav('overview');
}
```

Replace with:
```javascript
  refreshDriveButtons();
  refreshAllAssetLists();
  nav('overview');
}
```

- [ ] **Step 3: Refresh assets in newProject**

Find this block at the end of `newProject()`:
```javascript
  document.getElementById('setupModal').classList.remove('hidden');
  projectData = {};
  refreshDriveButtons();
}
```

Replace with:
```javascript
  document.getElementById('setupModal').classList.remove('hidden');
  projectData = {};
  refreshDriveButtons();
  refreshAllAssetLists();
}
```

- [ ] **Step 4: Inject asset lists on initial page load**

Find this block:
```javascript
window.onload = function() {
  const saved = localStorage.getItem('crispAssessment');
  if (saved) document.getElementById('resumeLink').classList.remove('hidden');
};
```

Replace with:
```javascript
window.onload = function() {
  const saved = localStorage.getItem('crispAssessment');
  if (saved) document.getElementById('resumeLink').classList.remove('hidden');
  injectAssetLists();
  refreshProgressCheckboxes();
};
```

- [ ] **Step 5: Verify in browser — empty state**

Hard-refresh `http://localhost:7788/Assessments%20Templates/Crisp-Assessment-AI-Process-Plan.html` (Ctrl+Shift+R).

Click "+ New Project" if a project is loaded. Enter Client Name "AssetTest" + Lead "Tester", click Launch. Navigate to Phase 1.

Expected:
- Below each of the 5 key activities is a small "+ Add asset" link
- All 5 checkboxes are visibly disabled (dimmed) and unclickable
- Hovering a disabled checkbox row shows the tooltip "Add an asset first"

- [ ] **Step 6: Verify in browser — add and check**

Click "+ Add asset" under "Client completes questionnaire".

Expected: an inline form appears with two inputs ("File name" + "https://...") and Save/Cancel buttons. Name input has focus.

Click Save with empty name.

Expected: alert "File name is required.", form stays open.

Type "Acme_Questionnaire_v2.docx" in name, leave URL blank, click Save.

Expected:
- Form replaced with a chip: 📄 Acme_Questionnaire_v2.docx ✕
- "+ Add asset" link is back below the chip
- The checkbox for that activity is now enabled (not dimmed)

Check the box.

Expected: line-through on the label (existing behavior), no errors.

- [ ] **Step 7: Verify in browser — add with URL and click**

Click "+ Add asset" under "Stakeholders & sample files identified".

Type "Stakeholder list" + URL "https://drive.google.com/file/d/TEST", click Save.

Expected: chip 📄 [Stakeholder list] (clickable link) ✕.

Click the chip's name link.

Expected: opens in new tab.

- [ ] **Step 8: Verify in browser — URL validation**

Click "+ Add asset" under any activity. Enter name "Test" and URL `javascript:alert(1)`. Click Save.

Expected: alert "URL must be a valid https:// link, or leave it blank.", form stays open, focus moves to URL field. No save happens.

- [ ] **Step 9: Verify in browser — auto-uncheck on last asset delete**

On the activity you checked in Step 6 (which now has 1 asset and is checked), click the ✕ on its chip.

Expected:
- Chip disappears
- Checkbox auto-unchecks
- Line-through is removed
- Checkbox is now disabled again

- [ ] **Step 10: Verify in browser — persistence**

Add an asset back to one activity. Check the box. Refresh the page (Ctrl+R, not Shift+R). Click Resume.

Expected: chip restored, checkbox checked, sidebar counter still shows "1/5".

- [ ] **Step 11: Verify in browser — backward compatibility**

Open browser DevTools → Application → Local Storage. Find the `crispAssessment` key. Edit the value to remove the `"assets":...` field entirely (leave the rest of the JSON intact). Hard-refresh and Resume.

Expected: page loads without errors. All checkboxes that were checked stay checked. Activities with no assets show "+ Add asset" + are enabled (since they were already checked — per spec migration rule).

- [ ] **Step 12: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: wire asset lifecycle into project launch/resume/new and page load"
```
