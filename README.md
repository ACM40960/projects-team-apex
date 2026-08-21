<div align="center">

# Conformal Uncertainty Quantification for Mammographic Malignancy Classification
---

**Split conformal prediction wrapped around a DenseNet-121 mammography classifier: turning raw softmax confidence into a provable, distribution-free coverage guarantee.**

</div>


## Table of Contents

- [Overview](#overview)
- [The Core Finding](#the-core-finding)
- [Headline Results](#headline-results)
- [Repository Structure](#repository-structure)
- [Method](#method)
- [Reproducing the Results](#reproducing-the-results)
- [Assessment Context](#assessment-context)
- [Team](#team)


## Overview
---

Deep learning classifiers for mammographic malignancy detection output a softmax probability, but that number is not a trustworthy confidence score on its own. This project's own base classifier has an **Expected Calibration Error of ~0.205**, meaning "80% confident" is not actually right 80% of the time.

This project wraps a **DenseNet-121** classifier, fine-tuned on **CBIS-DDSM** (benign vs. malignant), with **split conformal prediction**, a post-hoc, model-agnostic calibration layer that produces *prediction sets* with a distribution-free, finite-sample coverage guarantee, valid regardless of whether the underlying model is well-calibrated.

> **Try it yourself →** [Run the notebook on Kaggle](https://www.kaggle.com/code/wandererray9/conformal-mammography-classification) · [View the dataset](https://www.kaggle.com/datasets/awsaf49/cbis-ddsm-breast-cancer-image-dataset)

Three nonconformity scores are implemented and compared:

| Method | What it does |
|:---|:---|
| **LAC** | Least Ambiguous set-valued Classifier — simplest, usually smallest sets |
| **APS** | Adaptive Prediction Sets — set size adapts to per-example ambiguity |
| **RAPS** | Regularized APS — penalizes unnecessary tail-class inclusion |

All three hit the target ~95% marginal coverage. That's the baseline result, not the headline one.

## The Core Finding
---

> **Marginal coverage can hide a clinically important failure.**

Pooled ("marginal") coverage looked fine at target, but broken down by class, **malignant-specific coverage undershot to ~0.893**, meaning roughly 1 in 10 true cancers fell outside their own prediction set. Benign's easier cases were propping up the average.

**Mondrian (class-conditional) calibration**: a separate threshold per class instead of one global threshold restores malignant coverage to **~0.960**. This is the project's headline clinical contribution: not just "conformal prediction works," but "the naive version of it can silently fail on the class that matters most, and there's a specific, correctable reason why."

<div align="center">

| | Marginal | Mondrian |
|:---|:---:|:---:|
| Malignant coverage | 0.893 | 0.960 |
| Target | 0.95 | 0.95 |

</div>

---

## Headline Results
---

| Metric | Value |
|:---|:---:|
| Base classifier AUC | **0.708** |
| Base classifier ECE | **0.205** |
| LAC / APS / RAPS marginal coverage | all **~0.95** (target hit) |
| LAC average set size | **~1.694** |
| LAC singleton rate | **~30.6%** |
| Marginal malignant coverage | **~0.893** ⚠️ under target |
| Mondrian malignant coverage | **~0.960** ✅ target restored |

Full detail and every supporting figure live in [`Outputs Received/`](Outputs%20Received/).

## Repository Structure
---

```
.
├── Code/                    → notebook, conformal module, test + module comparison
├── Outputs Received/        → results.json + all result figures
├── Poster/                  → poster drafts and final A0 PDF
├── Literature Review/       → source materials for the lit review
├── Lit. Review Latex/       → LaTeX build for the lit review writeup
└── README.md                → you are here
```

## Method
---

<table>
<tr><td width="36px" align="center"><b>1</b></td><td><b>Data & splits</b>: CBIS-DDSM JPEGs (via Kaggle), labels binarized from pathology. Splits are done at the <b>patient level</b>, not per-image, so no patient appears in more than one of train/val/calibration/test: this is what keeps the conformal exchangeability assumption honest.</td></tr>
<tr><td align="center"><b>2</b></td><td><b>Model</b>: DenseNet-121, ImageNet-pretrained, finely tuned end-to-end with a 2-class head. The conformal layer never sees the architecture, only the softmax output.</td></tr>
<tr><td align="center"><b>3</b></td><td><b>Conformal calibration</b>: LAC / APS / RAPS, all calibrated using the finite-sample-corrected quantile <code>⌈(n+1)(1−α)⌉ / n</code>, which is what makes the coverage guarantee <i>exact</i> rather than approximate. Evaluated over 100 resampled calibration/test splits to report a coverage <i>distribution</i>, not a point estimate.</td></tr>
<tr><td align="center"><b>4</b></td><td><b>Mondrian extension</b>: the same LAC logic, calibrated separately per class, to fix the marginal-coverage blind spot on malignant cases described above.</td></tr>
</table>

## Reproducing the Results
---

See [`Code/README.md`](Code/README.md) for exact run instructions: Kaggle, GPU T4×2, dataset mount path, smoke test vs. full run.

```bash
# Validate the conformal implementation independently of any specific run
python Code/test_coverage.py
python Code/test_notebook_matches_conformal.py
```

## Team
---

**Programme:** MSc Data & Computational Science, University College Dublin, 2026–27

**Course:** ACM 40960 — Project Presentation (Disease Modelling)

**Module Coordinator:** Dr. Sarp Akçay

**Authors:** Darshan S Gowda [25219951], Rakshith K B [25235067]

<div align="center">

</div>
