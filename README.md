# Explainable AI for Food Safety Risk Prediction with Uncertainty Quantification Using RASFF Data

Supplementary code, results, tables, and figures for the manuscript submitted to **_Discover Food_** (Springer Nature).

> **Authors:** Shinyclimensa C and Parthiban A (corresponding: parthiban.a@vit.ac.in)
> School of Advanced Sciences, Department of Mathematics, Vellore Institute of Technology (VIT), Vellore, Tamil Nadu, India

---

## Overview

This repository contains the full implementation and reproducibility materials for an explainable AI framework that predicts food-safety risk severity from European RASFF (Rapid Alert System for Food and Feed) notifications. The framework integrates:

- **Ensemble classification** — 12 model configurations benchmarked with bootstrap 95% confidence intervals and McNemar paired tests.
- **SHAP feature attribution** — interpretable feature importance on the primary 18-feature model.
- **Granger temporal-precedence testing** — with ADF stationarity checks and Bonferroni correction across 30 hazard-pair tests (interpreted strictly as predictive temporal precedence, **not** causation).
- **What-if sensitivity analysis** — feature-perturbation exploration of model-predicted risk shifts (reported as both percentage-point and relative changes).
- **Multi-target prediction** — independent classifiers for risk severity, hazard type, and notification type (a multi-output design, not a joint multi-task architecture).
- **Conformal prediction** — distribution-free uncertainty quantification with a temporally ordered calibration set.

A strict 80/20 **temporal** train–test split is used throughout, and all historical/aggregate features are computed on **training data only** to prevent temporal leakage.

---

## Repository structure

```
.
├── README.md
├── requirements.txt
├── code/
│   └── xai_causal_revised.py        # Full pipeline (Colab-ready)
├── results/
│   ├── RESULTS_SUMMARY.md           # Key findings at a glance
│   └── tables/                      # Result tables (CSV)
│       ├── table01_dataset_overview.csv
│       ├── table02_model_comparison.csv
│       ├── table03_ablation.csv
│       ├── table04_shap_importance.csv
│       ├── table05_granger.csv
│       ├── table06_conformal.csv
│       ├── table07_multitarget.csv
│       ├── table08_whatif_sensitivity.csv
│       └── table09_hyperparameters.csv
└── figures/                         # Publication figures (PDF)
    ├── fig1_model_comparison_ablation.pdf
    ├── fig2_shap_importance.pdf
    ├── fig3_shap_summary.pdf
    ├── fig4_forecasting.pdf
    ├── fig5_counterfactual_corrected.pdf
    ├── fig6_country_profiles.pdf
    ├── fig7_granger_network.pdf
    ├── fig8_hazard_correlation.pdf
    ├── fig9_multitask.pdf
    └── fig10_conformal.pdf
```

---

## Data

The study uses publicly available notification records from the **European Commission RASFF Portal**:
https://webgate.ec.europa.eu/rasff-window/portal/

The data are **not redistributed** in this repository. To reproduce the results, export the RASFF notifications (2019–2025) to a CSV named `RASFF.csv`.

The script expects the file at (Google Colab default):

```
/content/drive/MyDrive/RASFF/RASFF.csv
```

Edit the path variable near the top of `code/xai_causal_revised.py` to point to your local copy if running outside Colab.

---

## Requirements

```bash
pip install -r requirements.txt
```

Core dependencies: `pandas`, `numpy`, `scikit-learn`, `xgboost`, `lightgbm`, `shap`, `statsmodels`, `scipy`, `matplotlib`, `seaborn`.

---

## How to run

**Google Colab (recommended):**
1. Upload `code/xai_causal_revised.py` (or paste it into a notebook cell).
2. Mount Google Drive and place `RASFF.csv` at `/content/drive/MyDrive/RASFF/RASFF.csv`.
3. Run the script. All output tables (CSV) and figures (PDF) are written to `/content/drive/MyDrive/RASFF/xai_output/`.

**Local:**
```bash
python code/xai_causal_revised.py
```
(after editing the input path and ensuring the output directory exists).

The pipeline runs in 18 stages: data loading → temporal split → train-only feature engineering → 12-model benchmarking → cross-validation → ablation → SHAP → Granger → forecasting → seasonal decomposition → what-if sensitivity → multi-target prediction → conformal prediction → country profiling → cross-hazard dependency → hyperparameter logging → table generation → figure generation.

---

## Key results

| Component | Result |
|---|---|
| Primary model (LightGBM, 18 features) | AUC-ROC = 0.8970 [0.8884, 0.9052], Acc = 0.8087, F1 = 0.8077 |
| Post-notification prioritization model (with notification type) | AUC-ROC = 0.8970 |
| Early-prediction deployment model (without notification type) | AUC-ROC = 0.7993 |
| Top SHAP feature | Notification Type (1.86) — a downstream regulatory variable |
| Granger (Bonferroni, 0.05/30) | 3 significant; strongest Pathogenic Microorganisms → Migration (F = 17.91) |
| What-if sensitivity (EU controls) | −3.90 pp (−5.89% relative) |
| Conformal coverage | 87.53% empirical at 90% nominal (2.47 pp under-coverage) |

See `results/RESULTS_SUMMARY.md` and `results/tables/` for full detail.

---

## Important interpretive notes

- **Two operating modes.** The post-notification model retains *notification type*, a variable available only after regulatory classification, and is appropriate for triaging already-lodged notifications. The early-prediction model excludes it and is the configuration recommended for real-world early warning.
- **Predictive, not causal.** Granger results denote temporal precedence under the chosen specification; what-if analysis perturbs a predictive model. Neither establishes causal mechanisms or intervention effects.
- **Leakage prevention.** All historical-risk features are computed on training data only; multi-target tasks exclude target-derived features at the feature level.

---

## Citation

If you use this code or results, please cite the *Discover Food* article (citation to be updated upon publication).

## License

Released under the MIT License (see `LICENSE`).
