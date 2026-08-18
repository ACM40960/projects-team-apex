**Conformal Uncertainty Quantification for Mammographic Breast Cancer Classification**

Course: ACM 40960: Project 8 (Disease Modelling) 
Programme: MSc Data & Computational Science, University College Dublin, 2026–27 
Supervisors: Dr. Sarp Akçay
Authors: Darshan S Gowda, Rakshith K B

**Overview**

Deep learning classifiers for mammographic malignancy detection output a softmax probability, but that probability is not a trustworthy confidence score on its own — this project's own base classifier has an Expected Calibration Error of ~0.205, meaning "80% confident" is not actually right 80% of the time. This project wraps a DenseNet-121 classifier, fine-tuned on CBIS-DDSM (benign vs. malignant), with split conformal prediction: a post-hoc, model-agnostic calibration layer that produces prediction sets with a distribution-free, finite-sample coverage guarantee — valid regardless of whether the underlying model is well-calibrated.

Three nonconformity scores are implemented and compared (LAC, APS, RAPS), all achieving the target ~95% marginal coverage. The project's core clinical finding is that marginal coverage can mask class-level failure: pooled coverage sits at target while malignant-specific coverage undershoots it (~0.893). Mondrian (class-conditional) calibration — a separate threshold per class instead of one global threshold — restores per-class coverage to ~0.960, which is the headline contribution.

**Method** [Summarized]

1) Data & splits — CBIS-DDSM JPEGs (via Kaggle), labels binarized from pathology. Splits are done at the patient level (not per-image), so no patient appears in more than one of train/val/calibration/test — this is what keeps the conformal exchangeability assumption honest.
2) Model — DenseNet-121, ImageNet-pretrained, fine-tuned end-to-end with a 2-class head. The conformal layer is model-agnostic — it only ever sees softmax output, not the architecture.
3) Conformal calibration — LAC (1 - p(true)), APS (adaptive, cumulative sorted mass), and RAPS (APS + a rank regularization penalty) are all calibrated using the standard finite-sample-corrected quantile ceil((n+1)(1-α))/n, which is what makes the coverage guarantee exact rather than approximate. Evaluated over 100 resampled calibration/test splits to report a coverage distribution, not a single point estimate.
4) Mondrian extension — the same LAC threshold logic, calibrated separately per class, to fix the marginal-coverage blind spot on malignant cases described above.

**Reproducing the results**

See Code/README.md for exact run instructions (Kaggle, GPU T4 ×2, dataset mount path, smoke test vs. full run). Outputs land in Outputs Received/; running Code/test_coverage.py and Code/test_notebook_matches_conformal.py verifies the conformal implementation independently of any specific run.
