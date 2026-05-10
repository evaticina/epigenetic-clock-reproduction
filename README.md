# Epigenetic clock reproduction (GSE40279)

## Project overview

This repository reproduces a methylation-based aging model from Illumina HumanMethylation450 beta values. The target is chronological age in a whole-blood cohort (GEO [GSE40279](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE40279)). The fitted learner is an **ElasticNetCV** regressor on a leakage-safe pipeline: univariate correlation is used only for dimension reduction, and the final model estimates **DNA methylation–predicted age** for comparison with observed age (**epigenetic aging** as a supervised prediction problem).

## Scientific motivation

DNA methylation at specific CpG sites tracks age across tissues and has been widely used to construct **epigenetic clocks**. Re-implementing such a model from public data clarifies preprocessing choices, quantifies generalization on a held-out test set, and supports post hoc interpretation (probe stability, overlap with published clocks, functional enrichment). This project emphasizes **biological age prediction** from methylation while documenting steps that prevent train-test **information leakage**.

## Dataset

| Property | Description |
|----------|-------------|
| GEO accession | [GSE40279](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE40279) |
| Tissue | Whole blood |
| Samples | 656 (after alignment of methylation profiles and age labels) |
| Assay | Illumina Infinium HumanMethylation450 (platform GPL13534) |
| Features | ~473k CpG beta values (473,034 probes in processed matrices) |

Sample-level metadata (age, batch/plate, sex, ethnicity) are parsed from GEO SOFT records; beta matrices are loaded and row/column indices are aligned with these labels before modeling.

## Repository structure

```
epigenetic-clock-reproduction/
├── data/
│   ├── raw/           # GEO SOFT, supplementary tables, reference downloads (not versioned by default)
│   └── processed/     # Serialized X, y, metadata, model artifacts from 02 (not versioned by default)
├── notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_model_training.ipynb
│   ├── 03_biological_interpretation.ipynb
│   ├── 04_external_validation.ipynb
│   └── 05_biological_and_batch_effect_analysis.ipynb
├── results/           # Full notebook outputs (mostly git-ignored)
│   └── public/        # Lightweight CSV summaries + PNG mirrors for GitHub browsing (see `results/public/README.md`)
├── scripts/           # Optional auxiliary scripts (may be empty)
├── requirements.txt
└── README.md
```

Large or regenerated artifacts under `data/raw`, `data/processed`, and most of `results/` are excluded from version control (see `.gitignore`). **`results/public/`** holds **mirrored** summary **CSVs** and **PNGs** for GitHub (see `results/public/README.md`). Model artifacts under `data/processed/` (including `scaler.joblib`, `elasticnet_model.joblib`, `selected_cpgs.csv`) are **not** committed; run **`02_model_training.ipynb`** before optional notebooks **04** and **05**.

## Methodology

The full analysis pipeline includes:

1. **Data loading** — Download/read GEO metadata and Illumina 450K beta values for GSE40279; align samples with chronological age (`01_data_loading.ipynb`).
2. **Leakage-safe Elastic Net clock** — Train/test split before feature selection; correlation-based candidate CpGs on the training set only; `StandardScaler` fit on training rows; **ElasticNetCV** (`02_model_training.ipynb`).
3. **Repeated split stability** — Multiple independent splits with the same pipeline to probe metric and probe-stability variability (`02_model_training.ipynb`).
4. **Permutation sanity test** — Label-shuffled control within the training workflow to verify dependence on true ages (`02_model_training.ipynb`).
5. **Biological annotation** — Manifest merge, **Horvath** overlap (Fisher exact), GO enrichment via **g:Profiler** REST (`03_biological_interpretation.ipynb`).
6. **External validation** — Apply the **frozen** trained model (no refit) to an independent whole-blood 450K cohort (`04_external_validation.ipynb`).
7. **Biological validation (cell composition & batch checks)** — Reference-based blood deconvolution vs **age acceleration**; **PCA / optional UMAP / cohort classifier** on combined GSE40279 + GSE87571 overlapping model CpGs (`05_biological_and_batch_effect_analysis.ipynb`; diagnostic only, no ComBat).

## Leakage prevention

- **Feature selection** uses only training samples.
- **Scaling** uses training statistics exclusively on held-out data.
- **Test labels** are not used during preprocessing.
- **Permutation sanity check** — See `results/public/permutation_test_metrics.csv`.
- **External evaluation** uses only frozen coefficients and training-derived scaling on external betas.

## External validation

Independent cohort validation matters because **within-study** test error can be optimistic when laboratory handling, normalization, and population structure match training. Applying the **same** GSE40279-fitted model to **another** blood methylation study tests transportability under batch and preprocessing drift.

This repository evaluates the frozen clock on **GSE87571** (whole blood, Illumina 450K) **without retraining** (`04_external_validation.ipynb`). Metrics below are taken from **`results/public/external_metrics.csv`** (mirrored from full `results/external_metrics.csv`).

| Quantity | Value |
|----------|--------|
| Cohort | GSE87571 |
| Samples evaluated | 515 |
| Overlapping training CpGs | 5000 |
| MAE | 4.415 years |
| RMSE | 5.513 years |
| R² | 0.930 |
| Pearson *r* | 0.983 |

**Interpretation:** Performance can shift versus the GSE40279 hold-out set because of **batch effects**, **different beta preprocessing** (e.g., SWAN vs study supplementary values), **cell composition**, and **population** differences. External metrics should be read as **compatibility checks** on public blood methylation, not as clinical validation.

## Biological robustness and batch diagnostics

Notebook **`05_biological_and_batch_effect_analysis.ipynb`** adds two layers on top of the frozen clock: a **cell-composition screen** and **batch/cohort diagnostics**. Outputs from notebook **05** include CSV summaries under **`results/`** (mirrored into **`results/public/`** where noted below); figures are written under **`results/`** and can be copied into **`results/public/`** for GitHub browsing.

### A. Cell-composition analysis

Whole-blood methylation clocks can be **confounded by leukocyte composition**: aggregated betas mix CD4, CD8, B cells, monocytes, granulocytes, and NK cells, each with distinct methylation profiles. This repository uses an **approximate Houseman-style constrained projection** against the **Reinius 450K reference** (Biolearn) as a **biological sanity check**, not as a substitute for dedicated immune profiling or a full **Houseman / EpiDISH** workflow.

Correlations between **`age_acceleration = predicted_age − chronological_age`** (from the frozen model) and estimated immune fractions are summarized in **`results/public/age_acceleration_celltype_correlations.csv`**. In the committed snapshot, associations are **weak**: the largest absolute Pearson correlation is **≈ 0.12** (CD8 T cell fraction). **Interpretation:** estimated immune composition does **not** appear to **dominate** the acceleration signal in this screen, but that does **not** rule out subtler composition effects or non-leukocyte biology.

### B. Batch and cohort diagnostics

To probe whether methylation space reflects **cohort structure** as much as aging-like signal, notebook **05** combines **GSE40279** (internal/training cohort) and **GSE87571** (external cohort) on **overlapping model CpGs**, runs **PCA** (and optional **UMAP**), compares chronological and predicted-age distributions, and trains a simple **cohort classifier** on overlapping beta values.

Hold-out metrics for that classifier are in **`results/public/cohort_classifier_metrics.csv`**. In this snapshot, **scaled logistic regression** achieves **ROC-AUC = 1.0** and **accuracy = 1.0** on the stratified hold-out split; **random forest** is effectively as high. That indicates **strong separability of cohort labels** in probe space using these features—consistent with **substantial cohort-specific structure** (technical and/or biological co-structure with study). It does **not** by itself invalidate external age prediction; it cautions that **naive cross-cohort comparisons** can align technical axes with outcomes.

**Careful summary:** External validation (notebook **04**) suggests **predictive transfer** of the frozen clock to an independent blood cohort under public supplementary betas. The batch diagnostics layer shows that **cohorts remain almost perfectly distinguishable** from methylation features alone in this setup. Results should be interpreted **cautiously**: performance does **not** imply **cohort invariance**, **full batch correction**, or **clinically validated biological age**.

PCA figures **`pca_by_cohort.png`** and **`pca_by_age.png`** are saved under **`results/`** when notebook **05** runs (copy to **`results/public/`** if you want them tracked on GitHub). **ComBat**, **SVA**, and similar harmonization steps are **not** applied in notebook **05**; that notebook remains **diagnostic only**.

## Key results

Values below come from **`results/public/*.csv`** (same files under `results/` after notebook runs).

### Internal hold-out (GSE40279, `model_metrics.csv`)

| Metric | Value |
|--------|--------|
| MAE | 3.704 years |
| RMSE | 4.934 years |
| R² | 0.893 |
| Pearson *r* | 0.945 |

### Repeated split stability (`repeated_split_metrics.csv`)

Five seeds (0, 1, 7, 21, 42): test-set MAE ranges approximately **3.37–3.85 years**, R² **0.87–0.90**, Pearson *r* **0.93–0.95**. **cg16867657** is selected with non-zero coefficient in every run in this snapshot.

### Permutation test (`permutation_test_metrics.csv`)

| Scenario | MAE | RMSE | R² | Pearson *r* |
|----------|-----|------|-----|-------------|
| Real labels | 3.704 | 4.934 | 0.893 | 0.945 |
| Shuffled labels | 14.051 | 17.503 | −0.240 | −0.100 |

### Horvath overlap (`enrichment_results.csv`, Fisher row)

| Quantity | Value |
|----------|--------|
| Fisher exact *p* | 3.90 × 10⁻⁹ |
| Odds ratio | 18.11 |

### External validation (`external_metrics.csv`)

See **External validation** section above.

### Cell composition vs acceleration (`age_acceleration_celltype_correlations.csv`)

Maximum **|**Pearson *r***|** across cell types ≈ **0.119** (CD8 T cell; *n* = 656).

## Figures

Notebooks write PNG figures under **`results/`**. **`results/public/`** holds **mirrored** CSV and PNG artifacts intended for GitHub (everything else under **`results/`** is git-ignored). Refresh mirrors after **Run All** by copying selected outputs from **`results/`** into **`results/public/`** (notebook **05** copies several files programmatically). Inventory is tracked in **`results/public/README.md`**.

| Notebook | Representative PNG outputs |
|----------|----------------------------|
| `02_model_training.ipynb` | `age_prediction_scatter.png`, `age_acceleration_hist.png`, `repeated_split_mae.png`, `cpg_stability_frequency_distribution.png` |
| `04_external_validation.ipynb` | `external_validation_scatter.png`, `external_validation_residual_hist.png` |
| `05_biological_and_batch_effect_analysis.ipynb` | `celltype_correlation_heatmap.png`, `age_accel_vs_celltype_scatters.png`, `pca_by_cohort.png`, `pca_by_age.png`, `cohort_age_prediction_distributions.png`; optional `umap_by_cohort.png`, `umap_by_age.png` if **umap-learn** is installed |

## Reproducibility

1. Create a Python 3.11+ environment and install dependencies (see **Installation**).
2. Run notebooks **in order**:
   - `notebooks/01_data_loading.ipynb`
   - `notebooks/02_model_training.ipynb`
   - `notebooks/03_biological_interpretation.ipynb`
   - *(optional)* `notebooks/04_external_validation.ipynb` — large GEO downloads.
   - *(optional)* `notebooks/05_biological_and_batch_effect_analysis.ipynb` — requires artifacts from **02**; external matrices download like **04** if missing.
3. Use **Restart Kernel & Run All** per notebook.

Random seeds are fixed where specified; reproducibility depends on library versions in `requirements.txt`.

## Installation

```bash
python -m venv .venv
.venv\Scripts\activate   # Windows
# source .venv/bin/activate   # macOS / Linux
pip install -r requirements.txt
```

## Usage

From the repository root, start Jupyter (`jupyter notebook` or `jupyter lab` if installed). Notebooks assume cwd **`notebooks/`** and resolve `PROJECT_DIR` via `Path.cwd().parent`.

## Output files (selected)

| Path | Contents |
|------|----------|
| `data/processed/X.pkl.gz`, `y.csv` | Training cohort matrix and ages |
| `data/processed/scaler.joblib`, `elasticnet_model.joblib`, `selected_cpgs.csv` | Frozen model for 04 / 05 |
| `results/model_metrics.csv` | Internal test metrics |
| `results/external_predictions.csv`, `external_metrics.csv` | External cohort outputs |
| `results/cell_composition_estimates.csv`, `age_acceleration_celltype_correlations.csv`, `cohort_batch_metadata.csv`, `cohort_classifier_metrics.csv`, PCA/batch PNGs (see **Figures**) | Deconvolution + batch diagnostics |
| `results/public/*.csv` | Lightweight metrics mirrored for GitHub (see `results/public/README.md`) |

## Limitations

- **Raw IDAT preprocessing** — The pipeline uses **GEO supplementary / averaged beta** tables, not **raw IDAT** files or a full Bioconductor **minfi** / **preprocessQuantile** (or similar) stack.
- **No full ComBat harmonization** — Study-wide **ComBat** (or equivalent M-value/Beta harmonization across cohorts) is **not** applied to training or validation betas here.
- **No surrogate variable analysis (SVA)** — Latent technical variation is **not** modeled with **SVA** / **RUV**-style approaches in this repository.
- **Plate / batch modeling in training** — Plate effects **within** GSE40279 are not absorbed by a formal batch-correction layer in the elastic-net workflow.
- **Cell composition** — Deconvolution in notebook **05** is **approximate reference projection**, not a publication-grade **Houseman** or **EpiDISH** replication or clinical immune profiling.
- **External validation** — Restricted to **compatible public** Illumina blood cohorts and dependent on GEO supplementary formatting.
- **Single training cohort** — Generalization beyond whole blood / GSE40279 is not established here.

## Future work

- Raw **IDAT** ingestion and preprocessing parity with standard Bioconductor workflows.
- Formal comparison to **Houseman / EpiDISH** implementations and reference panels.
- **Batch harmonization** (e.g., ComBat on betas) building on the diagnostic PCA/cohort-classifier layer in notebook **05**.
- Additional **external cohorts** and quantitative comparison of cohort-shift corrections.
- **Nested cross-validation** or expanded **permutation** ensembles where feasible.

## References

- Horvath S. DNA methylation age of human tissues and cell types. *Genome Biol.* 2013;14(10):R115. [https://doi.org/10.1186/gb-2013-14-10-r115](https://doi.org/10.1186/gb-2013-14-10-r115)
- Houseman EA, et al. DNA methylation arrays as surrogate measures of cell mixture distribution. *BMC Bioinformatics.* 2012;13:86.
- NCBI GEO series GSE40279. [https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE40279](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE40279)
- Illumina HumanMethylation450 BeadChip documentation (GPL13534).
- Raudvere U, et al. g:Profiler (2019 update). *Nucleic Acids Res.* 2019;47(W1):W191–W198. [https://doi.org/10.1093/nar/gkz369](https://doi.org/10.1093/nar/gkz369)
