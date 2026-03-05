# Human Reconstruction Paper Library

A personal, taxonomy-driven library of animatable Gaussian splatting and human reconstruction papers — published as a GitHub Pages site.

## 🔗 Live site
`https://sajal-sony.github.io/GSA_paperlist/`

---

## Workflow

### Adding a new paper

1. **Analyze the paper** using the `human-recon-paper-analyzer` skill in Claude.ai or GitHub Copilot.
   - Drop the PDF in, get back a structured 18-dimension analysis block.

2. **Update the site** using the `human-recon-gh-pages-updater` skill in GitHub Copilot (or any Claude session with `docs/index.html` as context).
   - Paste the analysis output + the skill instructions.
   - The agent edits only the `PAPERS[]` array inside `docs/index.html`.

3. **Review and push.**
   ```bash
   git diff docs/index.html   # review the change
   git add docs/index.html
   git commit -m "add: PaperName et al. VENUE YEAR"
   git push
   ```

4. GitHub Actions deploys automatically. Done.

---

## Repo structure

```
.
├── docs/
│   └── index.html          ← The entire site. All paper data lives in PAPERS[] inside this file.
├── skills/
│   ├── human-recon-paper-analyzer/
│   │   └── SKILL.md        ← Skill 1: analyze a paper → structured taxonomy output
│   └── human-recon-gh-pages-updater/
│       └── SKILL.md        ← Skill 2: take analyzer output → update docs/index.html
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

The site tracks 18 fixed dimensions per paper:

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

The analyzer skill also proposes new dimensions when it encounters something that doesn't fit. You decide whether to adopt them — if you do, backfill previous entries manually and add the new key to the taxonomy object in `index.html`.
