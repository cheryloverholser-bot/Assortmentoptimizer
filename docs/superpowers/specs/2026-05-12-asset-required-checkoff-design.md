# Asset-Required Check-Off — Design Spec

**Date:** 2026-05-12
**Feature:** Require users to log at least one asset (filename + optional URL) per key activity before they can check the activity complete.

---

## Goal

Force consultants to record what they've uploaded to the project folder for each key activity, providing a lightweight audit trail and preventing premature check-off. Honor-system only — the page does not verify the asset exists in Drive.

---

## Architecture

Three changes to `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`:

1. **Data model** — add `projectData.assets` keyed by `phase` then `task index`, each value is an array of `{name, url}` objects.
2. **UI per activity** — beneath each of the 20 `.progress-task` rows, render an expandable asset list with a "+ Add asset" trigger and per-row delete buttons.
3. **Checkbox gating** — disable each `.progress-check` until ≥1 asset is logged for that activity; auto-uncheck when the last asset is deleted from a checked task.

---

## Components

### 1. Data Model

```javascript
projectData.assets = {
  1: { 0: [{name, url}], 1: [...], 2: [...], 3: [...], 4: [...] },
  2: { ... },
  3: { ... },
  4: { ... }
};
```

- `phase` keys: 1–4
- `task` keys: 0–4 (one per activity, mirroring existing `data-task` attribute)
- Each asset: `{name: string (required), url: string (optional, must be https:// if present)}`

Persisted to `localStorage` alongside the rest of `projectData` on every change.

### 2. UI

**Below each `.progress-task` row** (existing markup unchanged), insert a sibling `.asset-list` container:

```html
<div class="asset-list" data-asset-phase="1" data-asset-task="0">
  <!-- one .asset-chip per saved asset, rendered by JS -->
  <a class="asset-add" onclick="openAssetForm(1, 0)">+ Add asset</a>
</div>
```

**Rendered chip** (per saved asset):
```html
<div class="asset-chip">
  <span>📄 <a href="URL" target="_blank" rel="noopener noreferrer">name</a></span>
  <button class="asset-delete" onclick="deleteAsset(1, 0, INDEX)" aria-label="Remove asset">✕</button>
</div>
```

If `url` is empty, the name is plain text (no `<a>` tag).

**Add form** (in-place, replaces the "+ Add asset" link when active):
```html
<div class="asset-form">
  <input type="text" placeholder="File name (e.g., questionnaire.docx)" data-asset-name>
  <input type="url" placeholder="https:// (optional)" data-asset-url>
  <button onclick="saveAsset(1, 0)">Save</button>
  <button onclick="cancelAssetForm(1, 0)">Cancel</button>
</div>
```

**Style:** Chips are compact (small icon + text + ✕), styled in brand teal/green palette. The "+ Add asset" link is small and unobtrusive. The form is a simple horizontal row.

### 3. Checkbox Gating

**On render / state change:**
- For each `.progress-check`, set `disabled = (asset count for that task === 0)`
- Add a class `.progress-task.locked` when disabled, which dims the row and adds a `title="Add an asset first"` attribute

**On asset delete:**
- If deleting the last asset of a checked task:
  - Uncheck the box
  - Strip the `.done` class
  - Update `projectData.progress[phase][task] = false`
  - Disable the checkbox again
  - Persist

**Checkbox click**: existing `updateProgress(phase)` continues to work — it just won't be reachable when disabled.

### 4. Migration / Backward Compatibility

When `resumeProject()` loads saved state:
- If `saved.assets` is missing, initialize `projectData.assets = {1:{},2:{},3:{},4:{}}`
- Existing `saved.progress` is honored — tasks that were already checked remain checked even if they have no assets logged
- However: the checkbox for any already-checked task with 0 assets is **enabled** (so user can uncheck if needed). Once unchecked, normal gating applies again.

When `launchProject()` creates a new project:
- Initialize `projectData.assets = {1:{},2:{},3:{},4:{}}` alongside `progress`

When `newProject()` resets:
- Clears all assets when `projectData = {}`

### 5. Functions

- `openAssetForm(phase, task)` — replaces the "+ Add asset" link with the inline form, focuses the name input
- `cancelAssetForm(phase, task)` — removes the form, restores the "+ Add asset" link
- `saveAsset(phase, task)` — reads name + url from form inputs, validates (name required, url optional but must pass `isValidHttpUrl` if present), pushes onto `projectData.assets[phase][task]`, re-renders the list, persists, refreshes checkbox state
- `deleteAsset(phase, task, index)` — splices out the asset, re-renders, persists, refreshes checkbox state (auto-unchecks if last asset)
- `renderAssetList(phase, task)` — rebuilds the chip list + add link inside `.asset-list[data-asset-phase][data-asset-task]`
- `refreshAllAssetLists()` — calls `renderAssetList` for all 20 task positions; called from `launchProject`, `resumeProject`, `newProject`
- `refreshProgressCheckboxes()` — recomputes `disabled` and `.locked` state for all 20 `.progress-check` based on asset counts; called whenever assets change

---

## Data Flow

```
User clicks "+ Add asset"
  → openAssetForm renders inline inputs
User fills name, optionally URL, clicks Save
  → saveAsset validates → push to projectData.assets → save to localStorage
  → renderAssetList rebuilds chips → refreshProgressCheckboxes enables checkbox
User checks the checkbox
  → existing updateProgress flow continues (sets done class, saves)

User clicks ✕ on a chip
  → deleteAsset splices → save → renderAssetList → 
  → if last asset and task was checked: uncheck, strip done, save progress
  → refreshProgressCheckboxes disables checkbox
```

---

## Testing

- [ ] Phase 1 task 0 with 0 assets → checkbox disabled, locked styling, hover shows "Add an asset first"
- [ ] Click "+ Add asset" → form appears with focused name input
- [ ] Save with empty name → validation message, form stays open
- [ ] Save with non-https URL → validation message, form stays open
- [ ] Save with valid name only → chip appears with plain-text name, no link
- [ ] Save with valid name + valid URL → chip's name is clickable (opens in new tab)
- [ ] Save multiple assets → all chips visible, each with own ✕
- [ ] With ≥1 asset → checkbox is enabled, can be checked
- [ ] Delete an asset (not the last) → that chip disappears, checkbox stays enabled
- [ ] Delete the last asset of a checked task → chip disappears, checkbox auto-unchecks and disables
- [ ] Refresh page, resume project → assets restored, checkboxes restored to correct state
- [ ] New Project → all assets cleared, all checkboxes disabled
- [ ] Existing saved project from before this feature → loads, no errors, previously-checked tasks remain checked
- [ ] Phase 4 gate is still respected — assets feature does not affect phase navigation gates from previous feature

---

## Out of Scope

- Verifying that the named file actually exists in the Drive folder (would require Drive API)
- Bulk import of assets (e.g., reading from a Drive folder listing)
- Drag-and-drop asset names
- Editing an existing asset (must delete + re-add)
- Asset reordering
- Per-activity required asset counts (>1) or required asset names
