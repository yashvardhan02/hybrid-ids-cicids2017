# Hybrid Network Intrusion Detection and Post-Breach Network Analysis

Code accompanying the paper **"A Hybrid Network Intrusion Detection and Post-Breach Network Analysis Framework Using Isolation Forest, XGBoost, and Graph-Based Heuristic Search on CICIDS-2017."**

Author: Yashvardhan Bhatnagar, NMIMS Mukesh Patel School of Technology Management & Engineering, Indore, India.

---

## Overview

This repository contains the full implementation of a two-stage intrusion detection pipeline combined with post-detection graph analysis:

1. **Stage 1 — Isolation Forest** (unsupervised): produces a per-flow anomaly score, appended as an engineered feature.
2. **Stage 2 — XGBoost** (supervised): multi-class classification across 27 traffic classes, using Stratified 5-Fold cross-validation and balanced class weights.
3. **Post-detection graph analysis**: BFS for blast-radius estimation, DFS for attack kill-chain reconstruction, and Hill Climbing for response-action selection.

---

## Dataset

CICIDS-2017, published by the Canadian Institute for Cybersecurity, University of New Brunswick:
https://www.unb.ca/cic/datasets/ids-2017.html

All five daily CSV files are combined, yielding **2,099,971 network flow samples** across **27 traffic classes** (26 attack categories plus benign traffic).

The dataset is **not** included in this repository. Download it from the link above and place the CSV files in a local directory, then update the data path at the top of the notebook.

---

## Main results

| Metric | Value |
|---|---|
| Overall accuracy (5-fold mean) | 99.81% ± 0.01% |
| Classes with F1 > 0.95 | 20 of 27 |
| BENIGN precision / recall | 1.0000 / 0.9999 |

### Ablation studies

| Configuration | Accuracy |
|---|---|
| Full pipeline (with Isolation Forest score) | 99.81% |
| Without Isolation Forest score | 99.82% |
| Without Source/Destination Port features | 99.34% |

The Isolation Forest anomaly score produces **no measurable aggregate accuracy gain**. This is reported openly in the paper rather than omitted.

Removing port features costs 0.47 percentage points, indicating the model retains most of its discriminative power from behavioural flow characteristics rather than the port shortcut identified in prior critiques of this dataset.

### Graph algorithm scaling

Benchmarked on synthetic layered enterprise topologies from 9 to 2,000 nodes (10 random topologies per size, 5 timing repetitions each). BFS runtime grows with an empirically fitted power-law exponent of ~1.42; Hill Climbing remains effectively constant (0.010–0.027 ms) since it operates on a fixed action table independent of graph size.

---

## Methodological notes

- **Leakage control.** `StandardScaler` and the Isolation Forest are refit independently inside each training fold. Nothing derived from a test fold — including feature means and variances — influences any stage of the pipeline. An earlier version fit the scaler globally before cross-validation; this was identified and corrected before final evaluation.
- **`Attempted Category` column dropped.** Direct inspection of the raw data showed this column is an integer code where every non-Attempted class is pure at `-1` across all 2,087,992 such samples, and every `- Attempted` class maps predominantly to one specific non-`-1` code (per-class purity 80–100%). It is a near-perfect proxy for attempt-status membership rather than a behavioural signal, so it was removed from the feature matrix, leaving 84 behavioural features.
- **Contamination parameter.** Isolation Forest uses `contamination=0.25`, approximating the dataset's known attack proportion (24.6%). This is calibrated to a known base rate rather than derived label-blind; a fully label-blind selection procedure is left for future work.

---

## Requirements

```
python >= 3.10
scikit-learn
xgboost
pandas
numpy
networkx
matplotlib
```

Install with:

```bash
pip install scikit-learn xgboost pandas numpy networkx matplotlib
```

A CUDA-capable GPU is optional. The main cross-validation loop uses `device='cuda'` where available; remove or change that parameter to run on CPU.

---

## Repository contents

| File | Description |
|---|---|
| `IDS_PORTAL.ipynb` | Full pipeline: preprocessing, both ML stages, cross-validation, ablation studies, graph analysis, and scaling benchmark |

---

## Reproducibility

Random seeds are fixed (`random_state=42`, `seed=42`) throughout. Reported figures are means across five stratified folds. Absolute runtimes in the scaling benchmark are hardware-dependent; the scaling *behaviour* rather than the absolute timings is the reproducible result.

---

## Citation

If you use this code, please cite the paper. A preprint is available on arXiv (link to be added once posted).

---

## License

MIT License — see `LICENSE`.
