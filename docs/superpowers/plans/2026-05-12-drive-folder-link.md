# Project Folder Link Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a per-engagement Google Drive folder link, captured in setup or set later, with an "Open Project Folder" button on all 4 phase pages.

**Architecture:** Two changes to the single HTML file: (1) add a Drive link input to the setup modal and persist it to `projectData.driveLink`; (2) render a Drive button on each phase page that opens the link in a new tab when set, or opens a prompt to set the URL when unset.

**Tech Stack:** Vanilla JS, CSS, single self-contained HTML file.

---

## File Structure

**Only file modified:**
- `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`
  - Line 280 area: insert new form field for Drive link
  - Lines 443, 514, 586, 655: insert Drive button before each Key Activities card
  - Add CSS for `.drive-link-btn`
  - `launchProject()`, `resumeProject()`, `newProject()`: wire in driveLink + button refresh
  - Add helpers `openOrSetDriveLink()`, `refreshDriveButtons()`

**Verify with:** `http://localhost:7788/Assessments%20Templates/Crisp-Assessment-AI-Process-Plan.html`

---

### Task 1: Setup modal input + storage

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`

- [ ] **Step 1: Add the Drive link input to the setup modal**

Find this exact block (lines ~277–280):
```html
    <div class="form-field">
      <label>Engagement Lead</label>
      <input type="text" id="engagementLead" placeholder="Your name" />
    </div>
```

Replace with:
```html
    <div class="form-field">
      <label>Engagement Lead</label>
      <input type="text" id="engagementLead" placeholder="Your name" />
    </div>
    <div class="form-field">
      <label>Google Drive Folder Link <span style="opacity:0.6; font-weight:400;">(optional)</span></label>
      <input type="url" id="driveLink" placeholder="https://drive.google.com/drive/folders/..." />
    </div>
```

- [ ] **Step 2: Save `driveLink` in `launchProject()`**

Find the current `launchProject()` function (lines ~1116–1128):
```javascript
function launchProject() {
  const client = document.getElementById('clientName').value.trim();
  if (!client) { document.getElementById('clientName').focus(); return; }
  const lead = document.getElementById('engagementLead').value.trim();
  if (!lead) { document.getElementById('engagementLead').focus(); return; }
  projectData = { client, lead, features: {...features}, progress: {1:{},2:{},3:{},4:{}} };
  try { localStorage.setItem('crispAssessment', JSON.stringify(projectData)); }
  catch { console.warn('localStorage unavailable; project state will not persist.'); }
  applyFeatures();
  document.getElementById('clientDisplay').textContent = client;
  document.getElementById('setupModal').classList.add('hidden');
  nav('overview');
}
```

Replace with:
```javascript
function launchProject() {
  const client = document.getElementById('clientName').value.trim();
  if (!client) { document.getElementById('clientName').focus(); return; }
  const lead = document.getElementById('engagementLead').value.trim();
  if (!lead) { document.getElementById('engagementLead').focus(); return; }
  const driveLink = document.getElementById('driveLink').value.trim();
  projectData = { client, lead, driveLink, features: {...features}, progress: {1:{},2:{},3:{},4:{}} };
  try { localStorage.setItem('crispAssessment', JSON.stringify(projectData)); }
  catch { console.warn('localStorage unavailable; project state will not persist.'); }
  applyFeatures();
  document.getElementById('clientDisplay').textContent = client;
  document.getElementById('setupModal').classList.add('hidden');
  refreshDriveButtons();
  nav('overview');
}
```

(`refreshDriveButtons` is defined in Task 2 — this is a forward reference but safe because the function exists by the time the user clicks Launch.)

- [ ] **Step 3: Clear the Drive link input in `newProject()`**

Find this line inside `newProject()` (line ~1156):
```javascript
  document.getElementById('engagementLead').value = '';
```

Replace with:
```javascript
  document.getElementById('engagementLead').value = '';
  document.getElementById('driveLink').value = '';
```

- [ ] **Step 4: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: add Google Drive folder link field to project setup"
```

---

### Task 2: Drive button on phase pages — HTML, CSS, helpers, wire-up

**Files:**
- Modify: `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`

- [ ] **Step 1: Add CSS for the Drive button**

Find this line in the stylesheet (near `.gate-pill` and `.gate-items` rules, after line 187):
```
.progress-task { display: flex; align-items: center; gap: 8px; padding: 4px 0; }
```

Insert these new rules immediately before that line:
```css
.drive-link-btn {
  display: flex; align-items: center; gap: 8px;
  background: var(--card); color: var(--teal-dark);
  border: 1px solid var(--green-border); border-left: 4px solid var(--green);
  border-radius: 10px; padding: 10px 14px;
  font-size: 13px; font-weight: 600;
  cursor: pointer; width: 100%;
  margin-bottom: 14px;
  text-decoration: none;
  transition: background 0.15s ease;
}
.drive-link-btn:hover { background: var(--green-light); }
.drive-link-btn.unset {
  border-left-style: dashed; color: var(--muted); font-weight: 500;
}
.drive-link-btn .drive-icon { font-size: 16px; }
```

- [ ] **Step 2: Insert Drive button into Phase 1 page**

Find this exact block (lines ~442–444):
```html
    <div class="card">
      <div class="card-label">Key Activities</div>
```

Replace with:
```html
    <button class="drive-link-btn" data-drive-btn onclick="openOrSetDriveLink()">
      <span class="drive-icon">📁</span><span data-drive-label>Set Project Folder Link</span>
    </button>
    <div class="card">
      <div class="card-label">Key Activities</div>
```

- [ ] **Step 3: Insert Drive button into Phase 2 page**

Find this exact block (lines ~513–515, just before Phase 2's Key Activities card):
```html
    <div class="card">
      <div class="card-label">Key Activities</div>
      <div class="progress-task" data-phase="2" data-task="0">
```

Replace with:
```html
    <button class="drive-link-btn" data-drive-btn onclick="openOrSetDriveLink()">
      <span class="drive-icon">📁</span><span data-drive-label>Set Project Folder Link</span>
    </button>
    <div class="card">
      <div class="card-label">Key Activities</div>
      <div class="progress-task" data-phase="2" data-task="0">
```

- [ ] **Step 4: Insert Drive button into Phase 3 page**

Find this exact block (lines ~585–587, just before Phase 3's Key Activities card):
```html
    <div class="card">
      <div class="card-label">Key Activities</div>
      <div class="progress-task" data-phase="3" data-task="0">
```

Replace with:
```html
    <button class="drive-link-btn" data-drive-btn onclick="openOrSetDriveLink()">
      <span class="drive-icon">📁</span><span data-drive-label>Set Project Folder Link</span>
    </button>
    <div class="card">
      <div class="card-label">Key Activities</div>
      <div class="progress-task" data-phase="3" data-task="0">
```

- [ ] **Step 5: Insert Drive button into Phase 4 page**

Find this exact block (lines ~654–656, just before Phase 4's Key Activities card):
```html
    <div class="card">
      <div class="card-label">Key Activities</div>
      <div class="progress-task" data-phase="4" data-task="0">
```

Replace with:
```html
    <button class="drive-link-btn" data-drive-btn onclick="openOrSetDriveLink()">
      <span class="drive-icon">📁</span><span data-drive-label>Set Project Folder Link</span>
    </button>
    <div class="card">
      <div class="card-label">Key Activities</div>
      <div class="progress-task" data-phase="4" data-task="0">
```

- [ ] **Step 6: Add `openOrSetDriveLink()` and `refreshDriveButtons()` helpers**

Find the existing `applyFeatures()` function (search for `function applyFeatures()`). Insert these two new functions immediately before `function applyFeatures()`:

```javascript
function refreshDriveButtons() {
  const hasLink = !!(projectData && projectData.driveLink);
  document.querySelectorAll('[data-drive-btn]').forEach(btn => {
    btn.classList.toggle('unset', !hasLink);
    const label = btn.querySelector('[data-drive-label]');
    if (label) label.textContent = hasLink ? 'Open Project Folder' : 'Set Project Folder Link';
  });
}

function openOrSetDriveLink() {
  if (projectData && projectData.driveLink) {
    window.open(projectData.driveLink, '_blank', 'noopener');
    return;
  }
  const url = window.prompt('Paste the Google Drive folder URL:');
  if (!url) return;
  const trimmed = url.trim();
  if (!trimmed) return;
  if (!projectData) projectData = {};
  projectData.driveLink = trimmed;
  try { localStorage.setItem('crispAssessment', JSON.stringify(projectData)); }
  catch { console.warn('localStorage unavailable; drive link will not persist.'); }
  refreshDriveButtons();
}

```

- [ ] **Step 7: Call `refreshDriveButtons()` in `resumeProject()` and `newProject()`**

Find this line inside `resumeProject()` (line ~1151):
```javascript
  nav('overview');
}
```

In the `resumeProject` function specifically (not in `launchProject`), insert the refresh call before `nav('overview')`:
```javascript
  refreshDriveButtons();
  nav('overview');
}
```

(`launchProject` already has this call from Task 1 Step 2.)

Then find the end of `newProject()` (line ~1166):
```javascript
  document.getElementById('setupModal').classList.remove('hidden');
}
```

Replace with:
```javascript
  document.getElementById('setupModal').classList.remove('hidden');
  refreshDriveButtons();
}
```

- [ ] **Step 8: Verify behavior in the browser**

Hard-refresh `http://localhost:7788/Assessments%20Templates/Crisp-Assessment-AI-Process-Plan.html` (Ctrl+Shift+R).

Test sequence:

1. Click "+ New Project" if a project is already loaded. Setup modal opens.
2. Enter Client Name "Test Co" and Engagement Lead "Tester". Leave Drive link blank. Click Launch.
3. Check all 5 Phase 1 activities, navigate to Phase 1.
   - Expected: A dashed button reads "📁 Set Project Folder Link" above the Key Activities card.
4. Click the button. A prompt appears asking for the URL. Enter `https://drive.google.com/drive/folders/TEST123`.
   - Expected: Button text changes to "📁 Open Project Folder" (no dashed border).
5. Navigate to Phase 2, 3, 4.
   - Expected: Each shows the solid "📁 Open Project Folder" button at the same position.
6. Click the button on Phase 2.
   - Expected: A new browser tab opens to the URL.
7. Hard-refresh the page. Click Resume.
   - Expected: All buttons show "📁 Open Project Folder" — link persisted.
8. Click "+ New Project". Setup modal opens with empty fields. Enter Client Name "Test 2", Lead "Tester", **and** Drive link `https://drive.google.com/drive/folders/SAVED`. Launch.
   - Expected: Phase 1–4 all show "📁 Open Project Folder" from the start. Clicking opens the saved URL.

- [ ] **Step 9: Commit**

```bash
git add "Assessments Templates/Crisp-Assessment-AI-Process-Plan.html"
git commit -m "feat: add Open/Set Project Folder button to all phase pages"
```
