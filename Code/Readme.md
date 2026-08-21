<div align="center">

# Code

**Everything needed to reproduce the model, the conformal calibration, and the tests that verify both. **
Four files, no hidden dependencies beyond what's imported at the top of each.

</div>

---

## Files

| File | Role |
|:---|:---|
| [`conformal-uq-for-mammography-classification.ipynb`](conformal-uq-for-mammography-classification.ipynb) | **End-to-end pipeline**: data loading → DenseNet-121 training → conformal calibration → all figures and `results.json`. This is what produced every number and figure in [`Outputs Received/`](../Outputs%20Received/). Also runnable directly on Kaggle: [**▶ Open Notebook**](https://www.kaggle.com/code/wandererray9/conformal-mammography-classification) |
| [`conformal.py`](conformal.py) | Standalone, **model-agnostic** conformal prediction module (LAC / APS / RAPS + finite-sample-corrected quantile). Same math as the notebook's Section 9, reusable outside this project. |
| [`test_coverage.py`](test_coverage.py) | Validates the **coverage guarantee itself** on synthetic data (200 resamples × 2 class counts). Proves the conformal math is correctly implemented, independent of the mammography model. |
| [`test_notebook_matches_conformal.py`](test_notebook_matches_conformal.py) | Extracts the notebook's inlined conformal functions and checks them **numerically against `conformal.py`**, closing the gap between "the notebook says it's self-contained" and a runnable proof of it. |

---

## Why the notebook doesn't just `import conformal`

Kaggle notebooks can't import a sibling `.py` file without extra setup (publishing it as a utility script/dataset and attaching it), so the notebook carries its own inlined copy of the LAC/APS/RAPS/quantile functions. `conformal.py` is the portable reference version; `test_notebook_matches_conformal.py` exists specifically because two copies of the same logic can silently drift, and nothing else in the repo would catch that.

---

## Running

### Notebook: Kaggle only
> Uses the CBIS-DDSM Kaggle dataset mount and needs a GPU. **[Open directly on Kaggle →](https://www.kaggle.com/code/wandererray9/conformal-mammography-classification)**

| Step | Action |
|:---:|:---|
| 1 | Attach the [CBIS-DDSM dataset](https://www.kaggle.com/datasets/awsaf49/cbis-ddsm-breast-cancer-image-dataset) (awsaf49's JPEG version) |
| 2 | Session options → Accelerator → **GPU T4 ×2** *(not P100, CUDA architecture mismatch with current PyTorch)* |
| 3 | Set `CFG.SMOKE_TEST = True` and **Run All** to confirm the pipeline executes end to end in a few minutes |
| 4 | Set `CFG.SMOKE_TEST = False` and **Run All** for the real run (~1–2 GPU-hours) |
| 5 | Outputs land in `/kaggle/working/`: `results.json` and six figures |

> **Path quirk:** `CFG.data_root` must include the `/datasets/awsaf49/` segment; Kaggle inserts a user-scoped path for datasets sourced from another user's account.

### Tests — run anywhere
> No GPU needed. Requires `numpy` and `nbformat`.

```bash
python test_coverage.py
python test_notebook_matches_conformal.py

# or, via pytest
pytest -v
```

`test_notebook_matches_conformal.py` expects the notebook file in the same directory; override with the `NOTEBOOK_PATH` environment variable if it lives elsewhere.

---

## Headline Results

*(from the last full run, see [`Outputs Received/`](../Outputs%20Received/) for full detail)*

| Metric | Value |
|:---|:---:|
| Base classifier AUC | 0.708 |
| Base classifier ECE | 0.205 |
| LAC coverage | 0.95 |
| LAC set size | 1.694 |
| LAC singleton rate | 30.6% |
| Marginal malignant coverage | 0.893 under target |
| Mondrian malignant coverage | 0.960 target restored |

<div align="center">

*Data: [CBIS-DDSM on Kaggle](https://www.kaggle.com/datasets/awsaf49/cbis-ddsm-breast-cancer-image-dataset) · Notebook: [Kaggle](https://www.kaggle.com/code/wandererray9/conformal-mammography-classification)*

</div>
