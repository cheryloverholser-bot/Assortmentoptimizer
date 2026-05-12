# Project Folder Link — Design Spec

**Date:** 2026-05-12
**Feature:** Per-engagement Google Drive folder link, visible on all 4 phase pages.

---

## Goal

Let consultants associate a Google Drive folder URL with each engagement and open it in one click from any phase page. Link is editable post-setup without resetting project data.

---

## Architecture

Three changes to `Assessments Templates/Crisp-Assessment-AI-Process-Plan.html`:

1. **Setup modal** — add an optional "Google Drive folder link" input after the engagement lead field.
2. **Storage** — save URL to `projectData.driveLink` in localStorage.
3. **Phase pages** — add a slim button between the phase hero card and the Key Activities card on Phase 1, 2, 3, and 4.

---

## Components

### 1. Setup Modal Input

New form field after the engagement lead input:
- Label: "Google Drive Folder Link (optional)"
- Type: `<input type="url" placeholder="https://drive.google.com/drive/folders/...">`
- Validation: none required (optional field); browser's native `type="url"` provides basic format hint
- Saved to `projectData.driveLink` on Launch

### 2. Project Folder Button

Inserted in each phase page (Phase 1–4), positioned between the phase hero card and the Key Activities card.

**Two states based on `projectData.driveLink`:**

| State | Button text | Click action |
|---|---|---|
| Link saved | `📁 Open Project Folder` | Opens URL in a new tab (`target="_blank"`, `rel="noopener"`) |
| No link | `📁 Set Project Folder Link` (lighter/dashed style) | Opens `window.prompt()` for URL; saves to localStorage; refreshes button state |

**Style:** Slim brand-green chip-style button, full-width on its row, matching existing card aesthetic. Subtle so it doesn't compete with phase content.

### 3. Update Logic

Single helper function `openOrSetDriveLink()`:
- If `projectData.driveLink` is truthy → `window.open(projectData.driveLink, '_blank', 'noopener')`
- Else → `const url = prompt('Paste the Google Drive folder URL:'); if (url) { projectData.driveLink = url; save; refreshDriveButtons(); }`

Helper `refreshDriveButtons()` updates the text/style of all four phase buttons based on current state.

Called from:
- `launchProject()` after saving form fields
- `resumeProject()` after restoring state
- `openOrSetDriveLink()` after a successful prompt entry
- `newProject()` after clearing data (to reset to "Set" state)

---

## Data Flow

```
Setup modal → driveLink stored in projectData → save() → localStorage
                                              ↓
                                    refreshDriveButtons()
                                              ↓
              Phase pages show "Open" or "Set" state on all 4 buttons

User clicks "Open" → window.open in new tab
User clicks "Set"  → prompt → save URL → refresh buttons
```

---

## Testing

- [ ] Setup modal shows the link input as optional
- [ ] Launching with a link saves it; reload restores it
- [ ] Launching without a link saves empty; reload still works
- [ ] Phase 1–4 each show the button in the correct position
- [ ] With link saved: button says "Open Project Folder" and opens in new tab
- [ ] Without link: button says "Set Project Folder Link" and opens a prompt
- [ ] After setting via prompt: button text on all 4 phases updates immediately
- [ ] New Project resets the link
- [ ] Empty/cancelled prompt does not clear an existing link

---

## Out of Scope

- Validating that the URL is actually a Google Drive folder (any URL accepted)
- Auto-creating the folder in Drive (separate idea, deferred)
- Multiple folders per engagement
- Sub-folders per phase
