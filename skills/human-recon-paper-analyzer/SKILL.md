---
name: human-recon-paper-analyzer
description: >
  Analyzes animatable Gaussian splatting / human reconstruction research papers
  and extracts a structured taxonomy of key technical dimensions. Use this skill
  whenever the user uploads or pastes a paper related to human body reconstruction,
  neural human avatars, animatable NeRFs, or Gaussian splatting for humans —
  even if they just say "analyze this paper", "what does this paper do", "classify
  this paper", or "add this to my paper list". Always produces a structured markdown
  analysis block ready to be appended to a paper registry file.
---

# Human Reconstruction Paper Analyzer

## Purpose
Extract a structured, consistent analysis of any animatable Gaussian splatting /
human reconstruction paper across a fixed 20-dimension taxonomy. Output is formatted
to be appended directly to the user's paper registry markdown file (`paper_registry.md`
or equivalent). Novel concepts outside the taxonomy are flagged explicitly.

---

## Workflow

### Step 1: Parse the Paper
Read the full paper carefully. Focus on:
- Method section (architecture, losses, deformation)
- Experiments section (datasets, metrics, baselines)
- Abstract and conclusion (claims about relightability, physics, editability)
- Any supplementary details visible in the document

### Step 2: Fill the Taxonomy
Extract all 20 dimensions below. For each:
- Be **concise and specific** — one to three sentences max per dimension
- Use `N/A` if the paper genuinely does not address the dimension
- Use `Not specified` if the paper likely has an approach but doesn't detail it
- **Never hallucinate** — if unsure, quote the paper or say "unclear"

### Step 3: Flag Novel Concepts
After the taxonomy, add a **"Novel / Out-of-Taxonomy Concepts"** section.
List anything technically interesting that doesn't fit the 20 dimensions —
new training tricks, unusual data representations, surprising results, etc.
These are the things worth flagging for the user to read manually.

### Step 3b: Mine for New Taxonomy Dimensions
After filling the 20 dimensions, ask yourself: **does this paper introduce a
recurring technical concern that the current taxonomy has no home for?**

Criteria for proposing a new dimension — it must be:
1. A **distinct, nameable technical axis** (not just a detail within an existing dimension)
2. Something a researcher would **consistently want to know** across future papers
3. **Not already captured** by any of the 20 existing dimensions

Good examples of what qualifies: "Generalization strategy (subject-specific vs. generalizable)",
"Scene / background handling", "Animation retargeting capability".

Bad examples (too specific, not recurring): "Uses trilinear interpolation",
"Trained on ZJU-MoCap only".

If you identify one or more candidate dimensions, add a **"Proposed New Taxonomy Dimensions"**
section to the registry entry. For each proposal:
- Give it a **name and one-line definition**
- Fill in what **this paper's answer** would be
- Leave a **`[ ] Backfill needed`** checkbox — this signals to the user that previous
  entries in the registry should be manually reviewed and filled for this dimension

The user decides whether to adopt the dimension permanently. This skill never
modifies the taxonomy itself — it only proposes.

### Step 4: Write the Registry Entry
Format the full output as a collapsible markdown block (using `<details>`)
so the registry file stays scannable over time.

### Step 5: Append to Registry
Save the formatted entry to `paper_registry.md` in the current working directory.
If the file doesn't exist yet, create it with a header. If it exists, append to the end.

---

## The 20-Dimension Taxonomy

```
1.  BODY MODEL
    What parametric body model is used? (SMPL, SMPL-X, FLAME, STAR, custom, none)
    Any modifications to the standard model?

2.  GAUSSIAN REPRESENTATION
    What form of Gaussian splatting? (3DGS, 2DGS, hybrid, other)
    Any novel properties added to each Gaussian (beyond standard mean/cov/color/opacity)?

3.  CANONICAL SPACE GAUSSIAN PLACEMENT / INITIALIZATION
    How are Gaussians initialized in canonical space?
    (SMPL mesh surface, random, point cloud, learned, etc.)
    How many Gaussians approximately?

4.  FORWARD KINEMATICS / LBS APPROACH
    How are Gaussians driven from canonical to posed space?
    Standard LBS? Dual quaternion? Learned blend weights? Corrective fields?

5.  DEFORMATION FIELD ARCHITECTURE
    Is there a neural deformation field on top of LBS? (MLP, hash grid, tri-plane, none)
    What does it predict? (residual offsets, full deformation, covariance correction?)

6.  POSE CONDITIONING METHOD
    How is pose information fed into the network?
    (Joint angles, bones, SMPL parameters, learned embeddings, etc.)

7.  DENSIFICATION / PRUNING STRATEGY
    Does the method use any Gaussian densification or pruning during training?
    Any domain-specific strategy for human body regions?

8.  LOSS FUNCTIONS
    List all loss terms: photometric (L1/L2/SSIM/LPIPS), perceptual, regularization,
    pose priors, geometry losses, any novel terms.
    Note any loss weightings explicitly mentioned.

9.  DYNAMIC / NEURAL PROPERTY PREDICTION
    Are Gaussian properties (color, opacity, covariance) predicted dynamically
    per-pose/per-frame by a network, or are they static in canonical space?

10. RELIGHTABILITY
    Can the model be relit under novel illumination?
    What decomposition is used? (diffuse/specular, SH, PBR, none)

11. APPEARANCE EDITABILITY / CUSTOM TEXTURE
    Can the user apply custom textures or edit appearance?
    How — UV map, style transfer, direct Gaussian color editing?

12. UV / GAUSSIAN MAPPING
    How are UVs generated for Gaussian-to-surface mapping?
    (SMPL UV unwrap, learned UV, none — Gaussians are free-floating)

13. CLOTH / HAIR / LOOSE GARMENT HANDLING
    Does the method handle loose clothing, hair, or accessories?
    Separate module? Implicit representation? Ignored?

14. PHYSICS / CLOTH SIMULATION
    Any physics simulation running under the hood?
    (MPM, position-based dynamics, physics-informed losses, none)

15. MULTI-VIEW vs MONOCULAR SETUP
    What input does training require? (monocular video, multi-view, RGBD, etc.)
    At test time, how many views are needed?

16. NUMBER OF TRAINING FRAMES / DATA REQUIREMENTS
    How many frames / how long a video does the method need per subject?
    Any few-shot or generalizable (cross-subject) capability?

17. INFERENCE / RENDERING SPEED
    Reported FPS or render time. On what hardware?
    Real-time capable?

18. DATASETS & BASELINES
    What datasets are used for evaluation?
    What are the main baseline methods compared against?
    Key metrics reported (PSNR, SSIM, LPIPS, FID, etc.)?

19. RECONSTRUCTION TARGET
    What body region does the method reconstruct?
    (full body, upper body, head-only frontal, head 360°, hands, face-only, etc.)
    Is the scope fixed by design or configurable?

20. TEXTURE CONTINUITY / EDITABILITY MECHANISM
    Does the method produce continuous, editable attribute maps (e.g. in UV space),
    or are Gaussian attributes purely discrete per-point?
    What mechanism ensures continuity? (CNN inductive bias, explicit smoothness loss,
    generative decoder, none)
    Can standard texture-editing workflows (sticker overlay, style transfer, painting)
    be applied directly?
```

---

## Output Format

Produce exactly this structure:

```markdown
<details>
<summary><strong>[PAPER TITLE]</strong> — [First Author et al., VENUE YEAR]</summary>

> **TL;DR:** One sentence summary of the core contribution.
> **Link / arXiv:** [if available in the paper]
> **Date Added:** [today's date]

| Dimension | Details |
|-----------|---------|
| Body Model | ... |
| Gaussian Representation | ... |
| Canonical Initialization | ... |
| FK / LBS Approach | ... |
| Deformation Field | ... |
| Pose Conditioning | ... |
| Densification / Pruning | ... |
| Loss Functions | ... |
| Dynamic Property Prediction | ... |
| Relightability | ... |
| Appearance Editability | ... |
| UV / Gaussian Mapping | ... |
| Cloth / Hair Handling | ... |
| Physics / Simulation | ... |
| Multi-view vs Monocular | ... |
| Training Data Requirements | ... |
| Inference Speed | ... |
| Datasets & Baselines | ... |
| Reconstruction Target | ... |
| Texture Continuity / Editability | ... |

### Novel / Out-of-Taxonomy Concepts
- [Concept 1]: Brief description of why it's interesting
- [Concept 2]: ...
- *(None flagged)* if nothing stands out

### Proposed New Taxonomy Dimensions
> Candidate dimensions surfaced from this paper that may be worth adding to the
> registry for all future (and past) entries. You decide whether to adopt them —
> if you do, backfill previous entries manually.

| Proposed Dimension | Definition | This Paper's Answer | Backfill |
|--------------------|------------|---------------------|----------|
| [Name] | [One-line definition] | [What this paper does] | [ ] Backfill needed |

*(No new dimensions proposed)* — replace table with this line if nothing qualifies

</details>
```

---

## Registry File Format

If `paper_registry.md` does not exist, create it with this header first:

```markdown
# Human Reconstruction Paper Registry

A running log of analyzed papers on animatable Gaussian splatting and human reconstruction.
Each entry is collapsed for readability — click to expand.

---
```

Then append each new paper entry below the `---`.
Never overwrite existing entries.

---

## Important Notes
- If the paper is a NeRF-based (not Gaussian) human method, still use the taxonomy —
  map "Gaussian Representation" to the volumetric representation used instead.
- If the paper is about a body part only (face, hands), note this clearly in the TL;DR
  and fill taxonomy dimensions that apply.
- Always tell the user: "Entry appended to `paper_registry.md`" after saving.
