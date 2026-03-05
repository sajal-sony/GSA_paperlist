---
name: human-recon-gh-pages-updater
description: >
  Inserts a pre-built JS paper object (produced by human-recon-paper-analyzer) into
  the PAPERS[] array in docs/index.html and pushes the change so GitHub Actions
  deploys it. Use this skill whenever the user says "add this paper to the site",
  "update the library", or pastes a JS block from the analyzer skill.
  Designed to run inside GitHub Copilot (VS Code) with full repo access.
---

# Human Reconstruction GitHub Pages Updater

## Purpose
Take the `javascript // PASTE INTO PAPERS[]` block produced by `human-recon-paper-analyzer`
and insert it into the `PAPERS[]` array in `docs/index.html`. No re-interpretation of
taxonomy values — the analyzer has already done all the work.

---

## Required Input
A fenced code block labelled `javascript // PASTE INTO PAPERS[]` from the analyzer skill.
If the user hasn't provided it, ask: *"Please paste the JS block from the analyzer output."*
Do **not** accept a markdown `<details>` registry entry as a substitute — those are for
`paper_registry.md` only. The JS block is the only input this skill needs.

---

## Workflow

### Step 1: Read docs/index.html
Open `docs/index.html` from the repo. Locate the `<script id="paper-data">` tag.
Inside it, find the `PAPERS` array opening:

```javascript
const PAPERS = [
```

### Step 2: Find the insertion point
The new entry goes at the **very beginning** of the array so the newest paper appears
first on the site.

- If the array is empty (only the example comment block), insert after the `*/` that
  closes the example comment.
- If the array already has entries, insert before the first `{`.
- Never reorder or touch existing entries.

### Step 3: Insert the JS object
Paste the object exactly as provided by the analyzer. Do not reformat, re-escape,
or summarise any values. Ensure there is a trailing comma after the closing `}` of
the new entry if any entries follow it.

### Step 4: Verify
Quick sanity checks before saving:
- `PAPERS` array brackets are balanced
- New entry has a trailing comma if it's not the last item
- No raw unescaped double quotes inside string values
- `null` (not `"null"`) for absent URLs
- All 20 taxonomy keys are present

### Step 5: Commit and push
Run:
```bash
git add docs/index.html
git commit -m "add: <paper title> <VENUE> <YEAR>"
git push
```
GitHub Actions will deploy automatically within ~1 minute.

### Step 6: Report to user
Tell the user:
1. ✅ **Paper added:** [title]
2. 🏷️ **Tags:** [list from the object's tags array]
3. ⚠️ **Proposed dimensions needing backfill** (if `proposedDimensions` array is non-empty — list the dimension names)
4. 🚀 **Deploying** — live at `https://sajal-sony.github.io/GSA_paperlist/` in ~1 min

---

## Important Notes
- **Never edit anything outside** `<script id="paper-data">...</script>`
- **Never reformat or reorder existing entries**
- `paper_registry.md` is a separate human-readable log — this skill does not touch it
- If the user provides a markdown `<details>` block instead of a JS block, tell them
  to re-run the analyzer in Claude.ai and copy the JS block it outputs at the end

