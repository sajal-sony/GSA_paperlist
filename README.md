# Human Reconstruction Paper Library

A personal, taxonomy-driven library of animatable Gaussian splatting and human reconstruction papers — published as a GitHub Pages site.

## 🔗 Live site
`https://sajal-sony.github.io/GSA_paperlist/`

---

## Workflow

### Adding a new paper

1. **Analyze the paper** using the `human-recon-paper-analyzer` skill in **Claude.ai**.
   - Drop the PDF in, get back:
     - A collapsible `<details>` registry entry — append this to `paper_registry.md` for your personal log.
     - A `javascript // PASTE INTO PAPERS[]` block — this is what you hand to the updater skill.

2. **Update the site** using the `human-recon-gh-pages-updater` skill in **GitHub Copilot**.
   - Paste only the JS block from step 1.
   - Copilot inserts it at the top of `PAPERS[]` in `docs/index.html`, commits, and pushes.
   - No manual git commands needed — the skill handles it.

3. GitHub Actions deploys automatically within ~1 minute. Done.

---

## Repo structure

```
.
├── docs/
│   └── index.html          ← The entire site. All paper data lives in PAPERS[] inside this file.
├── skills/
│   ├── human-recon-paper-analyzer/
│   │   └── SKILL.md        ← Skill 1 (Claude.ai): PDF → 20-dim analysis + ready-to-paste JS block
│   └── human-recon-gh-pages-updater/
│       └── SKILL.md        ← Skill 2 (Copilot): paste JS block → insert into PAPERS[], commit, push
└── .github/
    └── workflows/
        └── deploy.yml      ← Auto-deploy to GitHub Pages on push to main
```

---

## GitHub Pages setup (first time)

1. Push this repo to GitHub (can be private — Pages works on private repos with the right plan, or make it public).
2. Go to **Settings → Pages**.
3. Source: **GitHub Actions**.
4. Push any change to `docs/` to trigger the first deploy.

---

## Taxonomy dimensions

The site tracks 20 fixed dimensions per paper:

| # | Dimension |
|---|-----------|
| 1 | Body Model |
| 2 | Gaussian Representation |
| 3 | Canonical Space Initialization |
| 4 | FK / LBS Approach |
| 5 | Deformation Field Architecture |
| 6 | Pose Conditioning Method |
| 7 | Densification / Pruning Strategy |
| 8 | Loss Functions |
| 9 | Dynamic / Neural Property Prediction |
| 10 | Relightability |
| 11 | Appearance Editability |
| 12 | UV / Gaussian Mapping |
| 13 | Cloth / Hair Handling |
| 14 | Physics / Simulation |
| 15 | Multi-view vs Monocular |
| 16 | Training Data Requirements |
| 17 | Inference Speed |
| 18 | Datasets & Baselines |
| 19 | Reconstruction Target |
| 20 | Texture Continuity / Editability Mechanism |

The analyzer skill also proposes new dimensions when it encounters something that doesn't fit. You decide whether to adopt them — if you do, backfill previous entries manually and add the new key to the taxonomy object in `index.html`.
