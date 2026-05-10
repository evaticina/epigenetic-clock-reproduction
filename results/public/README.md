# Public results (lightweight)

This folder holds **GitHub-friendly** mirrors of notebook outputs: summary **CSV** tables and selected **PNG** figures. Bulk `results/*.csv` (outside this folder) remains ignored by `.gitignore`.

## CSV tables (metrics & summaries)

| File | Source notebook |
|------|-----------------|
| `model_metrics.csv` | `02_model_training.ipynb` |
| `repeated_split_metrics.csv` | `02_model_training.ipynb` |
| `permutation_test_metrics.csv` | `02_model_training.ipynb` |
| `horvath_overlap.csv` | `03_biological_interpretation.ipynb` |
| `enrichment_results.csv` | `03_biological_interpretation.ipynb` |
| `external_metrics.csv` | `04_external_validation.ipynb` |
| `cell_composition_estimates.csv` | `05_biological_and_batch_effect_analysis.ipynb` |
| `age_acceleration_celltype_correlations.csv` | `05_biological_and_batch_effect_analysis.ipynb` |
| `cohort_classifier_metrics.csv` | `05_biological_and_batch_effect_analysis.ipynb` |

## PNG figures (mirrored from `results/`)

| File | Source notebook |
|------|-----------------|
| `age_prediction_scatter.png` | `02` |
| `age_acceleration_hist.png` | `02` |
| `repeated_split_mae.png` | `02` |
| `cpg_stability_frequency_distribution.png` | `02` |
| `external_validation_scatter.png` | `04` |
| `external_validation_residual_hist.png` | `04` |
| `celltype_correlation_heatmap.png` | `05` |
| `age_accel_vs_celltype_scatters.png` | `05` |
| `pca_by_cohort.png` | `05` |
| `pca_by_age.png` | `05` |
| `cohort_age_prediction_distributions.png` | `05` |

Optional after **Run All** with **umap-learn**: `umap_by_cohort.png`, `umap_by_age.png`. Optional cell-adjustment outputs: `cell_adjusted_*.csv` / PNG if those notebook cells are executed.

**Removed legacy name:** `cpg_stability_top50.png` is **not** produced by the current `02_model_training.ipynb`; stability visualization is **`cpg_stability_frequency_distribution.png`**.

## What stays out of Git

Raw GEO supplements, full beta matrices, `data/processed/` serialized **X** / **y**, and bulk `results/` paths outside **`results/public/`** are ignored.

**Model artifacts** (`data/processed/scaler.joblib`, `elasticnet_model.joblib`, `selected_cpgs.csv`) are produced by **`02_model_training.ipynb`** and are not versioned; run notebook **02** before **04** or **05**.

## Regeneration

1. Run `01_data_loading.ipynb` → `02_model_training.ipynb` → `03_biological_interpretation.ipynb`.
2. Optionally run `04_external_validation.ipynb` and `05_biological_and_batch_effect_analysis.ipynb`.
3. Copy or refresh mirrors from `results/` into `results/public/` (notebook **05** copies batch diagnostics automatically when run).
