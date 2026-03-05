---
name: human-recon-gh-pages-updater
description: >
  Updates the human reconstruction paper library GitHub Pages site (docs/index.html)
  by converting a human-recon-paper-analyzer output into a correctly formatted
  JavaScript entry and inserting it into the PAPERS[] array in index.html.
  Use this skill whenever the user says "add this paper to the site", "update the
  library", "add to GitHub Pages", or pastes a paper analysis and wants it published.
  Designed to run inside GitHub Copilot (VS Code) with access to the repo, or in
  any Claude session where index.html is provided as context.
---

# Human Reconstruction GitHub Pages Updater

## Purpose
Take the structured analysis output from `human-recon-paper-analyzer` and translate
it into a valid JavaScript object entry in the `PAPERS[]` array inside `docs/index.html`.
The site renders entirely from that array — this is the only file that ever needs editing.

---

## Required Inputs
Before making any edits, confirm you have:

1. **The paper analysis** — the full taxonomy output from `human-recon-paper-analyzer`
   (the markdown block with the 18-dimension table + novel concepts + proposed dimensions)
2. **`docs/index.html`** — the current site file, either open in the editor or pasted in

If either is missing, ask the user to provide it.

---

## Workflow

### Step 1: Parse the Analysis
From the paper analyzer output, extract:

| Field | Where to find it |
|-------|-----------------|
| `title` | `<summary>` tag — the bold paper title |
| `authors` | `<summary>` tag — after the em dash |
| `venue` | `<summary>` tag — venue name |
| `year` | `<summary>` tag — year number |
| `dateAdded` | `> **Date Added:**` line → convert to YYYY-MM-DD |
| `arxiv` | `> **Link / arXiv:**` line |
| `projectPage` | `> **Link / arXiv:**` line if it's a project URL, not arXiv |
| `tldr` | `> **TL;DR:**` line — strip the prefix |
| taxonomy rows | The 18-row markdown table — map to JS object keys (see mapping below) |
| `novelConcepts` | Bullet points under `### Novel / Out-of-Taxonomy Concepts` |
| `proposedDimensions` | Rows in the `### Proposed New Taxonomy Dimensions` table |

### Step 2: Derive the `id`
Generate a slug from the paper title and year:
- Take first meaningful word of title + last word + year
- Lowercase, hyphens only, no special chars
- Examples: `"GaussianAvatar: Towards Realistic Human Avatars, 2023"` → `"gaussianavatar-2023"`
- If unsure, ask the user to confirm

### Step 3: Derive the `tags` array
Set tags based on taxonomy values:
- `"relightable"` → if Relightability row is NOT "N/A"
- `"physics"` → if Physics/Simulation row is NOT "N/A"
- `"editable"` → if Appearance Editability row is NOT "N/A"
- `"realtime"` → if Inference Speed mentions FPS > 20 or "real-time"
- `"monocular"` → if Multi-view vs Mono mentions "monocular"

### Step 4: Build the JS Object
Produce exactly this format — pay close attention to escaping:

```javascript
{
  id: "SLUG",
  title: "FULL TITLE",
  authors: "First Author et al.",
  venue: "VENUE",
  year: YEAR,
  dateAdded: "YYYY-MM-DD",
  arxiv: "URL_OR_NULL",
  projectPage: "URL_OR_NULL",
  tldr: "One sentence. No quotes that would break JS strings — escape with \\' if needed.",
  tags: ["tag1", "tag2"],
  taxonomy: {
    "Body Model":               "...",
    "Gaussian Repr.":           "...",
    "Canonical Init.":          "...",
    "FK / LBS":                 "...",
    "Deformation Field":        "...",
    "Pose Conditioning":        "...",
    "Densification / Pruning":  "...",
    "Loss Functions":           "...",
    "Dynamic Prop. Prediction": "...",
    "Relightability":           "...",
    "Appearance Editability":   "...",
    "UV / Gaussian Mapping":    "...",
    "Cloth / Hair":             "...",
    "Physics / Simulation":     "...",
    "Multi-view vs Mono":       "...",
    "Training Data Req.":       "...",
    "Inference Speed":          "...",
    "Datasets & Baselines":     "..."
  },
  novelConcepts: [
    "Concept one description",
    "Concept two description"
  ],
  proposedDimensions: [
    { name: "Dimension Name", definition: "One-line def.", answer: "This paper's value", backfill: true }
  ]
},
```

**String escaping rules:**
- Use double quotes for all JS string values
- Escape any double quotes inside values with `\"`
- Escape any backticks with `\``
- If a field is absent/null, use `null` (no quotes)
- If `novelConcepts` is empty, use `[]`
- If `proposedDimensions` is empty, use `[]`

### Step 5: Taxonomy Key Mapping
Map from the markdown table's left column to the exact JS key:

| Markdown dimension name | JS taxonomy key |
|------------------------|-----------------|
| Body Model | `"Body Model"` |
| Gaussian Representation | `"Gaussian Repr."` |
| Canonical Space Gaussian Placement / Initialization | `"Canonical Init."` |
| Forward Kinematics / LBS Approach | `"FK / LBS"` |
| Deformation Field Architecture | `"Deformation Field"` |
| Pose Conditioning Method | `"Pose Conditioning"` |
| Densification / Pruning Strategy | `"Densification / Pruning"` |
| Loss Functions | `"Loss Functions"` |
| Dynamic / Neural Property Prediction | `"Dynamic Prop. Prediction"` |
| Relightability | `"Relightability"` |
| Appearance Editability / Custom Texture | `"Appearance Editability"` |
| UV / Gaussian Mapping | `"UV / Gaussian Mapping"` |
| Cloth / Hair / Loose Garment Handling | `"Cloth / Hair"` |
| Physics / Cloth Simulation | `"Physics / Simulation"` |
| Multi-view vs Monocular | `"Multi-view vs Mono"` |
| Number of Training Frames / Data Requirements | `"Training Data Req."` |
| Inference / Rendering Speed | `"Inference Speed"` |
| Datasets & Baselines | `"Datasets & Baselines"` |

If the paper's analysis contains **additional adopted taxonomy dimensions** (ones the user
decided to keep from a prior proposed-dimensions suggestion), add them to the taxonomy
object with their exact adopted key name after the 18 standard keys.

### Step 6: Insert into index.html

Locate the `PAPERS` array in `index.html`. It lives inside the `<script id="paper-data">` tag:

```javascript
const PAPERS = [
  /* existing entries */
];
```

**Insertion rule:** Add the new object at the **beginning** of the array (after the opening `[`),
so the most recently added paper appears first on the page.

Insert immediately after `const PAPERS = [`, before any existing entries. If there's an
example entry comment block at the top, insert after the closing `*/` of that comment.

Do **not** touch anything outside the `<script id="paper-data">` block.

### Step 7: Verify

After inserting, do a quick sanity check:
- The `PAPERS` array is still valid JS (brackets balanced, trailing commas correct)
- The new entry has all 18 taxonomy keys
- No raw double quotes inside string values (they'd break JSON parsing)
- `null` (not `"null"`) used for absent URLs

### Step 8: Report to User

Tell the user:
1. ✅ Paper added: **[title]**
2. 📋 Tags assigned: `[list]`
3. ⚠️ Any proposed new dimensions that need backfilling across existing entries
4. 📝 Next step: review the change in VS Code / your editor, then `git commit` and `git push`
   — GitHub Actions will deploy automatically.

---

## Important Notes
- **Never reformat or reorder existing entries** in the PAPERS array
- **Never edit anything outside** `<script id="paper-data">...</script>`
- If a taxonomy value contains line breaks, collapse to a single line with ` · ` as separator
- If `arxiv` and `projectPage` are both available, populate both fields
- If only one link is available, set the other to `null`
- The `tldr` field must be plain text — no markdown, no asterisks, no backticks
