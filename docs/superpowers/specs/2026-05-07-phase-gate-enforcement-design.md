# Phase Gate Enforcement — Design Spec

**Date:** 2026-05-07
**Feature:** Require all key activities in a phase to be completed before navigating to the next phase.

---

## Goal

Prevent users from advancing to a later phase until all 5 key activity checkboxes in the current phase are checked. Applies regardless of whether the Progress Tracker feature is enabled.

---

## Architecture

Three targeted changes to `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`:

1. **Checkbox visibility** — Remove `hidden` class from all 20 `.progress-check` inputs so they are always visible. Remove the code that toggles checkbox visibility based on `features.progress`.
2. **Gate logic in `nav()`** — Before navigating to a phase page, check that all checkboxes in the preceding phase are checked. If not, show a toast and return early.
3. **Toast component** — New CSS + JS for a dismissible amber warning bar at the top of `#mainContent`.

---

## Components

### 1. Checkbox Visibility

- **Change:** Remove `class="progress-check hidden"` → `class="progress-check"` on all 20 checkbox inputs (phases 1–4, tasks 0–4).
- **Change:** Remove any JS that calls `.classList.add('hidden')` or `.classList.remove('hidden')` on `.progress-check` elements.
- **No change:** `features.progress` still controls the sidebar progress counters (`prog1`–`prog4`).

### 2. Gate Logic

New helper function `isPhaseComplete(phase)`:
```javascript
function isPhaseComplete(phase) {
  const checks = document.querySelectorAll(`[data-phase="${phase}"] .progress-check`);
  return Array.from(checks).every(cb => cb.checked);
}
```

Modified `nav(page)` — insert gate check at the top:
```javascript
// Gate: forward phase navigation requires prior phase complete
const phaseOrder = { phase2: 1, phase3: 2, phase4: 3 };
if (phaseOrder[page] !== undefined) {
  const requiredPhase = phaseOrder[page];
  if (!isPhaseComplete(requiredPhase)) {
    showToast('Complete all key activities in this phase before moving on.');
    return;
  }
}
```

Rules:
- Navigating to `phase2` → phase 1 must be complete
- Navigating to `phase3` → phase 2 must be complete
- Navigating to `phase4` → phase 3 must be complete
- Backward navigation (any phase to a lower-numbered phase) → always allowed
- Navigation to `overview`, `prompts`, `raci`, `gates` → always allowed
- Phase 4 has no outbound gate

### 3. Toast Component

**CSS** (added to the stylesheet):
```css
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
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  animation: toastSlideIn 0.2s ease;
}
.toast.hidden { display: none; }
.toast-close { cursor: pointer; font-size: 16px; line-height: 1; background: none; border: none; color: white; }
@keyframes toastSlideIn {
  from { transform: translateY(-100%); opacity: 0; }
  to   { transform: translateY(0);    opacity: 1; }
}
```

**HTML** (inside `#mainContent`, before all `.page-section` divs):
```html
<div id="toast" class="toast hidden">
  <span id="toastMsg"></span>
  <button class="toast-close" onclick="hideToast()">×</button>
</div>
```

**JS**:
```javascript
let toastTimer = null;
function showToast(msg) {
  const t = document.getElementById('toast');
  document.getElementById('toastMsg').textContent = msg;
  t.classList.remove('hidden');
  clearTimeout(toastTimer);
  toastTimer = setTimeout(hideToast, 3000);
}
function hideToast() {
  document.getElementById('toast')?.classList.add('hidden');
}
```

---

## Data Flow

```
User clicks nav link → nav(page) called
  → Is target phase2/3/4? 
      YES → isPhaseComplete(requiredPhase)?
                NO  → showToast() → return (no navigation)
                YES → proceed with normal nav()
      NO  → proceed with normal nav()
```

---

## Testing

- [ ] Phase 1 with 0/5 checked → clicking Phase 2 nav shows toast, stays on Phase 1
- [ ] Phase 1 with 4/5 checked → clicking Phase 2 nav shows toast, stays on Phase 1
- [ ] Phase 1 with 5/5 checked → clicking Phase 2 nav navigates successfully
- [ ] Phase 2 complete → Phase 3 nav works; Phase 3 incomplete → Phase 4 nav blocked
- [ ] Clicking Phase 1 nav from Phase 2 (backward) → always works
- [ ] Overview, Prompts, RACI, Gates nav → always works regardless of phase state
- [ ] Toast auto-dismisses after 3 seconds
- [ ] Toast × button dismisses immediately
- [ ] Toast re-appears correctly on repeated blocked attempts
- [ ] Checkboxes visible without enabling Progress Tracker
- [ ] Progress Tracker sidebar counters still work independently

---

## Out of Scope

- Gating navigation to non-phase pages (overview, prompts, raci, gates)
- Listing specific incomplete items in the toast message
- Locking/graying out future nav links proactively
- Any changes to the gate card visual content
