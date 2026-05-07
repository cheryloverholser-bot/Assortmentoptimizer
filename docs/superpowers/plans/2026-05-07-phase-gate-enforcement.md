# Phase Gate Enforcement Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Block navigation to a later phase until all 5 key activity checkboxes in the current phase are checked, showing an amber toast when the gate blocks navigation.

**Architecture:** Three targeted edits to the single HTML file: (1) add toast CSS + HTML, (2) remove `hidden` from all 20 checkboxes and strip the checkbox-visibility line from `applyFeatures()`, (3) add `isPhaseComplete()` + `showToast()` + `hideToast()` helpers and wire gate logic into `nav()`.

**Tech Stack:** Vanilla JS, CSS, single self-contained HTML file — no build step, no dependencies.

---

## File Structure

**Only file modified:**
- `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`
  - Line 239: insert toast CSS before `</style>`
  - Line 334: insert toast HTML inside `#mainContent`
  - Lines 415–419, 486–490, 558–562, 627–631: remove `hidden` from 20 checkbox `class` attributes
  - Lines 1118–1119: delete the `.progress-check` toggle from `applyFeatures()`
  - Line 1128 (before `function nav`): insert `isPhaseComplete`, `showToast`, `hideToast`
  - Lines 1129–1138: insert gate check at top of `nav()`

**Verify with:** `http://localhost:7788/Assessments%20Templates/Crisp-Assessment-AI-Process-Plan.html`
(server already running via `npx http-server . -p 7788 --cors -c-1`)

---

### Task 1: Toast component — CSS and HTML

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html` (lines 239, 334)

- [ ] **Step 1: Insert toast CSS before `</style>` at line 239**

Find this exact line (line 239):
```
</style>
```

Replace with:
```html
.toast {
  position: sticky;
  top: 0;
  z-index: 200;
  background: #d97706;
  color: white;
  padding: 10px 16px;
  font-size: 13px;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  border-radius: 0 0 8px 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.18);
  animation: toastSlideIn 0.2s ease;
}
.toast.hidden { display: none !important; }
.toast-close {
  cursor: pointer; font-size: 18px; line-height: 1;
  background: none; border: none; color: white; padding: 0;
}
@keyframes toastSlideIn {
  from { transform: translateY(-100%); opacity: 0; }
  to   { transform: translateY(0);     opacity: 1; }
}
</style>
```

- [ ] **Step 2: Insert toast HTML inside `#mainContent` after the searchResults div (line 334)**

Find this exact line (line 334):
```html
    <div id="searchResults" class="hidden"></div>
```

Replace with:
```html
    <div id="searchResults" class="hidden"></div>
    <div id="toast" class="toast hidden">
      <span id="toastMsg"></span>
      <button class="toast-close" onclick="hideToast()">×</button>
    </div>
```

- [ ] **Step 3: Verify toast CSS renders correctly in browser**

Open `http://localhost:7788/Assessments%20Templates/Crisp-Assessment-AI-Process-Plan.html` (hard-refresh with Ctrl+Shift+R).

In the browser console (F12 → Console), run:
```javascript
document.getElementById('toast').classList.remove('hidden');
```

Expected: An amber bar appears at the top of the main content area with an × button. It should not displace the page content below — it overlaps sticky-style.

Then run:
```javascript
document.getElementById('toast').classList.add('hidden');
```

Expected: Bar disappears.

- [ ] **Step 4: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: add toast component CSS and HTML for gate enforcement"
```

---

### Task 2: Make key activity checkboxes always visible

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html` (lines 415–419, 486–490, 558–562, 627–631, 1118–1119)

- [ ] **Step 1: Remove `hidden` from all 20 checkbox class attributes**

In the file, every key activity checkbox currently reads:
```html
class="progress-check hidden"
```

There are exactly 20 occurrences (5 per phase × 4 phases). Use find-and-replace-all to change:

**Find:** `class="progress-check hidden"`
**Replace with:** `class="progress-check"`

Verify the count: after the replace, searching for `class="progress-check hidden"` should return 0 results.

- [ ] **Step 2: Remove the checkbox visibility toggle from `applyFeatures()`**

Find this exact 2-line block (lines 1118–1119 after Task 1 edits — line numbers may shift slightly; search by content):
```javascript
  document.querySelectorAll('.progress-check').forEach(cb =>
    cb.classList.toggle('hidden', !features.progress));
```

Delete those two lines entirely. The `applyFeatures()` function should now start:
```javascript
function applyFeatures() {
  [1,2,3,4].forEach(p => {
    const el = document.getElementById(`prog${p}`);
    if (el) el.classList.toggle('hidden', !features.progress);
  });
  document.getElementById('printBtn')?.classList.toggle('hidden', !features.print);
  document.getElementById('searchWrap')?.classList.toggle('hidden', !features.search);
  document.getElementById('changelogSection')?.classList.toggle('hidden', !features.changelog);
}
```

- [ ] **Step 3: Verify checkboxes are visible without enabling Progress Tracker**

Hard-refresh the page (Ctrl+Shift+R).

Complete the setup modal (enter any client name and engagement lead, click Launch). Do NOT enable the Progress Tracker toggle.

Expected: On Phase 1 page, all 5 key activity checkboxes are visible and clickable.
Expected: Checking a checkbox still strikes through the label text (the `.done` class still applies).
Expected: The sidebar progress counters (e.g., "2/5") are still hidden — they are controlled separately by the Progress Tracker feature.

- [ ] **Step 4: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: make key activity checkboxes always visible regardless of Progress Tracker setting"
```

---

### Task 3: Gate logic — isPhaseComplete, toast functions, nav() update

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html` (before `function nav(`, and inside `function nav(`)

- [ ] **Step 1: Insert helper functions before `function nav(`**

Find this exact line (search by content):
```javascript
function nav(page) {
```

Insert the following three functions immediately before it:
```javascript
function isPhaseComplete(phase) {
  const checks = document.querySelectorAll(`[data-phase="${phase}"] .progress-check`);
  return checks.length > 0 && Array.from(checks).every(cb => cb.checked);
}

let _toastTimer = null;
function showToast(msg) {
  const t = document.getElementById('toast');
  if (!t) return;
  document.getElementById('toastMsg').textContent = msg;
  t.classList.remove('hidden');
  clearTimeout(_toastTimer);
  _toastTimer = setTimeout(hideToast, 3000);
}
function hideToast() {
  document.getElementById('toast')?.classList.add('hidden');
}

```

- [ ] **Step 2: Add gate check at the top of `nav()`**

The current `nav()` function reads:
```javascript
function nav(page) {
  document.getElementById('searchResults')?.classList.add('hidden');
  document.querySelectorAll('.page-section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('.nav-item, .nav-link').forEach(i => i.classList.remove('active'));
  const section = document.getElementById('page-' + page);
  if (section) section.classList.add('active');
  const navEl = document.getElementById('nav-' + page);
  if (navEl) navEl.classList.add('active');
  document.getElementById('mainContent').scrollTop = 0;
}
```

Replace it with:
```javascript
function nav(page) {
  const phaseGates = { phase2: 1, phase3: 2, phase4: 3 };
  if (phaseGates[page] !== undefined && !isPhaseComplete(phaseGates[page])) {
    showToast('Complete all key activities in this phase before moving on.');
    return;
  }
  document.getElementById('searchResults')?.classList.add('hidden');
  document.querySelectorAll('.page-section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('.nav-item, .nav-link').forEach(i => i.classList.remove('active'));
  const section = document.getElementById('page-' + page);
  if (section) section.classList.add('active');
  const navEl = document.getElementById('nav-' + page);
  if (navEl) navEl.classList.add('active');
  document.getElementById('mainContent').scrollTop = 0;
}
```

- [ ] **Step 3: Verify gate blocks navigation with 0/5 checked**

Hard-refresh (Ctrl+Shift+R). Complete setup modal. Navigate to Phase 1.

Click the **Phase 2 (Discovery)** nav link in the sidebar with 0 activities checked.

Expected:
- Page stays on Phase 1
- Amber toast appears at top of content area: *"Complete all key activities in this phase before moving on."*
- Toast auto-disappears after ~3 seconds
- Clicking × on toast dismisses it immediately

- [ ] **Step 4: Verify gate allows navigation with 5/5 checked**

Check all 5 Phase 1 checkboxes.

Click the **Phase 2 (Discovery)** nav link.

Expected: Page navigates to Phase 2 successfully. No toast appears.

- [ ] **Step 5: Verify gate blocks Phase 2 → Phase 3 independently**

On Phase 2, with 0 activities checked, click **Phase 3 (Analysis)** nav link.

Expected: Page stays on Phase 2. Toast appears.

Check all 5 Phase 2 activities, click Phase 3 nav link.

Expected: Navigation succeeds.

- [ ] **Step 6: Verify backward navigation is always allowed**

From Phase 3 (with Phase 3 activities unchecked), click **Phase 1 (Intake)** nav link.

Expected: Navigation succeeds — no gate, no toast.

- [ ] **Step 7: Verify non-phase navigation is always allowed**

From Phase 1 with 0 activities checked, click **Overview**, **AI Prompt Library**, **RACI Quick Reference**, and **Decision Gates** nav links.

Expected: All navigate successfully — no gate, no toast.

- [ ] **Step 8: Verify toast re-fires on repeated blocked attempts**

From Phase 1 with 0 activities checked, click Phase 2 nav link twice in quick succession (within 3 seconds).

Expected: Toast resets its 3-second countdown and stays visible without flickering.

- [ ] **Step 9: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: enforce phase gate — require all key activities complete before advancing to next phase"
```
