# ConjunctivaGuard

**A Task-Aware, Uncertainty-Calibrated Accept–Correct–Reject Framework for Smartphone-Based Conjunctival Anemia Screening with Cross-Dataset Validation**

Final-year project, BEng (Hons) Software Engineering (6SENG010C/W)
Student: Manusha Upekshana (UoW W1998738 / IIT 20221575)
Supervisor: Mr Guhanathan Poravi

> Research prototype only. Not a medical device. Results must not be used to make decisions about real people.

---

## Project status

| Stage | Status | Where |
|---|---|---|
| Project proposal and ethics declaration | Submitted 5 Nov 2026 | (submitted separately) |
| **Proof of concept (PoC)** | **Done, 18–22 Nov 2026** | [`poc/`](poc/) |
| Full implementation | Planned Dec 2026 – Mar 2027 | will be added to this repository |

This repository currently holds the **proof of concept only**. It shows that each main part of the proposed system can work on real public data and that the parts connect end to end. It is **not** the final system: the calibrated risk estimator, the colour-safety gate, the full damage benchmark, all experiments (E1–E5), the external test on D02 and the web prototype are part of the full implementation.

## What ConjunctivaGuard does

Smartphone photos of the inner eyelid (conjunctiva) can screen for anemia without a blood test, but glare, blur and poor lighting make AI predictions unreliable. ConjunctivaGuard checks the quality of the eyelid area in each photo **before** answering, and decides:

- **Accept**: good quality, give the screening result;
- **Correct**: a fixable problem (glare, exposure), repair it with colour-safety limits, check again, then give the result;
- **Reject**: too damaged or too uncertain, give **no result** and ask for a new photo.

## Proof of concept: what was built

All PoC work was run in Google Colab (T4 GPU for training). Each notebook is saved **with its outputs**, so the results can be read without running anything.

| # | Notebook | Module (proposal) | What it shows | Key result |
|---|---|---|---|---|
| 1 | [`01_setup_and_look`](poc/notebooks/01_setup_and_look.ipynb) | - | Opens D01 and D03, inspects folders, labels and images | D01: 4,262 files (labels in file names); D03: 547 photos + masks from 2 annotators |
| 2 | [`02_data_and_splits_M1`](poc/notebooks/02_data_and_splits_M1.ipynb) | M1 Data + validation | Finds copies, builds photo groups, group-based split, leakage check | 503 photo groups, 0 label conflicts, split 368 / 60 / 75, **0 leakage** |
| 3 | [`03_eyelid_finder_M2`](poc/notebooks/03_eyelid_finder_M2.ipynb) | M2 Eyelid finder | LinkNet (ResNet-34) segments the inner eyelid | Test Dice 0.848 / 0.789 vs annotators; human agreement 0.726; eyelid found in 82/82 |
| 4 | [`04_damage_M9`](poc/notebooks/04_damage_M9.ipynb) | M9 Damage generator | Glare, blur, dark at 3 levels, clean reference kept, fixed seeds | 1,350 val/test images; reproducible; damaged copies keep their split |
| 5 | [`05_quality_M3_and_baseline_M4`](poc/notebooks/05_quality_M3_and_baseline_M4.ipynb) | M3 Quality profiler, M4 Anemia predictor | Eyelid-only quality profile; EfficientNet-B0 baseline; damage effect (E1 preview) | Clean test AUROC 0.778 (95% CI 0.670–0.883); dark and blur push AUROC toward 0.5 |
| 6 | [`06_accept_correct_reject`](poc/notebooks/06_accept_correct_reject.ipynb) | M6–M8 (basic rules) | Accept / Correct / Reject with glare repair, exposure fix and colour check | Answers changed by bad photos 20.4% → 14.4%; glare repair cuts colour error ~85% |

A short written summary of all results, with a chart, limits and next steps: [`poc/ConjunctivaGuard_PoC_Results_Pack.pdf`](poc/ConjunctivaGuard_PoC_Results_Pack.pdf).

All result files (splits, predictions, decisions, thresholds, training histories) are in [`poc/results/`](poc/results/).

## Main findings

1. **Data:** D01's 4,262 files are 503 distinct eyelids (augmented and renamed copies). A pixel-verified duplicate check replaced the proposal's outline-based audit: groups changed from about 404 to 503, and the 14 apparent label conflicts were different children with similar eyelid outlines. Borderline pairs were reviewed by eye and recorded (`D01_manual_decisions.csv`).
2. **Eyelid finder:** agrees with each annotator at least as well as the two annotators agree with each other.
3. **Quality must be measured carefully:** a generic sharpness measure was fooled by grain and glare edges (dark photos scored 13–40× "sharper"). The fixed eyelid-only profile makes each number react only to its own problem (supports RQ1).
4. **Damage changes answers:** dark and blur push the baseline toward chance; glare barely moves AUROC but changes the yes/no answer for up to 35% of eyelids. Average scores hide per-photo harm.
5. **Colour safety is real:** glare repair restores colour (ΔE00 to the clean original 5.15 → 0.85) and never changes untouched tissue (ΔE00 = 0.0), but a naive brightness fix moved eyelid colour by about 20 ΔE00 and still passed a quality re-check. This is the evidence for the proposal's colour-safety gate (ΔE00 ≤ ε) and learned risk model.

## Honest limits of the PoC

- Synthetic damage; small test sets (75 D01 eyelids, 82 D03 photos); most confidence intervals overlap at mild damage.
- Baseline probabilities are not calibrated (sensitivity 0.81, specificity 0.45 at a threshold of 0.26 chosen on validation).
- A/C/R uses fixed rules chosen on validation photos, not the proposed risk estimator (M6) and target risk τ; blur is detected poorly and 20% of clean photos are rejected.
- The colour-safety gate is **not** enforced in the PoC; colour is only checked afterwards against the clean original.
- D03 was split by photo, not by person.

## How to run the notebooks

1. Download the datasets yourself (see [`DATA.md`](DATA.md)); they are not included here.
2. Put the D01 and D03 zip files in your Google Drive and share them as "Anyone with the link", or use their Drive paths.
3. Open a notebook in Google Colab, replace `YOUR_D01_FILE_ID` / `YOUR_D03_FILE_ID` with your file IDs, and run the cells top to bottom (Runtime → T4 GPU for notebooks 3 and 5).
4. Run the notebooks in order (1 → 6): later notebooks read the result files that earlier ones save to `ConjunctivaGuard/poc_outputs` in Google Drive.

Python packages: see [`requirements.txt`](requirements.txt) (all are pre-installed in Colab except `segmentation-models-pytorch`, which notebook 3 installs).

## Data and ethics

Only public, secondary datasets are used and no new data is collected (ethics declaration: Class 1). D02 (EYES-DEFY), the locked external test set, is **not used anywhere in the PoC** and is kept offline on the student's laptop. Details: [`DATA.md`](DATA.md).

## Use of AI assistance

The PoC notebooks were developed with an AI assistant (Claude, Anthropic) as a coding and learning aid, as declared in the project proposal. The student ran every notebook, checked the outputs, made the borderline data decisions by eye and is responsible for the work and its conclusions.

## Repository layout

```
ConjunctivaGuard/
├── README.md                     this file
├── DATA.md                       datasets, licences, what is and is not stored here
├── requirements.txt              Python packages used by the PoC
├── .gitignore                    keeps datasets, images and model weights out
└── poc/
    ├── ConjunctivaGuard_PoC_Results_Pack.pdf
    ├── notebooks/                6 Colab notebooks, saved with outputs
    └── results/                  CSV / JSON result files
```

The full implementation (Python package, tests, web prototype) will be added alongside `poc/` from December 2026.
